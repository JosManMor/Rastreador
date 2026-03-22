# Plan de Correcciones - Rastreador

Este documento detalla los cambios necesarios para abordar los hallazgos del reporte de pruebas de caja negra (PDF). Se ha analizado el código fuente para identificar los puntos exactos de intervención.

## 1. Validación de Entradas (Frontend)

**Problema:** El sistema acepta valores extremadamente largos sin validación en formularios de login y registro.
**Archivos Afectados:**
- `rastreadorFrontend/src/utils/validators.ts`
- `rastreadorFrontend/app/(auth)/login.tsx`
- `rastreadorFrontend/app/(auth)/register.tsx`

**Cambios Necesarios:**
1.  **Actualizar `validators.ts`:**
    - Agregar validación de longitud máxima en las funciones `nombre`, `correo`, `password` y `telefono`.
    - Ejemplo: Nombre máx 50 caracteres, Correo máx 100, Password máx 50.
2.  **Actualizar Vistas de Autenticación:**
    - Agregar la propiedad `maxLength` a los componentes `TextInput` en `login.tsx` y `register.tsx` para prevenir la entrada de datos excesivos desde la interfaz.

## 2. Gestión de Roles y Permisos

**Problema:** Inconsistencias visuales en opciones según el rol. Supervisores pueden ver filtros o acciones destinados solo a Administradores (aunque el backend proteja, la UI no debería mostrarlos).
**Archivos Afectados:**
- `rastreadorFrontend/app/(web)/users.tsx`

**Cambios Necesarios:**
1.  **Filtro de Roles:**
    - Modificar el selector de "Filtrar por rol". Si el usuario es `SUPERVISOR`, no debería poder filtrar por `ADMIN` (o ver la opción).
    - Validar que las acciones de edición/eliminación estén correctamente ocultas o deshabilitadas visualmente si el usuario no tiene permisos sobre el objetivo.

## 3. Geocercas y Mapas

**Problema:**
- No se validan coordenadas (permitiendo valores irreales como latitud 500).
- No se valida el radio (permitiendo valores negativos o excesivos).
- Mapas "infinitos" o renderizado incorrecto por coordenadas inválidas.
**Archivos Afectados:**
- `rastreadorFrontend/app/(web)/geofences.tsx`

**Cambios Necesarios:**
1.  **Validación en `save()`:**
    - Implementar rangos estrictos para coordenadas:
        - Latitud: `-90` a `90`.
        - Longitud: `-180` a `180`.
    - Validar radio: Mínimo `10m`, Máximo `50,000m` (ejemplo).
    - Validar polígonos: Asegurar que todos los puntos estén dentro de los rangos permitidos.
2.  **Input Filtering:**
    - Evitar caracteres no numéricos en los campos de coordenadas y radio.

## 4. Auditoría y Reportes

**Problema:**
- Registros de auditoría con valores `null` o visualización rota con nombres largos.
- Reportes generados sin identificadores únicos o fechas claras en el nombre del archivo.
- Búsqueda local en lugar de API.
**Archivos Afectados:**
- `rastreadorFrontend/app/(web)/audit.tsx`
- `rastreadorFrontend/app/(web)/reports.tsx`
- `rastreadorFrontend/app/(web)/users.tsx`

**Cambios Necesarios:**
1.  **Auditoría (`audit.tsx`):**
    - Manejar valores `null` en `usuario_nombre` mostrando un texto por defecto (ej. "Usuario Eliminado" o "Sistema").
    - Asegurar que las columnas de la tabla tengan restricciones de estilo (`numberOfLines`, `ellipsizeMode`) para que textos largos no rompan el layout.
2.  **Reportes (`reports.tsx`):**
    - Modificar la generación del nombre del archivo en `exportFile`.
    - Formato propuesto: `reporte_[Usuario]_[FechaInicio]_[Timestamp].pdf` para garantizar unicidad.
3.  **Búsqueda de Usuarios (`users.tsx`):**
    - *Nota:* Actualmente la búsqueda es local (`filtered`). Para corregir esto "realmente" se requiere soporte en el backend (`GET /users?search=...`). Si no se toca el backend, se debe optimizar el filtro local y añadir *debounce*.

## 5. Inconsistencias Visuales Generales

**Problema:** Botones sin función clara (ej. botón de velocidad), filtros poco intuitivos.
**Archivos Afectados:**
- `rastreadorFrontend/app/(mobile)/home.tsx` (posible ubicación del botón de velocidad)
- `rastreadorFrontend/app/(web)/*.tsx`

**Cambios Necesarios:**
1.  **Revisión de UI:**
    - Eliminar o implementar la funcionalidad del "botón de velocidad" mencionado si no tiene uso.
    - Mejorar la UX de los filtros: Hacer que el filtrado sea automático al escribir/seleccionar, o hacer el botón de búsqueda más evidente.

---
*Este documento sirve como guía para la implementación de las correcciones de calidad de software.*
