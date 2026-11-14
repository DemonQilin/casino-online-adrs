# Fase 5 — Tácticas y efecto

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 5 (bloques de táctica de cada ADR: T1, T2…)
**Criterios de la rúbrica:** Tácticas como mecanismos · Efecto estimado · Costo cuantificado. Nivel "Logrado": táctica del catálogo con un mecanismo concreto (qué componente, qué dato, en qué paso); cada táctica mueve una métrica de un valor a otro, con la estimación explicada y su tipo; costo en número y atributo que lo paga.

**Insumos:**
- `bloque-2/07-adrs.md`: las tres opciones de cada ADR y lo que quedó en "Depende".
- `02-linea-base.md` §3 y §4: los pasos de la operación de saldo y del retiro, y el piso regulatorio que no cuenta como táctica.
- `03-perfil-de-carga.md`: 3.000 operaciones de saldo/s (4.000 en ráfaga), 24 M de operaciones al día, ~3.000 M de identificadores por conservar, 3.000 retiros al día, presupuestos de tiempo.
- `04-cuello-de-botella.md`: base única con límite de ~10.000 transacciones/s (uso 0,30), servidor de aplicación de 16.000 peticiones/s, proveedor de listas a 5 consultas/s (uso 3,24).
- `docs/class-material/PedalVa Tácticas.html`: catálogo de tácticas (nombres, familias y responsabilidades).
- `docs/class-material/Tácticas y sizing · Bloque 3.html`: campos del bloque de táctica de la plantilla.

**Qué fija esta fase:**
1. Qué tácticas del catálogo hay detrás de cada opción del Bloque 2.
2. Los números que cierran cada "Depende" del Bloque 2.
3. Una ficha por táctica, con los campos de la plantilla en su orden.
4. Una opción recomendada por ADR, con variantes. **La elección es decisión del equipo** (punto de control, sección 9: aprobadas las cuatro recomendaciones).

Las Decision, el "hasta dónde" y las Consequences de cada ADR se escriben en la fase 6, sobre la opción elegida. Esta fase agrega cuatro fuentes (F39 a F42) y siete supuestos (S37 a S43), registrados en la sección 7.

---

## 1. Opciones del Bloque 2 y catálogo (tarea 5.1)

### 1.1 Vocabulario

La equivalencia propuesta en `00` §5 se valida sin cambios: Disponibilidad → Fiabilidad, Rendimiento → Eficiencia de desempeño, Seguridad → Seguridad, Modificabilidad y Testabilidad → Mantenibilidad, Despliegue → Portabilidad.

**El catálogo no tiene familia de Compatibilidad.** Lo que el Bloque 2 anotó como "Gestionar interfaces" en ADR-03 y ADR-04 (traducir el protocolo de cada proveedor a un modelo común) es, en el catálogo, **Usar un intermediario** ("un componente traduce entre quien llama y quien es llamado, para que no se conozcan"), de la familia Reducir el acoplamiento de Modificabilidad. Se usa con ese nombre. El atributo que mejora en el escenario sigue siendo Compatibilidad; la familia del catálogo solo dice dónde está la táctica.

### 1.2 Qué táctica hay detrás de cada opción

| ADR | Opción del Bloque 2 | Tácticas del catálogo que la forman |
|-----|---------------------|-------------------------------------|
| ADR-02 | A · Dos sitios antes de confirmar | Redundancia activa (con la réplica en el otro sitio), Transacciones |
| | B · Grabación local y copia posterior | Redundancia pasiva. **Es la línea base** (piso de E1) |
| | C · Libreta confirmada por mayoría | Redundancia activa (tres copias, confirma la mayoría), Latido, Reconfiguración |
| ADR-03 | A · Buscar en el historial | Ninguna nueva: usa la Auditoría del piso. **Es la línea base** |
| | B · Registro junto con el saldo | Transacciones (el registro entra al paso indivisible), Validar entradas |
| | C · Traductor por proveedor que filtra repetidos | Usar un intermediario, más una memoria de 24 h (Múltiples copias de datos) |
| ADR-04 | A · Clave fija y direcciones permitidas | Identificar y Autenticar actores, Limitar el acceso. **Es la línea base** |
| | B · Firma por petición, una pieza por proveedor | Verificar integridad del mensaje, Usar un intermediario |
| | C · Esquema único del operador | Autenticar actores (certificado propio), Verificar integridad del mensaje con un formato único |
| ADR-01 | A · Retener hasta terminar la reconsulta | Ninguna nueva. **Es la línea base** |
| | B · Consultar al proveedor en cada retiro | Ninguna de rendimiento: agrega una dependencia externa al camino del retiro |
| | C · Lista vigente propia, reconsulta aparte | Múltiples copias de datos (la lista, cerca de la evaluación), Limitar la tasa de eventos (en la reconsulta) |

Las tácticas que llegan por el piso regulatorio (`02` §4) no se cuentan como mejora: Redundancia pasiva, Transacciones (en su forma de la línea base), Auditoría, Identificar y Autenticar actores, Limitar el acceso y Cifrar datos.

### 1.3 Resumen de los "Depende" (tarea 5.3)

| ADR | Opción | Lo que faltaba medir en el Bloque 2 | Número que lo cierra | Veredicto |
|-----|--------|-------------------------------------|----------------------|-----------|
| ADR-02 | A | Latencia hasta el otro sitio | Ida y vuelta de ~5–10 ms: cabe en 250 ms. Pero el enlace entre sitios, a 99,9 %, puede detener el juego ~43 min al mes, todo el margen de S13 (§2.1) | **No**, con un solo enlace |
| ADR-02 | C | Latencia dentro del centro de datos | +~1 ms por operación; la referencia de la fase 4 ya mide con la espera de una réplica: p99 ≤ 40 ms saturada [F34] (§2.1) | **Sí** |
| ADR-03 | B | Costo de la escritura adicional en el pico | 3.000 inserciones/s más, ≤ 1 ms; el límite de la base no cambia (~10.000/s). ~300 GB de registro (§2.2) | **Sí** |
| ADR-04 | B | ¿Los proveedores firman el contenido? ¿Cabe en 20 ms? | Las dos integraciones documentadas firman el contenido [F24][F26]. Verificar cuesta ~0,05 ms de 20 ms (§3) | **Sí**, con S42 |
| ADR-01 | B | Latencia y disponibilidad del proveedor | El proveedor admite 5 consultas/s y la reconsulta ya le pide 16,2/s: cada retiro haría fila detrás de ella o sería rechazado. Con fallo cerrado, su caída detiene todos los retiros (§4) | **No** |

---

## 2. ADR-02 y ADR-03: el paso de escritura del saldo

Se deciden juntos porque los dos cambian la misma transacción (`bloque-2/07-adrs.md`, relación entre ADR-02 y ADR-03). Con las dos opciones recomendadas, la transacción de cada operación de saldo queda así:

```
en una transacción:  buscar e insertar el identificador en el registro (ADR-03)
                     comprobar que el saldo alcanza · cambiar el saldo · anotar en el historial
confirmar cuando:    2 de los 3 servidores de base de datos la tienen grabada (ADR-02)
```

### 2.1 ADR-02 · Permanencia de las operaciones confirmadas

**Cierre de la opción A (dos sitios antes de confirmar).**

```
distancia por fibra al alterno       ≈ 400 km                                  [S38 · supuesto]
ida y vuelta por la fibra            = 2 × 400 km × 5 µs/km = 4 ms             [F39 · referencia]
con equipos de red                   ≈ 5 a 10 ms; se usa 10 ms                 [S38 · supuesto]
efecto en la respuesta               +10 ms de 250 ms                          → cabe
operaciones esperando el otro sitio  3.000 /s × 0,010 s = 30 a la vez          → cabe
```

La latencia no descarta la opción A. La descarta la disponibilidad: cada operación depende del enlace entre sitios.

```
enlace entre sitios a 99,9 %         0,001 × 30 días × 1.440 min = 43,2 min al mes sin enlace   [S39 · supuesto]
meta de juego y saldo (S13)          99,9 %  → 43,2 min al mes de caída permitida en total
```

Con un solo enlace, la opción A gasta en el enlace todo el margen de caída del juego. En esos minutos hay que detener el juego o confirmar sin la segunda copia, que es volver a la línea base. **Veredicto: no cumple** con un enlace; con dos enlaces independientes sería otra decisión, con su costo (variante en §2.3).

**Cierre de la opción C (mayoría de tres en el mismo centro de datos).** La prueba de la fase 4 ya confirma cada transacción solo después de que una réplica la recibe [F34]: con esa espera da p99 de 29 a 39 ms con el servidor saturado. Dentro del mismo centro de datos la espera suma ~1 ms [S40]. El límite de ~10.000/s no cambia: S36 había ignorado a favor de la línea base que no esperaba réplica, y ese efecto desaparece ahora sin mover el número. **Veredicto: cumple.**

#### TAC-01 · Redundancia activa

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Redundancia activa (Disponibilidad → Recuperarse de fallas) |
| **Mecanismo** | El sitio principal pasa de 1 a **3 servidores de base de datos**, del tamaño de S32. Cada transacción de saldo (paso 5 de `02` §3.1) se anota en orden en una libreta de solo agregar que los tres reciben. El servidor que atiende responde al proveedor (paso 6) cuando **al menos 2 de los 3** tienen la operación grabada en disco. El saldo de cada jugador se calcula desde esa libreta. La copia al sistema alterno (paso 7) sigue igual, leyendo la libreta |
| **Responsabilidad** | Previene (la pérdida de lo confirmado) |
| **Métrica que mueve** | Operaciones de saldo confirmadas que se pierden cuando falla el servidor de base de datos |
| **Valor en la línea base** | **~3.000** (hasta 4.000 en la ráfaga; hasta 15.000 si la copia se atrasa 5 s) |
| **Valor con la táctica** | **0**, mientras sobreviva 1 de los otros 2 servidores |
| **Cómo se estimó** | Línea base: lo confirmado y todavía no copiado al alterno = retraso de la copia × tasa = 1 s × 3.000 operaciones/s = 3.000 (4.000/s × 1 s en la ráfaga; 5 s × 3.000/s = 15.000). Con la táctica: ninguna operación se confirma sin estar grabada en un segundo servidor; si cae uno, la otra copia la tiene |
| **Tipo de la estimación** | Supuesto [S37] sobre el retraso de la copia; la cuenta con la tasa es de la fase 3 [S6] |
| **Costo cuantificado** | **Dinero:** 2 servidores más ≈ 2 × USD 1.002 al mes = **~USD 24.000 al año** [F41], ≈ COP 96 M al año a COP 4.000 por dólar [S9], **1,3 %** del presupuesto de tecnología (COP 7.300 M). **Latencia:** **+~1 ms** por operación [S40]. **Red interna:** 3.000/s × 350 bytes × 2 copias ≈ 2,1 MB/s ≈ 17 Mbit/s [S34][S41] |
| **Atributo que paga** | Eficiencia de desempeño (utilización de recursos) y Mantenibilidad (tres servidores que operar y recertificar en lugar de uno) |

#### TAC-02 · Latido (heartbeat)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Latido (Disponibilidad → Detectar fallas) |
| **Mecanismo** | El servidor de base de datos que atiende envía una señal a los otros dos varias veces por segundo. Si un servidor deja de recibirla durante un intervalo aleatorio de **150 a 300 ms**, declara caído al que atendía |
| **Responsabilidad** | Detecta |
| **Métrica que mueve** | Tiempo desde la caída del servidor de saldo hasta que el sistema la detecta |
| **Valor en la línea base** | **10 a 30 min**: en la línea base nada detecta la caída; la detectan y deciden personas (`02` §3.1 no tiene detección) |
| **Valor con la táctica** | **≤ 0,3 s** |
| **Cómo se estimó** | El intervalo de 150–300 ms es el que recomienda la referencia del algoritmo de consenso por mayoría [F40]. La línea base es S40 |
| **Tipo de la estimación** | Referencia [F40] para la táctica; supuesto [S40] para la línea base |
| **Costo cuantificado** | **Caídas falsas:** una pausa de red de más de 300 ms dispara un cambio de servidor innecesario, con el costo de TAC-03 (~30 s sin saldo). **Tráfico:** despreciable (mensajes de pocos bytes) |
| **Atributo que paga** | Fiabilidad (disponibilidad, por las caídas falsas) |

#### TAC-03 · Reconfiguración

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Reconfiguración (Disponibilidad → Recuperarse de fallas) |
| **Mecanismo** | Cuando TAC-02 declara caído al servidor que atendía, los dos que quedan eligen al que tiene la libreta completa, y ese pasa a atender. El servidor de aplicación (paso 4) se reconecta al nuevo y retoma las operaciones. Las que estaban en curso sin confirmar no tienen efecto; el proveedor las revierte o las reenvía [F24] |
| **Responsabilidad** | Recupera |
| **Métrica que mueve** | Tiempo con el juego y el saldo detenidos tras la caída del servidor de base de datos |
| **Valor en la línea base** | **10 a 30 min**: pasar a mano al sistema alterno en el otro sitio [S40] |
| **Valor con la táctica** | **≤ 30 s** |
| **Cómo se estimó** | Detección ≤ 0,3 s (TAC-02) + elección en menos de 1 s [F40] + reconexión del servidor de aplicación y vuelta a atender ≤ 30 s [S40]. La medida de ESC-02 es ≤ 15 min: margen de 15 min ÷ 0,5 min = 30 veces |
| **Tipo de la estimación** | Supuesto [S40] apoyado en una referencia [F40] |
| **Costo cuantificado** | **Operaciones afectadas en cada cambio:** 3.000/s × 30 s = **hasta ~90.000** operaciones sin respuesta, que los proveedores revierten (descuentos) o reenvían (abonos) [F24]. **Capacidad:** mientras no se reemplace el servidor caído, quedan 2 de 3 y una segunda falla detiene el juego |
| **Atributo que paga** | Mantenibilidad (procedimiento de cambio que hay que probar y recertificar) y, durante 30 s, Eficiencia de desempeño |

**Interacción que se anota para la fase 7:** los ~90.000 reenvíos de TAC-03 son exactamente el estímulo de ESC-03. Sin la decisión de ADR-03, un cambio de servidor duplicaría abonos.

### 2.2 ADR-03 · Peticiones repetidas

**Cierre de la opción B (registro junto con el saldo).**

```
escrituras adicionales            1 inserción por operación × 3.000 /s = 3.000 inserciones/s     [S6]
tiempo adicional                  ≤ 1 ms por operación (una entrada más en un índice ya en uso)  [S41]
volumen del registro              3.000 M identificadores × ~100 bytes = ~300 GB                [S41]
datos de la base                  1,1 TB + 0,3 TB = 1,4 TB  (2,8 veces los 500 GB de F34)
```

**El límite de la base no cambia (~10.000/s).** Por transacción, la línea base con la opción B hace 1 cambio de saldo, 1 lectura y 2 inserciones; la transacción de la referencia hace 3 cambios de saldo, 1 lectura y 1 inserción [F35]: sigue siendo menos trabajo que la prueba. Los datos crecen de 2,2 a 2,8 veces los de la prueba, todavía por debajo del escalón de 5 veces que S36 ya descontó completo (`04` §2.1). Así se cumple la observación de la fase 4: la cuenta del límite se rehízo con el mismo método y da lo mismo. **Veredicto: cumple.**

**Cierre de la opción C (traductor que filtra repetidos).** La memoria de 24 h no ve las repeticiones tardías: un proveedor puede reenviar una reversión días después, dentro de los 4 meses de conciliación [F24], y una caída del traductor borra la memoria. **Veredicto: parcial**, como en el Bloque 2. Su parte de traducción sí sirve y se toma como TAC-06; su memoria de repetidos se descarta.

#### TAC-04 · Transacciones (registro de identificadores en el paso indivisible)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Transacciones (Disponibilidad → Prevenir fallas). En la línea base la táctica ya existe por el piso de C6 (`02` §4); aquí se **refuerza**: entra un registro más al paso indivisible |
| **Mecanismo** | Un registro de identificadores procesados en la base única, con clave única **proveedor + jugador + identificador** [F26]. En el paso 5 de `02` §3.1, la misma transacción que cambia el saldo inserta la clave, un resumen de los datos de la petición (tipo y monto) y la respuesta dada (resultado y saldo). Si la clave ya existe, la transacción no toca el saldo y devuelve la respuesta guardada. El registro se guarda por día y se borra por días completos a los 4 meses [F24]. Reemplaza la búsqueda del paso 4c |
| **Responsabilidad** | Previene (el doble abono) |
| **Métrica que mueve** | Abonos aplicados dos veces cuando dos copias de la misma petición se procesan a la vez |
| **Valor en la línea base** | **1 de más por cada par de copias que se solapan**: las dos pasan la búsqueda antes de que alguna quede grabada (`02` §1). Ocurre cuando la primera copia tarda más que los ~3 s del proveedor: con la base saturada (`04` §6.1) o tras un cambio de servidor (TAC-03, hasta ~90.000 reenvíos) |
| **Valor con la táctica** | **0**: la segunda inserción de la misma clave falla dentro de su transacción y no cambia el saldo |
| **Cómo se estimó** | Razonamiento sobre la unicidad: dos transacciones no pueden insertar la misma clave única; la que llega segunda ve la primera y devuelve su respuesta. No depende de la carga ni del momento en que llegan las copias |
| **Tipo de la estimación** | Referencia para la regla de unicidad del identificador [F26]; el costo es supuesto [S41] |
| **Costo cuantificado** | **Disco:** ~300 GB [S41]. **Escrituras:** +3.000 inserciones/s. **Latencia:** ≤ 1 ms por operación [S41]. **Límite de la base:** sin cambio, ~10.000/s |
| **Atributo que paga** | Eficiencia de desempeño (utilización de recursos) |

#### TAC-05 · Validar entradas (misma clave, datos distintos)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Validar entradas (Seguridad → Resistir ataques) |
| **Mecanismo** | Cuando la clave ya existe en el registro de TAC-04, la transacción compara el resumen guardado (tipo y monto) con el de la petición nueva. Iguales: devuelve la respuesta guardada. Distintos: **rechaza la petición** sin tocar el saldo y deja un evento de alerta en el mismo canal de TAC-08 |
| **Responsabilidad** | Detecta y resiste |
| **Métrica que mueve** | Porcentaje de peticiones con la misma clave y datos distintos que se rechazan, y tiempo hasta la alerta |
| **Valor en la línea base** | **0 %**: la búsqueda del paso 4c encuentra el identificador y devuelve la respuesta original sin comparar los datos. Sin alerta |
| **Valor con la táctica** | **100 %** rechazadas; alerta en **≤ 1 min** (ventana de agrupación de TAC-08), frente a ≤ 5 min de la medida [S25] |
| **Cómo se estimó** | La comparación es exacta sobre un resumen de 32 bytes: no hay casos grises. La clave incluye al jugador, así que dos jugadores con el mismo identificador no se confunden [F26] |
| **Tipo de la estimación** | Razonamiento sobre el mecanismo; ventana de alerta de TAC-08 [S43] |
| **Costo cuantificado** | **Disco:** 32 bytes por registro × 3.000 M = ~96 GB, ya contados en los ~300 GB de TAC-04. **Cómputo:** un resumen por petición, microsegundos. **Riesgo:** si un proveedor reutiliza un identificador para otra operación del mismo jugador, una petición legítima se rechaza; cuenta dentro del ≤ 0,001 % de S27 |
| **Atributo que paga** | Compatibilidad (un proveedor que no respete la unicidad por jugador ve rechazos) |

#### TAC-06 · Usar un intermediario (una pieza de traducción por proveedor)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Usar un intermediario (Modificabilidad → Reducir el acoplamiento). Hace lo que el Bloque 2 llamó "Gestionar interfaces" |
| **Mecanismo** | En el servidor de aplicación, el paso 4b de `02` §3.1 se separa en **5 piezas, una por proveedor**. Cada pieza recibe el protocolo de su proveedor y entrega a la interfaz de saldo una petición común: proveedor, jugador, identificador, tipo, monto y resultado de la verificación de origen (TAC-07). Las piezas no guardan memoria de repetidos: eso lo hace TAC-04. Se ejecutan dentro del mismo proceso, sin salto de red. **La comparten ADR-03 y ADR-04** |
| **Responsabilidad** | Desacopla |
| **Métrica que mueve** | Partes del sistema que cambian al integrar un proveedor nuevo |
| **Valor en la línea base** | La interfaz de saldo completa: el código de cada protocolo está mezclado con el del saldo (`02` §6) |
| **Valor con la táctica** | **1 pieza nueva y 0 cambios** en la interfaz de saldo, para cada uno de los 1 o 2 proveedores nuevos al año [S11] |
| **Cómo se estimó** | Por construcción: la interfaz de saldo solo conoce la petición común |
| **Tipo de la estimación** | Razonamiento sobre el mecanismo; frecuencia de proveedores nuevos [S11 · supuesto] |
| **Costo cuantificado** | **Mantenimiento:** 5 piezas que mantener y recertificar; cada cambio en una pieza pasa por el laboratorio, **2 a 4 semanas** [S10]. **Latencia:** una llamada dentro del proceso, < 0,1 ms |
| **Atributo que paga** | Mantenibilidad (más piezas, cada una con su validación) |

### 2.3 Propuesta para ADR-02 y ADR-03

| ADR | Recomendación | Tácticas | Por qué |
|-----|---------------|----------|---------|
| ADR-02 | **Opción C** | TAC-01, TAC-02, TAC-03 | Lleva la pérdida de ~3.000 a 0 operaciones con +1 ms y ~1,3 % del presupuesto de tecnología. Además baja el restablecimiento de 10–30 min a ≤ 30 s. A queda descartada por la disponibilidad del enlace, B es la línea base |
| ADR-03 | **Opción B, con la traducción de C** (combinación que el Bloque 2 dejó por evaluar) | TAC-04, TAC-05, TAC-06 | B da la garantía de 0 duplicados sin mover el límite de la base. De C se toma solo la traducción por proveedor, que sirve también a ADR-04; su memoria de 24 h se descarta porque no ve repeticiones tardías |

**Variantes:**

| Variante | Qué cambia | A favor | En contra |
|----------|-----------|---------|-----------|
| ADR-02 · **A sobre C** | Además de la mayoría local, esperar al alterno antes de confirmar, con dos enlaces independientes entre sitios | Cubre también la pérdida del sitio completo (que C no cubre: quedan ~3.000 operaciones en riesgo, S37) | +10 ms por operación y un segundo enlace; el juego sigue dependiendo de la red entre ciudades |
| ADR-02 · **A sola** | Un servidor en cada sitio, confirmar en los dos | Un servidor menos que C | Con un enlace, gasta los 43 min mensuales de S13; sin servidor local de reemplazo, el restablecimiento sigue siendo de minutos |
| ADR-03 · **B sola** | Sin piezas por proveedor; el protocolo sigue mezclado con el saldo | Menos piezas que mantener | Cada proveedor nuevo toca la interfaz de saldo, y ADR-04 B necesita igual las piezas |

---

## 3. ADR-04 · Verificación del origen de las peticiones

**Cierre de la opción B (firma por petición).** Faltaban dos cosas:

1. **¿Los proveedores firman el contenido?** Las dos integraciones de billetera documentadas en el Bloque 2 lo hacen. Una firma el cuerpo de cada petición con una clave privada y una firma de 2.048 bits; el operador la verifica con la clave pública del proveedor [F24]. La otra agrega a cada petición un resumen calculado sobre los parámetros (entre ellos el monto) más un secreto compartido [F26]. **Ninguna de las dos incluye la hora** en lo firmado. Con eso se fija S42: los 5 proveedores firman el contenido con su propio esquema, sin garantía de que firmen la hora.
2. **¿Cabe en 20 ms?** Verificar una firma de 2.048 bits cuesta 1 ÷ 22.220 por segundo ≈ **0,045 ms** en un núcleo [F36]. El resumen con secreto compartido cuesta menos. Se usa 0,045 ms como cota de los cinco esquemas.

**Veredicto: cumple**, con S42.

**Cierre de la opción C (esquema único del operador).** No se mide: S29 dice que los proveedores no aceptan cambiar su esquema. **Veredicto: no cumple**, como en el Bloque 2.

**Copia exacta de una petición legítima.** La firma de una copia exacta es válida, y como los esquemas no firman la hora, no se puede rechazar por vieja. No hace falta: tiene el mismo proveedor, jugador e identificador que la original, así que TAC-04 devuelve la respuesta guardada sin mover saldo. La batería de ESC-05 no la incluye; se anota porque es la táctica de ADR-03 la que la cubre.

#### TAC-07 · Verificar integridad del mensaje

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Verificar integridad del mensaje (Seguridad → Detectar ataques) |
| **Mecanismo** | En la pieza de su proveedor (TAC-06), antes de que la petición llegue al saldo (paso 4a de `02` §3.1), se verifica la firma del contenido con el esquema y la clave **de ese proveedor**: clave pública para quien firma con clave privada, secreto compartido para quien firma con resumen. Si la firma no corresponde, la petición se rechaza sin tocar el saldo y se registra con su origen. La clave fija y la lista de direcciones de la línea base se conservan como primera barrera |
| **Responsabilidad** | Detecta y resiste |
| **Métrica que mueve** | Porcentaje de la batería de falsificación de ESC-05 que se rechaza |
| **Valor en la línea base** | **2 de 3 tipos (~67 %)**: se rechaza la petición sin credencial y la que trae la credencial de otro proveedor; **pasa la copia legítima con el monto modificado**, porque la clave fija no depende del contenido |
| **Valor con la táctica** | **3 de 3 tipos (100 %)**: cambiar el monto invalida la firma; sin la clave del proveedor no se puede firmar; la firma de otro proveedor no se verifica con la clave de este |
| **Cómo se estimó** | Recorrido de los tres tipos de la batería contra el mecanismo. Tiempo: 1 ÷ 22.220 verificaciones/s = 0,045 ms por petición [F36] |
| **Tipo de la estimación** | Referencia [F24][F26][F36]; que los 5 proveedores firmen es supuesto [S42] |
| **Costo cuantificado** | **Latencia:** +0,05 ms de los 20 ms de S26. **Cómputo:** 3.000/s × 0,045 ms = 135 ms por segundo = 0,135 núcleos, **0,8 %** del servidor de aplicación (16 núcleos). **Operación:** 5 claves que guardar y rotar; cada rotación o cambio de esquema es un cambio en la seguridad de las comunicaciones y pasa por el laboratorio (B3) [F1], **2 a 4 semanas** [S10] |
| **Atributo que paga** | Mantenibilidad (claves y validaciones) y, en mínima parte, Eficiencia de desempeño |

**Táctica que no se usa: Detectar retraso del mensaje.** Rechazar peticiones con hora vieja exige que el proveedor firme la hora, y S42 no lo garantiza. Además, un reloj desfasado rechazaría peticiones legítimas, y la medida solo tolera ≤ 0,001 % [S27]. Las copias exactas ya las neutraliza TAC-04.

#### TAC-08 · Informar a los actores

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Informar a los actores (Seguridad → Reaccionar) |
| **Mecanismo** | Cada rechazo de TAC-07 (firma inválida) y de TAC-05 (misma clave, datos distintos) deja un evento con el origen de red, el proveedor que dice ser y el motivo. Un proceso aparte agrupa los eventos por origen en ventanas de **1 minuto** [S43]: el primer evento de un origen abre un incidente y alerta al equipo de seguridad; los siguientes del mismo origen se suman a ese incidente |
| **Responsabilidad** | Hace observable y reacciona |
| **Métrica que mueve** | Tiempo desde la primera petición falsificada hasta la alerta al equipo de seguridad |
| **Valor en la línea base** | **Sin alerta**: la línea base rechaza lo que no trae la clave, pero nadie se entera hasta revisar el registro |
| **Valor con la táctica** | **≤ 1 min**, frente a ≤ 5 min de la medida [S28] y de ESC-03 [S25] |
| **Cómo se estimó** | El incidente se abre con el primer evento; la ventana de agrupación acota el resto a 1 min [S43] |
| **Tipo de la estimación** | Supuesto [S43] |
| **Costo cuantificado** | **Alertas falsas:** hasta **240 al día** si las peticiones legítimas rechazadas llegan al tope de S27 (24 M × 0,001 %). **Operación:** un proceso más que vigilar; si falla, los rechazos siguen pero sin alerta |
| **Atributo que paga** | Mantenibilidad (proceso y guardia del equipo de seguridad) |

### 3.1 Propuesta para ADR-04

**Recomendación: opción B**, con TAC-06 (compartida con ADR-03), TAC-07 y TAC-08. Lleva la batería del 67 % al 100 % con +0,05 ms de 20 ms, respeta el esquema de cada proveedor y alerta en ≤ 1 min.

| Variante | Qué cambia | A favor | En contra |
|----------|-----------|---------|-----------|
| **B + C donde se pueda** | Además, los proveedores que acepten conectarse con un certificado propio lo hacen en una entrada exclusiva | El tráfico no autorizado no llega a la aplicación para esos proveedores | Dos esquemas que mantener; un certificado vencido corta a un proveedor completo. Con S29, quizá ningún proveedor lo acepte |
| **B con hora firmada** | Exigir la hora firmada y rechazar peticiones de más de 5 min | Rechaza copias viejas antes del saldo | Contradice S29 y S42: los esquemas documentados no firman la hora [F24][F26] |

**Riesgo para la sustentación:** si uno de los 5 proveedores usara solo una clave fija (S42 falla), para ese proveedor la copia con el monto modificado seguiría pasando. ADR-04 tendría que decir qué se hace con él: no integrarlo, o aceptarlo con un tope de monto por abono y alerta en cada abono atípico. Se resuelve en las Consequences de la fase 6.

---

## 4. ADR-01 · Retiros y listas restrictivas

**Cierre de la opción B (consultar al proveedor en cada retiro).** La fase 4 dejó el número: el proveedor de listas admite **5 consultas/s** [F37] y la reconsulta ya le pide 16,2/s (uso 3,24). Los ~3.000 retiros al día son 0,035 consultas/s, poco, pero durante la reconsulta llegan a un proveedor que ya rechaza peticiones: cada retiro espera detrás de la reconsulta o es rechazado y reintentado. El p95 ≤ 1 s depende de quién gane esa fila, no del operador. Además, con fallo cerrado, una caída del proveedor detiene todos los retiros. **Veredicto: no cumple.**

**Cierre de la opción C (lista vigente propia).** El Bloque 2 pedía saber cuánto tarda cargar la lista y si la reconsulta cabe en 6 h sin afectar el juego. Con la lista en poder del operador, **la reconsulta deja de consultar al proveedor**: compara cada cuenta con la lista propia. El proveedor de listas solo atiende los registros nuevos.

```
uso del proveedor de listas   línea base:  16,2 consultas/s ÷ 5 /s = 3,24
                              opción C:    ~1.000 registros al día ÷ 86.400 s = 0,012 /s ÷ 5 /s = 0,002
```

**Veredicto: cumple.** Las cuentas están en las fichas.

#### TAC-09 · Múltiples copias de datos (la lista vigente junto a la evaluación)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Múltiples copias de datos (Rendimiento → Gestionar recursos) |
| **Mecanismo** | Cuando llega la actualización del proveedor (paso 4 de `02` §3.2), la evaluación de riesgo carga la lista completa en la memoria del servidor de aplicación, indexada por número de documento, y guarda el archivo recibido para la auditoría. Solo cuando la carga termina, la lista nueva reemplaza a la anterior. Desde ese momento, cada retiro (paso 3) se evalúa contra la lista nueva, **haya o no terminado la reconsulta**. Mientras carga, los retiros esperan (fallo cerrado) |
| **Responsabilidad** | Gestiona recursos y previene (el pago a una persona listada) |
| **Métrica que mueve** | Tiempo de evaluación de un retiro mientras corre la reconsulta |
| **Valor en la línea base** | **Hasta ~19,4 h** (el retiro espera a que termine la reconsulta al ritmo del proveedor, `04` §2.7); **~2.430 retiros** retenidos por reconsulta |
| **Valor con la táctica** | **≤ 10 ms** por retiro. Solo esperan los retiros que llegan durante la carga: ≤ 1 min, **~2 retiros** (125 por hora ÷ 60) |
| **Cómo se estimó** | Evaluar es una búsqueda en un índice en memoria más la lectura del perfil de riesgo en la base única, del mismo orden que una operación de saldo (p99 ≤ 40 ms saturada [F34]); se usa ≤ 10 ms con la base al 30 % [S42]. Carga: 1 M de registros × 500 bytes = 500 MB, en ≤ 1 min [S42]. Retenidos: 3.000 retiros al día ÷ 24 h = 125 por hora × 1 min ÷ 60 min ≈ 2 |
| **Tipo de la estimación** | Supuesto [S42] apoyado en una referencia [F34] |
| **Costo cuantificado** | **Memoria:** ~500 MB de 128 GB (0,4 %) [S32][S42]. **Disco:** una copia del archivo por actualización, varias veces al mes. **Operación:** dos procesos que mantener coherentes (lista vigente y reconsulta); si el servidor de aplicación se reinicia, la lista se recarga y los retiros esperan ≤ 1 min |
| **Atributo que paga** | Mantenibilidad y Eficiencia de desempeño (utilización de memoria) |

#### TAC-10 · Limitar la tasa de eventos (reconsulta contra la lista propia)

| Campo | Contenido |
|-------|-----------|
| **Táctica (catálogo)** | Limitar la tasa de eventos (Rendimiento → Controlar la demanda) |
| **Mecanismo** | La reconsulta (paso 5 de `02` §3.2) recorre las 350.000 cuentas, compara el documento de cada una con la lista en memoria de TAC-09 y guarda la marca solo si cambió. Lo hace con un **ritmo máximo de 50 cuentas/s**, en lugar de "sin ritmo máximo" como en la línea base |
| **Responsabilidad** | Controla la demanda |
| **Métrica que mueve** | (1) Duración de la reconsulta. (2) Uso de la base única durante la reconsulta |
| **Valor en la línea base** | (1) **~19,4 h**, al ritmo del proveedor. (2) Sin ritmo máximo y sin el proveedor de por medio, la reconsulta leería tan rápido como responda la base: tomaría la capacidad libre, 10.000 − 3.000 = 7.000 lecturas/s, y llevaría el uso de **0,30 a ~1,0** |
| **Valor con la táctica** | (1) **~1,9 h**, frente a ≤ 6 h. (2) Uso de **0,30 a 0,31** |
| **Cómo se estimó** | 350.000 cuentas ÷ 50 /s = 7.000 s ÷ 3.600 s/h = 1,94 h. Carga: 50 lecturas/s + hasta 50 escrituras/s = 100 ÷ 10.000 = +0,01 de uso. El ritmo de 50/s es ~3 veces el mínimo de 16,2/s [S19] |
| **Tipo de la estimación** | Supuesto [S42] con el límite de la base de la fase 4 [F34][S36] |
| **Costo cuantificado** | **Tiempo:** la reconsulta dura ~1,9 h en lugar de los minutos que tardaría sin ritmo máximo. Los retiros no esperan por eso (TAC-09), pero una marca de riesgo que cambie llega a su cuenta hasta 1,9 h después de recibir la lista |
| **Atributo que paga** | Eficiencia de desempeño (comportamiento temporal de la reconsulta) |

### 4.1 Propuesta para ADR-01

**Recomendación: opción C**, con TAC-09 y TAC-10. Baja la evaluación del retiro durante la reconsulta de hasta ~19,4 h a ≤ 10 ms, la reconsulta de ~19,4 h a ~1,9 h, y el uso del proveedor de listas de 3,24 a 0,002. Es la única opción que resuelve el cuello que la fase 4 encontró en la otra dimensión.

| Variante | Qué cambia | A favor | En contra |
|----------|-----------|---------|-----------|
| **C con ritmo de 100/s** | Reconsulta a 100 cuentas/s | Termina en ~1 h | Duplica la carga sobre la base (+0,02 de uso). Diferencia pequeña; se deja a criterio del equipo |
| **C con la lista en la base única** | La lista vive en una tabla de la base y no en memoria | Sobrevive al reinicio del servidor de aplicación sin recargar | Cada evaluación y cada cuenta de la reconsulta suman lecturas a la base que es el cuello del saldo |

**Riesgo para la sustentación:** la opción C supone que el proveedor entrega la lista completa, no solo consultas por documento (así lo describe la línea base, `02` §3.2). La lista vinculante de Naciones Unidas tiene 877 registros [F42]; el millón de S42 es una cota para listas más amplias. Si el proveedor solo respondiera consultas, la opción C dejaría de ser posible con él.

---

## 5. Efecto conjunto sobre los límites de la fase 4

Adelanto para las fases 6 y 7, que calculan el "hasta dónde" sobre la opción elegida.

| Componente | Línea base | Con las tácticas recomendadas | Qué lo mueve |
|-----------|-----------|-------------------------------|--------------|
| Base única: transacciones de saldo | ~10.000/s, uso 0,30 | **~10.000/s, uso 0,31** | TAC-01 y TAC-04 no mueven el límite (§2.1 y §2.2); TAC-10 suma 0,01 |
| Servidor de aplicación | 16.000 peticiones/s, uso 0,26 | Uso **0,27** | TAC-07: +135 ms de cómputo por segundo; (4.200 + 135) ÷ 16.000 = 0,27 |
| Proveedor de listas | 5 consultas/s, uso **3,24** | Uso **0,002** | TAC-09 |
| Tiempo de la operación de saldo | p99 < 40 ms en la base | +~2 ms (1 de TAC-01, ≤ 1 de TAC-04, 0,05 de TAC-07) | La suma completa contra los 250 ms es tarea 7.2 |

**Lectura:** las tácticas de la operación de saldo compran garantías (pérdida cero, duplicados cero, origen verificado), no capacidad. El cuello de la fase 4 sigue en la base única con ~10.000/s. En cambio, en el retiro, la táctica de ADR-01 sí elimina el límite que ya estaba superado.

---

## 6. Resumen de tácticas

| ID | Táctica (catálogo) | ADR | Métrica | Línea base → con la táctica | Costo principal |
|----|--------------------|-----|---------|-----------------------------|-----------------|
| TAC-01 | Redundancia activa | ADR-02 | Operaciones confirmadas perdidas | ~3.000 → 0 | USD ~24.000/año; +1 ms |
| TAC-02 | Latido (heartbeat) | ADR-02 | Tiempo de detección de la caída | 10–30 min → ≤ 0,3 s | Cambios falsos ante pausas de red > 300 ms |
| TAC-03 | Reconfiguración | ADR-02 | Tiempo con el saldo detenido | 10–30 min → ≤ 30 s | ~90.000 operaciones revertidas o reenviadas por cambio |
| TAC-04 | Transacciones | ADR-03 | Abonos duplicados con copias simultáneas | 1 por par solapado → 0 | ~300 GB; +3.000 inserciones/s; ≤ 1 ms |
| TAC-05 | Validar entradas | ADR-03 | Rechazo de misma clave con datos distintos | 0 % → 100 %, alerta ≤ 1 min | Rechazo si un proveedor reutiliza identificadores |
| TAC-06 | Usar un intermediario | ADR-03 y ADR-04 | Partes que cambian con un proveedor nuevo | Interfaz de saldo → 1 pieza nueva | 5 piezas; 2–4 semanas de laboratorio por cambio |
| TAC-07 | Verificar integridad del mensaje | ADR-04 | Batería de falsificación rechazada | 67 % → 100 % | +0,05 ms; 0,8 % de CPU; 5 claves con validación |
| TAC-08 | Informar a los actores | ADR-04 (y ADR-03) | Tiempo hasta la alerta de seguridad | Sin alerta → ≤ 1 min | Hasta 240 alertas falsas al día |
| TAC-09 | Múltiples copias de datos | ADR-01 | Evaluación del retiro durante la reconsulta | ≤ 19,4 h → ≤ 10 ms | 500 MB de memoria; dos procesos coherentes |
| TAC-10 | Limitar la tasa de eventos | ADR-01 | Duración de la reconsulta · uso de la base | 19,4 h → 1,9 h · 0,30 → 0,31 | Marca de riesgo hasta 1,9 h tarde |

Diez tácticas: 3 en ADR-02, 3 en ADR-03 (una compartida con ADR-04), 2 propias en ADR-04 y 2 en ADR-01. Todas dentro del rango de 2 a 3 por ADR del plan.

---

## 7. Cifras nuevas

### 7.1 Fuentes

| ID | Fuente | Qué aporta |
|----|--------|-----------|
| F39 | CommScope, "Latency in optical fiber systems", libro blanco WP-111432. https://www.commscope.com/globalassets/digizuite/2799-latency-in-optical-fiber-systems-wp-111432-en.pdf | La luz viaja por la fibra ~1,5 veces más lento que en el vacío: ~5 µs por kilómetro en un sentido |
| F40 | D. Ongaro y J. Ousterhout, "In Search of an Understandable Consensus Algorithm", USENIX ATC 2014, pp. 305–319. https://raft.github.io/raft.pdf | Algoritmo de consenso por mayoría: el servidor que atiende envía latidos; un intervalo de elección aleatorio de 150–300 ms detecta su caída y elige a otro en menos de un segundo |
| F41 | Holori, calculadora de precios de servidores en la nube: instancia de 16 núcleos virtuales, 128 GB y disco local de estado sólido, región us-east-1. https://calculator.holori.com/aws/ec2/i4i.4xlarge/us-east-1 | ~USD 1,373 por hora, ~USD 1.002 al mes bajo demanda, para un servidor del tamaño de S32 |
| F42 | Naciones Unidas, Consejo de Seguridad, "Composition of the List" (Lista consolidada de sanciones). https://www.un.org/sc/suborg/en/sanctions/un-sc-consolidated-list.html | 684 personas y 193 entidades: 877 registros |

Se reutilizan sin cambios: F24 (una integración firma el cuerpo de cada petición con clave privada), F26 (otra integración agrega un resumen con secreto compartido sobre los parámetros; unicidad del identificador por jugador), F34 y F35 (límite de la base y forma de la transacción de referencia) y F36 (~22.220 verificaciones de firma de 2.048 bits por segundo en un núcleo).

**Sobre las fuentes.** F41 es un agregador de precios, no la lista oficial del proveedor de nube, y el precio es de una región de Estados Unidos; se usa como orden de magnitud del costo de un servidor. F42 no muestra fecha y la lista cambia; se usa para mostrar que la lista vinculante es pequeña. Los nombres de productos solo aparecen en esta lista (`00` §6).

### 7.2 Supuestos

| ID | Supuesto | Razón |
|----|----------|-------|
| S37 | En la línea base, la copia al sistema alterno va **~1 s** atrasada en el pico (rango de 1 a 5 s) | La línea base copia "segundos después" (`02` §3.1). El enlace usa 6 Mbit/s de su capacidad (`04` §2.6), así que el atraso lo fija el proceso de copia, no la red; 1 s es el extremo optimista y 5 s el pesimista. Agregado el 13 de noviembre de 2026. |
| S38 | El sistema alterno está a **~400 km de fibra** del sitio principal, en otra ciudad. Ida y vuelta: **5 a 10 ms** con los equipos de red | E5 exige un sitio físicamente separado [F1]. 400 km es el orden de la distancia por carretera entre las dos ciudades más grandes del país; la fibra sigue rutas parecidas. 2 × 400 km × 5 µs/km = 4 ms [F39], más equipos. Agregado el 13 de noviembre de 2026. |
| S39 | El enlace entre sitios tiene **99,9 %** de disponibilidad | Valor usual de un enlace contratado sin redundancia. Es el mismo número que la meta de juego y saldo de S13, lo que hace visible el choque de la opción A. Agregado el 13 de noviembre de 2026. |
| S40 | Dentro del centro de datos, esperar la grabación en un segundo servidor suma **~1 ms**. Con la mayoría de tres, el servidor de aplicación vuelve a atender **≤ 30 s** después de la caída. En la línea base, pasar al alterno a mano toma **10 a 30 min** | Ida y vuelta dentro de un centro de datos menor a 1 ms más una escritura en disco local; la referencia F34 ya mide con esa espera. Los 30 s cubren la detección y la elección [F40] con amplio margen para reconectar. Los 10–30 min de la línea base: alguien debe notar la caída, decidir el cambio, activar el alterno y redirigir a los proveedores. Agregado el 13 de noviembre de 2026. |
| S41 | Cada registro de identificador procesado ocupa **~100 bytes** con su índice (proveedor, jugador, identificador, resumen de 32 bytes de los datos y respuesta). Insertarlo suma **≤ 1 ms** a la transacción | Los campos son cortos y el resumen tiene tamaño fijo. Una inserción en un índice que ya está en memoria por los jugadores conectados (`04` §2.2) es del orden de microsegundos; ≤ 1 ms es una cota amplia. Agregado el 13 de noviembre de 2026. |
| S42 | **(a)** Los 5 proveedores firman el contenido de cada petición con su propio esquema, sin garantía de que firmen la hora. **(b)** La lista del proveedor tiene **hasta 1 M de registros** de ~500 bytes y se carga en memoria en **≤ 1 min**. **(c)** La evaluación local de un retiro tarda **≤ 10 ms**. **(d)** La reconsulta contra la lista propia corre a **50 cuentas/s** | (a) Las dos integraciones documentadas lo hacen [F24][F26] y S29 dice que cada proveedor trae su esquema. (b) La lista vinculante tiene 877 registros [F42]; 1 M es una cota para listas comerciales más amplias. Construir un índice de 1 M de entradas en memoria toma segundos; 1 min es margen. (c) Una búsqueda en memoria y una lectura en la base, del orden de la operación de saldo [F34]. (d) ~3 veces el mínimo de 16,2/s [S19], con +0,01 de uso de la base. Agregado el 13 de noviembre de 2026. |
| S43 | Los rechazos se agrupan por origen en ventanas de **1 minuto** para alertar | Con el primer rechazo de un origen se abre el incidente; agrupar por minuto evita una alerta por petición y queda 5 veces por debajo del plazo de ≤ 5 min [S25][S28]. Agregado el 13 de noviembre de 2026. |

**S42 junta cuatro cifras** porque todas nacen de la opción C de ADR-01 y de la B de ADR-04 y se discuten juntas en el punto de control. Si el equipo prefiere un supuesto por cifra, se separan en S42 a S45 antes de cerrar la fase.

---

## 8. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| Cada opción del Bloque 2 se relacionó con tácticas del catálogo | ✔ §1.2, las doce opciones |
| Cada táctica tiene el nombre exacto del catálogo | ✔ Redundancia activa, Latido (heartbeat), Reconfiguración, Transacciones, Validar entradas, Usar un intermediario, Verificar integridad del mensaje, Informar a los actores, Múltiples copias de datos, Limitar la tasa de eventos |
| Cada mecanismo dice qué componente, qué dato y en qué paso | ✔ Cada ficha cita el paso de `02` §3.1 o §3.2 |
| Cada táctica mueve una métrica de un valor a otro, con la cuenta | ✔ §6, diez tácticas con antes y después |
| Cada estimación declara su tipo | ✔ Campo "Tipo de la estimación" en cada ficha |
| Cada costo es un número y tiene un atributo que lo paga | ✔ Dinero, milisegundos, gigabytes, núcleos, semanas de laboratorio, alertas al día |
| Se cerraron los "Depende" del plan (ADR-01 B, ADR-02 A y C, ADR-03 B, ADR-04 B) | ✔ §1.3 y el cierre en cada sección |
| Ninguna táctica de esta fase estaba en la línea base, salvo como piso reforzado | ✔ TAC-04 refuerza Transacciones (declarado); las demás están en la lista de libres de `02` §6 |
| Las cuentas cuadran | ✔ 3.000 × 1 s = 3.000; 2 × 400 × 5 µs = 4 ms; 0,001 × 43.200 min = 43,2 min; 2 × 1.002 × 12 = 24.048 USD; 24.048 × 4.000 = COP 96 M = 1,3 % de 7.300 M; 3.000 × 30 s = 90.000; 3.000 M × 100 B = 300 GB; 1 ÷ 22.220 = 0,045 ms; 3.000 × 0,045 = 135 ms/s; (4.200 + 135) ÷ 16.000 = 0,27; 350.000 ÷ 50 = 7.000 s = 1,94 h; 1.000 ÷ 86.400 ÷ 5 = 0,002; 125 ÷ 60 ≈ 2 |
| Las cifras que llegan de las fases 3 y 4 no cambiaron | ✔ 3.000/s, 4.000/s, 3.000 M identificadores, 3.000 retiros al día, 10.000/s, 16.000/s, 5 consultas/s, 3,24, 19,4 h, 2.430 |
| Ningún mecanismo nombra tecnologías ni marcas | ✔ Los productos solo aparecen en la lista de fuentes (§7.1) |
| Cada cifra nueva tiene tipo y origen | ✔ F39 a F42 con enlace; S37 a S43 con razón |
| No se escribieron Decision ni Consequences | ✔ Quedan para la fase 6, sobre la opción elegida |

**Observaciones para las fases siguientes:**
- **Fase 6:** la Decision de ADR-04 debe decir qué se hace con un proveedor que no firme el contenido (riesgo de S42 a, §3.1). Las Consequences de ADR-02 deben decir que la pérdida del sitio completo sigue dejando ~3.000 operaciones en riesgo (S37).
- **Fase 7:** TAC-03 genera hasta ~90.000 reenvíos, el estímulo de ESC-03, y TAC-04 los absorbe. TAC-01 y TAC-04 suman tiempo al mismo paso de escritura que ADR-04; la suma contra los 250 ms es la tarea 7.2. ESC-04 (720 abonos en un segundo) pasa por TAC-04 y TAC-07.

---

## 9. Punto de control (cerrado el 13 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las cinco recomendaciones. Es la decisión central del bloque: la opción de cada ADR.

| # | Decisión | Aprobado | Variantes registradas |
|---|----------|----------|-----------------------|
| 1 | ADR-02 | **C** · mayoría de tres en el sitio principal (TAC-01, TAC-02, TAC-03) | A sobre C (con dos enlaces); A sola (§2.3) |
| 2 | ADR-03 | **B con la traducción de C** (TAC-04, TAC-05, TAC-06) | B sola (§2.3) |
| 3 | ADR-04 | **B** · firma por petición en la pieza de cada proveedor (TAC-06, TAC-07, TAC-08) | B + C donde se pueda; B con hora firmada (§3.1) |
| 4 | ADR-01 | **C** · lista vigente en memoria y reconsulta a 50 cuentas/s (TAC-09, TAC-10) | Ritmo de 100/s; lista en la base única (§4.1) |
| 5 | Cifras nuevas | **F39 a F42 y S37 a S43 aprobados**, con S42 como un solo supuesto de cuatro cifras | Separar S42 en cuatro supuestos (§7.2) |

**Para la sustentación,** los números más débiles de esta fase son S37 (atraso de la copia, de él depende el "~3.000 operaciones perdidas") y S40 (los 10–30 min del cambio manual en la línea base). Los dos son de la línea base, no de las tácticas: si fueran mayores, las tácticas ganarían más.

La fase 6 (ADR completados) puede arrancar.
