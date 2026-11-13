# Lista de tareas con cuentas de usuario

Aplicación web de lista de tareas en la que cada persona tiene su propia cuenta: se registra, inicia sesión y solo ve sus propias tareas. El proyecto se construye con **desarrollo guiado por especificaciones**: primero se acuerdan las reglas, después se escribe el backlog y las historias, se descompone la primera historia en tareas con su especificación y solo entonces se escribe el código, una tarea por rama.

El repositorio es un **monorepo** con tres piezas:

| Pieza | Qué es | Carpeta |
|---|---|---|
| Frontend | Lo que la persona ve y toca en el navegador | `frontend/` |
| API | Las reglas del negocio expuestas por HTTP | `backend/` |
| Base de datos | Donde se guardan usuarios y tareas | `database/` |

Además, `backlog/` guarda la idea, las historias de usuario y las especificaciones de cada tarea.

---

## Stack tecnológico

| Componente | Tecnología | Versión mínima |
|---|---|---|
| Lenguaje | Python | 3.10 (desarrollado con 3.12) |
| Framework de la API | Flask | 3.x |
| Base de datos | SQLite (módulo `sqlite3` de la biblioteca estándar) | 3 |
| Frontend | HTML, CSS y JavaScript sin framework (`fetch`) | Navegador moderno |
| Gestor de dependencias | pip + entorno virtual (`venv`) y `requirements.txt` | La de Python |
| Pruebas de la API | pytest | 8.x |
| Pruebas de pantalla | Recorrido manual en el navegador, escrito paso a paso | — |
| Control de versiones | Git + GitHub | — |

---

## Decisiones del proyecto

Cada decisión se tomó con un criterio concreto, no por preferencia.

### 1. Lenguaje y framework de la API: Python + Flask
- **Qué:** la API se escribe en Python con Flask.
- **Criterio:** Flask es mínimo, sin piezas obligatorias que la IA tenga que decidir por su cuenta. Python es el lenguaje que asumen las herramientas que exige la actividad (pytest y entorno virtual), así que no hace falta adaptar ninguna regla.

### 2. Base de datos: SQLite
- **Qué:** una base SQLite en un solo archivo, accedida con el módulo `sqlite3` que ya trae Python.
- **Criterio:** no necesita instalar ni levantar un servidor, no agrega dependencias y permite usar una base aparte para las pruebas con solo cambiar la ruta del archivo.

### 3. Frontend: HTML, CSS y JavaScript con `fetch`
- **Qué:** páginas HTML estáticas que llaman a la API con `fetch`. Flask sirve esas páginas desde `frontend/`, así frontend y API comparten la misma dirección.
- **Criterio:** sin compilación, sin Node y sin servicios extra. Al servirse desde el mismo origen no hay que configurar CORS, y la sesión por cookie funciona directamente.

### 4. Estructura del monorepo
- **Qué:** cuatro carpetas en la raíz: `frontend/`, `backend/`, `database/` y `backlog/`.
- **Criterio:** separa lo que ve la persona, las reglas, los datos y la especificación. Cada tarea del login recorre las tres primeras.

### 5. Gestor de dependencias y herramienta de pruebas: pip + venv y pytest
- **Qué:** las dependencias se instalan en un entorno virtual con pip y se listan en `requirements.txt`. Las pruebas se escriben con pytest.
- **Criterio:** venv aísla las dependencias del sistema. pytest es la herramienta que pide la actividad y el cliente de pruebas de Flask permite probar la API sin levantar el servidor.

### 6. Arquitectura de la API: tres capas
| Capa | Carpeta | Responsabilidad |
|---|---|---|
| Rutas | `backend/rutas/` | Reciben la petición HTTP, validan el formato, llaman al servicio y devuelven la respuesta. Sin lógica de negocio. |
| Servicios | `backend/servicios/` | Aplican las reglas de negocio (por ejemplo, correo único o longitud de la contraseña). No saben nada de HTTP ni de SQL. |
| Repositorios | `backend/repositorios/` | Son el único lugar que lee y escribe en SQLite. |

- **Regla de dependencia:** `rutas → servicios → repositorios`. Una capa solo llama a la siguiente; nunca al revés ni saltándose capas.
- **Criterio:** cada capa tiene una sola responsabilidad, la lógica se prueba sin HTTP y cambiar la base de datos solo afecta a los repositorios.

### 7. Estrategia de git
- **Rama principal:** `main`. No se hacen commits ni cambios directos en ella, salvo el commit inicial que crea el repositorio.
- **Una rama por tarea**, creada desde `main`:
  - `feat/tN-nombre` para código (por ejemplo `feat/t1-registro`).
  - `docs/nombre` para documentación (por ejemplo `docs/backlog`).
- **Un commit por tarea.**
- **Fusión a `main`:** solo con autorización explícita de la persona responsable después de revisar el diff. Claude Code no fusiona.
- **Criterio:** el historial queda revisable y cada tarea se puede revertir por separado.

### 8. Formato de los mensajes de commit: Conventional Commits
- **Formato:** `tipo: descripción corta en minúsculas`.

| Tipo | Uso | Ejemplo |
|---|---|---|
| `feat` | Funcionalidad nueva | `feat: registro de usuario` |
| `fix` | Corrección de un error | `fix: rechaza correo duplicado` |
| `docs` | Documentación, backlog y especificaciones | `docs: backlog e historias completas` |
| `test` | Pruebas | `test: criterios de inicio de sesión` |
| `chore` | Estructura o configuración | `chore: estructura inicial del monorepo` |

- **Criterio:** es el formato que usa la guía de la actividad y deja ver el propósito de cada commit de un vistazo.

---

## Reglas de pruebas

1. Cada criterio de la API tiene al menos una prueba con pytest.
2. Cada criterio de pantalla tiene un recorrido manual en el navegador, escrito paso a paso.
3. La prueba se escribe antes y se ve fallar antes de implementar.
4. Nunca se borra ni se debilita una prueba para que pase.
5. Las pruebas usan una base de datos aparte, no la de la aplicación.

---

## Estructura de carpetas

```
ListaDeTareas/
├── CLAUDE.md            # reglas del proyecto para Claude Code
├── README.md
├── .gitignore
├── requirements.txt     # dependencias de Python (se crea en la fase 4)
├── backlog/             # idea, backlog, historias y especificaciones
├── frontend/            # páginas HTML, estilos y JavaScript
├── backend/
│   ├── rutas/           # capa HTTP
│   ├── servicios/       # reglas de negocio
│   ├── repositorios/    # acceso a SQLite
│   └── pruebas/         # pruebas con pytest
└── database/            # esquema SQL (el archivo .db local no se versiona)
```

---

## Instalación y ejecución

> Pendiente: se completará cuando exista el código (fase 4). Se espera algo así:

```bash
python -m venv venv
venv\Scripts\activate          # Windows (en Linux/macOS: source venv/bin/activate)
pip install -r requirements.txt
python -m pytest               # ejecutar las pruebas
flask --app backend.app run    # levantar la aplicación
```

---

## Dependencias instaladas

Toda dependencia nueva se anota aquí con el motivo.

| Dependencia | Motivo | Tarea |
|---|---|---|
| _Ninguna todavía_ | | |

---

## Estado

En construcción: fase 1 (fundamentos del proyecto). El login todavía no está implementado.
