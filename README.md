# Casino online regulado en Colombia — Arquitectura por bloques

Proyecto final de Arquitectura de Software (ITM, Equipo 8). Desarrolla de forma teórica la arquitectura de *Casino Andino*, un casino online hipotético (tragamonedas, ruleta, blackjack en vivo y *crash*; sin apuestas deportivas), bloque a bloque: de la regulación y los atributos de calidad a cuatro ADR (Bloque 2), y de ahí a las tácticas, el sizing y los ADR aceptados (Bloque 3).

## Empieza por aquí

- **La entrega más reciente:** la [presentación del Bloque 3](./bloque-3/Casino%20Andino.html) (ábrela en el navegador) y los [ADR aceptados](./bloque-3/06-adrs.md), con su Decision, su "hasta dónde" y sus Consequences.
- **De dónde salen los ADR:** los [escenarios](./bloque-2/06-escenarios.md) y las [opciones consideradas](./bloque-2/07-adrs.md) del Bloque 2.
- **Cómo se llegó:** lee las fases de cada bloque en orden.
- **Estado de la revisión:** [Bloque 2](./bloque-2/08-revision.md) y [Bloque 3](./bloque-3/09-revision.md).

## Estado

| Bloque | Tema | Estado | Presentación |
|:-:|---|---|---|
| 2 | Atributos de calidad, escenarios y ADR | Entregado | [Casino Andino.html](./bloque-2/Casino%20Andino.html) |
| 3 | Tácticas y sizing | En curso: falta la plantilla (10.2) | [Casino Andino.html](./bloque-3/Casino%20Andino.html) |
| 4 | Estilos arquitectónicos | Pendiente | — |

## Índice del Bloque 3 · Tácticas y sizing

| Fase | Documento | Qué responde |
|:-:|---|---|
| 0 | [Plan de trabajo](./bloque-3/00-plan-de-trabajo.md) | Qué hay que entregar y cómo se organiza el trabajo |
| 1 | [Punto de partida](./bloque-3/01-punto-de-partida.md) | Tensión dominante, operación crítica y tipos de cifra |
| 2 | [Línea base](./bloque-3/02-linea-base.md) | La versión más simple que funcionaría, sin tácticas |
| 3 | [Perfil de carga](./bloque-3/03-perfil-de-carga.md) | De 350.000 cuentas a 3.000 operaciones de saldo por segundo en el pico |
| 4 | [Cuello de botella](./bloque-3/04-cuello-de-botella.md) | Qué componente se satura primero y con cuánta carga |
| 5 | [Tácticas y efecto](./bloque-3/05-tacticas-y-efecto.md) | Diez tácticas: mecanismo, efecto estimado y costo |
| 6 | [ADR completados](./bloque-3/06-adrs.md) | Decision, descartes, hasta dónde y Consequences de cada ADR |
| 7 | [Interacción y límites](./bloque-3/07-interaccion-y-limites.md) | Qué táctica empeora otro escenario y cuál es el límite con todas |
| 8 | [Extensión X1](./bloque-3/08-extension.md) | Qué se sostiene y qué se rompe con el pico multiplicado por 5 |
| 9 | [Revisión integral](./bloque-3/09-revision.md) | Autoevaluación, coherencia y lo que el equipo reconoce |

## Índice del Bloque 2 · Atributos de calidad, escenarios y ADR

| Fase | Documento | Qué responde |
|:-:|---|---|
| 0 | [Plan de trabajo](./bloque-2/00-plan-de-trabajo.md) | Qué hay que entregar y cómo se organiza el trabajo |
| 1 | [Investigación del dominio](./bloque-2/01-investigacion-dominio.md) | Qué exige la regulación y cómo opera un casino online |
| 2 | [Ficha de contexto](./bloque-2/02-ficha-contexto.md) | Requisitos no funcionales con cifras, restricciones y supuestos |
| 3 | [Mapa de tensiones](./bloque-2/03-mapa-tensiones.md) | Qué atributos de calidad chocan entre sí |
| 4 | [Priorización](./bloque-2/04-priorizacion.md) | Cómo se reparten los 18 puntos entre los 8 atributos |
| 5 | [Selección de escenarios](./bloque-2/05-seleccion-escenarios.md) | Qué escenarios son críticos y por qué |
| 6 | [Escenarios de calidad](./bloque-2/06-escenarios.md) | Los escenarios redactados (fuente, estímulo… medida) |
| 7 | [ADRs](./bloque-2/07-adrs.md) | Opciones consideradas para cada decisión |
| 8 | [Revisión integral](./bloque-2/08-revision.md) | Autoevaluación, coherencia y riesgos pendientes |

## Cómo está armado

- Cada fase usa como insumo a las anteriores y lo declara al inicio.
- Cada fase cierra con un **punto de control** donde el equipo valida lo decidido.
- La presentación HTML de cada bloque se integra al final desde los markdown, que son la fuente.
- Los ADR no se reescriben: el Bloque 2 los deja en Propuesto y el Bloque 3 los pasa a Aceptado con una nota fechada.
