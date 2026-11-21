# SITP · Sistema de Trazabilidad de Proyectos de Investigación Formativa (UCC Medellín)

Prototipo del Formato E8 (Grupo 04).
- **Backend:** API REST con **Flask (Python)** + **MySQL** (`mysql-connector-python`), como **monolito modular por capas**.
- **Frontend:** HTML, CSS y **JavaScript vanilla** (módulos ES, sin frameworks) organizado como un proyecto React.

## Ejecutar en local
1. Tener MySQL 8 (o MariaDB) encendido.
2. Instalar dependencias y configurar la conexión:
```bash
cd backend
python -m venv .venv
.venv\Scripts\Activate.ps1          # Windows  (Mac/Linux: source .venv/bin/activate)
pip install -r requirements.txt
copy .env.example .env              # Mac/Linux: cp .env.example .env  -> edita DB_PASSWORD
python main.py                      # http://localhost:8000
```
Al arrancar, el backend crea la base `sitp_ucc`, las tablas (`app/database/schema.sql`) y los datos de prueba si la base está vacía.
La guía paso a paso está en el PDF «Guía de instalación y ejecución del SITP».

## Usuarios de prueba (datos ficticios) — contraseña: `Ucc2026*`
| Rol global | Correo | Rol dentro de cada proyecto |
|---|---|---|
| Estudiante | laura.restrepo@campusucc.edu.co · samuel.ortiz@campusucc.edu.co | Proyecto G-04 |
| Estudiante | valentina.rios@campusucc.edu.co | Proyecto G-02 (Civil) |
| Estudiante | mateo.zapata@campusucc.edu.co | Proyecto G-01 (finalizado) |
| Docente | carolina.mesa@ucc.edu.co | Técnica en G-04 y G-01 · **Jurado en G-02** |
| Docente | andres.gomez@ucc.edu.co | Jurado en G-04 y G-01 · **Técnico en G-02** |
| Docente | julian.arango@ucc.edu.co | Metodológico en todos |
| Administrador | admin.sitp@ucc.edu.co | Gestión de usuarios, catálogos y asignaciones |

## Roles (5)
La cuenta tiene un rol global (estudiante, docente o administrador). El docente toma un rol **contextual** según el proyecto: asesor técnico, asesor metodológico o jurado (columnas `asesor_tecnico_id`, `asesor_metodologico_id`, `jurado_id` de `proyectos`). La interfaz lo muestra como «Tu rol: …».
| Acción | Estudiante | Técnico | Metodológico | Jurado | Admin |
|---|---|---|---|---|---|
| Subir PDF de entregable | Sí | | | | |
| Revisar entregable | | Sí | Sí | | |
| Registrar acta con compromisos | | Sí | Sí | | |
| Validar firma del acta | | | Sí | | |
| Marcar compromiso cumplido | Sí | Sí | Sí | | |
| Comentar (hilo compartido) | Sí | Sí | Sí | | |
| Objetivos / editar proyecto | Sí | | Sí | | |
| Avanzar etapa / cancelar | | | Sí | | |
| Evaluar con rúbrica | | | | Sí | |
| Usuarios, catálogos y asignaciones | | | | | Sí |

El administrador ve todos los proyectos en solo lectura. Avanzar de etapa solo cambia `curso_actual`: no duplica registros.

## Estructura
```
backend/
  main.py                    punto de entrada (create_app)
  app/__init__.py            fábrica: registra el Blueprint de cada módulo
  app/core/                  config (.env), database, errors, security, validation, serializers, http
  app/database/              schema.sql y seed.py
  app/modules/<modulo>/      auth · proyectos · entregables · actas · objetivos · seguimiento · evaluacion · usuarios · catalogo
      routes.py              capa de rutas (URL + permisos por rol)
      service.py             capa de negocio (reglas)
      repository.py          capa de datos (SQL parametrizado)
      schemas.py             validación de entrada
frontend/
  index.html                 carga src/main.js
  public/img/                logo
  src/main.js · src/App.js   arranque y enrutamiento (como index.js / App.jsx)
  src/pages/                 LoginPage, PanelPage, ProyectosPage, AdminPage, proyecto/ProyectoPage + tabs/ (Info, Actas, Historial, Objetivos, Evaluacion)
  src/components/            ui/, layout/, filters/, comments/, entregables/, projects/ (componentes reutilizables)
  src/services/api.js        única puerta a la API Flask
  src/store/                 state.js (estado global) y selectors.js (datos derivados)
  src/core/ · src/utils/ · src/constants/ · src/styles/
```
Regla de dependencias: `routes → service → repository → core`. Un módulo usa a otro solo por su `service` o `repository` (por ejemplo, todos validan el acceso con `proyectos.service.accesible`).

## Detalles
- **Base de datos MySQL:** 18 tablas relacionales (usuarios, sesiones, materias, programas, semestres, criterios, entregables_def, proyectos, proyecto_estudiantes, objetivos, objetivo_versiones, actas, compromisos, entregables, revisiones, comentarios, historial, evaluaciones). Si encuentra una base de una versión anterior (sin `evaluaciones`), la recrea con los datos de prueba.
- **Archivos:** los PDF subidos se guardan en `backend/uploads/` (se crea solo) con nombre automático `G{grupo}_M{materia}_{E1|E2}_{nombre}_v{n}.pdf`. Los PDF de observaciones del asesor técnico se generan con `reportlab` (incluido en `requirements.txt`).
- **Endpoints:** `POST /api/auth/login` · `GET /api/panel` · `GET|PATCH /api/proyectos/<id>` · `POST /api/proyectos/<id>/{objetivos,actas,entregables,comentarios,avanzar,cancelar,evaluacion}` · `POST /api/actas/<id>/firma` · `PATCH /api/compromisos/<id>` · `POST /api/entregables/<id>/revisiones` · `GET /api/entregables/<id>/archivo` · `GET /api/revisiones/<id>/pdf` · `DELETE /api/evaluaciones/<id>` · `GET /api/catalogos` · `POST /api/catalogos/<programas|semestres>` · `PATCH /api/catalogos/<tipo>/<id>` · `GET|POST /api/admin/usuarios` · `PATCH /api/admin/usuarios/<id>` · `PATCH /api/admin/proyectos/<id>/asignacion`.
- **Rúbrica:** cada criterio se califica de 0 a 5; el total es el promedio ponderado por peso. El concepto «Observado / Plan de mejora» exige observaciones.
- **Seguridad:** contraseñas con PBKDF2-SHA256 + sal, sesiones guardadas en MySQL con expiración (8 h), consultas parametrizadas (sin inyección SQL), escape de HTML (sin XSS).
