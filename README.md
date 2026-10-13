# test-manager-library-3145556-infra

> **Prueba técnica · Ficha ADSO 3145556**
> Entrega: **martes 13 de octubre de 2026, a las 5:00 p. m.** (17:00, hora Colombia).

Este repositorio es la **infraestructura** de *Manager Library*: contenedores, red, volúmenes y el motor de base de datos vacío. **Empieza vacío a propósito**: se construye en el fork de cada aprendiz.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Entrega

Cuenta lo que quede **subido a los forks antes de las 5:00 p. m.** (17:00). Lo que llegue después no se revisa.
No hay prórroga: suban avances parciales con commits pequeños durante la prueba.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-manager-library-3145556-api`](https://github.com/code-sena/test-manager-library-3145556-api) | Backend en PHP (Laravel) |
| [`test-manager-library-3145556-app`](https://github.com/code-sena/test-manager-library-3145556-app) | Frontend en React |
| [`test-manager-library-3145556-page`](https://github.com/code-sena/test-manager-library-3145556-page) | Sitio público estático de presentación |
| [`test-manager-library-3145556-infra`](https://github.com/code-sena/test-manager-library-3145556-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-manager-library-3145556-tool`](https://github.com/code-sena/test-manager-library-3145556-tool) | Utilidades: sembrador de datos de demostración |

---

## Running the project

Requirements: Docker with Compose v2.

```bash
cp .env.example .env   # then set JWT_SECRET and ADMIN_PASSWORD
docker compose up --build
```

| Service | URL |
|---|---|
| Frontend (`app`) | http://localhost:5173 |
| API (`api`) | http://localhost:8000/api/v1 |
| PostgreSQL (`db`) | localhost:55433 (user/password/database `library`) |

The `db` service starts **empty**: no tables and no DDL. The API runs its Laravel migrations on startup
(ADR-001: the schema is owned only by the application migrations, never by `infra`).

### Verify that the database starts empty

```bash
docker compose up -d db
docker compose exec db psql -U library -d library -c "SELECT table_schema, table_name FROM information_schema.tables WHERE table_schema NOT IN ('pg_catalog', 'information_schema');"
```

The query must return zero rows. Then stop the project with `docker compose down`.

### Decisions

- The compose project name is `manager-library`. Other containers on the same host are not touched.
- The database is published on host port `55433` to avoid clashing with a local PostgreSQL on `5432`.
- `.env` is optional for `docker compose config`, but the `api` service needs it for `JWT_SECRET`, `ADMIN_*` and `CORS_ORIGIN`.

## Status

| Task | Status | Evidence |
|---|---|---|
| T-20 `docker compose up` brings up an empty database, API and frontend | Done | Fresh volume (`down -v` for project `manager-library` only), 13-oct ~15:20: `up -d db` -> `information_schema.tables` outside system schemas returned 0 rows. Then `docker compose -p manager-library up -d --build` built and started `db` (healthy), `api`, `app` in ~21 s. API migrations ran and created the admin; `GET http://localhost:8000/api/v1/health` -> 200 `{"status":"ok"}`; `GET http://localhost:5173/` -> 200 with the built `index.html` (`<title>Manager Library</title>`); schema `library` then holds `category, member, loan, loan_item, book, user`. |
