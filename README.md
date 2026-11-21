# FrontEnd SITP · Sistema de Trazabilidad de Proyectos de Investigación Formativa

Frontend desarrollado para la Universidad Cooperativa de Colombia (UCC - Campus Medellín).

## Arquitectura
Aplicación **SPA (Single Page Application)** desarrollada con **JavaScript Vanilla moderno (ES Modules)**, HTML5 y CSS3, adoptando un patrón de componentes y flujo de datos unidireccional similar a React:
- **`index.html`:** Única página HTML de la aplicación.
- **`src/main.js` y `src/App.js`:** Punto de entrada y orquestador del renderizado de vistas.
- **`src/core/`:** Enrutador cliente (`router.js`) y utilidades para manipulación reactiva del DOM (`dom.js`).
- **`src/store/`:** Estado global centralizado (`state.js`) y selectores de datos derivados (`selectors.js`).
- **`src/components/`:** Componentes reutilizables modulares (`layout/`, `ui/`, `filters/`, `comments/`, `entregables/`, `projects/`).
- **`src/pages/`:** Vistas completas de la aplicación:
  - `LoginPage.js`: Autenticación con token Bearer.
  - `PanelPage.js`: Dashboard principal adaptativo según el rol activo.
  - `AdminPage.js`: Módulo de administración descongestionado en 5 vistas independientes con Modal centralizado para gestión y multi-rol.
  - `ProyectosPage.js` y `proyecto/`: Detalle del proyecto, objetivos, actas, entregables y evaluación con rúbrica.
- **`src/services/api.js`:** Cliente HTTP unificado basado en `fetch()` que gestiona la comunicación con la API REST y el encabezado `Authorization: Bearer <token>`.
- **`src/styles/main.css`:** Sistema de diseño institucional UCC (colores teal, verde, lima y tipografías Inter / DM Serif Display).

## Funcionalidades Clave
1. **Sistema Multi-Rol y Role Switcher (Swap):** Selector en la barra de navegación que permite al usuario conmutar entre sus roles asignados (Estudiante, Jurado, Asesor Técnico, Asesor Metodológico).
2. **Modo Super-Acceso del Administrador:** Permite al administrador utilizar el Swap para operar como cualquier rol sobre todos los proyectos institucionales sin filtros de pertenencia.
3. **Módulo Administrativo Descongestionado:** 
   - 5 vistas dedicadas: *Gestión de Usuarios*, *Cursos*, *Programas y Semestres*, *Entregables por Curso* y *Auditoría global*.
   - **Modal centralizado:** Creación con múltiples roles simultáneos, edición, desactivación (soft delete) y eliminación.

## Conexión con el Backend
Por defecto se conecta al backend en `http://localhost:8000`. Puede configurarse mediante la variable global `window.__API_BASE__` en `index.html`.
