# Actores y Roles

## Actores del sistema

| Actor | Descripción | Cómo interactúa |
|---|---|---|
| Paciente (sin cuenta) | Persona que reserva atención; NO tiene usuario ni contraseña | Portal público: reserva con nombre + teléfono (+ email para recibir token de gestión); gestiona sus turnos con token por email con expiración 48-72h |
| Recepcionista | Staff que gestiona la agenda y administra el consultorio | Login con sesión server-side; agenda diaria/semanal de todos los profesionales; CRUD de turnos, profesionales, sillones, prestaciones y pacientes |
| Odontólogo | Profesional que atiende y registra prestaciones | Login con sesión server-side; ve su agenda; registra prestación en ficha + odontograma |

No hay rol admin separado en v1: la recepcionista administra (ver `09_decisiones_y_supuestos.md`).

## RBAC — Matriz de permisos

| Rol | Agenda (ver) | Turnos (crear/cancelar/reprogramar) | Profesionales/Sillones/Prestaciones | Pacientes/Fichas | Odontograma |
|---|---|---|---|---|---|
| Paciente (token) | Solo disponibilidad pública | Solo sus propios turnos (con token vigente) | — | — | — |
| Recepcionista | Toda (diaria/semanal, todos) | Todos | CRUD completo | CRUD completo | Lectura |
| Odontólogo | Propia (diaria/semanal) | Solo reprogramar/cancelar los propios | Lectura | Lectura + registro de prestación | Lectura + escritura de estados |

## Rutas públicas

Rutas accesibles sin autenticación (portal paciente):

- `GET /` — inicio del portal.
- `GET /reservar` — elegir profesional, prestación, fecha y hora.
- `GET /disponibilidad` — horarios libres (solo lectura, JSON/HTML parcial para HTMX).
- `POST /turnos` — crear reserva (nombre + teléfono + email, RN-04).
- `GET /mis-turnos?token=...` — ver turnos propios con token vigente (RN-09).
- `POST /mis-turnos/{id}/cancelar?token=...` — cancelar con token (RN-03).
- `POST /mis-turnos/{id}/reprogramar?token=...` — reprogramar con token (RN-03, RN-04).

Todo lo staff (`/staff/...`, `/agenda/...`, `/fichas/...`, `/admin/...`) requiere sesión server-side con rol Recepcionista u Odontólogo.
