# Descripción General

## Stack tecnológico

| Capa | Tecnologías | Versión mínima |
|---|---|---|
| Lenguaje | Python | 3.12 |
| Backend | FastAPI | 0.110 |
| Datos | SQLite + SQLAlchemy 2.x + Alembic | SQLAlchemy 2.0 / Alembic 1.13 |
| Frontend | Jinja2 + HTMX (server-rendered, sin SPA) | Jinja2 3.1 / HTMX 1.9 |
| Auth staff | Sesiones server-side + bcrypt | bcrypt 4.x |
| Scheduler | APScheduler embebido (recordatorios) | 3.10 |
| Email | SMTP propio configurable por env | — |
| Tests | pytest + httpx (TDD estricto desde día 1) | pytest 8.x |

Migración futura a Postgres sin reescribir: la capa de datos va desacoplada (ORM + migraciones versionadas con Alembic) desde el día 1.

## Arquitectura general

Monolito modular (ver `08_arquitectura_propuesta.md`): un solo deploy FastAPI con paquetes por dominio (`turnos`, `pacientes`, `fichas`, `auth`) y plantillas Jinja + HTMX servidas por el mismo backend. Sin frontend separado ni servicios externos obligatorios en v1.

```
[Portal público] ─┐
                   ├─→ FastAPI (monolito modular) ─→ SQLite (SQLAlchemy/Alembic)
[Staff Sesiones] ─┘         │ └→ APScheduler ─→ SMTP propio
                            └→ Jinja + HTMX (mismo proceso)
```

Justificación: consultorio 1-3 profesionales/sillones y portal público de bajo volumen → un monolito es más mantenible (calidad prioritaria) que microservicios; HTMX evita una SPA; el scheduler embebido evita infraestructura de colas en v1.

## Integraciones externas

| Servicio | Propósito | Tipo |
|---|---|---|
| SMTP propio (configurable) | Envío de tokens de gestión y recordatorios | SMTP (env) |
| (Ninguna obligatoria en v1) | — | — |

Post-v1 (no v1): WhatsApp Business API, Mercado Pago (pagos/señas), obras sociales, ARCA.

## API REST (si aplica)

El backend expone rutas HTML (Jinja + HTMX) y endpoints JSON para HTMX/validaciones, agrupados por recurso:

- `auth`: login/logout staff por sesiones; sin registro público.
- `profesionales` / `sillones` / `prestaciones`: CRUD administrativo (recepcionista).
- `disponibilidad`: horarios libres por profesional/prestación/fecha (público, solo lectura).
- `turnos`: crear (público + staff), reprogramar/cancelar (token paciente o staff), agenda diaria/semanal (staff).
- `pacientes` / `fichas` / `odontograma`: ficha básica y estados por pieza (staff según rol).
- `recordatorios`: disparo y estado de envíos (interno, scheduler).
