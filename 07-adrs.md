# Fase 7 — Registro de decisiones de arquitectura (ADRs)

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Insumos:**
- `06-escenarios.md`: el Context de cada ADR es su escenario, copiado tal cual.
- `05-seleccion-escenarios.md`: criterio de selección.
- `02-ficha-contexto.md`: restricciones y cifras.

**Estado de todos los ADR: Propuesto.** Siguiendo el material del curso, cada ADR llega **hasta las opciones consideradas**. Decision y Consequences se escriben en el Bloque 3, cuando contemos con las tácticas para elegir con fundamento.

**Convenciones para las opciones:**
- Se describen como **mecanismos del sistema**, sin nombres de tácticas ni marcas.
- Cada opción indica qué gana, qué paga, qué atributo sacrifica y si cumple la medida de la respuesta.
- La medida de la respuesta del escenario es el criterio para compararlas.

---

## Selección de los ADR

> Hay un ADR por cada escenario crítico: ESC-01, ESC-02 y ESC-03. Son las únicas tensiones fuertes entre dos atributos priorizados altos (T1 Seguridad × Desempeño, T2 Fiabilidad × Desempeño y T3 Compatibilidad × Fiabilidad). ESC-04 no lleva ADR porque su costo cae en un atributo priorizado bajo (Adecuación funcional, 1).

| ADR | Escenario | Atributo dominante | Atributo que paga |
|-----|-----------|--------------------|-------------------|
| ADR-01 | ESC-01 | Seguridad (4) | Desempeño (3) |
| ADR-02 | ESC-02 | Fiabilidad (4) | Desempeño (3) |
| ADR-03 | ESC-03 | Compatibilidad (3) | Fiabilidad (4) |

---

# ADR-01: Evaluación de los retiros contra listas restrictivas recién recibidas

### Estado

**Propuesto.** Pasa a Aceptado en el Bloque 3, cuando se complete la Decision.

### Context

*El escenario ESC-01, tal cual.*

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Seguridad, con una restricción de desempeño |
| **Fuente del estímulo** | Un jugador cuyo documento aparece en una actualización de listas restrictivas que el operador recibió hace 30 minutos, y cuya cuenta todavía no ha sido reconsultada |
| **Estímulo** | Solicita un retiro de COP 3.000.000 hacia su billetera digital |
| **Artefacto** | El proceso de solicitud de retiros y su evaluación de riesgo de lavado de activos (SARLAFT) |
| **Entorno** | Viernes de quincena a las 21:00. 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo, ~1.800 depósitos en la hora. Reconsulta de las 350.000 cuentas contra la lista nueva en curso, iniciada hace 30 minutos |
| **Respuesta** | Evalúa el retiro contra la versión de la lista recién recibida, aunque la cuenta no haya sido reconsultada todavía. Si el jugador está en la lista: no paga el retiro, suspende la cuenta por precaución, alerta al oficial de cumplimiento, deja el caso listo para el reporte a la UIAF y le muestra al jugador el retiro "en revisión", sin revelarle el motivo. Si el jugador no está en la lista y su riesgo es normal: el retiro sigue su curso habitual. El juego y los depósitos de los demás jugadores siguen operando |
| **Medida de la respuesta** | **0 retiros pagados** a personas incluidas en una lista ya recibida, verificado cruzando pagos y listas por marca de tiempo. Evaluación del retiro en **p95 ≤ 1 s**, también durante la reconsulta. Alerta al oficial de cumplimiento en **≤ 15 min**. Durante la reconsulta, las operaciones de saldo del juego se mantienen en **p99 ≤ 250 ms**. Reconsulta completa en **≤ 6 h**. Los retiros de riesgo normal mantienen la orden de transferencia en **≤ 24 h** |

**Restricciones del proyecto que también fuerzan esta decisión:**
- **Fallo cerrado (ficha, restricción 9):** si la evaluación de riesgo o la consulta de listas no están disponibles, los retiros se detienen. Legalmente es más grave pagar dinero de lavado que demorar un retiro. El plazo máximo sigue siendo de 72 h para la orden de transferencia [F1].
- SARLAFT, cumpliendo los mínimos del SIPLAFT [F9]; ~3.000 retiros al día (S8); las listas cambian varias veces al mes (S19).

### Opciones consideradas

**A · Retener los retiros hasta que termine la reconsulta.** Mientras corre la reconsulta masiva, toda solicitud de retiro queda en espera. Al terminar la reconsulta, cada retiro se evalúa con todas las cuentas ya marcadas.

**B · Consultar al proveedor de listas en cada retiro.** Cada solicitud de retiro dispara, en ese momento, una consulta individual al proveedor externo de listas por el documento del jugador. La reconsulta masiva sigue por su lado.

**C · Lista vigente consultable por el operador, independiente de la reconsulta.** Al recibir la actualización, el operador guarda la lista nueva en un registro propio de documentos listados, consultable por documento. Cada retiro se evalúa contra ese registro desde el primer minuto. La reconsulta masiva de cuentas avanza aparte, con un ritmo máximo para no competir con el juego, y debe cumplir las 6 h (al menos ~16 cuentas por segundo).

| Opción | Gana | Paga | Atributo sacrificado | ¿Cumple la medida? |
|--------|------|------|----------------------|--------------------|
| A · Retener hasta terminar la reconsulta | Es la más simple: ningún retiro se evalúa con una lista incompleta | Los retiros de riesgo normal esperan hasta 6 h para ser evaluados, sin culpa del jugador | Desempeño (comportamiento temporal); de paso, Usabilidad | **No**: incumple la evaluación en p95 ≤ 1 s |
| B · Consultar al proveedor en cada retiro | Cada retiro ve la lista más reciente del proveedor, sin depender de la reconsulta | Cada retiro depende de la latencia y la disponibilidad de un tercero. Con fallo cerrado, una caída del proveedor **detiene todos los retiros** | Fiabilidad (disponibilidad de los retiros) y Compatibilidad (dependencia externa en el camino crítico) | **Depende**: hay que medir la latencia del proveedor (¿p95 ≤ 1 s?) y su disponibilidad |
| C · Lista vigente propia, separada de la reconsulta | Evaluación en tiempo constante contra la lista más reciente desde el minuto 0. La reconsulta no compite con el juego gracias a su ritmo máximo | Almacenamiento adicional y dos procesos que mantener coherentes (lista vigente y reconsulta). La reconsulta tarda más por su ritmo limitado | Mantenibilidad; Desempeño (utilización de recursos) | **Sí**, si la lista nueva se carga en pocos minutos y el ritmo de la reconsulta (≥ 16 cuentas/s) cabe sin afectar el p99 ≤ 250 ms del juego |

### Pendiente para el Bloque 3

- **Decision:** abierta. No elegimos todavía.
- **Para decidir con fundamento hace falta:**
  - saber cuánto tarda cargar una lista nueva en la opción C;
  - la latencia y la disponibilidad reales del proveedor de listas (opción B);
  - verificar que las 6 h de la reconsulta caben dentro del ritmo máximo.
- **Familias de tácticas por revisar:** *Detectar ataques* y *Resistir ataques* (Seguridad), y *Controlar la demanda de recursos* (Desempeño).
- **Consequences:** se escriben al decidir: qué se gana, qué se paga y qué queda sin resolver. Por ejemplo, el manejo de coincidencias parciales de nombre (homonimia).

---

# ADR-02: Permanencia de las operaciones de saldo confirmadas ante la caída de un servidor

### Estado

**Propuesto.** Pasa a Aceptado en el Bloque 3, cuando se complete la Decision.

### Context

*El escenario ESC-02, tal cual.*

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Fiabilidad, con una restricción de desempeño |
| **Fuente del estímulo** | Una falla de hardware en el servidor que registra las operaciones de saldo de los jugadores |
| **Estímulo** | El servidor deja de responder abruptamente, con operaciones de saldo en curso |
| **Artefacto** | La cuenta de juego: el registro del saldo de cada jugador y de sus operaciones de descuento, abono y reversión |
| **Entorno** | Sábado de quincena a las 21:30, en el pico de diseño. 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo, confirmadas a los proveedores con p99 ≤ 250 ms. Una ronda de *crash* en curso con 2.400 apuestas registradas. Sistema técnico de juego alterno certificado, exigido por el regulador |
| **Respuesta** | Conserva todas las operaciones que ya había confirmado a los proveedores. Las operaciones que estaban en curso sin confirmar quedan sin efecto sobre el saldo, de forma que el proveedor y el operador terminen con el mismo saldo. Avisa a los jugadores afectados que el juego no está disponible. Restablece el juego y el saldo. Al reconectarse, cada jugador ve el resultado de la jugada que quedó incompleta. Registra el inicio, la duración y los servicios afectados, e informa a Coljuegos |
| **Medida de la respuesta** | **0 operaciones confirmadas perdidas**, verificado por la conciliación operador–proveedores posterior al evento. **0 saldos negativos.** Juego y saldo restablecidos en **≤ 15 min**. **100 %** de las jugadas incompletas con su resultado visible al reconectarse. Aviso a Coljuegos en **≤ 30 min**. A los **5 min** del restablecimiento, las operaciones de saldo vuelven a **p99 ≤ 250 ms** |

**Restricciones del proyecto que también fuerzan esta decisión:**
- **Sistema alterno** con la misma certificación que el principal; todo cambio en el principal se replica de inmediato en el alterno (E1) [F1].
- Centro de datos Tier III como mínimo (A1); copias en un sitio físicamente separado (E5) [F1].
- Saldo nunca negativo (C6); toda jugada debe poder reconstruirse (C11) [F1].
- Los proveedores dan por fallida una operación sin respuesta en ~3 s y la revierten por su cuenta (S14, F24).

### Opciones consideradas

**A · Confirmar solo después de escribir en dos sitios físicos.** El operador responde al proveedor únicamente cuando la operación quedó grabada de forma permanente en el servidor principal **y** en una segunda ubicación física independiente, como el sistema alterno.

**B · Confirmar tras la grabación local y copiar después.** El operador responde en cuanto la operación queda grabada de forma permanente en el servidor principal. La copia a la segunda ubicación se hace a continuación, segundos después.

**C · Libreta de operaciones de solo agregar, confirmada por mayoría.** Cada operación se anota primero, en orden, en una libreta a la que solo se le agregan entradas, y que mantienen tres servidores del mismo centro de datos. El operador responde al proveedor cuando al menos dos de los tres la tienen grabada. El saldo de cada jugador se calcula desde esa libreta. El sistema alterno recibe la libreta aparte.

| Opción | Gana | Paga | Atributo sacrificado | ¿Cumple la medida? |
|--------|------|------|----------------------|--------------------|
| A · Dos sitios antes de confirmar | Cero pérdida incluso si se pierde todo el sitio principal | Cada una de las ~3.000 operaciones por segundo espera el viaje de red hasta el otro sitio. Si el segundo sitio no responde, el juego se detiene o hay que relajar la garantía | Desempeño (comportamiento temporal) y Fiabilidad (disponibilidad, por depender de dos sitios) | **Depende**: la pérdida cero se cumple, pero el p99 ≤ 250 ms depende de la distancia al segundo sitio. Hay que medirlo |
| B · Grabación local y copia posterior | La respuesta más rápida; el viaje al segundo sitio no está en el camino de la jugada | Si el servidor falla antes de copiar, se pierden las últimas operaciones ya confirmadas (una ventana de segundos) | Fiabilidad (integridad y recuperación) | **No**: deja una ventana de pérdida, y la medida exige 0 operaciones confirmadas perdidas |
| C · Libreta confirmada por mayoría | Tolera la caída de un servidor completo sin perder lo confirmado y sin esperar al más lento. La libreta permite reconstruir saldos y jugadas, como exige el reglamento | Más infraestructura (tres servidores), mayor complejidad de operación y de recertificación, y un viaje de red dentro del centro de datos en cada operación | Mantenibilidad; Desempeño (utilización de recursos) | **Sí**, si la mayoría se obtiene dentro de p99 ≤ 250 ms con los tres servidores en el mismo centro de datos. Hay que medirlo |

### Pendiente para el Bloque 3

- **Decision:** abierta. No elegimos todavía.
- **Para decidir con fundamento hace falta:**
  - la latencia de red real entre los sitios (opción A) y dentro del centro de datos (opción C), bajo ~3.000 operaciones por segundo;
  - el costo de la infraestructura adicional frente al presupuesto (~15 % del GGR).
- **Familias de tácticas por revisar:** *Detectar fallas*, *Recuperarse de fallas* y *Prevenir fallas* (Fiabilidad), y *Gestionar los recursos* (Desempeño).
- **Pregunta abierta:** cómo se mantiene el sistema alterno al día "de inmediato" (E1) sin que eso entre al camino de cada jugada.
- **Consequences:** se escriben al decidir.

---

# ADR-03: Tratamiento de las peticiones de saldo repetidas de los proveedores de juego

### Estado

**Propuesto.** Pasa a Aceptado en el Bloque 3, cuando se complete la Decision.

### Context

*El escenario ESC-03, tal cual.*

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Compatibilidad (interoperabilidad), con una restricción de fiabilidad |
| **Fuente del estímulo** | El proveedor externo del juego *crash* |
| **Estímulo** | Envía tres veces la misma petición de abono de COP 50.000, con el mismo identificador de transacción, porque no recibió respuesta a la primera dentro de su tiempo de espera de ~3 s |
| **Artefacto** | La interfaz de operaciones de saldo que el operador ofrece a los proveedores de juego |
| **Entorno** | Pico de diseño: ~3.000 operaciones de saldo por segundo. La red entre el proveedor y el operador está congestionada y pierde respuestas. Ronda de *crash* en curso con 2.400 jugadores. 5 proveedores conectados, cada uno con su propio protocolo |
| **Respuesta** | Abona los COP 50.000 una sola vez. A la segunda y a la tercera petición les responde con el mismo resultado y el mismo saldo que a la primera. Si llegara una petición del mismo proveedor, para el mismo jugador y con el mismo identificador, pero con datos distintos (otro monto u otro tipo de operación), la rechaza y genera una alerta. Registra cada petición recibida para la conciliación con el proveedor. Reporta la transacción una sola vez al sistema de inspección de Coljuegos |
| **Medida de la respuesta** | **0 abonos duplicados.** **100 %** de las peticiones repetidas reciben una respuesta idéntica a la original, en **p99 ≤ 250 ms**. **0 diferencias** en la conciliación diaria de saldos operador–proveedor. **0 transacciones duplicadas** en el reporte a Coljuegos. **100 %** de las peticiones que repiten proveedor, jugador e identificador con datos distintos rechazadas, con alerta en **≤ 5 min** |

**Restricciones del proyecto que también fuerzan esta decisión:**
- Los identificadores de transacción con proveedores se conservan **≥ 4 meses** para conciliación (ficha §4.5, F24).
- El alcance de unicidad del identificador lo define cada proveedor: en algunos protocolos es único solo por jugador, y dos jugadores pueden compartir el mismo identificador (F26). Una petición repetida se reconoce por la combinación de proveedor, jugador e identificador.
- Registros de toda operación del jugador con ≥ 6 meses disponibles (C11) [F1].
- Un proveedor nuevo sobre otra plataforma obliga a recertificar la integración (B4) [F1]; se suman 1 o 2 proveedores al año (S11).
- Volumen: ~24 millones de operaciones de saldo al día (ficha §4.5).

### Opciones consideradas

**A · Buscar el identificador en el historial de operaciones.** Ante cada petición, se busca en el historial completo de operaciones del jugador si ese identificador ya se aplicó. Si aparece, se reconstruye la respuesta; si no, se aplica la operación.

**B · Registro de identificadores procesados, guardado junto con el cambio de saldo.** Al llegar una petición, el operador anota la combinación de proveedor, jugador e identificador, junto con la respuesta que dio, **en el mismo paso indivisible** en que modifica el saldo. Si el identificador ya estaba anotado, devuelve la respuesta guardada sin tocar el saldo. Si estaba anotado con datos distintos, rechaza y alerta. Las anotaciones se conservan durante el periodo de conciliación (≥ 4 meses).

**C · Una pieza de traducción por proveedor que descarta repetidos antes de llegar al saldo.** Cada proveedor tiene su propia pieza que lleva su protocolo al modelo del operador y descarta las peticiones repetidas que recuerda de las últimas 24 horas. Al saldo solo le llegan operaciones nuevas.

| Opción | Gana | Paga | Atributo sacrificado | ¿Cumple la medida? |
|--------|------|------|----------------------|--------------------|
| A · Buscar en el historial | No añade estado nuevo: usa el historial que ya existe por obligación | El tiempo de búsqueda crece con el historial (~24 M de operaciones al día). Dos copias que llegan al mismo tiempo pueden pasar ambas la búsqueda antes de que alguna quede registrada, y aplicarse dos veces | Desempeño y Fiabilidad (integridad) | **No**: no garantiza 0 abonos duplicados cuando las repeticiones llegan a la vez, y su tiempo crece |
| B · Registro de identificadores junto con el saldo | Tiempo constante. Cada operación se aplica exactamente una vez, incluso con copias simultáneas. Respuesta idéntica garantizada. Sirve igual para los 5 proveedores | Almacenamiento: ~24 M de anotaciones al día durante ≥ 4 meses (~2.900 M de registros). Cada operación de saldo hace una escritura más en su mismo paso | Desempeño (utilización de recursos) | **Sí**, si la escritura adicional cabe en p99 ≤ 250 ms |
| C · Traductor por proveedor que filtra repetidos | Aísla las particularidades de cada protocolo, lo que facilita sumar proveedores nuevos. El núcleo del saldo no conoce protocolos externos | Su memoria de 24 h no detecta repeticiones tardías (por ejemplo, una reversión días después). Si el traductor se cae, pierde su memoria. Son 5 piezas que mantener y recertificar | Fiabilidad (integridad) y Mantenibilidad | **Parcial**: no garantiza 0 duplicados fuera de la ventana de 24 h ni tras una caída del traductor |

### Pendiente para el Bloque 3

- **Decision:** abierta. No elegimos todavía.
- **Para decidir con fundamento hace falta:**
  - medir el costo de la escritura adicional de la opción B en el pico;
  - definir el plan de almacenamiento de ~2.900 M de anotaciones;
  - evaluar si B y C se combinan: traducción por proveedor para el protocolo y registro junto al saldo para la garantía.
- **Familias de tácticas por revisar:** *Gestionar interfaces* (Compatibilidad), *Prevenir fallas* (Fiabilidad) y *Reducir el acoplamiento* (Mantenibilidad).
- **Relación con ADR-02:** las dos decisiones afectan el mismo paso de escritura del saldo, así que deben tomarse juntas en el Bloque 3.
- **Consequences:** se escriben al decidir.

---

## Verificación de la fase (rúbrica)

| Criterio | Nivel buscado | Cómo se cumple |
|----------|---------------|----------------|
| Selección de escenarios críticos | Logrado | Cada ADR responde a un escenario con dominante alto que cuesta otro atributo alto; la selección está justificada en una línea |
| ADR: contexto y estado | Logrado | Context idéntico al escenario (7 filas copiadas sin cambios), título numerado que nombra la decisión y estado Propuesto con lo pendiente declarado |
| ADR: opciones y trade-offs | Logrado | Tres opciones reales por ADR, descritas como mecanismos, cada una con lo que gana, lo que paga, el atributo sacrificado y si cumple la medida |

**Otros chequeos:**
- **Ninguna opción usa nombres de tácticas ni marcas.** Por ejemplo, se escribe "una libreta de solo agregar confirmada por mayoría", no el nombre técnico del patrón.
- **Ningún ADR elige.** La sección Decision queda abierta en los tres.
- **Las opciones no son relleno.** En cada ADR hay al menos una opción que no cumple la medida (A en ADR-01, B en ADR-02, A en ADR-03). Así se ve que la medida sirve para descartar.

---

## Punto de control (cerrado el 26 de octubre de 2026)

1. **Opciones aprobadas.** El equipo no discutió alternativas adicionales.
2. **Las respuestas "Depende" se mantienen:** se resuelven midiendo en el Bloque 3.

> **Fase 8:** el Context de ADR-03 se actualizó junto con ESC-03 (regla de identificadores repetidos). Ver `08-revision.md`, hallazgo H1.
