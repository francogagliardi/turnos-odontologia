# Funcionalidades

Organizadas por **épica** y luego por **historia de usuario** (formato US-NNN).

## Épica 1: Agenda multi-profesional

### US-001 — Ver agenda diaria por profesional
**Como** recepcionista
**Quiero** ver la agenda diaria de cada profesional (con sillón y estado de cada turno)
**Para** gestionar los turnos del consultorio sin solapamientos

**Criterios de aceptación**:
- [ ] CA-1: La vista diaria muestra todos los turnos del profesional con hora, paciente, prestación, sillón y estado.
- [ ] CA-2: Los turnos cancelados se distinguen visualmente y no ocupan disponibilidad.

**Reglas relacionadas**: RN-01, RN-06

### US-002 — Ver agenda semanal
**Como** recepcionista
**Quiero** ver la agenda semanal (por profesional y sillón)
**Para** detectar huecos y planificar la semana

**Criterios de aceptación**:
- [ ] CA-1: Vista semana por profesional con bloques según duración de prestación.
- [ ] CA-2: Vista semana por sillón para detectar colisiones de box.

**Reglas relacionadas**: RN-01, RN-05, RN-06

### US-003 — Ver mi agenda del día
**Como** odontólogo
**Quiero** ver mi agenda del día
**Para** saber a quién atiendo y en qué sillón

**Criterios de aceptación**:
- [ ] CA-1: Solo muestra los turnos del profesional logueado.
- [ ] CA-2: Acceso directo a la ficha del paciente de cada turno.

**Reglas relacionadas**: RN-10

### US-004 — Crear turno desde staff
**Como** recepcionista
**Quiero** crear un turno (profesional + sillón + prestación + paciente + hora)
**Para** agendar reservas telefónicas/presenciales

**Criterios de aceptación**:
- [ ] CA-1: Rechaza solapamientos de profesional o sillón con mensaje claro.
- [ ] CA-2: Rechaza sobreturnos en cualquier caso.
- [ ] CA-3: El fin se calcula desde la duración de la prestación.

**Reglas relacionadas**: RN-01, RN-02, RN-05, RN-06

### US-005 — Cancelar/reprogramar desde staff
**Como** recepcionista
**Quiero** cancelar o reprogramar cualquier turno
**Para** mantener la agenda actualizada

**Criterios de aceptación**:
- [ ] CA-1: Respeta la anticipación mínima de 48h (RN-03).
- [ ] CA-2: Reprogramar revalida solapamientos y recalcula fin si cambia la prestación.

**Reglas relacionadas**: RN-01, RN-03, RN-05

## Épica 2: Reserva online (portal paciente, sin cuenta)

### US-006 — Ver horarios libres
**Como** paciente
**Quiero** ver los horarios libres de un profesional para una prestación
**Para** reservar sin llamar ni escribir por WhatsApp

**Criterios de aceptación**:
- [ ] CA-1: Muestra solo huecos reales (descuenta turnos existentes y respeta duración).
- [ ] CA-2: No ofrece horarios dentro de las próximas 72h.
- [ ] CA-3: No expone datos de otros pacientes.

**Reglas relacionadas**: RN-01, RN-02, RN-04, RN-05

### US-007 — Reservar turno online
**Como** paciente
**Quiero** reservar un turno con nombre + teléfono + email
**Para** asegurar mi atención sin cuenta

**Criterios de aceptación**:
- [ ] CA-1: Crea el turno y envía token de gestión por email (expiración 48-72h).
- [ ] CA-2: Rechaza reservas con <72h de anticipación.
- [ ] CA-3: Rechaza el hueco si otro paciente lo ocupó en simultáneo (condición de carrera).

**Reglas relacionadas**: RN-04, RN-07, RN-09

### US-008 — Gestionar mis turnos con token
**Como** paciente
**Quiero** ver/cancelar/reprogramar mis turnos con el token del email
**Para** liberar o mover mi horario sin llamar

**Criterios de aceptación**:
- [ ] CA-1: Token vigente permite ver solo los turnos asociados.
- [ ] CA-2: Cancelar/reprogramar exige ≥48h de anticipación.
- [ ] CA-3: Token vencido → mensaje de reemisión, sin filtrar datos ajenos.

**Reglas relacionadas**: RN-03, RN-04, RN-09

## Épica 3: Recordatorios propios

### US-009 — Recibir recordatorio por email
**Como** paciente
**Quiero** recibir un recordatorio de mi turno
**Para** no olvidarlo y reducir ausentes

**Criterios de aceptación**:
- [ ] CA-1: Se programa automáticamente al crear el turno (scheduler embebido).
- [ ] CA-2: Se envía por SMTP configurable; el fallo queda en estado `fallido` con reintento acotado.
- [ ] CA-3: No se envía para turnos cancelados; no incluye datos clínicos.

**Reglas relacionadas**: RN-07

## Épica 4: Ficha clínica y odontograma mínimo

### US-010 — Ficha básica del paciente
**Como** odontólogo
**Quiero** consultar y actualizar la ficha básica (antecedentes, alergias)
**Para** atender con contexto clínico mínimo

**Criterios de aceptación**:
- [ ] CA-1: Una ficha por paciente, visible según RBAC.
- [ ] CA-2: Cambios con fecha y autor registrado.

**Reglas relacionadas**: RN-10

### US-011 — Odontograma mínimo interactivo
**Como** odontólogo
**Quiero** marcar el estado de cada pieza permanente
**Para** registrar la situación dental actual

**Criterios de aceptación**:
- [ ] CA-1: 32 piezas permanentes (FDI) con estados sano/cariado/obturado/ausente/en-tratamiento.
- [ ] CA-2: Sin dentición temporal ni periodontograma.
- [ ] CA-3: La recepcionista solo lee.

**Reglas relacionadas**: RN-08, RN-10

### US-012 — Registrar prestación realizada
**Como** odontólogo
**Quiero** registrar la prestación realizada en la ficha (vinculada al turno)
**Para** dejar constancia clínica y marcar el turno como realizado

**Criterios de aceptación**:
- [ ] CA-1: Registra prestación + nota + autor + fecha.
- [ ] CA-2: El turno pasa a `realizado` y libera la agenda.

**Reglas relacionadas**: RN-10

## Épica 5: Administración y roles

### US-013 — Administrar profesionales, sillones y prestaciones
**Como** recepcionista
**Quiero** crear/editar/desactivar profesionales, sillones y prestaciones (con duración)
**Para** mantener la oferta del consultorio actualizada

**Criterios de aceptación**:
- [ ] CA-1: Desactivar no borra historial ni turnos pasados.
- [ ] CA-2: No se agenda sobre recursos inactivos.

**Reglas relacionadas**: RN-05, RN-06

### US-014 — Login staff por sesiones
**Como** recepcionista u odontólogo
**Quiero** iniciar/cerrar sesión con usuario y contraseña
**Para** acceder según mi rol

**Criterios de aceptación**:
- [ ] CA-1: Contraseñas con bcrypt; sesiones server-side con expiración.
- [ ] CA-2: RBAC aplicado en cada ruta staff.
