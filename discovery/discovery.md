# Discovery — turnos-odontologia

**Fecha**: 2026-09-29
**Fuentes investigadas**: 17 sistemas (ver `discovery/sources/` y `discovery/market-study.md` con tabla comparativa, matriz ponderada y recomendación de MVP)

## 1. Problema que resuelve

Los consultorios odontológicos chicos (1-3 profesionales, 1-3 sillones) gestionan
sus turnos por teléfono, WhatsApp o papel, lo que genera turnos perdidos,
solapamientos y horas semanales perdidas en confirmar y reprogramar manualmente.
El sistema da una agenda digital con reserva online y recordatorios automáticos
que reduce ese trabajo manual y los ausentes.

## 2. Usuarios / roles

- **Paciente**: reserva, reprograma y cancela sus turnos online sin llamar.
- **Recepcionista**: gestiona la agenda diaria de todos los profesionales.
- **Odontólogo**: ve su agenda y registra la prestación en la ficha del paciente.

## 3. Casos de uso

1. Como paciente, quiero ver los horarios libres de un profesional para reservar
   sin llamar ni escribir por WhatsApp.
2. Como paciente, quiero reprogramar o cancelar mi turno (respetando la
   anticipación mínima) para liberar el horario.
3. Como paciente, quiero recibir un recordatorio de mi turno para no olvidarlo.
4. Como recepcionista, quiero ver la agenda diaria de cada profesional para
   gestionar los turnos del consultorio.
5. Como odontólogo, quiero ver mi agenda del día y registrar la prestación
   realizada en la ficha del paciente.

## 4. Competidores / soluciones existentes

| Competidor | Problema que resuelve | Pricing | Diferenciadores |
|---|---|---|---|
| DentalSoft | Gestión integral odontológica argentina | Publicado en sitio oficial | Local AR, primero en ranking ponderado (4,45) |
| Dentiqa | Gestión dental con foco normativo | Publicado en sitio oficial | Menciona Ley 26.529, segundo en ranking (4,20) |
| Dentalink | Operación fragmentada (agenda, ficha, cobros) | Por cotización | Dental-nativo 15 años, IA en cada módulo |
| tab32 | Gestión cloud para clínicas dentales | Publicado en sitio oficial | Referente cloud internacional |
| Doctoralia | Citas perdidas + gestión manual | Desde ~89 €/mes + add-ons IA | Marketplace de pacientes, 13 países |
| AgendaPro | Citas por WhatsApp, ausentismo, caja | USD 10–199, pricing localizado LATAM | Todo-en-uno multi-rubro |
| DoctoCliq | Agenda médica online | Publicado en sitio oficial | Foco LATAM |

Detalle completo de los 17 sistemas en `discovery/market-study.md` (secciones
A–D: comparativa, matriz ponderada, análisis competitivo y recomendación).

**Notas**: ningún competidor integra en un solo flujo ARCA + odontograma +
obras sociales argentinas + Mercado Pago; casi ninguno ofrece exportación
abierta de datos ni declara normativa argentina de salud. Las métricas de los
proveedores (porcentajes de ausentismo, cantidad de clínicas) se trataron como
afirmación comercial, no evidencia.

## 5. Funcionalidades necesarias

- Agenda multi-profesional y multi-sillón con vista diaria/semanal.
- Reserva online simple (elegir profesional, fecha y hora disponible).
- Recordatorios automáticos con medios propios (email/push, sin proveedores
  externos en v1).
- Ficha básica del paciente con odontograma mínimo.
- Roles y permisos (recepcionista, odontólogo, paciente).

## 6. Funcionalidades opcionales

- Pagos y señas online (a diferencia de Fresha/AgendaPro, quedan fuera de v1 —
  decisión consciente para no descontrolar el alcance).
- Integración con WhatsApp Business API.
- Obras sociales y prepagas argentinas.
- Reportes y métricas avanzadas.
- Circuito de laboratorio protésico.

## 7. Reglas de negocio

- Un profesional no puede tener dos turnos superpuestos.
- No existen los sobreturnos: el sistema no permite crearlos.
- No se puede cancelar un turno con menos de 48 horas de anticipación.
- No se puede pedir un turno para dentro de menos de 72 horas.
- La duración del turno depende de la prestación (duración variable).

## 8. Integraciones

- Ninguna integración externa obligatoria para la v1 (a diferencia de la
  mayoría de los competidores, que cobran WhatsApp o pagos aparte — decisión
  consciente para cerrar el alcance).

## 9. Restricciones

- Stack definido: backend Python + FastAPI, datos SQLite, front Jinja + HTMX.
- Sistema multi-profesional con diseño escalable: la capa de datos va
  desacoplada (ORM + migraciones versionadas) para migrar a Postgres sin
  reescribir cuando escale.
- Sin plazo de entrega (proyecto académico, Programación 3).

## 10. Riesgos

- **Supuesto sin probar**: que los pacientes prefieran reservar online en vez
  de seguir escribiendo por WhatsApp por costumbre — no validado con usuarios
  reales todavía.
- **Riesgo**: descontrol de alcance (odontograma completo, pagos,
  multi-sucursal) — mitigado con el corte v1 cerrado en el punto 5.
- **Riesgo**: tensión SQLite vs requisito escalable — mitigada con capa de
  datos desacoplada y migraciones desde el día 1.
- **Supuesto sin probar**: el alcance exacto del "odontograma mínimo" — se
  define en la etapa de diseño, no acá.

## 11. Preguntas abiertas

- ¿Qué incluye exactamente el odontograma mínimo de v1?
- ¿Push propio = solo email, o también web push?
- Lista de espera: ¿alcanza con avisar el hueco libre (sin sobreturnos)?
