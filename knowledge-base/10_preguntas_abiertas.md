# Preguntas Abiertas

## Inconsistencias detectadas

Ninguna inconsistencia bloqueante entre `discovery/discovery.md`, `discovery/market-study.md` y las decisiones cerradas: el corte v1 (sin pagos/WhatsApp/OS/reportes/laboratorio) es coherente con el MVP sugerido en market-study §D una vez recortado a alcance académico; la tensión SQLite-vs-escalable está mitigada por DD-02.

## Preguntas abiertas (priorizadas)

| Prioridad | Pregunta | Bloquea | Decisor |
|---|---|---|---|
| Alta | Canal push propio: ¿solo email, o también web push en v1? Define el campo `canal` de Recordatorio y el trabajo de `recordatorios/` | Sprint 1 (recordatorios) | Equipo técnico + consultorio |
| Alta | Odontograma: ¿el set de 5 estados (sano/cariado/obturado/ausente/en-tratamiento) es clínicamente suficiente? Confirmar con un odontólogo real | Sprint 1 (odontograma) | Odontólogo referente |
| Media | Lista de espera: ¿alcanza con avisar el hueco libre, o se necesita reserva prioritaria? (Sin sobreturnos en cualquier caso, RN-02) | Sprint 2 | Consultorio |
| Media | Excepción a RN-03/RN-04: ¿el staff puede saltarse los 48/72h en casos justificados, con registro de motivo? Hoy la regla aplica a todos | Sprint 1 (turnos staff) | Consultorio |
| Media | Expiración exacta del token: ¿48h o 72h? (rango ya decidido; falta el valor operativo y su reemisión) | Sprint 1 (token) | Equipo técnico |
| Baja | Anticipación del recordatorio: ¿24h antes, o doble aviso (48h + 24h)? Define `REMINDER_HOURS_BEFORE` | Sprint 2 | Consultorio |
| Baja | Post-v1: ¿qué va primero, WhatsApp API o Mercado Pago (seña)? El mercado (market-study §C) premia ambos; hay que secuenciarlos | Post-v1 | Consultorio |
