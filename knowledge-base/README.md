# turnos-odontologia — Base de Conocimiento

Base de conocimiento generada a partir de `discovery/discovery.md`, `discovery/market-study.md` y las decisiones cerradas con el usuario (stack, arquitectura, roles, reglas, alcance v1). Fuente: interactiva (`source: "interactive"`, `created_by: "kb-creator"`).

## Índice de Archivos

| Archivo | Contenido |
|---------|-----------|
| [01_vision_y_objetivos.md](01_vision_y_objetivos.md) | Propósito, objetivos por actor, alcance v1, fuera de alcance, métricas |
| [02_descripcion_general.md](02_descripcion_general.md) | Stack (FastAPI+SQLite+Jinja+HTMX), arquitectura general, integraciones, API |
| [03_actores_y_roles.md](03_actores_y_roles.md) | Actores, matriz RBAC, rutas públicas del portal paciente |
| [04_modelo_de_datos.md](04_modelo_de_datos.md) | Dominios, ERD en texto, 12 entidades, seed inicial |
| [05_reglas_de_negocio.md](05_reglas_de_negocio.md) | RN-01…RN-10 (solapamientos, sobreturnos, 48/72h, duración, token, odontograma) |
| [06_funcionalidades.md](06_funcionalidades.md) | 5 épicas, US-001…US-014 con criterios de aceptación y RN trazadas |
| [07_flujos_principales.md](07_flujos_principales.md) | Reserva online con token, gestión recepcionista, registro prestación, recordatorios |
| [08_arquitectura_propuesta.md](08_arquitectura_propuesta.md) | Monolito modular, estructura de directorios, seguridad, env vars |
| [09_decisiones_y_supuestos.md](09_decisiones_y_supuestos.md) | DD-01…DD-11 y SU-01…SU-04 |
| [10_preguntas_abiertas.md](10_preguntas_abiertas.md) | 7 preguntas priorizadas (push, odontograma, lista de espera, excepciones, token) |

## Quick Start para Desarrolladores

1. Entender el dominio → [01](01_vision_y_objetivos.md), [03](03_actores_y_roles.md)
2. Entender los datos → [04](04_modelo_de_datos.md)
3. Entender las reglas → [05](05_reglas_de_negocio.md)
4. Entender la arquitectura → [02](02_descripcion_general.md), [08](08_arquitectura_propuesta.md)
5. Implementar → [07](07_flujos_principales.md), [06](06_funcionalidades.md)
6. Antes de codificar → [10](10_preguntas_abiertas.md)

## Resumen Ejecutivo

Agenda digital para consultorios chicos con reserva online sin cuenta (token por email 48-72h), recordatorios propios vía SMTP y ficha con odontograma mínimo. Monolito modular FastAPI + SQLite + Jinja + HTMX, migrable a Postgres, con TDD estricto desde día 1 y corte v1 cerrado (pagos, WhatsApp, obras sociales y reportes quedan post-v1).
