# Fase 6 — Escenarios de calidad

> [← Anterior](./05-seleccion-escenarios.md) · [Índice](./README.md) · [Siguiente →](./07-adrs.md)

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Insumos:**
- [`05-seleccion-escenarios.md`](./05-seleccion-escenarios.md): selección aprobada.
- [`04-priorizacion.md`](./04-priorizacion.md): puntajes.
- [`03-mapa-tensiones.md`](./03-mapa-tensiones.md): tensiones.
- [`02-ficha-contexto.md`](./02-ficha-contexto.md): cifras.

**Formato:** cada escenario tiene el atributo y sus seis partes (fuente, estímulo, artefacto, entorno, respuesta y medida de la respuesta). Ese bloque de 7 filas es el que pasa **tal cual** al Context de su ADR en la fase 7. Debajo de cada escenario hay notas de apoyo que **no** forman parte del escenario.

**Regla de esta fase:** ninguna respuesta nombra tácticas ni tecnologías. Se describe lo que se observa desde afuera.

---

## Resumen

| ID | Atributo dominante (puntaje) | Paga | Tensión | ¿Crítico? | ADR |
|----|------------------------------|------|---------|:---------:|-----|
| ESC-01 | Seguridad (4) | Desempeño (3) | T1 | ✔ | ADR-01 |
| ESC-02 | Fiabilidad (4) | Desempeño (3) | T2 | ✔ | ADR-02 |
| ESC-03 | Compatibilidad (3) | Fiabilidad (4) | T3 | ✔ | ADR-03 |
| ESC-04 | Eficiencia de desempeño (3) | Adecuación funcional (1) | T6 | ✘ | — |

---

## ESC-01 · Retiro de un jugador incluido en una lista recién recibida

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Seguridad, con una restricción de desempeño |
| **Fuente del estímulo** | Un jugador cuyo documento aparece en una actualización de listas restrictivas que el operador recibió hace 30 minutos, y cuya cuenta todavía no ha sido reconsultada |
| **Estímulo** | Solicita un retiro de COP 3.000.000 hacia su billetera digital |
| **Artefacto** | El proceso de solicitud de retiros y su evaluación de riesgo de lavado de activos (SARLAFT) |
| **Entorno** | Viernes de quincena a las 21:00. 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo, ~1.800 depósitos en la hora. Reconsulta de las 350.000 cuentas contra la lista nueva en curso, iniciada hace 30 minutos |
| **Respuesta** | Evalúa el retiro contra la versión de la lista recién recibida, aunque la cuenta no haya sido reconsultada todavía. Si el jugador está en la lista: no paga el retiro, suspende la cuenta por precaución, alerta al oficial de cumplimiento, deja el caso listo para el reporte a la UIAF y le muestra al jugador el retiro "en revisión", sin revelarle el motivo. Si el jugador no está en la lista y su riesgo es normal: el retiro sigue su curso habitual. El juego y los depósitos de los demás jugadores siguen operando |
| **Medida de la respuesta** | **0 retiros pagados** a personas incluidas en una lista ya recibida, verificado cruzando pagos y listas por marca de tiempo. Evaluación del retiro en **p95 ≤ 1 s**, también durante la reconsulta. Alerta al oficial de cumplimiento en **≤ 15 min**. Durante la reconsulta, las operaciones de saldo del juego se mantienen en **p99 ≤ 250 ms**. Reconsulta completa en **≤ 6 h**. Los retiros de riesgo normal mantienen la orden de transferencia en **≤ 24 h** |

**Casos que distingue la respuesta:**
- persona en la lista recibida → retiro retenido, cuenta suspendida, alerta;
- persona fuera de la lista y de riesgo normal → retiro normal;
- el resto de jugadores → sin afectación.

**Por qué es crítico (T1):** la Seguridad (4) exige evaluar contra la lista más reciente en el camino de cada retiro, mientras una reconsulta masiva consume recursos en pleno pico. Cumplirlo presiona el Desempeño (3): la evaluación del retiro y las operaciones de saldo del juego deben mantener sus tiempos.

**Trazabilidad:** ficha §4.2, §4.4, S18–S21; restricción 9 (fallo cerrado, que irá al Context de ADR-01); investigación §1.3 y obligación C4.

---

## ESC-02 · Caída del servidor de saldo en pleno pico

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Fiabilidad, con una restricción de desempeño |
| **Fuente del estímulo** | Una falla de hardware en el servidor que registra las operaciones de saldo de los jugadores |
| **Estímulo** | El servidor deja de responder abruptamente, con operaciones de saldo en curso |
| **Artefacto** | La cuenta de juego: el registro del saldo de cada jugador y de sus operaciones de descuento, abono y reversión |
| **Entorno** | Sábado de quincena a las 21:30, en el pico de diseño. 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo, confirmadas a los proveedores con p99 ≤ 250 ms. Una ronda de *crash* en curso con 2.400 apuestas registradas. Sistema técnico de juego alterno certificado, exigido por el regulador |
| **Respuesta** | Conserva todas las operaciones que ya había confirmado a los proveedores. Las operaciones que estaban en curso sin confirmar quedan sin efecto sobre el saldo, de forma que el proveedor y el operador terminen con el mismo saldo. Avisa a los jugadores afectados que el juego no está disponible. Restablece el juego y el saldo. Al reconectarse, cada jugador ve el resultado de la jugada que quedó incompleta. Registra el inicio, la duración y los servicios afectados, e informa a Coljuegos |
| **Medida de la respuesta** | **0 operaciones confirmadas perdidas**, verificado por la conciliación operador–proveedores posterior al evento. **0 saldos negativos.** Juego y saldo restablecidos en **≤ 15 min**. **100 %** de las jugadas incompletas con su resultado visible al reconectarse. Aviso a Coljuegos en **≤ 30 min**. A los **5 min** del restablecimiento, las operaciones de saldo vuelven a **p99 ≤ 250 ms** |

**Casos que distingue la respuesta:**
- operación ya confirmada → se conserva;
- operación en curso sin confirmar → sin efecto, con ambos saldos coincidiendo;
- jugada incompleta → resultado visible al volver.

**Por qué es crítico (T2):** la Fiabilidad (4) exige que ninguna de las ~3.000 operaciones por segundo que se confirmaron se pierda en la caída. Esa garantía tiene que cumplirse **sin que la confirmación supere p99 ≤ 250 ms** en el pico, y por eso presiona el Desempeño (3).

**Trazabilidad:** ficha §4.2, §4.3, S6, S13, S14; investigación C6, D6, E1–E5. El aviso a Coljuegos en ≤ 30 min concreta el "de inmediato" del reglamento **[nuevo supuesto S22]**. La vuelta a p99 ≤ 250 ms en 5 min se respalda con S24 (agregado en la fase 8).

---

## ESC-03 · El proveedor del *crash* envía tres veces el mismo abono

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Compatibilidad (interoperabilidad), con una restricción de fiabilidad |
| **Fuente del estímulo** | El proveedor externo del juego *crash* |
| **Estímulo** | Envía tres veces la misma petición de abono de COP 50.000, con el mismo identificador de transacción, porque no recibió respuesta a la primera dentro de su tiempo de espera de ~3 s |
| **Artefacto** | La interfaz de operaciones de saldo que el operador ofrece a los proveedores de juego |
| **Entorno** | Pico de diseño: ~3.000 operaciones de saldo por segundo. La red entre el proveedor y el operador está congestionada y pierde respuestas. Ronda de *crash* en curso con 2.400 jugadores. 5 proveedores conectados, cada uno con su propio protocolo |
| **Respuesta** | Abona los COP 50.000 una sola vez. A la segunda y a la tercera petición les responde con el mismo resultado y el mismo saldo que a la primera. Si llegara una petición del mismo proveedor, para el mismo jugador y con el mismo identificador, pero con datos distintos (otro monto u otro tipo de operación), la rechaza y genera una alerta. Registra cada petición recibida para la conciliación con el proveedor. Reporta la transacción una sola vez al sistema de inspección de Coljuegos |
| **Medida de la respuesta** | **0 abonos duplicados.** **100 %** de las peticiones repetidas reciben una respuesta idéntica a la original, en **p99 ≤ 250 ms**. **0 diferencias** en la conciliación diaria de saldos operador–proveedor. **0 transacciones duplicadas** en el reporte a Coljuegos. **100 %** de las peticiones que repiten proveedor, jugador e identificador con datos distintos rechazadas, con alerta en **≤ 5 min** |

**Casos que distingue la respuesta:**
- petición nueva → se aplica;
- petición repetida idéntica → misma respuesta, sin volver a aplicarse;
- mismo proveedor, jugador e identificador con datos distintos → se rechaza y genera alerta;
- mismo identificador para otro jugador → es una operación nueva, porque algunos proveedores solo garantizan identificadores únicos por jugador [F26].

**Por qué es crítico (T3):** la Compatibilidad (3) exige aceptar el protocolo de cada proveedor tal como viene, con sus repeticiones, sus reversiones y sus tiempos de espera. Cada variante es una forma de descontar o abonar dos veces, así que cumplirla presiona la Fiabilidad (4) del saldo.

**Trazabilidad:** ficha §3, §4.2, S11, S14, S25 (alerta en ≤ 5 min, agregado en la fase 8); investigación A3, C6; F24, F26.

---

## ESC-04 · Ráfaga de retiros del *crash* (no crítico)

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Eficiencia de desempeño |
| **Fuente del estímulo** | El proveedor externo del juego *crash*, después de que 720 jugadores de una misma ronda se retiran con el multiplicador en 2,00x |
| **Estímulo** | Envía al operador, en el mismo segundo, 720 peticiones de abono de premios |
| **Artefacto** | El proceso de abono de premios del *crash* al saldo de los jugadores |
| **Entorno** | Pico de diseño: 2.400 jugadores en la ronda y ~3.000 operaciones de saldo por segundo de los demás juegos |
| **Respuesta** | Abona cada premio por el monto que liquidó el proveedor. Cada jugador ve su saldo actualizado. Las operaciones de los demás juegos siguen atendiéndose |
| **Medida de la respuesta** | Los 720 abonos responden en **p99 ≤ 250 ms**. Saldo actualizado visible al jugador en **p95 ≤ 2 s**. **0 diferencias** entre el monto abonado y el liquidado por el proveedor. Durante la ráfaga, las operaciones de los demás juegos se mantienen en **p99 ≤ 250 ms** |

**Por qué NO es crítico:** el dominante, Desempeño (3), es alto. Pero su costo cae en la corrección del premio (Adecuación funcional, 1). Esa corrección la calcula el proveedor y la certifica el laboratorio, así que cumplir este escenario no obliga al operador a sacrificar un atributo que priorizamos alto. **Se escribe sin ADR, a propósito:** muestra el criterio de selección.

**Trazabilidad:** ficha §4.1, §4.2, S5, S6, S15, S23 (saldo visible en p95 ≤ 2 s, agregado en la fase 8); mapa T6; priorización §2.2.

---

## Lista de chequeo

Revisión según el Constructor y la Clínica del taller GPSMetro.

| Verificación | ESC-01 | ESC-02 | ESC-03 | ESC-04 |
|--------------|:------:|:------:|:------:|:------:|
| La fuente es un actor concreto, no una condición | ✔ | ✔ | ✔ | ✔ |
| Un solo estímulo, observable por el sistema | ✔ | ✔ | ✔ | ✔ |
| Artefacto nombrado por su función, sin componentes de solución | ✔ | ✔ | ✔ | ✔ |
| Entorno con cifras (carga, franja, modo) | ✔ | ✔ | ✔ | ✔ |
| Respuesta observable, sin tácticas ni tecnologías | ✔ | ✔ | ✔ | ✔ |
| La respuesta distingue los casos relevantes | ✔ | ✔ | ✔ | ✔ |
| Medida con número, unidad y percentil o umbral; sin adjetivos ni promedios | ✔ | ✔ | ✔ | ✔ |
| Atributo correcto; no se confunde disponibilidad con corrección del dato | ✔ | ✔ | ✔ | ✔ |
| Dominante rastreable a un puntaje alto | 4 | 4 | 3 | 3 |
| Tensión explícita con otro atributo | Desempeño (3) | Desempeño (3) | Fiabilidad (4) | Adec. funcional (1) |

**Notas de revisión:**
- **ESC-02 es de Fiabilidad y no solo de disponibilidad.** Además de volver a estar arriba, el sistema debe tener los saldos correctos. Es la lección del caso 6 de la Clínica: un sistema que está arriba pero dice algo falso falla en Fiabilidad.
- **ESC-03 declara Compatibilidad, aunque mide integridad del saldo.** El escenario trata de usar correctamente la información intercambiada con un sistema externo, que es la definición de interoperabilidad en el widget de ISO 25010. La integridad aparece como restricción porque es el atributo que paga.
- **Ninguna respuesta nombra mecanismos.** Las palabras que el Constructor marca como tácticas (caché, réplica, cola, reintento…) no aparecen en artefactos ni en respuestas. "Reintento" solo describiría el estímulo, y aun ahí se evitó.
- **Correcciones de la fase 8.** En ESC-04, la fuente y el estímulo pasaron a ser lo que el operador observa: las peticiones del proveedor, no el clic del jugador, que ocurre en el sistema del proveedor. En ESC-03 se precisó cuándo una petición es "la misma": misma combinación de proveedor, jugador e identificador.

---

## Nuevo supuesto

| # | Supuesto | Razón |
|---|----------|-------|
| S22 | El aviso a Coljuegos ante una falla grave se da en ≤ 30 min | El reglamento dice "de inmediato" (E3, E5), y "inmediato" no es verificable. Esta cifra lo vuelve medible. Agregado a la ficha. |

---

## Verificación de la fase (rúbrica)

| Criterio | Nivel buscado | Cómo se cumple |
|----------|---------------|----------------|
| Estímulo y entorno | Logrado | Estímulos observables por el sistema; entornos con carga, franja horaria y modo de operación |
| Respuesta | Logrado | Respuestas concretas que distinguen dos o tres casos cada una |
| Medida de la respuesta | Logrado | Cifras verificables con su método de verificación (conciliación, cruce por marca de tiempo, percentiles) |
| Coherencia con la priorización | Logrado | Cada dominante sale de un puntaje alto, y la tensión con otro atributo queda explícita |

---

## Punto de control (cerrado el 25 de octubre de 2026)

1. **Textos aprobados.** Pasan tal cual al Context de los ADR.
2. **S22 aprobado** y agregado a la ficha (§4.3 y supuestos).

---

> [← Anterior](./05-seleccion-escenarios.md) · [Índice](./README.md) · [Siguiente →](./07-adrs.md)
