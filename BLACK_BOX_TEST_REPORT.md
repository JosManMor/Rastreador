# Reporte de Pruebas de Caja Negra - Rastreador

Este reporte documenta el comportamiento actual del sistema ante diversos tipos de entrada, basándose en el análisis del código fuente (backend) y las evidencias del PDF proporcionado.

## 1. Módulo de Autenticación (Registro)

| Campo | Tipo de Dato | Validación Actual (Backend) | Valores Aceptados (Ejemplos) | Resultado Esperado vs Real |
| :--- | :--- | :--- | :--- | :--- |
| **Nombre** | String | `/^[a-zA-Z0-9\s]+$/` | `"Juan"`, `"A" * 5000`, `"12345"` | **Real:** Acepta longitudes infinitas (rompe la UI). **Esperado:** Máx 50-100 carac. |
| **Correo** | String | `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` | `"a@b.c"`, `"muy_largo@empresa.com" * 100` | **Real:** Sin límite de longitud. **Esperado:** Máx 100 carac. |
| **Password** | String | Solo verifica que no esté vacío | `"1"`, `"password" * 500`, `"abc"` | **Real:** Sin límite y sin complejidad mínima. **Esperado:** Min 8, Máx 50 carac. |
| **Teléfono** | String | `/^[0-9+()\s-]+$/` | `"123"`, `"999" * 100` | **Real:** Sin límite de longitud. **Esperado:** 7-15 dígitos. |
| **Código Sup.**| Number | Existencia en BD (Role=SUPERVISOR) | `1`, `999999` | **Real:** Correcto (validación de integridad). |

---

## 2. Módulo de Geocercas (Creación)

| Campo | Tipo de Dato | Validación Actual (Backend) | Valores Aceptados (Ejemplos) | Resultado Esperado vs Real |
| :--- | :--- | :--- | :--- | :--- |
| **Nombre** | String | Solo verifica que no esté vacío | `"Zona" * 1000` | **Real:** Acepta longitudes infinitas. **Esperado:** Máx 50 carac. |
| **Tipo** | Enum | `'CIRCLE'` o `'POLYGON'` | `'CIRCLE'`, `'POLYGON'` | **Real:** Correcto. |
| **Coordenadas**| JSON | Ninguna (JSON.stringify) | `{"lat": 500, "lng": -999}`, `{"x": "y"}` | **Real:** Acepta coordenadas fuera de rango y campos inválidos. **Esperado:** Lat (-90,90), Lng (-180,180). |
| **Radio** | Number | Ninguna | `-100`, `999999999` | **Real:** Acepta valores negativos o excesivos. **Esperado:** 10m a 50,000m. |

---

## 3. Módulo de Auditoría (Logs)

| Campo | Validación Actual | Valores Aceptados | Observación |
| :--- | :--- | :--- | :--- |
| **ID Usuario** | `id || null` | `null`, `1`, `999` | Permite registros huérfanos si no se provee el ID. |
| **Entidad** | `entity || null` | `null`, `"Users"`, `"Geofences"` | Difícil de rastrear si el valor es nulo. |
| **Detalles** | Solo no vacío | Cualquier string largo | Puede corromper la vista de tabla si el detalle es muy extenso. |

---

## 4. Scripts de Simulación de Fallos (Pseudo-Pruebas)

A continuación se describen los casos de prueba "extremos" que el sistema permite actualmente:

### Caso 1: Desbordamiento de Buffer Visual (Buffer Overflow UI)
*   **Input:** En el campo `nombre` del registro, enviar una cadena de 10,000 caracteres.
*   **Resultado:** El backend lo guarda exitosamente en la base de datos (si el tipo de columna lo permite, ej. TEXT) o falla silenciosamente si excede el límite de la BD, pero no hay validación previa. En el frontend, al listar usuarios, el nombre "rompe" el contenedor y empuja otros elementos fuera de la pantalla.

### Caso 2: Coordenadas Imposibles (Impossible Coordinates)
*   **Input:** `{"lat": 999.99, "lng": 888.88}`
*   **Resultado:** El sistema guarda la geocerca. Al intentar renderizarla en el mapa (Leaflet/Google Maps), el mapa se vuelve blanco o entra en un bucle infinito intentando centrar una posición inexistente.

### Caso 3: Inyección de Atributos en Coordenadas
*   **Input:** `{"lat": 19, "lng": -103, "malicious_script": "<script>alert(1)</script>"}`
*   **Resultado:** El backend guarda todo el objeto JSON. Si el frontend renderiza detalles de la geocerca sin sanitizar, podría ejecutarse código arbitrario (XSS).

---

## 5. Conclusiones de la Prueba
El sistema presenta una **ausencia crítica de validaciones de negocio** en el backend. Confía plenamente en que el frontend enviará datos limpios, lo cual es una vulnerabilidad de Caja Negra, ya que un atacante puede bypassear el frontend usando herramientas como Postman o cURL.

**Acciones Recomendadas:**
1. Implementar librerías de validación como `Joi` o `Zod` en el backend.
2. Definir esquemas estrictos para el objeto `coordenadas`.
3. Establecer límites de caracteres (`maxLength`) en todos los campos de texto.
