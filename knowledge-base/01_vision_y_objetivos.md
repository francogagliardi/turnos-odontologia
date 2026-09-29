# Visión y Objetivos

## Propósito del sistema

Dar a consultorios odontológicos chicos (1-3 profesionales, 1-3 sillones) una agenda digital con reserva online y recordatorios automáticos, que elimine la gestión por teléfono/WhatsApp/papel y reduzca turnos perdidos, solapamientos y horas semanales de confirmación manual.

El sistema centraliza agenda multi-profesional/multi-sillón, reserva online simple para pacientes sin cuenta, recordatorios propios y ficha clínica básica con odontograma mínimo, con datos desacoplados para escalar sin reescribir.

## Objetivos por actor

| Actor | Objetivo principal | Objetivos secundarios |
|---|---|---|
| Paciente (sin cuenta) | Reservar, reprogramar y cancelar turnos online sin llamar | Recibir recordatorios; gestionar sus turnos con token por email |
| Recepcionista | Gestionar la agenda diaria de todos los profesionales | Crear/cancelar turnos presenciales y telefónicos; administrar profesionales, sillones y prestaciones |
| Odontólogo | Ver su agenda del día y registrar la prestación realizada | Consultar ficha básica y odontograma mínimo del paciente |

## Alcance v1

- Agenda multi-profesional y multi-sillón con vista diaria y semanal.
- Reserva online simple: elegir profesional, prestación, fecha y hora disponible.
- Paciente sin cuenta: reserva con nombre + teléfono (+ email para gestión con token con expiración 48-72h).
- Recordatorios automáticos propios: SMTP configurable por env + scheduler embebido (APScheduler); sin proveedores externos.
- Ficha básica del paciente + odontograma interactivo mínimo: 32 piezas permanentes, estados sano/cariado/obturado/ausente/en-tratamiento.
- Roles y permisos: Recepcionista (gestión total de agenda y administración), Odontólogo (su agenda + registro de prestación).
- Auth staff por sesiones server-side + bcrypt; tests pytest desde día 1 (TDD estricto).
- Duración variable del turno según prestación.

## Fuera de alcance

- Pagos y señas online (post-v1).
- Integración con WhatsApp Business API (post-v1).
- Obras sociales y prepagas argentinas (post-v1).
- Reportes y métricas avanzadas (post-v1).
- Circuito de laboratorio protésico (post-v1).
- Dentición temporal y periodontograma (fuera del odontograma mínimo v1).
- Rol admin separado (v1: la recepcionista administra).
- Multi-sucursal, facturación ARCA, chatbot IA, PACS/imagen avanzada, API pública.

## Métricas de éxito

- Reducción de ausentes vs. gestión manual (medible cuando haya recordatorios activos).
- Cero solapamientos de turnos por profesional/sillón (garantizado por RN-01/RN-02).
- Tiempo semanal de confirmación/reprogramación manual tendiendo a cero.
- 100% de reservas online con anticipación ≥72h y cancelaciones con ≥48h (RN-03/RN-04).
