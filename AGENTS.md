# turnos-odontologia — Instrucciones para Agentes

> Este archivo (y su copia `CLAUDE.md`) es lo PRIMERO que todo agente lee al entrar al repo.
> Generado a partir de `knowledge-base/` y `CHANGES.md`. No editar a mano sin re-sincronizar ambos archivos.

---

## Stack Tecnológico

| Capa | Tecnología | Versión mínima |
|---|---|---|
| Lenguaje | Python | 3.12 |
| Backend | FastAPI | 0.110 |
| Datos | SQLite + SQLAlchemy 2.x + Alembic | SQLAlchemy 2.0 / Alembic 1.13 |
| Frontend | Jinja2 + HTMX (server-rendered, sin SPA) | Jinja2 3.1 / HTMX 1.9 |
| Auth staff | Sesiones server-side + bcrypt | bcrypt 4.x |
| Scheduler | APScheduler embebido (recordatorios) | 3.10 |
| Email | SMTP propio configurable por env | — |
| Tests | pytest + httpx (TDD estricto desde día 1) | pytest 8.x |

Migración futura a Postgres sin reescribir: capa de datos desacoplada (ORM + Alembic) desde el día 1.

Detalle completo: [knowledge-base/02_descripcion_general.md](knowledge-base/02_descripcion_general.md)

---

## Base de Conocimiento

La fuente de verdad del dominio vive en `knowledge-base/`. **Leé el archivo relevante ANTES de implementar.**

| Archivo | Cuándo leerlo |
|---|---|
| [01_vision_y_objetivos.md](knowledge-base/01_vision_y_objetivos.md) | Entender propósito y alcance |
| [03_actores_y_roles.md](knowledge-base/03_actores_y_roles.md) | Auth, RBAC, permisos |
| [04_modelo_de_datos.md](knowledge-base/04_modelo_de_datos.md) | Entidades, ERD, migraciones |
| [05_reglas_de_negocio.md](knowledge-base/05_reglas_de_negocio.md) | Reglas codificadas (RN-01…RN-10) |
| [06_funcionalidades.md](knowledge-base/06_funcionalidades.md) | Historias de usuario por épica (US-001…US-014) |
| [07_flujos_principales.md](knowledge-base/07_flujos_principales.md) | Flujos E2E |
| [08_arquitectura_propuesta.md](knowledge-base/08_arquitectura_propuesta.md) | Patrones, estructura, env vars |
| [10_preguntas_abiertas.md](knowledge-base/10_preguntas_abiertas.md) | ⚠️ Inconsistencias a resolver ANTES de codear |

> ⚠️ Resolver las preguntas de prioridad **Alta** de `10_preguntas_abiertas.md` (canal push propio, set de estados del odontograma) antes de arrancar el primer change.

---

## Skills Disponibles

| Agente | Rol | Skills que carga |
|---|---|---|
| **Backend Core** | FastAPI / SQLAlchemy / Alembic / migraciones | `fastapi-python`, `python-design-patterns`, `supabase-postgres-best-practices`, `postgresql-table-design`, `sql-optimization-patterns` |
| **Calidad / Testing** | TDD estricto, E2E, review | `python-testing-patterns`, `tdd`, `playwright-best-practices`, `code-review` |
| **Frontend** | Jinja / HTMX server-rendered | `frontend-design`, `playwright-best-practices` |
| **Transversal** | Seguridad, async, recordatorios | `security-and-hardening`, `async-python-patterns` |
| **Orquestación** | OPSX / SDD | `openspec-propose`, `openspec-apply-change`, `openspec-archive-change` (project) |

Cargá la skill correspondiente al contexto ANTES de escribir código.

> Los compact rules de cada skill los resuelve el orquestador desde `.atl/skill-registry.md` (generado por `skill-registry`; no versionado — no está en el repo). Esta tabla solo mapea skill→rol.

---

## Roadmap de Changes

El plan de implementación completo está en [CHANGES.md](CHANGES.md). Resumen:

- **Total**: 12 changes en 6 fases (0 Cimientos · 1 Auth+Admin · 2 Agenda staff · 3 Portal paciente · 4 Recordatorios+Clínica · 5 Cierre v1).
- **Camino crítico**: `C-01 foundation-setup → C-02 core-models → C-03 auth-staff-sesiones → C-04 admin-catalogo → C-05 agenda-turnos-staff → C-07 disponibilidad-reserva-online → C-08 gestion-token-paciente → C-12 cierre-v1`.
- **Primer change**: `C-01 foundation-setup`.

**Antes de cualquier `/opsx:propose`**: leé [CHANGES.md](CHANGES.md), identificá las dependencias del change y los archivos de "Leer antes".

---

## Reglas Duras

> No hay `~/.claude/CLAUDE.md` global en este entorno: las universales viven acá también. Son contrato; romperlas es un defecto. Formato `NUNCA X → hacer Y`:

- **R1.** NUNCA SQL crudo en routers/servicios → pasar por SQLAlchemy (capa desacoplada, migrable a Postgres).
- **R2.** NUNCA cambiar modelos sin migración Alembic → `revision` + `upgrade` probado.
- **R3.** NUNCA schema Pydantic de entrada sin `extra="forbid"` → rechazar campos no declarados.
- **R4.** NUNCA crear/reprogramar/cancelar turno sin validar RN-01…RN-05 → cada regla con su test.
- **R5.** NUNCA exponer datos de paciente sin sesión staff válida o token vigente → verificar auth en cada ruta.
- **R6.** Python: `snake_case` + type hints en todo código nuevo.
- **R7.** NUNCA commitear ni pushear sin pedido explícito → commits convencionales (`tipo(alcance): descripción`), sin co-autoría IA.
- **R8.** TDD estricto: test que falla primero, mínimo código para pasar, sin asserts triviales.
- **R9.** Archivos siempre UTF-8 sin BOM (Windows) → verificar tras escribir.
- **R10.** NUNCA adivinar artefactos OpenSpec → `openspec status` primero; no crear estructura `openspec/` a mano.

---

## Flujo de Trabajo

```
1. Leer la KB relevante (knowledge-base/)        → entender el dominio
2. Identificar el change en CHANGES.md           → respetar dependencias
3. /opsx:propose <nombre-del-change>             → proposal + design + specs + tasks
4. Implementar las tasks (cargando skills)       → respetando las reglas duras
5. /opsx:archive <nombre-del-change> + marcar [x] → cerrar el change
```

Aplicar TODAS las reglas duras en cada paso. Ante conflicto entre la KB y este archivo, las reglas duras prevalecen.
