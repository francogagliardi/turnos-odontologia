# Skill Registry

**Delegator use only.** Any agent that launches sub-agents reads this registry to resolve compact rules, then injects them directly into sub-agent prompts. Sub-agents do NOT read this registry or individual SKILL.md files.

Proyecto: turnos-odontologia (Python + FastAPI + SQLite + Jinja + HTMX, monolito modular, TDD estricto, migrable a Postgres via ORM + Alembic).

## User Skills

| Trigger | Skill | Path |
|---------|-------|------|
| FastAPI Python development, APIs and async operations | fastapi-python | C:\Users\Franco Gagliardi\.agents\skills\fastapi-python\SKILL.md |
| Writing Python tests, test suites, pytest, fixtures, mocking, TDD | python-testing-patterns | C:\Users\Franco Gagliardi\.agents\skills\python-testing-patterns\SKILL.md |
| Test-first features/bugfixes, red-green-refactor, integration tests | tdd | C:\Users\Franco Gagliardi\.agents\skills\tdd\SKILL.md |
| Slow queries, schema design, EXPLAIN analysis, indexing, N+1 | sql-optimization-patterns | C:\Users\Franco Gagliardi\.agents\skills\sql-optimization-patterns\SKILL.md |
| New UI or reshaping existing UI, aesthetic direction, typography | frontend-design | C:\Users\Franco Gagliardi\.agents\skills\frontend-design\SKILL.md |
| Playwright tests, flaky tests, POM, CI/CD, auth, a11y, E2E flows | playwright-best-practices | C:\Users\Franco Gagliardi\.agents\skills\playwright-best-practices\SKILL.md |
| New service/component design, layering, refactor God class, composition vs inheritance | python-design-patterns | C:\Users\Franco Gagliardi\.agents\skills\python-design-patterns\SKILL.md |
| Review branch/PR/WIP since fixed point, Standards vs Spec axes | code-review | C:\Users\Franco Gagliardi\.agents\skills\code-review\SKILL.md |
| Asyncio, concurrent programming, async APIs, I/O-bound apps | async-python-patterns | C:\Users\Franco Gagliardi\.agents\skills\async-python-patterns\SKILL.md |
| Anything living in Postgres: tables, columns, schema, migrations, RLS, indexes, triggers, functions, EXPLAIN, connections, locking | supabase-postgres-best-practices | C:\Users\Franco Gagliardi\.agents\skills\supabase-postgres-best-practices\SKILL.md |
| Designing or reviewing a PostgreSQL-specific schema | postgresql-table-design | C:\Users\Franco Gagliardi\.agents\skills\postgresql-table-design\SKILL.md |
| Input-handler audit, user input, auth, sessions, external integrations, OWASP Top Ten, PII/GDPR | security-and-hardening | C:\Users\Franco Gagliardi\.agents\skills\security-and-hardening\SKILL.md |

## Compact Rules

Pre-digested rules per skill. Delegators copy matching blocks into sub-agent prompts as `## Project Standards (auto-resolved)`.

### fastapi-python
- Router-service-schemas: routers delgados, logica en services/repositorios, validacion en Pydantic v2 BaseModel, nunca dicts crudos.
- Type hints en todas las firmas; `def` para puro, `async def` para I/O (DB/API); RORO (recibe objeto, retorna objeto).
- DI de FastAPI (Depends) para sesion DB, auth y permisos; lifespan context manager para startup/shutdown.
- Errores esperados con HTTPException + response_model tipado; guard clauses + early return, happy path al final, sin else innecesarios.
- Nunca I/O bloqueante en path async; lazy loading para datasets grandes; nombrar archivos `routers/user_routes.py` (minusculas + underscores).
- Rutas: router exportado, sub-rutas, utils, tipos (models/schemas) al final del modulo.

### python-testing-patterns
- Estructura AAA (Arrange/Act/Assert); tests independientes, sin estado compartido, cada test limpia lo suyo.
- Nombres `test_<unidad>_<escenario>_<esperado>` (ej. `test_create_turno_solapado_rechaza_409`); layout `tests/` con `conftest.py`, `test_unit/`, `test_integration/`, `test_e2e/`.
- Fixtures compartidas en conftest.py; `@pytest.mark.parametrize` para RN-01..RN-10; markers `@slow`/`@integration` y `-m "not slow"` en loop rapido.
- APIs FastAPI con httpx TestClient/TestClient async; mockear SMTP/APScheduler con `unittest.mock` (side_effect para reintentos).
- Tiempo con freezegun (`@freeze_time`) para anticipacion 48/72h y recordatorios 24h; nunca `time.sleep` real en tests.
- Coverage con pytest-cov (`--cov-fail-under=80 --cov-report=term-missing`); cobertura significativa, no solo porcentaje.

### tdd
- Red antes que green: un test fallando primero, luego solo el codigo minimo para pasar; nada especulativo ni features futuras.
- Un slice vertical por ciclo (un seam, un test, una implementacion tracer-bullet); nunca escribir todos los tests primero (horizontal slicing prohibido).
- Testear solo en seams publicos acordados (interfaces/API); preguntar "cual es la interfaz publica y que seams testeamos" antes de escribir.
- Expected values de fuente independiente (literal conocido, ejemplo trabajado, spec RN): nunca recomputar como el codigo ni snapshots tautologicos.
- Nombres de test en lenguaje del dominio (GLOSSARY.md si existe); respetar ADRs del area tocada.
- Refactor fuera del loop red->green; va en etapa review (skill code-review), no dentro del ciclo.

### sql-optimization-patterns
- Leer `EXPLAIN (ANALYZE, BUFFERS)`: Seq Scan = sospechoso en tablas grandes; Index/Index-Only Scan = objetivo; mirar costo, rows y actual time.
- Indices B-Tree por defecto; compuestos con regla leftmost-prefix y mas selectivo primero (ej. `(profesional_id, inicio)` para disponibilidad).
- Indexar siempre FKs y claves de join; parciales para subsets calientes (`WHERE estado = 'activo'`); covering con INCLUDE para agenda diaria sin tocar tabla.
- Nunca `SELECT *`; nunca funcion sobre columna indexada en WHERE sin indice funcional; `LIKE '%x'` no usa indice.
- Filtrar antes de joinear; evitar N+1 (joins/batching); cada indice frena INSERT/UPDATE: indexar selectivamente.
- Monitorear `pg_stat_statements` (slow queries) y `pg_stat_user_tables` (seq_scan alto = indice faltante); `ANALYZE` regular; tipos chicos = mejor performance.

### frontend-design
- Portal publico + staff server-rendered (Jinja + HTMX): una decision memorable por vista, resto quieto y disciplinado; nada de SaaS-card-kit generico.
- Hero con lo mas caracteristico del consultorio (reserva en 1 clic); tipografia 1-2 familias deliberadas, lineas < 80 chars, sentence case, voz activa ("Reservar", no "Enviar").
- Nunca: una sola palabra acentuada en headlines, labels ALL-CAPS, eyebrow labels en cada heading, `A - B - C` con middle dots, `->` en botones.
- Estructura visual = informacion: bordes/numeracion solo si el contenido es secuencia real; motion solo como respuesta a accion HTMX (abrir/expandir/confirmar).
- Copy desde el usuario ("Mis turnos", no "Gestion de entidades"); errores/estados vacios con direccion ("Elegi otro horario"), nunca vagos ni con disculpas.
- Piso de calidad sin anunciarlo: responsive mobile, foco de teclado visible, reduced-motion, contraste accesible; criticar con screenshot si hay browser.

### playwright-best-practices
- Localizadores por rol/accesibilidad (`getByRole`, `getByLabel`), nunca selectores CSS fragiles ni waits explicitos; assertions con auto-wait.
- POM para flujos reserva/reprogramacion/cancelacion + agenda HTMX; fixtures para setup/teardown y datos aislados por worker.
- Auth una vez en setup global (storageState por rol: paciente/recepcionista/odontologo); reutilizar, no loguear en cada test.
- Tiempo con clock-mocking para ventanas 48/72h y recordatorios; network interception para SMTP propio; tags `@smoke`/`@critical` y `--grep` para subsets.
- Flaky: aislar estado, `test.describe.serial` solo si es imprescindible, repetir criticos `--repeat-each=5`; debug con trace viewer.
- Validar con `npx playwright test --reporter=list`; solo avanzar con todo verde; CI con sharding + reporte de artefactos.

### python-design-patterns
- KISS: la solucion mas simple que funciona; complejidad solo con requerimiento concreto; regla de tres antes de abstraer.
- SRP + capas estrictas: API -> Service -> Repository; service nunca importa de handlers; tipos compartidos en capa models.
- Composicion sobre herencia; funciones chicas (20-50 lineas, un proposito); inyeccion por constructor para testabilidad.
- Explicito sobre astuto: dict de dispatch (`FORMATTERS = {...}`) antes que factory/registry; borrar codigo muerto antes de abstraer.
- Test por capa aislada (con python-testing-patterns); si un constructor supera 7 params, partir la clase, no parchear DI.
- Composicion plana (2-3 niveles); si dos capas divergen peligrosamente, extraer ya + test del comportamiento compartido.

### code-review
- Dos ejes separados, nunca mezclar: Standards (repo + smell baseline) vs Spec (CHANGES.md / C-XX / RN); reportar bajo `## Standards` y `## Spec`.
- Fijar punto base primero: `git diff <base>...HEAD` (tres puntos) + `git log <base>..HEAD --oneline`; verificar ref y diff no vacio antes de lanzar sub-agentes.
- Spec source en orden: refs en commits (#123), path pasado por usuario, archivo en `docs/`/`specs/`/`.scratch/` que matchee branch; si no hay, Spec reporta "no spec available".
- Standards = repo docs (si existen) + smell baseline Fowler (Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession, Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality, Message Chains, Middle Man, Refused Bequest); repo siempre gana al baseline; smells = judgement calls.
- Lanzar ambos sub-agentes en paralelo (< 400 palabras c/u); citar archivo + regla por hallazgo; ignorar lo que ya cubre tooling.
- Cierre con una linea: totales por eje + peor issue de cada eje; no elegir un ganador global.

### async-python-patterns
- Regla de oro: totalmente sync o totalmente async dentro de un call path; mezclar crea bloqueo oculto.
- FastAPI + SQLAlchemy async para concurrencia; CPU-bound a `multiprocessing` o `asyncio.to_thread()`; scripts simples = sync.
- Concurrencia con `asyncio.gather(*tasks)`; background (APScheduler/SMTP recordatorios) con `asyncio.create_task` + cancelacion limpia (`CancelledError` + `raise`).
- Nunca `time.sleep` en async (usar `await asyncio.sleep`); nunca olvidar `await` (retorna coroutine sin ejecutar).
- Timeouts con `asyncio.wait_for(coro, timeout=...)`; errores con `gather(..., return_exceptions=True)` + filtrado de exitos/fallos.
- Tests async con pytest-asyncio (`@pytest.mark.asyncio`); recursos con `async with` / `async for`.

### supabase-postgres-best-practices
- Cargar ANTES de tocar cualquier cosa que viva en la DB, aunque sea una columna o un query: schema, migraciones, RLS, indices, triggers, funciones, jobs.
- Prioridad: query performance + connections + RLS (CRITICAL), luego schema (HIGH), locking (MEDIUM-HIGH); leer el rule file de `references/` del caso.
- RLS: politica por rol/tabla + test que verifique filas visibles por tenant; nunca logica de tenant solo en app.
- Conexiones: pooling obligatorio (PgBouncer/Supavisor); nunca conexion nueva por request; vigilar exhaustion y timeouts.
- Migraciones declarativas versionadas (Alembic en este proyecto); nunca DDL manual divergente; restores con `pg_restore`, imports validados.
- Diagnostico con EXPLAIN, `pg_stat_statements`, locks y bloat; vector/pg_cron/pgmq solo si el caso lo pide (LOW).

### postgresql-table-design
- PK `BIGINT GENERATED ALWAYS AS IDENTITY` (tablas ref: Turno/Paciente/Ficha); UUID solo si hace falta unicidad global/opacidad; toda tabla de negocio con PK.
- Normalizar a 3NF primero; desnormalizar solo con lectura medida que lo justifique; `NOT NULL` + `DEFAULT` donde aplique.
- Tipos: `TIMESTAMPTZ` tiempo, `NUMERIC` dinero, `TEXT` (+`CHECK (LENGTH<=n)`) strings, `BOOLEAN NOT NULL`, `JSONB` (no JSON) solo opcional/semiestructurado con GIN.
- Nunca `timestamp` sin TZ, `varchar(n)`/`char(n)`, `money`, `serial`; enums solo sets chicos estables, resto `TEXT+CHECK` o lookup table.
- FKs con `ON DELETE/UPDATE` explicito + indice manual en columna referenciante (PG no lo crea); `UNIQUE NULLS NOT DISTINCT` si un solo NULL.
- Anti-solapamiento con `EXCLUDE USING gist (profesional WITH =, periodo WITH &&)`; `snake_case` sin quoted identifiers; gaps de secuencia e heap/MVCC son normales.
- Constraints: `CHECK` + `NOT NULL` juntos (`precio > 0`); composite leftmost-prefix; partial/expression/covering segun access path real.

### security-and-hardening
- Threat model de 5 minutos primero: trust boundaries (HTTP/forms/webhooks/ outputs), activos (credenciales/PII/turnos/admin), STRIDE + abuse case como primer test.
- Siempre: validar input en boundary con schema Pydantic (422 estructurado); queries parametrizadas; auto-escape (nunca `innerHTML`/f-strings a SQL/shell); HTTPS; bcrypt>=12/argon2; cookies `httpOnly+secure+sameSite=lax/strict`; headers (helmet/CSP `default-src 'self'`); CORS allowlist explicita.
- AuthZ en cada request (IDOR: el usuario debe poseer el recurso); errores genericos al cliente, detalle solo en logs; strip `passwordHash`/tokens en respuestas.
- Nunca: secretos en git/logs, validacion solo cliente, `eval()`, tokens en localStorage, stack traces al usuario, `*` CORS con credenciales.
- Secretos desde entorno (`.env.example` con placeholders, `.env*` gitignored); secreto que llega a remoto = rotar primero, luego purgar historial.
- Rate limit general + auth estricto (~10/15min) con store compartido; uploads con allowlist MIME + tope + magic bytes; SSRF con allowlist host + rechazo RFC1918/link-local.
- PII (Ley 26.529): clasificar, minimo necesario, TTL + borrado real (incluye backups/caches); auditoria nativa del lockfile antes de cada release.

## Project Conventions

| File | Path | Notes |
|------|------|-------|
| (none) | — | CLAUDE.md / AGENTS.md aun no existen (los genera agent-instruction); seccion arranca vacia como esperado |

Leer los archivos listados arriba para patrones del proyecto. No hay indice que expandir en esta fase.
