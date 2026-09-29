# CHANGES — Secuencia de Implementación

> Índice canónico de todos los changes del proyecto **turnos-odontologia**.
> Cada change es atómico: un agente puede implementarlo en una sesión (~4-6 horas).
> **Leer este archivo antes de ejecutar cualquier `/opsx:propose`.**

---

## Cómo usar este documento

1. Identificar el change a implementar (verificar que sus dependencias están en `openspec/changes/archive/`).
2. Leer los docs de la knowledge-base indicados en "Leer antes".
3. Ejecutar `/opsx:propose <nombre-del-change>`.
4. Al terminar el change, archivarlo con `/opsx:archive <nombre-del-change>`.
5. Marcar el checkbox `[x]` en este archivo.

---

## Árbol de dependencias

```
C-01 foundation-setup
  └── C-02 core-models
        └── C-03 auth-staff-sesiones          ← desbloquea TODO lo demás
              │
              ├── C-04 admin-catalogo
              │     └── C-05 agenda-turnos-staff
              │           │
              │           ├── C-06 agenda-semanal-reprogramacion
              │           │
              │           ├── C-07 disponibilidad-reserva-online
              │           │     └── C-08 gestion-token-paciente
              │           │
              │           └── C-09 recordatorios-propios
              │
              └── C-10 ficha-odontograma      ← paralelo con C-04
                    └── C-11 agenda-odontologo-prestacion-realizada  ← + C-05
                          │
C-12 cierre-v1 ◄──────────┴── C-06 + C-08 + C-09 + C-11 ✓
```

### Paralelismo por fase

> Cada "gate" es un punto de sincronización. Los changes dentro de un grupo pueden ejecutarse en paralelo.

```
GATE 0: ninguna
  → C-01 foundation-setup (solo)

GATE 1: C-01 ✓
  → C-02 core-models (solo)

GATE 2: C-02 ✓
  → C-03 auth-staff-sesiones (solo)

GATE 3: C-03 ✓                     ← FORK (2 paralelos)
  → C-04 admin-catalogo            [Agente A]
  → C-10 ficha-odontograma         [Agente C]

GATE 4: C-04 ✓
  → C-05 agenda-turnos-staff       [Agente A]

GATE 5: C-05 ✓                     ← FORK (4 paralelos)
  → C-06 agenda-semanal-reprogramacion   [Agente A]
  → C-07 disponibilidad-reserva-online   [Agente B]
  → C-09 recordatorios-propios           [Agente C]
  → C-11 agenda-odontologo-prestacion-realizada  [Agente C — si C-10 ✓]

GATE 6: C-07 ✓
  → C-08 gestion-token-paciente    [Agente B]

GATE 7: C-06 + C-08 + C-09 + C-11 ✓
  → C-12 cierre-v1 (solo)
```

### Camino crítico (8 changes — mínimo irreducible)

```
C-01 → C-02 → C-03 → C-04 → C-05 → C-07 → C-08 → C-12
```

### Plan óptimo con 3 agentes

```
Paso │ Agente A (Backend Core)            │ Agente B (Portal Paciente)           │ Agente C (Clínica + Recordatorios)
─────┼────────────────────────────────────┼──────────────────────────────────────┼──────────────────────────────────────
  1  │ C-01 foundation-setup              │         —                            │         —
  2  │ C-02 core-models                   │         —                            │         —
  3  │ C-03 auth-staff-sesiones           │         —                            │         —
  4  │ C-04 admin-catalogo                │         —                            │ C-10 ficha-odontograma
  5  │ C-05 agenda-turnos-staff           │         —                            │         —
  6  │ C-06 agenda-semanal-reprogramacion │ C-07 disponibilidad-reserva-online   │ C-09 recordatorios-propios
  7  │         —                          │ C-08 gestion-token-paciente          │ C-11 agenda-odontologo-prestacion-realizada
  8  │ C-12 cierre-v1                     │         —                            │         —
```

---

## FASE 0 — Cimientos

### [C-01] `foundation-setup`
- **Estado**: `[ ]` pendiente
- **Scope**: Scaffolding completo del monolito modular + infraestructura base
  - Estructura `app/` según KB: `main.py`, `core/`, `turnos/`, `pacientes/`, `fichas/`, `auth/`, `recordatorios/`, `templates/`, `static/`, `db/`, `tests/`
  - `app/main.py`: FastAPI app mínima con `GET /api/health`, montaje de routers vacíos y hook de scheduler
  - `app/core/config.py`: settings por env (`DATABASE_URL`, `SECRET_KEY`, `SESSION_EXPIRE_MIN`, `TOKEN_EXPIRE_HOURS`, `SMTP_*`, `REMINDER_HOURS_BEFORE`, `BASE_URL`)
  - `app/core/security.py`: helpers bcrypt + sesiones server-side (vacío funcional, firma lista)
  - `app/core/deps.py`: dependencias `get_db`, `usuario_actual`, `require_rol`
  - `app/db/base.py` + Alembic inicializado (`app/db/versions/`), `DATABASE_URL=sqlite:///./turnos.db` por defecto
  - `pyproject.toml`: fastapi, sqlalchemy 2.x, alembic, jinja2, htmx (static), bcrypt, apscheduler, pytest + httpx
  - `.env.example` con las 10 variables de `08_arquitectura_propuesta.md` §Variables de entorno
  - Layout Jinja base (`templates/base.html`) + HTMX vía `static/`
  - Tests: `test_health.py` (GET /api/health → 200), TDD de referencia
- **Dependencias**: ninguna
- **Governance**: BAJO
- **Leer antes**:
  - `knowledge-base/01_vision_y_objetivos.md` (alcance v1 y fuera de alcance)
  - `knowledge-base/02_descripcion_general.md` §Stack tecnológico
  - `knowledge-base/08_arquitectura_propuesta.md` §Estructura de directorios
  - `knowledge-base/08_arquitectura_propuesta.md` §Variables de entorno
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-01, §DD-02, §DD-03

---

### [C-02] `core-models`
- **Estado**: `[ ]` pendiente
- **Scope**: Modelos SQLAlchemy + migraciones iniciales + repositorios genéricos + seed
  - Modelos: `Profesional`, `Sillon`, `Prestacion`, `Paciente`, `Turno`, `TokenGestion`, `Recordatorio`, `Ficha`, `EstadoPieza`, `PrestacionRealizada`, `UsuarioStaff` (+ tabla `sesiones` server-side)
  - Constraints a nivel modelo: matrícula única, `duracion_min > 0`, email formato válido, `fin = inicio + duracion`, pieza FDI 11-18/21-28/31-38/41-48, set cerrado de estados (RN-08)
  - Índices: `profesional(matricula, activo)`, `sillon(activo)`, `turno(profesional_id,inicio)`, `turno(sillon_id,inicio)`, `turno(paciente_id,inicio)`, `turno(estado)`, `token(token único)`, `recordatorio(estado,programado_para)`, `ficha(paciente_id único)`
  - Capa base: `BaseRepository[T]` + `UnitOfWork` por dominio (patrón capas de `08_arquitectura_propuesta.md`)
  - Migración Alembic 001: todas las tablas core (disciplina DD-02 desde día 1)
  - Seed demo/tests: 1 recepcionista + 1 odontólogo (bcrypt), 2 profesionales, 2 sillones, 5 prestaciones (limpieza 30, consulta 30, obturación 60, endodoncia 90, extracción 60)
  - Tests: creación de cada entidad, constraints (matrícula duplicada, FDI inválido, ficha duplicada por paciente), índices presentes
- **Dependencias**: C-01
- **Governance**: CRITICO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Entidades
  - `knowledge-base/04_modelo_de_datos.md` §ERD
  - `knowledge-base/04_modelo_de_datos.md` §Seed data inicial
  - `knowledge-base/08_arquitectura_propuesta.md` §Patrones aplicados
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-02

---

## FASE 1 — Auth staff + Administración

> C-04 y C-10 pueden proponerse en paralelo tras C-03 (dominios independientes).

### [C-03] `auth-staff-sesiones`
- **Estado**: `[ ]` pendiente
- **Scope**: Login/logout staff con sesiones server-side + RBAC (US-014)
  - `POST /staff/login` (username + password, bcrypt verify, crea sesión server-side con expiración `SESSION_EXPIRE_MIN=480`)
  - `POST /staff/logout` (borra sesión, revocación inmediata)
  - `GET /staff/me` (usuario actual + rol + profesional asociado)
  - `PermissionContext`: `require_rol("recepcionista")`, `require_rol("odontologo")`, lectura vs escritura odontograma (RN-10)
  - Templates Jinja: `templates/staff/login.html` (form + HTMX error parcial)
  - Migración 002: tabla `sesiones` (id, usuario_id, expira_en, creada_en) si no incluida en 001
  - Tests: login correcto, password incorrecta, usuario inactivo 403, sesión expirada 401, RBAC por ruta (recepcionista vs odontólogo), hash nunca en plaintext
- **Dependencias**: C-02
- **Governance**: CRITICO
- **Leer antes**:
  - `knowledge-base/03_actores_y_roles.md` §RBAC — Matriz de permisos
  - `knowledge-base/05_reglas_de_negocio.md` §Excepciones globales (bcrypt, sesiones)
  - `knowledge-base/06_funcionalidades.md` §US-014
  - `knowledge-base/07_flujos_principales.md` §Flujo 2 (paso login)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-07

---

### [C-04] `admin-catalogo`
- **Estado**: `[ ]` pendiente
- **Scope**: CRUD administrativo de profesionales, sillones y prestaciones (US-013)
  - Modelos ya en C-02; aquí servicio + router + UI: `turnos/service.py` validación RN-06 (no agendar sobre inactivos)
  - Endpoints staff (recepcionista): `GET/POST /admin/profesionales`, `GET/POST /admin/sillones`, `GET/POST /admin/prestaciones` + activar/desactivar (soft, `activo=false`, sin borrar historial)
  - Endpoints lectura (odontólogo): `GET /admin/profesionales`, `/admin/prestaciones` solo lectura
  - Templates HTMX: `templates/admin/catalogo.html` + parciales por tabla (crear/editar inline sin recarga)
  - Migración: ninguna nueva (usa tablas C-02); si falta columna `activo`, migración 003 mínima
  - Tests: CRUD por rol (odontólogo 403 en escritura), desactivar no borra turnos pasados, recurso inactivo rechaza turno (422, RN-06)
- **Dependencias**: C-03
- **Governance**: BAJO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Profesional, §Sillon, §Prestacion
  - `knowledge-base/05_reglas_de_negocio.md` §RN-05, §RN-06
  - `knowledge-base/06_funcionalidades.md` §US-013
  - `knowledge-base/03_actores_y_roles.md` §RBAC — Matriz de permisos

---

## FASE 2 — Agenda staff (núcleo)

### [C-05] `agenda-turnos-staff`
- **Estado**: `[ ]` pendiente
- **Scope**: Agenda diaria por profesional + alta de turnos staff con validación completa (US-001, US-004)
  - Servicio `turnos/service.py`: `crear_turno()` valida RN-01 (solape profesional + sillón), RN-02 (cero sobreturnos), RN-05 (`fin = inicio + prestacion.duracion_min`), RN-06 (recursos activos)
  - Endpoints staff: `GET /agenda/dia?profesional_id=&fecha=` (turnos con hora, paciente, prestación, sillón, estado; cancelados distinguidos, no ocupan disponibilidad — CA US-001)
  - `POST /agenda/turnos` (origen `staff`: profesional + sillón + prestación + paciente + inicio; 409 con turno en conflicto sin datos sensibles)
  - Templates HTMX: `templates/staff/agenda_dia.html` + parcial `turno_row.html`
  - Estados Turno: `reservado/confirmado/cancelado/realizado/ausente`; alta staff crea `reservado` o `confirmado`
  - Tests parametrizados TDD: solapamientos borde (fin==inicio ajeno OK, 1 min overlap 409), sillón cruzado, sobreturno, duración por prestación, RN-04 no aplica a staff (hueco existente)
- **Dependencias**: C-03, C-04
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Turno
  - `knowledge-base/05_reglas_de_negocio.md` §RN-01, §RN-02, §RN-05, §RN-06
  - `knowledge-base/06_funcionalidades.md` §US-001, §US-004
  - `knowledge-base/07_flujos_principales.md` §Flujo 2
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-06

---

### [C-06] `agenda-semanal-reprogramacion`
- **Estado**: `[ ]` pendiente
- **Scope**: Vista semanal + cancelar/reprogramar staff (US-002, US-005)
  - `GET /agenda/semana?profesional_id=&semana=` (bloques según `duracion_min`) + `GET /agenda/semana/sillon?sillon_id=` (detección colisiones box)
  - `POST /agenda/turnos/{id}/cancelar` (respeta RN-03 ≥48h; staff sin excepción salvo motivo registrado — ver pregunta abierta)
  - `POST /agenda/turnos/{id}/reprogramar` (nuevo inicio ± nueva prestación; revalida RN-01 y recalcula fin RN-05)
  - Si paciente tiene email: envía confirmación/cancelación por SMTP propio (reúsa mailer de C-09 si ya existe; si no, stub que C-09 completa)
  - Templates HTMX: `templates/staff/agenda_semana.html` (grid profesional + grid sillón)
  - Tests: cancelación <48h → 422, reprogramación con solape → 409, cambio de prestación recalcula fin, vistas semanales descuentan cancelados
- **Dependencias**: C-05
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/05_reglas_de_negocio.md` §RN-01, §RN-03, §RN-05
  - `knowledge-base/06_funcionalidades.md` §US-002, §US-005
  - `knowledge-base/07_flujos_principales.md` §Flujo 2 (pasos 3-5)
  - `knowledge-base/10_preguntas_abiertas.md` (excepción RN-03/RN-04 staff)

---

## FASE 3 — Portal paciente (reserva sin cuenta)

> C-07 y C-09 pueden proponerse en paralelo tras C-05 (uno es portal, otro es scheduler).

### [C-07] `disponibilidad-reserva-online`
- **Estado**: `[ ]` pendiente
- **Scope**: Disponibilidad pública + reserva online con token por email (US-006, US-007, Flujo 1 pasos 1-5)
  - `GET /reservar` (portal: elige profesional + prestación + fecha) + `GET /disponibilidad?profesional_id=&prestacion_id=&fecha=` (solo huecos reales: descuenta turnos, respeta duración RN-05, oculta <72h RN-04, sin datos de otros pacientes)
  - `POST /turnos` (público: nombre + teléfono + email obligatorio RN-07; revalida hueco anti-carrera en transacción; crea Paciente si nuevo + Turno `reservado` + TokenGestion expira `TOKEN_EXPIRE_HOURS`; 409 si carrera, 422 si <72h o email inválido)
  - Email con token vía SMTP propio (`BASE_URL/mis-turnos?token=...`); SMTP caído → turno creado igual + email pendiente de reintento (Flujo 1 caso de error)
  - Scheduler hook: al crear turno programa recordatorio (fila `Recordatorio` pendiente; worker en C-09)
  - Templates: `templates/portal/reservar.html`, `templates/portal/disponibilidad_parcial.html`, `templates/portal/confirmacion.html`
  - Tests: huecos exactos por duración, corte 72h borde, carrera simultánea (2 POST mismo hueco → 1×201 + 1×409), email inválido 422 sin crear turno
- **Dependencias**: C-05
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/03_actores_y_roles.md` §Rutas públicas
  - `knowledge-base/05_reglas_de_negocio.md` §RN-04, §RN-07, §RN-09
  - `knowledge-base/06_funcionalidades.md` §US-006, §US-007
  - `knowledge-base/07_flujos_principales.md` §Flujo 1
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-08

---

### [C-08] `gestion-token-paciente`
- **Estado**: `[ ]` pendiente
- **Scope**: Gestión de turnos propios con token vigente (US-008, Flujo 1 pasos 6-7)
  - `GET /mis-turnos?token=...` (solo turnos del token; token vencido/desconocido → 403 sin revelar datos ajenos + mensaje de reemisión)
  - `POST /mis-turnos/{id}/cancelar?token=...` (exige ≥48h RN-03)
  - `POST /mis-turnos/{id}/reprogramar?token=...` (nuevo hueco; revalida RN-01/RN-04/RN-05; reemite token al reprogramar RN-09)
  - `POST /mis-turnos/reemitir` (email → nuevo token 48-72h; valor operativo default 72h hasta cerrar pregunta abierta)
  - Templates: `templates/portal/mis_turnos.html` + parciales cancelar/reprogramar
  - Tests: token ajeno no ve turnos, vencido 403 + reemisión OK, cancelar <48h 422, reprogramar a hueco ocupado 409
- **Dependencias**: C-07
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §TokenGestion
  - `knowledge-base/05_reglas_de_negocio.md` §RN-03, §RN-04, §RN-09
  - `knowledge-base/06_funcionalidades.md` §US-008
  - `knowledge-base/07_flujos_principales.md` §Flujo 1 (casos de error token)
  - `knowledge-base/10_preguntas_abiertas.md` (expiración 48h vs 72h)

---

## FASE 4 — Recordatorios + Clínica

### [C-09] `recordatorios-propios`
- **Estado**: `[ ]` pendiente
- **Scope**: Scheduler embebido + SMTP propio + estado de envíos (US-009, Flujo 4)
  - `recordatorios/mailer.py`: envío SMTP por env (`SMTP_HOST/PORT/USER/PASSWORD/FROM`), sin datos clínicos en el cuerpo, log sin PII excedente
  - `recordatorios/scheduler.py`: job APScheduler por tick — busca `pendientes` con `programado_para <= ahora` y turno en reservado/confirmado; envía; marca `enviado(+enviado_en)` o `fallido` (reintento acotado); turno cancelado → descarta
  - Anticipación `REMINDER_HOURS_BEFORE` (default 24h hasta cerrar pregunta abierta simple vs doble aviso)
  - Endpoint staff lectura: `GET /staff/recordatorios?estado=` (recepcionista ve pendientes/enviados/fallidos)
  - Programación al crear turno (hook usado por C-05/C-07): `programado_para = inicio - REMINDER_HOURS_BEFORE`
  - Tests: programa al crear, envía pendientes, no envía cancelados, SMTP caído → fallido + reintento, sin datos clínicos en email
- **Dependencias**: C-05
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Recordatorio
  - `knowledge-base/06_funcionalidades.md` §US-009
  - `knowledge-base/07_flujos_principales.md` §Flujo 4
  - `knowledge-base/08_arquitectura_propuesta.md` §Patrones aplicados (scheduler embebido)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-04, §DD-05

---

### [C-10] `ficha-odontograma`
- **Estado**: `[ ]` pendiente
- **Scope**: Ficha básica + odontograma mínimo interactivo (US-010, US-011)
  - Servicio `fichas/service.py` (RN-08, RN-10): una ficha por paciente; `actualizada_en` + autor en cada cambio
  - Endpoints: `GET /fichas/{paciente_id}` (ficha + odontograma actual), `POST /fichas/{paciente_id}` (antecedentes/alergias; odontólogo escribe, recepcionista 403 en escritura clínica), `POST /fichas/{paciente_id}/piezas` (pieza FDI permanente + estado del set cerrado)
  - RBAC: odontólogo lectura + escritura; recepcionista lectura (odontograma) + CRUD administrativo de paciente; paciente sin acceso
  - Templates HTMX: `templates/staff/ficha.html` + `odontograma.svg/parcial` (32 piezas clicables, 5 estados con color)
  - Tests: pieza fuera de FDI → 422, estado inválido → 422, recepcionista escribe → 403, una ficha por paciente, autor+fecha registrados
- **Dependencias**: C-03
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Ficha, §EstadoPieza
  - `knowledge-base/05_reglas_de_negocio.md` §RN-08, §RN-10
  - `knowledge-base/06_funcionalidades.md` §US-010, §US-011
  - `knowledge-base/07_flujos_principales.md` §Flujo 3 (pasos 1-3)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-11

---

### [C-11] `agenda-odontologo-prestacion-realizada`
- **Estado**: `[ ]` pendiente
- **Scope**: Agenda propia del odontólogo + registro de prestación realizada (US-003, US-012, Flujo 3 pasos 4-5)
  - `GET /staff/mi-agenda?fecha=` (solo turnos del profesional logueado vía `profesional_id`; acceso directo a ficha por turno — CA US-003)
  - `POST /fichas/{paciente_id}/prestaciones` (prestación + nota + turno vinculado; autor = odontólogo del turno; recepcionista por excepción registra — RN-10; marca turno `realizado` y libera agenda)
  - Template: `templates/staff/mi_agenda.html` (reúsa parciales de C-05/C-10)
  - Tests: odontólogo solo ve sus turnos, registro crea `PrestacionRealizada` + turno pasa a `realizado`, recepcionista escribe odontograma → 403 pero puede registrar prestación por excepción
- **Dependencias**: C-05, C-10
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §PrestacionRealizada, §Turno
  - `knowledge-base/05_reglas_de_negocio.md` §RN-10
  - `knowledge-base/06_funcionalidades.md` §US-003, §US-012
  - `knowledge-base/07_flujos_principales.md` §Flujo 3
  - `knowledge-base/03_actores_y_roles.md` §RBAC — Matriz de permisos

---

## FASE 5 — Cierre v1

### [C-12] `cierre-v1`
- **Estado**: `[ ]` pendiente
- **Scope**: Endurecimiento v1, seed productivo y verificación E2E (sin post-v1)
  - Validación global: toda escritura valida RN-01…RN-09 antes de persistir; errores 409/422/403 sin exponer datos de otros pacientes (auditoría Flujo 1-4 casos de error)
  - Seed productivo (C-02 demo → real): recepcionista + odontólogos del consultorio, profesionales/sillones/prestaciones reales; sin pacientes ni turnos de ejemplo en producción
  - Estados `ausente` y `confirmado`: transición recepcionista (no-show y confirmación manual) + descarte de recordatorios en cancelados
  - Tests E2E pytest+httpx: Flujo 1 completo (reservar → token → mis-turnos → reprogramar), Flujo 2 (login → agenda → crear → cancelar), Flujo 3 (ficha → odontograma → prestación → realizado), Flujo 4 (recordatorio enviado)
  - Docs operativos: `README` despliegue (env, Alembic upgrade, APScheduler, SMTP), verificación `REMINDER_HOURS_BEFORE` y `TOKEN_EXPIRE_HOURS` operativos
  - Explícito NO v1: pagos/señas, WhatsApp API, obras sociales, reportes avanzados, laboratorio, temporal/periodontograma (DD-10)
  - Tests: suite E2E verde, cobertura RN-01…RN-10, checklist preguntas abiertas Alta respondidas o diferidas a Sprint 2
- **Dependencias**: C-06, C-08, C-09, C-11
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/01_vision_y_objetivos.md` §Alcance v1, §Fuera de alcance, §Métricas de éxito
  - `knowledge-base/05_reglas_de_negocio.md` §Excepciones globales
  - `knowledge-base/07_flujos_principales.md` (los 4 flujos, casos de error)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-10
  - `knowledge-base/10_preguntas_abiertas.md` (las 7, priorizadas)
