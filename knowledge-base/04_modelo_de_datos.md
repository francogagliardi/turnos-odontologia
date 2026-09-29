# Modelo de Datos

## Dominios

- **turnos**: agenda, disponibilidad, reservas y tokens de gestión del paciente.
- **pacientes**: datos mínimos del paciente (sin cuenta de usuario).
- **fichas**: ficha clínica básica + odontograma mínimo por paciente.
- **auth**: usuarios staff (recepcionista/odontólogo), sesiones server-side y tokens.

## ERD (Entity Relationship Diagram)

```
Profesional 1───* Turno *───1 Sillon
Prestacion  1───* Turno *───1 Paciente
Paciente    1───1 Ficha 1───* EstadoPieza
UsuarioStaff *───1 Profesional (nullable: recepcionista sin profesional asociado)
TokenGestion *───1 Turno
Recordatorio *───1 Turno
```

Reglas estructurales: un turno ocupa exactamente 1 profesional + 1 sillón + 1 prestación + 1 paciente; sin solapamientos por profesional ni por sillón (RN-01); un profesional no atiende dos sillones a la vez.

## Entidades

### Profesional
- Atributos: `id` (int PK), `nombre` (str), `apellido` (str), `matricula` (str, única), `activo` (bool).
- Relaciones: 1───* Turno; 0..1───1 UsuarioStaff.
- Constraints: matrícula única; solo profesionales activos reciben turnos.
- Índices: `matricula`, `activo`.

### Sillon
- Atributos: `id` (int PK), `nombre` (str, ej. "Sillón 1"), `activo` (bool).
- Relaciones: 1───* Turno.
- Constraints: solo sillones activos reciben turnos; sin solapamientos por sillón (RN-01).
- Índices: `activo`.

### Prestacion
- Atributos: `id` (int PK), `nombre` (str), `duracion_min` (int > 0), `activa` (bool).
- Relaciones: 1───* Turno.
- Constraints: la duración del turno = `duracion_min` de su prestación (RN-05).
- Índices: `activa`.

### Paciente
- Atributos: `id` (int PK), `nombre` (str), `telefono` (str), `email` (str, para token y recordatorios), `dni` (str, nullable), `creado_en` (datetime).
- Relaciones: 1───* Turno; 1───1 Ficha.
- Constraints: email con formato válido si se informa (requerido para reserva online con gestión por token).
- Índices: `email`, `telefono`.

### Turno
- Atributos: `id` (int PK), `profesional_id` (FK), `sillon_id` (FK), `prestacion_id` (FK), `paciente_id` (FK), `inicio` (datetime), `fin` (datetime), `estado` (reservado/confirmado/cancelado/realizado/ausente), `origen` (online/staff), `creado_en` (datetime).
- Relaciones: *───1 Profesional, Sillon, Prestacion, Paciente; 1───* TokenGestion, Recordatorio.
- Constraints: `fin = inicio + prestacion.duracion_min` (RN-05); sin solapamientos por profesional ni sillón (RN-01); sin sobreturnos (RN-02); reserva con ≥72h (RN-04); cancelación con ≥48h (RN-03).
- Índices: `(profesional_id, inicio)`, `(sillon_id, inicio)`, `(paciente_id, inicio)`, `estado`.

### TokenGestion
- Atributos: `id` (int PK), `turno_id` (FK), `token` (str, único, aleatorio), `expira_en` (datetime, 48-72h), `usado` (bool).
- Relaciones: *───1 Turno.
- Constraints: token único; expiración 48-72h desde emisión (RN-09); un solo uso lógico por operación (se reemite al reprogramar).
- Índices: `token` (único).

### Recordatorio
- Atributos: `id` (int PK), `turno_id` (FK), `canal` (email; +web push si se decide en `10_preguntas_abiertas.md`), `programado_para` (datetime), `enviado_en` (datetime, nullable), `estado` (pendiente/enviado/fallido).
- Relaciones: *───1 Turno.
- Constraints: solo para turnos no cancelados; reintentos acotados ante fallo SMTP.
- Índices: `(estado, programado_para)`.

### Ficha
- Atributos: `id` (int PK), `paciente_id` (FK única), `antecedentes` (text), `alergias` (text), `actualizada_en` (datetime).
- Relaciones: 1───1 Paciente; 1───* EstadoPieza; 1───* PrestacionRealizada.
- Constraints: una ficha por paciente.
- Índices: `paciente_id` (único).

### EstadoPieza
- Atributos: `id` (int PK), `ficha_id` (FK), `pieza` (int 11-48 FDI, solo permanentes), `estado` (sano/cariado/obturado/ausente/en-tratamiento), `registrado_en` (datetime), `registrado_por` (FK UsuarioStaff).
- Relaciones: *───1 Ficha.
- Constraints: pieza en rango FDI permanente 11-18, 21-28, 31-38, 41-48; estado dentro del set cerrado v1 (RN-08); sin dentición temporal.
- Índices: `(ficha_id, pieza)`.

### PrestacionRealizada
- Atributos: `id` (int PK), `ficha_id` (FK), `turno_id` (FK), `prestacion_id` (FK), `nota` (text), `realizada_en` (datetime), `registrada_por` (FK UsuarioStaff).
- Relaciones: *───1 Ficha, Turno, Prestacion.
- Constraints: solo el odontólogo del turno (o recepcionista por excepción) registra.
- Índices: `(ficha_id, realizada_en)`.

### UsuarioStaff
- Atributos: `id` (int PK), `username` (str único), `password_hash` (bcrypt), `rol` (recepcionista/odontologo), `profesional_id` (FK nullable), `activo` (bool).
- Relaciones: 0..1───1 Profesional; 1───* sesiones server-side.
- Constraints: hash bcrypt, nunca plaintext; solo usuarios activos pueden iniciar sesión.
- Índices: `username` (único).

## Seed data inicial

- 1 usuario recepcionista (`recepcion` / hash bcrypt) y 1 usuario odontólogo de ejemplo.
- 2 profesionales activos, 2 sillones activos.
- Prestaciones base: limpieza (30 min), consulta (30 min), obturación (60 min), endodoncia (90 min), extracción (60 min).
- Sin pacientes ni turnos de ejemplo en producción (solo en demo/tests).
