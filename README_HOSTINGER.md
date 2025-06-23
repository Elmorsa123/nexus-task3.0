# NEXUS Task - Guía para Desarrolladores de Hostinger

Este documento proporciona una visión general del estado actual de la aplicación NEXUS Task, las funcionalidades pendientes y sugerencias para su implementación.

## 1. Estado Actual de la Aplicación

NEXUS Task es una aplicación frontend desarrollada con React (usando Vite) y TailwindCSS para la gestión de incidencias, tareas y planes de limpieza. Actualmente, la aplicación cuenta con las siguientes características:

- **Interfaz de Usuario Moderna:** Un diseño "Tech Abyss" oscuro, con componentes de shadcn/ui y animaciones de Framer Motion.
- **Gestión de Incidencias:**
    - Creación, visualización, edición y filtrado de incidencias.
    - Asignación a técnicos.
- **Gestión de Tareas:**
    - Creación, visualización, edición (detalles y rápida desde tarjeta) y filtrado de tareas.
    - Asignación a técnicos.
    - Cambio de estado (Pendiente, En Progreso, Completada) directamente desde la tarjeta de tarea (simulando el botón "Finalizar Tarea").
- **Gestión de Usuarios (Básica):**
    - Creación, visualización, edición y eliminación de usuarios.
    - Cambio de estado (Activo/Inactivo).
    - Roles básicos (administrador, responsable, técnico) definidos en el frontend.
- **Vista de Calendario:**
    - Muestra tareas (por fecha de entrega) e incidencias pendientes (por fecha de reporte).
    - Navegación mensual y acceso directo a edición.
- **Gestión de Limpieza:**
    - Visualización y filtrado de planes de limpieza.
    - Cambio de estado (Pendiente/Realizada).
- **Persistencia de Datos:** Actualmente implementada mediante `localStorage`. Todos los datos de incidencias, tareas, usuarios y limpieza se guardan localmente en el navegador.
- **Notificaciones Toast:** Para feedback de acciones del usuario.

## 2. Funcionalidades Críticas Pendientes y Puntos de Integración

La principal carencia de la aplicación es la **falta de un backend robusto** para la persistencia de datos, autenticación y gestión de roles avanzada. Los siguientes puntos indican dónde se deben realizar las integraciones clave:

### 🔁 Conexión de Base de Datos (Backend)

- **Ubicación Principal:** `src/hooks/useDataManagement.js`.
    - Actualmente, este hook utiliza `useLocalStorage` para simular la persistencia.
    - **Acción Requerida:** Reemplazar las llamadas a `useLocalStorage` con llamadas a la API del backend elegido (Supabase, Firebase, API REST propia con MySQL/PostgreSQL, etc.).
    - Todas las funciones CRUD (addIncident, updateTask, addUser, deleteUser, etc.) dentro de `useDataManagement.js` deben ser modificadas para interactuar con el backend en lugar del estado local y localStorage.
    - Los datos iniciales (`initialIncidentsData`, etc.) deben eliminarse una vez que los datos se carguen desde la base de datos.
- **Carga de Datos Inicial:** El `useEffect` comentado en `useDataManagement.js` (línea ~35) es un ejemplo de cómo se podrían cargar los datos iniciales desde Supabase.
- **Tecnologías Sugeridas:**
    - **Supabase (Recomendado):** Ofrece base de datos PostgreSQL, autenticación, almacenamiento y APIs auto-generadas. Es una excelente opción "Backend como Servicio" (BaaS) que se integra bien con React.
    - **Firebase (Alternativa BaaS):** Similar a Supabase, con Firestore (NoSQL) o Realtime Database, autenticación y Cloud Functions.
    - **Backend Propio (Node.js/Express + MySQL/PostgreSQL):** Si se requiere un control total sobre el backend. Se necesitaría construir una API REST.

### 🧑‍🔧 Aplicación de Roles y Permisos

- **Ubicación Principal:**
    - `src/App.jsx`:
        - En la función `renderView()` (línea ~100): Aquí se debe implementar la lógica para restringir el acceso a ciertas vistas según el rol del usuario autenticado. Por ejemplo, la vista `ManageUsersView` debería ser accesible solo por administradores.
        - En las funciones `handleAddUserAndCloseModal`, `handleUpdateUserAndCloseModal` (líneas ~78, ~90): Al crear o editar usuarios, el rol asignado debe guardarse en la base de datos.
    - `src/hooks/useDataManagement.js`: Al obtener el usuario autenticado, su rol debe determinar qué datos puede ver o modificar.
    - `src/components/views/ManageUsersView.jsx`: (línea ~11) Comentario sobre restricción de acceso.
- **Lógica de Roles:**
    - **Administrador:** Acceso total. Puede gestionar usuarios, todas las incidencias, tareas, etc.
    - **Responsable:** Puede ver todas las incidencias/tareas, asignarlas, gestionar usuarios de su equipo (potencialmente).
    - **Técnico:** Solo ve las incidencias/tareas asignadas a él. No puede gestionar usuarios.
- **Autenticación:**
    - **Tecnologías Sugeridas:**
        - **Supabase Auth:** Se integra directamente con la base de datos de Supabase y ofrece gestión de usuarios y RLS (Row Level Security).
        - **Firebase Authentication:** Similar a Supabase Auth.
        - **Auth0 / Clerk:** Servicios de autenticación especializados si se necesita una solución más desacoplada o con características avanzadas de identidad.
    - El estado de autenticación del usuario (y su rol) deberá gestionarse globalmente en la aplicación, posiblemente a través de un Context API de React o un gestor de estado como Zustand/Redux.

### ✅ Botón "Finalizar Tarea" (y cambio de estado general)

- **Ubicación Principal:** `src/components/views/TasksView.jsx` (en el componente `TaskCard`, función `handleStatusToggle`, línea ~56) y `src/hooks/useDataManagement.js` (función `updateTaskStatus`, línea ~95).
- **Funcionalidad Actual:** El botón ya existe y cambia el estado de la tarea en el frontend y `localStorage`.
- **Acción Requerida:**
    - La función `updateTaskStatus` en `useDataManagement.js` debe modificarse para actualizar el estado de la tarea en la base de datos.
    - Considerar la posibilidad de añadir un campo para notas o comentarios al finalizar una tarea.
- **Consideraciones Adicionales:** La lógica de "Finalizar Tarea" es un caso específico de actualización de estado. Principios similares aplican para actualizar el estado de incidencias. El botón "Actualizar Estado" en `IncidentsView.jsx` (línea ~78) es otro punto a considerar.

### 🗂️ Filtros y Dashboard Avanzado

- **Ubicación Filtros:**
    - `src/components/views/IncidentsView.jsx` (líneas ~90-130)
    - `src/components/views/TasksView.jsx` (líneas ~130-170)
- **Ubicación Dashboard:** `src/components/views/DashboardView.jsx` (línea ~5)
- **Funcionalidad Actual:** Los filtros se aplican en el frontend sobre los datos cargados en `localStorage`. El dashboard muestra estadísticas básicas calculadas en el frontend.
- **Acción Requerida:**
    - **Filtros:** Para un rendimiento óptimo con grandes volúmenes de datos, los filtros deberían realizar consultas al backend. Esto implica modificar las funciones que obtienen los datos para que acepten parámetros de filtrado y los pasen a la API del backend (ej. `supabase.from('tasks').select('*').eq('status', filterStatus)`).
    - **Dashboard:** Los KPIs y gráficos del dashboard (ej. incidencias por técnico, tiempo medio de resolución) deberían calcularse en el backend o mediante consultas optimizadas a la base de datos para evitar la carga excesiva de datos en el cliente.
    - **Librerías de Gráficos:** Se pueden usar librerías como `Recharts` o `Chart.js` para visualizar los datos del dashboard.

## 3. Carga de Datos Masiva o Vía Archivo (Funcionalidad Futura)

- Esta funcionalidad no está implementada.
- **Sugerencia de Implementación:**
    1.  Un componente de UI para subir archivos (CSV/Excel).
    2.  Parseo del archivo en el frontend (usando librerías como `papaparse` para CSV o `xlsx` para Excel).
    3.  Envío de los datos parseados a un endpoint específico en el backend.
    4.  El backend se encargaría de validar e insertar los datos en la base de datos de forma masiva.
- **Tecnologías:** `papaparse`, `xlsx`.

## 4. Seguridad y Protección de Datos (RGPD)

- **Acción Requerida (con Backend):**
    - **Cifrado:** Asegurarse de que la conexión a la base de datos sea segura (SSL/TLS). Si se almacenan datos sensibles, considerar el cifrado a nivel de campo en la base de datos.
    - **Gestión Segura de Roles:** Implementar RLS (Row Level Security) si se usa Supabase/PostgreSQL, o lógica de autorización robusta en el backend.
    - **RGPD:**
        - Obtener consentimiento explícito para el tratamiento de datos.
        - Permitir a los usuarios acceder, rectificar y eliminar sus datos.
        - Implementar políticas de retención de datos.
        - Anonimización/pseudoanonimización si es necesario.
        - **Es crucial consultar a un experto legal para el cumplimiento total del RGPD.**

## 5. Resumen Tecnológico y Próximos Pasos

- **Frontend:** React 18, Vite, TailwindCSS, shadcn/ui, Framer Motion.
- **Persistencia Actual:** `localStorage` (temporal).
- **Sugerencias para Backend y Servicios Adicionales:**
    - **Base de Datos y Autenticación:** Supabase (PostgreSQL + Auth) o Firebase (Firestore/RTDB + Auth).
    - **Alternativa Backend:** API REST personalizada (Node.js/Express + MySQL/PostgreSQL) con un servicio de identidad como Auth0 o Clerk.
    - **Notificaciones:** Firebase Cloud Messaging (FCM) o servicios de email/SMS (ej. SendGrid, Twilio) integrados con el backend.
    - **Carga de Archivos:** Supabase Storage, Firebase Storage, o almacenamiento en el servidor si es un backend propio.

**Prioridad Inmediata:**
1.  **Seleccionar e integrar una solución de backend (Base de Datos + Autenticación).** Supabase es una excelente opción para empezar rápidamente.
2.  **Refactorizar `useDataManagement.js` para usar el backend.**
3.  **Implementar la lógica de roles y permisos en `App.jsx` y en las consultas al backend.**

¡Esperamos que esta guía sea de utilidad para el equipo de Hostinger!
