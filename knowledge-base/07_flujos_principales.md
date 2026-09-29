# Flujos Principales

Cada flujo se documenta extremo a extremo, mostrando interacciones entre componentes.

## Flujo 1: Reserva online con token

**Disparador**: un paciente entra al portal público.
**Actor**: Paciente (sin cuenta).

**Pasos**:
1. [Portal] Paciente elige profesional + prestación + fecha.
2. [API] Calcula disponibilidad: descuenta turnos existentes, respeta duración (RN-05) y oculta horarios con <72h (RN-04).
3. [Portal] Paciente elige hora e informa nombre + teléfono + email (RN-07).
4. [API] Revalida hueco (anti-carrera), crea Paciente (si nuevo) + Turno `reservado` + TokenGestion (expira 48-72h, RN-09).
5. [API] Envía email con token de gestión vía SMTP propio.
6. [Scheduler] Programa el recordatorio del turno.
7. [Portal] Paciente gestiona después en `/mis-turnos?token=...`: ver/cancelar/reprogramar (RN-03/RN-04/RN-09).

**Diagrama de secuencia** (ASCII):
```
Paciente → Portal → API → DB (Turno+Token)
                       ← hueco confirmado
             API → SMTP → email con token
             API → Scheduler → Recordatorio programado
Paciente → /mis-turnos?token → API → DB (solo sus turnos)
```

**Casos de error**:
- Hueco ocupado en simultáneo → 409, se pide elegir otro horario.
- Reserva con <72h → 422 con mensaje de anticipación (RN-04).
- Email inválido → 422, no se crea el turno.
- SMTP caído → el turno se crea igual; el email queda pendiente de reintento y se avisa.
- Token vencido/desconocido → 403 sin revelar datos ajenos.

## Flujo 2: Gestión de agenda por recepcionista

**Disparador**: la recepcionista abre la agenda del día.
**Actor**: Recepcionista (sesión server-side).

**Pasos**:
1. [Staff UI] Login con usuario + contraseña (sesión server-side, bcrypt).
2. [API] Devuelve agenda diaria/semanal por profesional y por sillón.
3. [Staff UI] Recepcionista crea, cancela o reprograma turnos (origen `staff`).
4. [API] Valida RN-01 (profesional y sillón), RN-02 (sin sobreturnos), RN-03 (≥48h), RN-05 (fin por prestación), RN-06 (recursos activos).
5. [API] Persiste y, si el paciente tiene email, envía confirmación/cancelación.

**Diagrama de secuencia**:
```
Recepcionista → Staff UI → API → DB (agenda)
                              ← turnos del día
Recepcionista → Staff UI → API → valida RN-01/02/03/05/06 → DB
                              → SMTP (aviso al paciente, si hay email)
```

**Casos de error**:
- Solapamiento → 409 indicando el turno en conflicto (sin datos sensibles).
- Cancelación con <48h → 422 (RN-03).
- Recurso inactivo → 422 (RN-06).

## Flujo 3: Registro de prestación por odontólogo

**Disparador**: el odontólogo atiende a un paciente de su agenda.
**Actor**: Odontólogo (sesión server-side).

**Pasos**:
1. [Staff UI] Odontólogo abre su agenda del día y entra a la ficha del paciente.
2. [API] Devuelve ficha + odontograma mínimo actual (RN-08).
3. [Staff UI] Odontólogo actualiza estados de piezas (32 permanentes, set cerrado).
4. [Staff UI] Registra prestación realizada + nota vinculada al turno (RN-10).
5. [API] Persiste PrestacionRealizada + EstadoPieza con autor y fecha; marca turno `realizado`.

**Diagrama de secuencia**:
```
Odontólogo → Staff UI → API → DB (Ficha+EstadoPieza)
Odontólogo → Staff UI → API → DB (PrestacionRealizada, Turno=realizado)
```

**Casos de error**:
- Pieza fuera de FDI permanente o estado inválido → 422 (RN-08).
- Recepcionista intenta escribir odontograma → 403 (RN-10).

## Flujo 4: Recordatorios automáticos propios

**Disparador**: scheduler embebido (APScheduler) por tick programado.
**Actor**: Sistema (sin actor humano).

**Pasos**:
1. [Scheduler] Busca recordatorios `pendientes` con `programado_para <= ahora` y turno en reservado/confirmado.
2. [Worker] Envía email por SMTP propio (config por env).
3. [Worker] Marca `enviado` (+`enviado_en`) o `fallido` para reintento acotado.
4. [API] Expone estado de envíos para la recepcionista (solo lectura administrativa).

**Diagrama de secuencia**:
```
APScheduler → Worker → DB (pendientes)
              Worker → SMTP → email al paciente
              Worker → DB (enviado/fallido)
```

**Casos de error**:
- SMTP caído → `fallido`, reintento acotado; la recepcionista ve el estado.
- Turno cancelado antes del envío → el recordatorio se descarta.
- Email rebotado → `fallido` sin datos clínicos en el log.
