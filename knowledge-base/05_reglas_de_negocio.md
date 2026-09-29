# Reglas de Negocio

Códigos `RN-NN` para trazabilidad (usados en `06_funcionalidades.md` y `07_flujos_principales.md`).

## Dominio: Agenda y turnos

- **RN-01**: Un profesional no puede tener dos turnos superpuestos (ni en dos sillones a la vez). Todo alta/reprogramación valida solapamiento por `(profesional_id, inicio, fin)` y se rechaza si colisiona.
- **RN-02**: No existen los sobreturnos: el sistema no permite crearlos por ningún rol ni canal (la lista de espera, si se define, solo avisa huecos — ver `10_preguntas_abiertas.md`).
- **RN-03**: No se puede cancelar ni reprogramar un turno con menos de 48 horas de anticipación al `inicio` (aplica a paciente con token y a staff; urgencias reales se gestionan fuera del sistema o por excepción registrada).
- **RN-04**: No se puede pedir un turno para dentro de menos de 72 horas desde la solicitud (aplica al portal online; el staff presencial puede registrar con menor anticipación solo si el hueco ya existe y respeta RN-01/RN-02).
- **RN-05**: La duración del turno depende de la prestación: `turno.fin = turno.inicio + prestacion.duracion_min`. Cambiar la prestación recalcula el fin y revalida RN-01.
- **RN-06**: Un turno ocupa exactamente un profesional y un sillón activos. No se agenda sobre profesionales o sillones inactivos; el solapamiento se valida también por `(sillon_id, inicio, fin)`.

## Dominio: Paciente sin cuenta

- **RN-07**: El paciente no tiene usuario ni contraseña. Reserva con nombre + teléfono (+ email obligatorio en el canal online para enviar token y recordatorios).
- **RN-09**: La gestión online de turnos (ver/cancelar/reprogramar) exige token por email vigente, con expiración 48-72h desde su emisión. Token vencido o desconocido → acceso denegado sin revelar datos de otros turnos.

## Dominio: Clínica y odontograma

- **RN-08**: Odontograma mínimo v1: 32 piezas permanentes (FDI 11-18, 21-28, 31-38, 41-48) con set cerrado de estados: sano, cariado, obturado, ausente, en-tratamiento. Sin dentición temporal ni periodontograma.
- **RN-10**: Solo el odontólogo registra prestaciones y estados de piezas; la recepcionista tiene lectura del odontograma y CRUD de datos administrativos y de agenda.

## Dominio: Excepciones globales

- Toda operación de escritura valida RN-01…RN-09 antes de persistir; el error se informa sin exponer datos de otros pacientes.
- Los recordatorios solo se envían para turnos en estado reservado/confirmado y nunca revelan datos clínicos.
- Las contraseñas staff solo existen como hash bcrypt; las sesiones son server-side con expiración.
