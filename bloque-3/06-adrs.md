# Fase 6 — ADR completados

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 5 (un bloque por ADR: encabezado, Decision, "hasta dónde" y Consequences; las tácticas T1, T2… están en `05`)
**Criterios de la rúbrica:** Decision · Hasta dónde. Nivel "Logrado": Decision en voz activa, justificada frente a la medida de la respuesta y frente a cada opción descartada; nuevo límite calculado, margen sobre el pico, siguiente componente que se satura y condición de revisión medible.

**Insumos:**
- `bloque-2/06-escenarios.md` y `bloque-2/07-adrs.md`: Context, medida de la respuesta y opciones de cada ADR.
- `03-perfil-de-carga.md`: 3.000 operaciones de saldo/s (4.000 en la ráfaga), 12.000 jugadores, 0,249 operaciones/s por jugador.
- `04-cuello-de-botella.md`: límites de la línea base (base única ~10.000/s; servidor de aplicación 16.000 peticiones/s; proveedor de listas 5 consultas/s).
- `05-tacticas-y-efecto.md`: fichas TAC-01 a TAC-10 y opción aprobada por ADR (§9).

**Qué fija esta fase:**
1. Los cuatro ADR pasan a **Aceptado**, con la opción que el equipo aprobó en la fase 5.
2. Cada Decision se justifica frente a la medida y frente a cada opción descartada, con los números de la fase 5.
3. Cada ADR calcula su "hasta dónde" con las cifras ya registradas. **No se agregan fuentes ni supuestos nuevos.**
4. Cada ADR cierra con una condición de revisión medible. **Los umbrales son propuestas para el punto de control** (sección 6).

Las fichas de las tácticas no se repiten aquí: cada ADR enlaza las suyas en `05`. El Context y la medida de la respuesta se copian de `bloque-2/07-adrs.md` (idénticos a `bloque-2/06-escenarios.md`, verificado en §8).

---

## 1. Resumen

| ADR | Opción elegida | Tácticas | Límite con las tácticas | Margen sobre el pico | ¿Cumple la medida? | Siguiente en saturarse |
|-----|----------------|----------|-------------------------|----------------------|--------------------|------------------------|
| ADR-01 | C · lista vigente propia | TAC-09, TAC-10 | Reconsulta: 1.080.000 cuentas en 6 h | 3,1 sobre las 350.000 cuentas | Sí | Base única (+0,01 de uso por la reconsulta) |
| ADR-02 | C · libreta confirmada por mayoría | TAC-01, TAC-02, TAC-03 | ~10.000 operaciones de saldo/s (sin cambio) | 3,3 (2,5 en la ráfaga) | Sí | Base única |
| ADR-03 | B con la traducción de C | TAC-04, TAC-05, TAC-06 | ~10.000 operaciones de saldo/s (sin cambio) | 3,3 (2,5 en la ráfaga) | Sí | Base única (por volumen: tope de 2,5 TB, 1,8 veces el actual) |
| ADR-04 | B · firma por petición | TAC-06, TAC-07, TAC-08 | ~44.400 jugadores en el servidor de aplicación (antes ~45.800) | 3,7 | Depende de S42 (a) | Base única (~40.000 jugadores) |

**Lectura:** las tácticas de la operación de saldo compran garantías (0 pérdidas, 0 duplicados, origen verificado), no capacidad: el límite sigue siendo el de la base única. Es la pista que la fase 7 deja para el Bloque 4.

---

## 2. ADR-01: Evaluación de los retiros contra listas restrictivas recién recibidas

- **Estado:** **Aceptado** (15 de noviembre de 2026).
- **Escenario que lo origina:** ESC-01.
- **Tácticas:** TAC-09 Múltiples copias de datos, TAC-10 Limitar la tasa de eventos (fichas en `05` §4).

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

### Opciones del Bloque 2

- **A ·** Retener los retiros hasta que termine la reconsulta.
- **B ·** Consultar al proveedor de listas en cada retiro.
- **C ·** Lista vigente consultable por el operador, independiente de la reconsulta.

**Opción elegida: C.**

### Decision

Usaremos una **lista vigente propia en la memoria del servidor de aplicación** (TAC-09) para evaluar cada retiro desde el minuto en que llega la actualización del proveedor, sin esperar a la reconsulta. Limitaremos la reconsulta de las 350.000 cuentas a **50 cuentas por segundo** (TAC-10), comparando cada cuenta con esa lista y no con el proveedor. La lista nueva solo reemplaza a la anterior cuando termina de cargarse; mientras carga, los retiros esperan (fallo cerrado).

### Por qué se descartan las demás opciones

| Opción | Frente a la medida | Número |
|--------|--------------------|--------|
| **A · Retener hasta terminar la reconsulta** | Incumple la evaluación en p95 ≤ 1 s | El retiro espera hasta ~19,4 h (la reconsulta corre al ritmo del proveedor, 350.000 ÷ 5 /s = 70.000 s) y se retienen ~2.430 retiros por reconsulta (`04` §2.7). Un retiro pedido al empezar consume casi toda la meta de 24 h |
| **B · Consultar al proveedor en cada retiro** | Incumple p95 ≤ 1 s durante la reconsulta y la disponibilidad de los retiros | El proveedor admite 5 consultas/s [F37] y la reconsulta ya le pide 16,2/s (uso 3,24): cada retiro haría fila detrás de ella o sería rechazado. Con fallo cerrado, una caída del proveedor detiene **todos** los retiros |

### Tácticas

| ID | Táctica (catálogo) | Métrica | Antes → después |
|----|--------------------|---------|-----------------|
| TAC-09 | Múltiples copias de datos | Evaluación del retiro durante la reconsulta | hasta ~19,4 h → ≤ 10 ms |
| TAC-10 | Limitar la tasa de eventos | Uso de la base durante la reconsulta · duración (con TAC-09) | ~1,0 sin tope → 0,31 · ~19,4 h → ~1,9 h |

### Hasta dónde llegan estas tácticas

- **Nuevo límite del sistema con las tácticas (tasa por segundo):** la reconsulta cumple las 6 h hasta **1.080.000 cuentas**, y el proveedor de listas queda a 0,002 de su límite. La evaluación del retiro ya no depende de la reconsulta.
- **Cómo se calculó:**

```
reconsulta a 50 cuentas/s en 6 h     50 /s × 21.600 s = 1.080.000 cuentas
margen sobre las cuentas de hoy      1.080.000 ÷ 350.000 = 3,1
duración de la reconsulta de hoy     350.000 ÷ 50 /s = 7.000 s = 1,94 h        ≤ 6 h
uso del proveedor de listas          ~1.000 registros al día ÷ 86.400 s = 0,012 /s ÷ 5 /s = 0,002
uso de la base única                 0,30 + (50 lecturas + 50 escrituras) ÷ 10.000 = 0,31
```

- **Siguiente componente que se satura:** la **base única**. La reconsulta le suma 0,01 de uso por cada 50 cuentas/s, así que subir el ritmo (la variante de 100/s suma 0,02) compite con el saldo.
- **¿Se cumple la medida con la carga del pico?** **Sí.**

| Parte de la medida | Valor con la decisión | Cumple |
|--------------------|-----------------------|--------|
| 0 retiros pagados a personas de una lista ya recibida | La evaluación usa la lista nueva desde que termina de cargarse (≤ 1 min); solo esperan ~2 retiros por actualización (125 por hora ÷ 60) y no se pagan | Sí |
| Evaluación p95 ≤ 1 s, también durante la reconsulta | ≤ 10 ms [S42] | Sí |
| Alerta al oficial de cumplimiento ≤ 15 min | No cambia: la alerta nace de la evaluación, que ahora tarda ≤ 10 ms | Sí |
| Saldo p99 ≤ 250 ms durante la reconsulta | La base sube de 0,30 a 0,31 de uso | Sí |
| Reconsulta completa ≤ 6 h | ~1,9 h | Sí |
| Orden de transferencia de retiros normales ≤ 24 h | Sin espera por la reconsulta | Sí |

### Consequences

- **Gana:** la evaluación del retiro pasa de hasta ~19,4 h a ≤ 10 ms; la reconsulta, de ~19,4 h a ~1,9 h; el uso del proveedor de listas, de 3,24 a 0,002, y los retiros dejan de depender de un tercero en el camino crítico.
- **Paga:** ~500 MB de memoria de 128 GB (0,4 %) [S42]; dos procesos que mantener coherentes (lista vigente y reconsulta); una marca de riesgo que cambie llega a la cuenta hasta ~1,9 h después de recibir la lista; si el servidor de aplicación se reinicia, la lista se recarga y los retiros esperan ≤ 1 min.
- **No resuelve:** (1) las coincidencias parciales de nombre (homonimia): la lista se indexa por número de documento; (2) la opción C supone que el proveedor entrega la **lista completa** y no solo respuestas a consultas por documento; la lista vinculante de Naciones Unidas tiene 877 registros [F42] y el millón de S42 es una cota; (3) la lista vive en memoria: no sobrevive a un reinicio sin recargarse.
- **Condición de revisión (propuesta):** revisar esta decisión si **(a)** las cuentas registradas superan **~1.000.000** (la reconsulta a 50/s llegaría a 5,6 h, cerca de las 6 h), **(b)** la lista del proveedor supera **1 M de registros** o su carga tarda más de **1 min** [S42], o **(c)** el proveedor deja de entregar la lista completa.

**Dónde está este ADR en la entrega:** presentación HTML, sección "ADR" (fase 10), y este archivo.

---

## 3. ADR-02: Permanencia de las operaciones de saldo confirmadas ante la caída de un servidor

- **Estado:** **Aceptado** (15 de noviembre de 2026).
- **Escenario que lo origina:** ESC-02.
- **Tácticas:** TAC-01 Redundancia activa, TAC-02 Latido (heartbeat), TAC-03 Reconfiguración (fichas en `05` §2.1).

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

### Opciones del Bloque 2

- **A ·** Confirmar solo después de escribir en dos sitios físicos.
- **B ·** Confirmar tras la grabación local y copiar después.
- **C ·** Libreta de operaciones de solo agregar, confirmada por mayoría.

**Opción elegida: C.**

### Decision

Usaremos una **libreta de operaciones de solo agregar, mantenida por tres servidores de base de datos del sitio principal** (TAC-01), y responderemos al proveedor cuando al menos dos de los tres la tengan grabada en disco. Detectaremos la caída del servidor que atiende con una señal de vida entre los tres, con un intervalo aleatorio de 150 a 300 ms (TAC-02), y los dos que quedan elegirán al que tiene la libreta completa para atender (TAC-03). El sistema alterno seguirá recibiendo la copia leyendo la libreta, fuera del camino de cada jugada.

### Por qué se descartan las demás opciones

| Opción | Frente a la medida | Número |
|--------|--------------------|--------|
| **A · Dos sitios antes de confirmar** | Cumple la pérdida cero y la latencia, pero **incumple la disponibilidad de juego y saldo** | La ida y vuelta al alterno cuesta ~10 ms de 250 ms [F39][S38] y 30 operaciones esperando a la vez: cabe. Pero con un solo enlace a 99,9 % [S39] hay 0,001 × 43.200 min = **43,2 min al mes** sin enlace, exactamente el margen de 99,9 % de S13. En esos minutos se detiene el juego o se vuelve a la línea base |
| **B · Grabación local y copia posterior** | **Incumple** 0 operaciones confirmadas perdidas | Es la línea base: ~3.000 operaciones perdidas (1 s de atraso × 3.000 /s; 4.000 en la ráfaga; hasta 15.000 con 5 s) [S37] y 10 a 30 min para restablecer [S40], contra ≤ 15 min de la medida |

### Tácticas

| ID | Táctica (catálogo) | Métrica | Antes → después |
|----|--------------------|---------|-----------------|
| TAC-01 | Redundancia activa | Operaciones confirmadas perdidas | ~3.000 → 0 |
| TAC-02 | Latido (heartbeat) | Tiempo de detección de la caída | manual, dentro de 10–30 min → ≤ 0,3 s |
| TAC-03 | Reconfiguración | Tiempo con el saldo detenido | 10–30 min → ≤ 30 s |

### Hasta dónde llegan estas tácticas

- **Nuevo límite del sistema con las tácticas (tasa por segundo):** **~10.000 operaciones de saldo/s**, sin cambio. TAC-01 no mueve el límite: la prueba de referencia ya confirma cada transacción después de que una réplica la recibe [F34], y la espera local suma ~1 ms [S40].
- **Cómo se calculó:**

```
margen sobre el pico            10.000 ÷ 3.000 = 3,3
margen sobre la ráfaga          10.000 ÷ 4.000 = 2,5
tolerancia a fallas             1 servidor de 3 (mayoría de 2)
cambio de servidor              3.000 /s × 30 s = ~90.000 operaciones sin respuesta
vuelta a p99 ≤ 250 ms           ~90.000 reenvíos ÷ (10.000 − 3.000) /s ≈ 13 s     (medida: ≤ 5 min; ~39 s si cada operación llega 3 veces, `07` I1)
```

- **Siguiente componente que se satura:** la **base única**, ahora el servidor de base de datos que atiende: todas las escrituras del saldo pasan por él, y los otros dos reciben la libreta pero no suman capacidad de escritura.
- **¿Se cumple la medida con la carga del pico?** **Sí**, para la caída de un servidor.

| Parte de la medida | Valor con la decisión | Cumple |
|--------------------|-----------------------|--------|
| 0 operaciones confirmadas perdidas | 0 mientras sobreviva 1 de los otros 2 servidores | Sí |
| 0 saldos negativos | No cambia: la regla de saldo no negativo sigue en la transacción (C6) | Sí |
| Juego y saldo restablecidos en ≤ 15 min | ≤ 30 s; margen 15 min ÷ 0,5 min = 30 veces | Sí |
| 100 % de jugadas incompletas con resultado visible | Las operaciones en curso sin confirmar no tienen efecto y el proveedor las revierte o reenvía [F24] | Sí |
| Aviso a Coljuegos ≤ 30 min | No cambia; la detección en ≤ 0,3 s adelanta el registro del inicio del evento | Sí |
| p99 ≤ 250 ms a los 5 min del restablecimiento | ≈ 13 s para absorber los reenvíos (~39 s con 3 copias por operación, `07` I1) | Sí |

### Consequences

- **Gana:** la pérdida de operaciones confirmadas pasa de ~3.000 a 0, y el restablecimiento de 10–30 min a ≤ 30 s.
- **Paga:** ~USD 24.000 al año por 2 servidores más (~COP 96 M, 1,3 % del presupuesto de tecnología) [F41][S9]; +~1 ms por operación; ~17 Mbit/s de red interna; hasta ~90.000 operaciones revertidas o reenviadas en cada cambio de servidor; una pausa de red de más de 300 ms dispara un cambio innecesario (~30 s sin saldo); tres servidores que operar y recertificar.
- **No resuelve:** (1) **la pérdida del sitio completo** sigue dejando ~3.000 operaciones en riesgo, porque el alterno recibe la copia con atraso (S37); (2) mientras no se reponga el servidor caído quedan 2 de 3 y una segunda falla detiene el juego; (3) el límite de escritura sigue siendo el de un solo servidor.
- **Condición de revisión (propuesta):** revisar si **(a)** el pico sostenido supera **~7.500 operaciones/s** (~30.000 jugadores, `04` §5: es el punto donde la ráfaga deja de caber), **(b)** hay más de **20 cambios de servidor al mes** (20 × 0,5 min = 10 min, casi una cuarta parte de los 43,2 min de S13), o **(c)** el atraso de la copia al alterno supera **5 s** (15.000 operaciones en riesgo).

**Dónde está este ADR en la entrega:** presentación HTML, sección "ADR" (fase 10), y este archivo.

---

## 4. ADR-03: Tratamiento de las peticiones de saldo repetidas de los proveedores de juego

- **Estado:** **Aceptado** (15 de noviembre de 2026).
- **Escenario que lo origina:** ESC-03.
- **Tácticas:** TAC-04 Transacciones, TAC-05 Validar entradas, TAC-06 Usar un intermediario (fichas en `05` §2.2).

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

### Opciones del Bloque 2

- **A ·** Buscar el identificador en el historial de operaciones.
- **B ·** Registro de identificadores procesados, guardado junto con el cambio de saldo.
- **C ·** Una pieza de traducción por proveedor que descarta repetidos antes de llegar al saldo.

**Opción elegida: una combinación, B con la traducción de C.** El Bloque 2 dejó esa combinación por evaluar (`bloque-2/07-adrs.md`, "Pendiente para el Bloque 3").

### Decision

Usaremos un **registro de identificadores procesados dentro de la misma transacción que cambia el saldo** (TAC-04), con clave única de proveedor, jugador e identificador. Si la clave ya existe, devolveremos la respuesta guardada sin tocar el saldo; si existe con datos distintos, rechazaremos la petición y alertaremos (TAC-05). Del traductor por proveedor tomaremos solo la **traducción del protocolo**: una pieza por proveedor entrega al saldo una petición común (TAC-06). Su memoria de repetidos de 24 h no se usa.

### Por qué se descartan las demás opciones

| Opción | Frente a la medida | Número |
|--------|--------------------|--------|
| **A · Buscar en el historial** | **Incumple** 0 abonos duplicados | Es la línea base: dos copias que llegan a la vez pasan ambas la búsqueda antes de que alguna quede grabada y se aplican dos veces. Ocurre cuando la primera copia tarda más que los ~3 s del proveedor: con la base saturada o tras un cambio de servidor (hasta ~90.000 reenvíos, ADR-02). Además la búsqueda recorre ~4.400 M de registros del historial |
| **C · Traductor que filtra repetidos** | **Parcial**: no garantiza 0 duplicados | Su memoria de 24 h no ve repeticiones tardías: un proveedor puede reenviar una reversión días después, dentro de los 4 meses de conciliación [F24], y una caída del traductor borra la memoria. Su parte de traducción sí se toma (TAC-06) |

### Tácticas

| ID | Táctica (catálogo) | Métrica | Antes → después |
|----|--------------------|---------|-----------------|
| TAC-04 | Transacciones | Abonos duplicados con copias simultáneas | 1 por par solapado → 0 |
| TAC-05 | Validar entradas | Rechazo de misma clave con datos distintos | 0 % → 100 %, alerta ≤ 1 min |
| TAC-06 | Usar un intermediario | Partes que cambian con un proveedor nuevo | interfaz de saldo → 1 pieza nueva |

### Hasta dónde llegan estas tácticas

- **Nuevo límite del sistema con las tácticas (tasa por segundo):** **~10.000 operaciones de saldo/s**, sin cambio. Por transacción, la línea base con el registro hace 1 cambio de saldo, 1 lectura y 2 inserciones; la prueba de referencia hace 3 cambios de saldo, 1 lectura y 1 inserción [F35]: sigue siendo menos trabajo que la prueba.
- **Cómo se calculó:**

```
escrituras adicionales          1 inserción × 3.000 /s = 3.000 inserciones/s          [S6]
tiempo adicional                ≤ 1 ms de 250 ms                                      [S41]
registro                        3.000 M identificadores × ~100 bytes = ~300 GB       [S41]
datos de la base                1,1 TB de historial + 0,3 TB = 1,4 TB = 2,8 × los 500 GB de F34
tope de datos del descuento     S36 ya descontó 5 × 500 GB = 2,5 TB
margen de datos                 2,5 TB ÷ 1,4 TB = 1,8
margen sobre el pico            10.000 ÷ 3.000 = 3,3   (ráfaga: 10.000 ÷ 4.000 = 2,5)
```

- **Siguiente componente que se satura:** la **base única**, por dos vías: la tasa (~10.000/s) y el **volumen**: a 2,5 TB se llega al escalón de datos que S36 descontó completo; con el registro y el historial creciendo en proporción, eso equivale a ~1,8 veces las operaciones diarias de hoy (~43 M al día).
- **¿Se cumple la medida con la carga del pico?** **Sí.**

| Parte de la medida | Valor con la decisión | Cumple |
|--------------------|-----------------------|--------|
| 0 abonos duplicados | 0: la segunda inserción de la misma clave falla dentro de su transacción | Sí |
| 100 % de repetidas con respuesta idéntica, p99 ≤ 250 ms | Se devuelve la respuesta guardada; +≤ 1 ms | Sí |
| 0 diferencias en la conciliación diaria | El registro guarda cada petición con su respuesta | Sí |
| 0 duplicados en el reporte a Coljuegos | Una sola aplicación por clave | Sí |
| 100 % de misma clave con datos distintos rechazadas, alerta ≤ 5 min | 100 %, alerta ≤ 1 min | Sí |

### Consequences

- **Gana:** 0 duplicados sin importar el momento en que lleguen las copias (incluidos los ~90.000 reenvíos de un cambio de servidor), alerta en ≤ 1 min en lugar de sin alerta, y un proveedor nuevo es 1 pieza nueva con 0 cambios en la interfaz de saldo [S11].
- **Paga:** ~300 GB de disco [S41]; +3.000 inserciones/s y ≤ 1 ms por operación; 5 piezas que mantener y recertificar, cada cambio entre **2 y 4 semanas** de laboratorio [S10]; hasta 240 alertas falsas al día **entre ADR-03 y ADR-04** si los rechazos legítimos llegan al tope de S27, que es común a los dos (`07` I5).
- **No resuelve:** (1) un proveedor que reutilice un identificador para otra operación del mismo jugador ve rechazada una petición legítima; (2) las repeticiones posteriores a los 4 meses de conservación [F24]; (3) el registro suma volumen a la misma base que es el cuello del saldo.
- **Condición de revisión (propuesta):** revisar si **(a)** los datos de la base superan **2,5 TB** o el promedio supera **~43 M de operaciones al día**, **(b)** los rechazos legítimos de ADR-03 y ADR-04, sumados, superan **0,001 %** de las peticiones (240 al día) [S27], o **(c)** un proveedor reutiliza identificadores entre operaciones distintas del mismo jugador.

**Dónde está este ADR en la entrega:** presentación HTML, sección "ADR" (fase 10), y este archivo.

---

## 5. ADR-04: Verificación del origen de las peticiones de saldo de los proveedores

- **Estado:** **Aceptado** (15 de noviembre de 2026).
- **Escenario que lo origina:** ESC-05.
- **Tácticas:** TAC-06 Usar un intermediario (compartida con ADR-03), TAC-07 Verificar integridad del mensaje, TAC-08 Informar a los actores (fichas en `05` §3).

### Context

*El escenario ESC-05, tal cual.*

| Parte | Contenido |
|-------|-----------|
| **Atributo de calidad** | Seguridad (autenticidad e integridad), con una restricción de compatibilidad |
| **Fuente del estímulo** | Un actor externo no autorizado que se hace pasar por uno de los proveedores de tragamonedas |
| **Estímulo** | Envía a la interfaz de saldo una petición de abono de COP 2.000.000 a una cuenta de jugador, con el formato de ese proveedor, pero que no fue emitida por él |
| **Artefacto** | La interfaz de operaciones de saldo que el operador ofrece a los proveedores de juego |
| **Entorno** | Viernes de quincena a las 21:00, en el pico de diseño: 12.000 jugadores simultáneos y ~3.000 operaciones de saldo legítimas por segundo de 5 proveedores, cada uno con su propio esquema de autenticación |
| **Respuesta** | Rechaza la petición sin modificar ningún saldo, la registra con su origen y alerta al equipo de seguridad. Si llegan más peticiones no emitidas por proveedores desde el mismo origen, las sigue rechazando y las agrupa en el mismo incidente. Las peticiones legítimas de los 5 proveedores se siguen aceptando, cada una con el esquema de autenticación propio de su proveedor |
| **Medida de la respuesta** | **100 %** de las peticiones de una batería de falsificación rechazadas (sin credencial, con la credencial de otro proveedor y copia de una petición legítima con el monto modificado), con **0 saldos modificados**, verificado en la prueba de carga y en las pruebas de penetración del laboratorio. **≤ 0,001 %** de las peticiones legítimas rechazadas por la verificación. La verificación añade **≤ 20 ms (p99)** a cada operación, y las operaciones de saldo siguen en **p99 ≤ 250 ms**. Alerta al equipo de seguridad en **≤ 5 min** |

### Opciones del Bloque 2

- **A ·** Clave fija y lista de direcciones de red permitidas por proveedor.
- **B ·** Firma de cada petición con un secreto por proveedor, verificada por una pieza de traducción por proveedor.
- **C ·** Un esquema único, definido por el operador, para todos los proveedores.

**Opción elegida: B.** La firma cubre el contenido de la petición con el esquema de cada proveedor; la hora firmada queda fuera (ver "No resuelve").

### Decision

Usaremos una **firma por petición verificada en la pieza de cada proveedor** (TAC-06), con el esquema y la clave de ese proveedor (TAC-07), antes de que la petición llegue al saldo. Conservaremos la clave fija y la lista de direcciones de la línea base como primera barrera. Cada rechazo de firma dejará un evento que un proceso aparte agrupa por origen en ventanas de 1 minuto y alerta al equipo de seguridad (TAC-08). **No integraremos a un proveedor que no firme el contenido de sus peticiones.**

### Por qué se descartan las demás opciones

| Opción | Frente a la medida | Número |
|--------|--------------------|--------|
| **A · Clave fija y direcciones permitidas** | **Incumple** el 100 % de la batería | Es la línea base: rechaza 2 de 3 tipos (~67 %); pasa la copia legítima con el monto modificado porque la clave no depende del contenido |
| **C · Esquema único del operador** | **Incumple** la aceptación de las peticiones legítimas de los 5 proveedores | Cada proveedor trae su esquema y no acepta cambiarlo [S29]; con ese supuesto, quizá ningún proveedor la acepte. Un certificado vencido cortaría a un proveedor completo |

### Tácticas

| ID | Táctica (catálogo) | Métrica | Antes → después |
|----|--------------------|---------|-----------------|
| TAC-06 | Usar un intermediario | Partes que cambian con un proveedor nuevo | interfaz de saldo → 1 pieza nueva |
| TAC-07 | Verificar integridad del mensaje | Batería de falsificación rechazada | 67 % → 100 % |
| TAC-08 | Informar a los actores | Tiempo hasta la alerta de seguridad | sin alerta → ≤ 1 min |

### Hasta dónde llegan estas tácticas

- **Nuevo límite del sistema con las tácticas (usuarios concurrentes):** el servidor de aplicación pasa de ~45.800 a **~44.400 jugadores conectados**. La verificación suma ~0,045 ms de cómputo a cada operación de saldo [F36].
- **Cómo se calculó:**

```
costo de verificar              1 ÷ 22.220 verificaciones/s = 0,045 ms                [F36]
cómputo en el pico              3.000 /s × 0,045 ms = 135 ms/s = 0,135 núcleos = 0,8 % de 16 núcleos
uso del servidor de aplicación  (3.000 + 1.200 + 135) ÷ 16.000 = 0,27                 (antes 0,26)
petición-equivalente por jugador 0,249 × (1 + 0,045) + 0,1 = 0,360 /s
límite en jugadores             16.000 ÷ 0,360 = ~44.400                              (antes 16.000 ÷ 0,349 = ~45.800)
margen sobre el pico            44.400 ÷ 12.000 = 3,7
presupuesto de verificación     20 ms ÷ 0,05 ms = 400 veces
```

- **Siguiente componente que se satura:** la **base única** (~40.000 jugadores, `04` §3). Sigue siendo anterior al servidor de aplicación, que además se multiplica sin tocar ningún ADR de la operación de saldo; solo obliga a replicar la lista vigente de ADR-01 (`07` I6).
- **¿Se cumple la medida con la carga del pico?** **Depende de S42 (a)**: sí, si los 5 proveedores firman el contenido con su propio esquema. Con un proveedor que no firme el contenido no se cumpliría, y por eso no se integra (Decision).

| Parte de la medida | Valor con la decisión | Cumple |
|--------------------|-----------------------|--------|
| 100 % de la batería rechazada, 0 saldos modificados | 3 de 3 tipos (sin credencial, credencial de otro proveedor, monto modificado) | Depende de S42 (a) |
| ≤ 0,001 % de legítimas rechazadas | Firma válida de un proveedor legítimo no se rechaza; un reloj desfasado no interviene porque no se verifica la hora | Sí |
| Verificación ≤ 20 ms (p99) | +0,05 ms | Sí |
| Saldo p99 ≤ 250 ms | +0,05 ms (más ≤ 1 ms de ADR-03 y ~1 ms de ADR-02: suma completa en la fase 7) | Sí |
| Alerta ≤ 5 min | ≤ 1 min | Sí |

### Consequences

- **Gana:** la batería de falsificación pasa de ~67 % a 100 % con +0,05 ms de 20 ms; cada proveedor conserva su esquema; el tráfico alterado se detecta y se alerta en ≤ 1 min.
- **Paga:** +0,05 ms por operación; 0,8 % de CPU del servidor de aplicación; 5 claves que guardar y rotar, y cada rotación o cambio de esquema pasa por el laboratorio (B3) [F1] en **2 a 4 semanas** [S10]; hasta 240 alertas falsas al día **entre ADR-03 y ADR-04** (tope común de S27, `07` I5).
- **No resuelve:** (1) **la filtración del secreto de un proveedor**: quien lo tenga firma como ese proveedor; (2) **un proveedor que no firme el contenido** (S42 (a) falla): para él la copia con el monto modificado pasaría, por eso no se integra; (3) **no hay rechazo por petición vieja**, porque los esquemas documentados no firman la hora [F24][F26]; las copias exactas las cubre TAC-04 (ADR-03), no este ADR; (4) un certificado propio por proveedor (opción C) queda como variante, no incluida.
- **Condición de revisión (propuesta):** revisar si **(a)** un proveedor actual o nuevo no firma el contenido de sus peticiones, **(b)** las peticiones legítimas rechazadas por ADR-03 y ADR-04, sumadas, superan **0,001 %** (240 al día) [S27], o **(c)** la verificación supera **20 ms (p99)**, por ejemplo con un esquema más costoso que la firma de 2.048 bits.

**Dónde está este ADR en la entrega:** presentación HTML, sección "ADR" (fase 10), y este archivo.

---

## 6. Respuestas a lo "pendiente para el Bloque 3"

| ADR | Pendiente del Bloque 2 | Respuesta |
|-----|------------------------|-----------|
| ADR-01 | Cuánto tarda cargar una lista nueva (opción C) | ≤ 1 min para 1 M de registros de ~500 bytes [S42] |
| ADR-01 | Latencia y disponibilidad del proveedor (opción B) | No hace falta medirlas: el límite de 5 consultas/s ya descarta la opción [F37], con la reconsulta en uso 3,24 |
| ADR-01 | Si las 6 h de la reconsulta caben dentro del ritmo máximo | Sí: 350.000 ÷ 50 /s = ~1,9 h; base en 0,31 |
| ADR-02 | Latencia entre sitios y dentro del centro de datos | ~10 ms al alterno [S38] y ~1 ms local [S40] |
| ADR-02 | Costo frente al presupuesto | ~USD 24.000 al año, 1,3 % de COP 7.300 M [F41] |
| ADR-02 | Cómo se mantiene el alterno al día sin entrar al camino de cada jugada | Lee la libreta aparte (paso 7 de `02` §3.1); su atraso es S37 |
| ADR-03 | Costo de la escritura adicional en el pico | +3.000 inserciones/s y ≤ 1 ms [S41] |
| ADR-03 | Plan de almacenamiento de las anotaciones | ~300 GB, por día, con borrado por días completos a los 4 meses [F24] (la cuenta es 3.000 M × ~100 bytes, no ~2.900 M como en el Bloque 2) |
| ADR-03 | Si B y C se combinan | Sí: B para la garantía y la traducción de C como TAC-06 |
| ADR-04 | Si los 5 proveedores firman el contenido y la hora | Contenido: sí [F24][F26], con S42 (a). Hora: no |
| ADR-04 | Qué hacer con un proveedor que solo tenga clave fija | No integrarlo (Decision). Variante: tope de monto por abono y alerta en cada abono atípico (exigiría un supuesto nuevo) |
| ADR-04 | Si B y C se combinan | Queda como variante (`05` §3.1); con S29, quizá ningún proveedor lo acepte |

---

## 7. Enlace con el Bloque 2

Nota agregada al final de `bloque-2/07-adrs.md` al cerrar el punto de control. El texto de los ADR del Bloque 2 no se edita:

> **15 de noviembre de 2026:** los cuatro ADR pasan de Propuesto a **Aceptado**. La opción elegida, la Decision, el "hasta dónde" y las Consequences están en `bloque-3/06-adrs.md`. Context, opciones y estado original de este archivo no cambian.

---

## 8. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| Context de cada ADR idéntico a su escenario | ✔ Las filas de cada tabla se extrajeron de `bloque-2/07-adrs.md` por rango de líneas y cada una se encontró textualmente en `bloque-2/06-escenarios.md` |
| Medida de la respuesta copiada literal | ✔ Es la última fila de cada Context |
| Estado Aceptado, opción elegida y Decision en voz activa | ✔ ADR-01 a ADR-04 ("Usaremos…") |
| Cada opción descartada, justificada frente a la medida y con número | ✔ ADR-01 A y B; ADR-02 A y B; ADR-03 A y C; ADR-04 A y C |
| Las opciones elegidas son las aprobadas en `05` §9 | ✔ ADR-02 C; ADR-03 B con la traducción de C; ADR-04 B; ADR-01 C |
| Solo nombres de táctica del catálogo y ningún mecanismo con marcas | ✔ Los productos solo aparecen en las fuentes de `05` §7.1 |
| "Hasta dónde" calculado con unidades y siguiente cuello | ✔ Cuentas en cada ADR; siguiente: base única en los cuatro |
| Ningún margen menor que 1 marcado como cumplido | ✔ Márgenes: 3,1; 3,3 (2,5 en la ráfaga); 3,3 y 1,8 en datos; 3,7 |
| Cuentas recalculadas | ✔ 50 × 21.600 = 1.080.000 y ÷ 350.000 = 3,1; 350.000 ÷ 50 = 7.000 s = 1,94 h; 10.000 ÷ 3.000 = 3,3 y ÷ 4.000 = 2,5; 90.000 ÷ 7.000 = 12,9 s; 0,001 × 43.200 = 43,2 min; 2,5 ÷ 1,4 = 1,8 y 24 M × 1,8 = 43 M; 16.000 ÷ (0,249 × 1,045 + 0,1) = 44.419; 44.400 ÷ 12.000 = 3,7; 20 ÷ 0,05 = 400 |
| Las cifras de las fases 3, 4 y 5 no cambiaron | ✔ Se reutilizan sin modificar; no hay fuentes ni supuestos nuevos |
| Cada ADR tiene condición de revisión con umbral numérico | ✔ Marcadas como **propuesta** del punto de control |
| Los ADR del Bloque 2 no se reescribieron | ✔ `bloque-2/07-adrs.md` no se tocó; la nota se agregó al cerrar el punto de control (§7) |

**Observaciones para la fase 7:**
- **ADR-02, ADR-03 y ADR-04 suman al mismo paso de escritura:** ~1 ms (TAC-01), ≤ 1 ms (TAC-04) y 0,05 ms (TAC-07): ~2 ms de los 250 ms. La suma completa contra la medida de tiempo es la tarea 7.2.
- **TAC-03 genera hasta ~90.000 reenvíos** que TAC-04 absorbe sin duplicar (estímulo de ESC-03). La vuelta a p99 ≤ 250 ms en ~13 s es una estimación gruesa que la fase 7 puede afinar.
- **Resumen de límites de la sección 7 de la plantilla:** el límite con todas las tácticas es ~10.000/s (margen 3,3) y el siguiente componente que se satura es la base única.
- **Decisión pendiente que condiciona la extensión X1 (fase 8):** a 15.000/s la base única se satura; es el caso en que "más servidores" no basta.

---

## 9. Punto de control (cerrado el 15 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las tres recomendaciones.

| # | Decisión | Aprobado | Variantes registradas |
|---|----------|----------|-----------------------|
| 1 | Umbrales de las condiciones de revisión | **Los propuestos en §2 a §5** | ADR-02 (b) con 10 cambios al mes; ADR-01 (a) con 900.000 cuentas |
| 2 | ADR-04: proveedor que no firme el contenido | **No integrarlo** | Integrarlo con tope de monto por abono y alerta en cada abono atípico (exigiría un supuesto nuevo) |
| 3 | Redacción de cada Decision | **La de §2 a §5** | — |

La fase 7 (interacción y límites) puede arrancar.
