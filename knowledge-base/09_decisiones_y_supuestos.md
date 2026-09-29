# Decisiones y Supuestos

## Decisiones documentadas

### DD-01 — Stack Python + FastAPI + SQLite + Jinja + HTMX
**Decisión**: Backend Python + FastAPI, datos SQLite, frontend Jinja + HTMX server-rendered.
**Contexto**: Proyecto académico (Programación 3), consultorio chico, sin plazo de entrega.
**Alternativas consideradas**: SPA separada (React/Vue); Postgres desde día 1; servicio de turnos externo (SaaS).
**Justificación**: Menor complejidad operativa, un solo deploy, sin API pública en v1; SQLite alcanza para el volumen.
**Trade-offs aceptados**: Migración futura a Postgres; sin tiempo real ni app móvil.

### DD-02 — Datos desacoplados: SQLAlchemy + Alembic (migrable a Postgres)
**Decisión**: Acceso a datos vía SQLAlchemy 2.x con migraciones Alembic desde día 1.
**Contexto**: Tensión entre SQLite (simple) y requisito de diseño escalable (discovery §9).
**Alternativas consideradas**: SQL crudo contra SQLite; Postgres directo desde día 1.
**Justificación**: Desacopla el dominio del motor; migrar a Postgres es cambiar `DATABASE_URL` + migración, sin reescribir servicios.
**Trade-offs aceptados**: Disciplina de migraciones obligatoria desde el primer cambio de schema.

### DD-03 — Monolito modular (`turnos`, `pacientes`, `fichas`, `auth`)
**Decisión**: Un deploy con paquetes por dominio y capas router → servicio → repositorio.
**Contexto**: Calidad prioritaria: mantenibilidad; equipo chico.
**Alternativas consideradas**: Microservicios; monolito sin modularizar.
**Justificación**: Las reglas (RN-01…RN-10) quedan testeables por servicio; el volumen no justifica distribución.
**Trade-offs aceptados**: Escalado vertical; extracción futura si un dominio lo exige.

### DD-04 — Scheduler embebido (APScheduler) para recordatorios
**Decisión**: Recordatorios con APScheduler en el mismo proceso.
**Contexto**: Sin proveedores externos en v1 (decisión de alcance); evitar colas/infra extra.
**Alternativas consideradas**: Celery + Redis; cron del sistema; servicio externo de emails.
**Justificación**: Cero infraestructura adicional para bajo volumen; suficiente para email programado.
**Trade-offs aceptados**: Si el proceso cae, los jobs caen con él; reintentos acotados; migrar a colas si el volumen crece.

### DD-05 — SMTP propio configurable por env
**Decisión**: Envío de tokens y recordatorios por SMTP propio (host/user/pass por variables de entorno).
**Contexto**: Recordatorios "con medios propios, sin proveedores externos en v1" (discovery §5).
**Alternativas consideradas**: SendGrid/Resend; WhatsApp Business API desde v1.
**Justificación**: Cierra el alcance v1 sin dependencias pagas ni integraciones; configurable por despliegue.
**Trade-offs aceptados**: Entregabilidad y rebotes a cargo del consultorio; WhatsApp queda post-v1.

### DD-06 — Tests pytest desde día 1 (TDD estricto)
**Decisión**: Desarrollo guiado por tests con pytest (+ httpx) desde el primer cambio.
**Contexto**: Reglas de agenda críticas (solapamientos, anticipaciones) donde un bug cuesta turnos reales.
**Alternativas consideradas**: Testear después; solo tests manuales.
**Justificación**: RN-01…RN-09 son propiedades ideales para tests parametrizados (solapes, bordes 48/72h).
**Trade-offs aceptados**: Velocidad inicial menor a cambio de regresión contenida.

### DD-07 — Auth staff por sesiones server-side + bcrypt
**Decisión**: Login staff con sesiones en servidor y hashes bcrypt; sin JWT ni OAuth en v1.
**Contexto**: Staff chico, mismo monolito que sirve el HTML.
**Alternativas consideradas**: JWT stateless; OAuth externo.
**Justificación**: Revocación inmediata (basta borrar la sesión), sin manejo de tokens en el cliente.
**Trade-offs aceptados**: Sesiones ligadas al servidor (sticky o store compartido si se escala horizontalmente).

### DD-08 — Paciente SIN cuenta, gestión con token por email (48-72h)
**Decisión**: El paciente reserva con nombre + teléfono + email y gestiona con token aleatorio por email con expiración 48-72h.
**Contexto**: Fricción mínima (patrón reserva-sin-registro, cf. Fresha en market-study §D); consultorio 1-3 profesionales.
**Alternativas consideradas**: Cuentas con contraseña; gestión solo telefónica; token sin expiración.
**Justificación**: Sin contraseñas que gestionar; el token con expiración acota la ventana de exposición.
**Trade-offs aceptados**: Dependencia del email (si no llega, el paciente llama); reemisión necesaria tras vencer.

### DD-09 — Sin rol admin separado en v1
**Decisión**: La recepcionista administra (profesionales, sillones, prestaciones, usuarios básicos).
**Contexto**: Consultorio 1-3 profesionales; simplificar RBAC v1.
**Alternativas consideradas**: Rol admin/owner separado.
**Justificación**: Menos roles = menos superficie de permisos para un equipo chico.
**Trade-offs aceptados**: Sin separación de deberes; se introduce admin si el consultorio crece o lo exige auditoría.

### DD-10 — Corte v1 cerrado (pagos, WhatsApp, OS, reportes y laboratorio fuera)
**Decisión**: Pagos/señas, WhatsApp API, obras sociales, reportes avanzados y laboratorio protésico quedan post-v1.
**Contexto**: Riesgo de descontrol de alcance (discovery §10); competidores cobran esos módulos aparte.
**Alternativas consideradas**: Incluir Mercado Pago o WhatsApp en v1.
**Justificación**: Cierra un MVP entregable: agenda + reserva + recordatorios + ficha/odontograma mínimo.
**Trade-offs aceptados**: Sin seña anti-ausentismo ni canal WhatsApp en v1 (el supuesto de adopción online queda sin probar).

### DD-11 — Odontograma mínimo v1 (32 permanentes, 5 estados)
**Decisión**: 32 piezas permanentes FDI con estados sano/cariado/obturado/ausente/en-tratamiento; sin temporal ni periodontograma.
**Contexto**: Pregunta abierta de discovery §11 cerrada a un mínimo implementable.
**Alternativas consideradas**: Odontograma completo (18 estados como DentalSoft); incluir temporal; periodontograma BOP/NIC.
**Justificación**: Cubre el registro básico de prestación; lo avanzado (perio, ortodoncia) es post-v1.
**Trade-offs aceptados**: El set de estados debe confirmarse con un odontólogo real (ver `10_preguntas_abiertas.md`).

## Supuestos inferidos

### SU-01 — Consultorio de 1-3 profesionales/sillones
**Supuesto**: El dimensionamiento (SQLite, scheduler embebido, sin multi-sucursal) alcanza para 1-3 profesionales y 1-3 sillones con portal de bajo volumen.
**Origen**: Discovery §1 y decisiones cerradas del usuario.
**Riesgo si es falso**: Degradación o necesidad de migrar a Postgres/colas antes de lo previsto.
**Cómo validar**: Medir turnos/día y concurrencia de reserva en las primeras semanas de uso.

### SU-02 — Token con expiración 48-72h es usable
**Supuesto**: Los pacientes gestionan sus turnos dentro de la ventana de 48-72h o aceptan reemitir el token.
**Origen**: Decisión cerrada del usuario (token con expiración).
**Riesgo si es falso**: Soporte manual por tokens vencidos; pacientes que prefieren llamar.
**Cómo validar**: Tasa de gestiones con token vencido vs. reemisiones en el primer mes.

### SU-03 — Pacientes adoptan la reserva online
**Supuesto**: Los pacientes prefieren reservar online en vez de seguir escribiendo por WhatsApp por costumbre.
**Origen**: Discovery §10 (riesgo explícito sin validar).
**Riesgo si es falso**: Portal subutilizado; la recepcionista sigue cargando todo.
**Cómo validar**: % de turnos de origen online vs. staff tras el lanzamiento; encuesta mínima a pacientes.

### SU-04 — Email basta como canal de token y recordatorio en v1
**Supuesto**: El email es suficiente para entregar tokens y recordatorios sin WhatsApp en v1.
**Origen**: "Recordatorios propios (email/push, sin proveedores externos)" + SMTP propio.
**Riesgo si es falso**: Baja lectura de emails → ausentes sin reducir.
**Cómo validar**: Tasa de apertura/entrega SMTP y ausentismo antes/después; decidir web push (ver `10_preguntas_abiertas.md`).
