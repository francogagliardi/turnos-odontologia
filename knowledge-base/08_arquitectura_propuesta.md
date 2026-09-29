# Arquitectura Propuesta

Monolito modular: un solo deploy FastAPI con paquetes por dominio, plantillas Jinja + HTMX en el mismo proceso, SQLite vía SQLAlchemy + Alembic, scheduler embebido y SMTP propio. Calidad prioritaria: mantenibilidad.

## Patrones aplicados

| Patrón | Dónde se usa | Por qué |
|---|---|---|
| Monolito modular | Paquetes `turnos`, `pacientes`, `fichas`, `auth` | Bajo volumen (staff chico + portal público); evita complejidad distribuida |
| Capas (router → servicio → repositorio) | Cada paquete de dominio | Reglas (RN-01…RN-10) testeables sin HTTP; TDD estricto |
| ORM + migraciones versionadas | SQLAlchemy 2.x + Alembic | Migrable a Postgres sin reescribir |
| Server-rendered + HTMX | Jinja2 + fragmentos parciales | Sin SPA ni API pública en v1 |
| Scheduler embebido | APScheduler en el mismo proceso | Sin colas/infra extra para recordatorios v1 |
| Sesiones server-side + RBAC | `auth` | Staff chico; sin JWT ni OAuth en v1 |

## Estructura de directorios

```
turnos-odontologia/
├── app/
│   ├── main.py                 # FastAPI app, montaje de routers y scheduler
│   ├── core/
│   │   ├── config.py           # settings por env
│   │   ├── security.py         # bcrypt + sesiones server-side
│   │   └── deps.py             # dependencias (DB, usuario actual, roles)
│   ├── turnos/
│   │   ├── router.py           # agenda, disponibilidad, reserva, token
│   │   ├── service.py          # RN-01…RN-06, RN-09
│   │   ├── models.py           # Turno, TokenGestion, Recordatorio
│   │   └── schemas.py
│   ├── pacientes/
│   │   ├── router.py
│   │   ├── service.py          # RN-07
│   │   ├── models.py           # Paciente
│   │   └── schemas.py
│   ├── fichas/
│   │   ├── router.py
│   │   ├── service.py          # RN-08, RN-10
│   │   ├── models.py           # Ficha, EstadoPieza, PrestacionRealizada
│   │   └── schemas.py
│   ├── auth/
│   │   ├── router.py           # login/logout staff
│   │   ├── service.py
│   │   └── models.py           # UsuarioStaff + sesiones
│   ├── recordatorios/
│   │   ├── scheduler.py        # APScheduler jobs
│   │   └── mailer.py           # SMTP propio
│   ├── templates/              # Jinja2 (portal + staff)
│   ├── static/                 # CSS/HTMX
│   └── db/
│       ├── base.py
│       └── versions/           # migraciones Alembic
├── tests/                      # pytest (TDD desde día 1)
├── knowledge-base/
├── discovery/
└── pyproject.toml
```

## Seguridad

- Autenticación: staff con usuario + contraseña (bcrypt), sesiones server-side con expiración; paciente con token aleatorio por email con expiración 48-72h (sin contraseña).
- Autorización: RBAC por rol en cada ruta staff (matriz en `03_actores_y_roles.md`); token solo autoriza sus propios turnos.
- Validación de input: schemas estrictos (Pydantic/FastAPI) + validación de RN-01…RN-09 en la capa de servicio; errores sin filtrar datos ajenos.
- Secrets management: todo secreto y credencial SMTP solo por variables de entorno; nunca en código ni en la KB.

## Variables de entorno

| Variable | Descripción | Ejemplo | Sensible |
|---|---|---|---|
| `DATABASE_URL` | URL de la base (SQLite v1, Postgres futuro) | `sqlite:///./turnos.db` | N |
| `SECRET_KEY` | Clave de firma de sesiones | `cambiar-en-produccion` | Y |
| `SESSION_EXPIRE_MIN` | Minutos de expiración de sesión staff | `480` | N |
| `TOKEN_EXPIRE_HOURS` | Horas de vigencia del token paciente | `72` | N |
| `SMTP_HOST` | Host SMTP propio | `mail.consultorio.com` | N |
| `SMTP_PORT` | Puerto SMTP | `587` | N |
| `SMTP_USER` | Usuario SMTP | `turnos@consultorio.com` | Y |
| `SMTP_PASSWORD` | Clave SMTP | `***` | Y |
| `SMTP_FROM` | Remitente de tokens y recordatorios | `turnos@consultorio.com` | N |
| `REMINDER_HOURS_BEFORE` | Anticipación del recordatorio (horas) | `24` | N |
| `BASE_URL` | URL pública (links con token en emails) | `https://turnos.consultorio.com` | N |
