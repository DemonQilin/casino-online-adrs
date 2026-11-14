# Fase 4 — Capacidad actual y cuello de botella

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 4 (Capacidad actual y cuello de botella)
**Criterio de la rúbrica:** Línea base y cuello de botella. Nivel "Logrado": línea base clara, componente que se satura primero, su límite con fuente y qué ve el usuario al superarlo. Esta fase aporta la segunda mitad del criterio; la primera la cerró la fase 2.

**Insumos:**
- `02-linea-base.md` §3 y §5: los pasos de la operación de saldo y del retiro, y los siete candidatos a saturarse.
- `03-perfil-de-carga.md`: 3.000 operaciones de saldo/s en el pico y 4.000/s en ráfaga, 0,249 operaciones/s por jugador, 12.000 concurrentes, 16,2 consultas/s de la reconsulta, volúmenes del historial y presupuestos de tiempo.
- `bloque-2/02-ficha-contexto.md` §4 y §6: S8, S14, S19 y el máximo legal de 72 h para el retiro [F1].
- `docs/class-material/Tácticas y sizing · Bloque 3.html`: sección 4 de la plantilla y la regla "si nada se satura, busquen el límite en otra dimensión".

**Qué fija esta fase:**
1. El límite de cada candidato de la línea base, con fuente o con un supuesto que lleva su razón.
2. El uso de cada candidato con la carga del pico y de la ráfaga (carga ÷ límite).
3. El componente que se satura primero y hasta cuánta carga aguanta hoy la línea base.
4. Lo que ve el jugador, el proveedor o el regulador cuando se supera el límite.

Esta fase agrega cinco fuentes (F34 a F38) y cinco supuestos (S32 a S36), registrados en la sección 7.

---

## 1. Candidatos y criterio (tarea 4.1)

Se revisan los siete candidatos de `02` §5, no solo el servidor. Para cada uno se escribe:

- **Límite:** la carga máxima que soporta, en su propia unidad, con su tipo y origen.
- **Carga en el pico:** la que le llega con 3.000 operaciones de saldo/s (fase 3).
- **Uso:** carga ÷ límite. Con un uso mayor que 1, el componente está saturado.
- **Concurrentes que aguanta:** el límite pasado a jugadores conectados con la carga por jugador de la fase 3. Es la unidad común para comparar candidatos que se miden en cosas distintas.

**Tamaño de los servidores.** S30 fija uno de aplicación y uno de base de datos, pero no su tamaño. Sin tamaño no hay límite. Se fija en S32: **16 núcleos virtuales, 128 GB de memoria y disco de estado sólido local**, para los dos servidores y en los dos sitios. La razón es que es el tamaño de la prueba pública que sirve de referencia (F34); la sección 4 explica por qué un servidor más pequeño no "funcionaría".

---

## 2. Límite de cada candidato (tarea 4.2)

### 2.1 Base de datos única: transacciones de saldo

Es el candidato que señaló la fase 3: toda operación de saldo termina en una transacción que se graba en disco antes de responder (`03` §6).

**Referencia que se usa.** Una prueba pública de julio de 2026 midió una base de datos relacional con una carga "tipo TPC-B": transacciones cortas, muy concurrentes, que cambian saldos y agregan una fila a un historial [F34]. El guion de esa carga actualiza el saldo de una cuenta, lo lee, actualiza otros dos saldos y agrega una fila al historial, todo en una transacción [F35]. Es casi la misma forma de la operación de saldo de la línea base.

| Configuración de la prueba [F34] | Datos | Transacciones/s | p99 |
|-----------------------------------|-------|-----------------|-----|
| 16 núcleos, 128 GB, disco local de estado sólido | ~100 GB | 16.887 y 25.429 (dos servicios) | 32,9 y 29,1 ms |
| 16 núcleos, 128 GB, disco local de estado sólido | ~500 GB | **14.046** y 21.694 (dos servicios) | 38,6 y 31,4 ms |
| 4 núcleos, 32 GB, disco local de estado sólido | ~100 GB | 4.676 y 6.242 (dos servicios) | 103,6 y 115,2 ms |

En todas las corridas la prueba confirma cada transacción solo después de que una réplica en espera la recibe, con 256 clientes simultáneos [F34].

**Diferencias entre la prueba y la línea base:**

| Diferencia | Efecto sobre el límite | Cómo se trata |
|-----------|------------------------|---------------|
| La prueba espera a una réplica antes de confirmar; la línea base confirma con la grabación local | La línea base aguantaría **más** | Se ignora (conservador) |
| La prueba cambia 3 saldos y agrega 1 fila; la línea base cambia 1 saldo y agrega 1 fila | La línea base aguantaría **más** | Se ignora (conservador) |
| Los datos de la línea base son más grandes: el historial de 6 meses pesa ~1,1 TB (4.400 millones de operaciones × 250 bytes [S34]), 2,2 veces los 500 GB de la prueba | La línea base aguanta **menos** | Se descuenta (S36) |
| La línea base busca el identificador en un índice del historial y lo mantiene en cada inserción; la prueba no indexa el historial | La línea base aguanta **menos** | Se descuenta (S36) |
| La copia al alterno, la extracción para Coljuegos y la reconsulta leen la misma base (~6.000 lecturas/s, `03` §6) | La línea base aguanta **menos** | Se descuenta (S36) |

**Cuánto se descuenta (S36).** En la prueba, pasar de 100 GB a 500 GB (5 veces más datos) bajó el rendimiento entre 15 % y 17 %:

```
16.887 → 14.046:  14.046 ÷ 16.887 = 0,83  →  −17 %
25.429 → 21.694:  21.694 ÷ 25.429 = 0,85  →  −15 %
```

Se descuenta otro 15 % por los datos de la línea base (2,2 veces más que la prueba; se repite el escalón completo aunque el salto sea menor) y otro 15 % por el índice del historial y las lecturas de fondo. Total: −30 %, sobre el menor de los dos servicios.

```
límite de referencia  = 14.046 transacciones/s                              [F34 · referencia]
límite de diseño      = 14.046 × (1 − 0,30) = 9.832 ≈ 10.000 transacciones/s  [F34 · referencia][S36 · supuesto]
```

El límite se da como un rango: **entre ~10.000 y ~14.000 transacciones de saldo por segundo**. Las cuentas usan el extremo bajo, ~10.000/s. Por la regla de la fase 1 (§5), la cifra toma el tipo de su insumo más débil: es un **supuesto apoyado en una referencia**.

**Tiempo de respuesta por debajo del límite.** La prueba, saturada, da p99 de 31 a 39 ms [F34]. Con la carga del pico, el paso de la base consume menos de ~40 ms de los 250 ms de S14. **La línea base cumple el tiempo de respuesta en el pico**; lo que no cumple es la pérdida cero (ESC-02), que no es un problema de carga.

| Carga | Límite | Uso |
|-------|--------|-----|
| Pico sostenido: 3.000 transacciones/s | 10.000/s (14.000/s) | 3.000 ÷ 10.000 = **0,30** (0,21) |
| Ráfaga: 4.000 transacciones/s | 10.000/s (14.000/s) | 4.000 ÷ 10.000 = **0,40** (0,29) |

```
concurrentes que aguanta = 10.000 operaciones/s ÷ 0,249 operaciones/s por jugador = 40.160 ≈ 40.000 jugadores
                         (14.000 ÷ 0,249 = 56.225 ≈ 56.000 con el límite de referencia)
```

### 2.2 Base de datos única: búsqueda del identificador y volumen

En la línea base el identificador se busca con el proveedor y el jugador (`02` §3.1, paso 4c). La búsqueda solo toca la parte del índice de ese jugador, y en el pico esa parte es la de los 12.000 conectados, que se consulta una y otra vez. El índice completo no tiene que caber en memoria para que la búsqueda sea rápida.

| Dimensión | Cuenta | Valor |
|-----------|--------|-------|
| Historial de 6 meses | 4.400 millones × 250 bytes [S34] | **~1,1 TB** |
| Crecimiento en un año | 8.850 millones × 250 bytes | **~2,2 TB por año** |
| Memoria del servidor | — | 128 GB [S32] |

**Conclusión:** no se satura en el pico. Su costo ya está dentro del descuento de S36. Es una dimensión de **crecimiento**: el historial no deja de crecer y la opción A de ADR-03 lo usa para buscar. Se retoma en la fase 5 al cerrar esa opción.

### 2.3 Base de datos única: lecturas de fondo

La copia al alterno y la extracción leen cada operación una vez, poco después de grabarla: 3.000 + 3.000 lecturas/s. La reconsulta agrega 16 lecturas y 16 escrituras por segundo (`03` §5.2). Son lecturas de datos recientes, que todavía están en memoria.

**Conclusión:** no tienen límite propio. Restan capacidad a la misma base y por eso entran en el descuento de S36.

### 2.4 Servidor de aplicación

**Límite.** El servidor tiene 16 núcleos [S32]. Cada petición, de un proveedor o del sitio de los jugadores, cuesta ~1 ms de cómputo: interpretar el protocolo, comparar la clave, llamar a la base y armar la respuesta [S33 · supuesto]. No hay una referencia pública para este número; es el supuesto más débil de la fase.

```
capacidad = 16 núcleos × 1.000 ms de cómputo por segundo ÷ 1 ms por petición = 16.000 peticiones/s
```

**Carga.** Las operaciones de saldo, más el sitio de los jugadores, a ~0,1 peticiones/s por jugador conectado [S33]. El saldo que ve el jugador durante el juego lo muestra el proveedor con la respuesta de cada operación, así que el sitio del operador no se consulta en cada jugada.

```
sitio de los jugadores = 12.000 × 0,1 = 1.200 peticiones/s
pico:    (3.000 + 1.200) ÷ 16.000 = 4.200 ÷ 16.000 = 0,26
ráfaga:  (4.000 + 1.200) ÷ 16.000 = 5.200 ÷ 16.000 = 0,33
```

**Canal cifrado.** Abrir una conexión cifrada cuesta una firma con la clave de 2.048 bits del servidor. Un núcleo hace ~1.009 firmas por segundo, ~1 ms por firma [F36]. En HTTP/1.1 las conexiones son persistentes por omisión [F38]: los proveedores envían muchas peticiones por la misma conexión y la firma se paga solo al abrirla. La línea base supone que así ocurre [S33].

**Caso pesimista.** Si un proveedor abriera una conexión por petición, cada operación de saldo costaría 1 ms + 1 ms:

```
pico:    (3.000 × 2 ms + 1.200 × 1 ms) ÷ 16.000 ms = 7.200 ÷ 16.000 = 0,45
```

En ese caso el servidor de aplicación se saturaría **antes** que la base. Se registra como riesgo en la sección 4.

```
concurrentes que aguanta = 16.000 ms/s ÷ (0,249 × 1 ms + 0,1 × 1 ms) por jugador
                         = 16.000 ÷ 0,349 = 45.845 ≈ 46.000 jugadores
caso pesimista           = 16.000 ÷ (0,249 × 2 ms + 0,1 × 1 ms) = 16.000 ÷ 0,598 = 26.756 ≈ 27.000 jugadores
```

### 2.5 Red pública hasta el sitio principal

**Límite.** Enlace del sitio principal a internet de 1 Gbit/s [S35]. Tamaños: ~2 KB por operación de saldo (petición y respuesta) y ~20 KB por petición del sitio de los jugadores, sin el contenido de los juegos, que sirve el proveedor [S35]. Aquí 1 KB = 1.000 bytes.

```
proveedores:  3.000 /s × 2 KB  =  6.000 KB/s =  6 MB/s × 8 =  48 Mbit/s
jugadores:    1.200 /s × 20 KB = 24.000 KB/s = 24 MB/s × 8 = 192 Mbit/s
total:        240 Mbit/s ÷ 1.000 Mbit/s = 0,24
```

```
por jugador conectado = 0,249 × 2 KB + 0,1 × 20 KB = 2,5 KB/s = 20 kbit/s
concurrentes que aguanta = 1.000.000 kbit/s ÷ 20 kbit/s = 50.000 jugadores
```

### 2.6 Copia al sistema alterno

**Enlace entre sitios.** 3.000 operaciones/s × 250 bytes [S34] = 750 KB/s ≈ 6 Mbit/s. No se acerca a ningún límite.

**Base del alterno.** Recibe las mismas 3.000 escrituras/s que la principal y tiene el mismo tamaño [S32]: su uso es el de la base principal, 0,30. No se satura antes que ella.

**Conclusión:** la copia no se satura en el pico. Su riesgo no es de carga, sino la ventana de operaciones confirmadas que se pierden si el principal cae antes de copiar. Es la medida de ESC-02 y se trata en ADR-02.

### 2.7 Proveedor de listas restrictivas (dependencia externa)

**Límite.** El personal técnico de un proveedor de listas de alcance mundial recomienda **un máximo de 5 consultas por segundo** a su servicio. Las peticiones por encima de ese ritmo pueden ser rechazadas, y el valor exacto varía con la capacidad de su plataforma [F37]. En otra respuesta del mismo foro, el mismo proveedor aconseja 1 consulta por segundo [F37]. Se usa 5/s, el valor más alto, para no exagerar el problema.

```
ritmo que necesita la reconsulta = 350.000 cuentas ÷ 21.600 s = 16,2 consultas/s   [S3][S19 · supuesto]
uso                              = 16,2 ÷ 5 = 3,24
```

**El proveedor de listas ya está saturado con la carga de diseño: 3,2 veces su límite.** El operador no lo controla, y comprar otro servidor no lo mueve.

Las ~1.000 consultas al día del registro de jugadores (ficha §4) son ~0,01/s y no cambian la cuenta.

| Con el límite del proveedor | Cuenta | Valor |
|-----------------------------|--------|-------|
| Duración de la reconsulta | 350.000 ÷ 5 /s = 70.000 s ÷ 3.600 s/h | **~19,4 h**, frente a la meta de ≤ 6 h [S19] |
| Cuentas que caben en 6 h | 5 /s × 21.600 s | **108.000**, menos de un tercio de las 350.000 |
| Retiros retenidos durante una reconsulta (opción A de ADR-01) | 3.000 al día × 19,4 h ÷ 24 h | **~2.430**, frente a ~750 de la fase 3 |
| Con 1 consulta/s | 350.000 s ÷ 3.600 s/h | **~97 h**: supera el máximo legal de 72 h para el retiro [F1] |

---

## 3. Uso en el pico (tarea 4.3)

Todos los candidatos en una tabla, ordenados por los jugadores conectados que aguantan.

| Candidato | Límite | Tipo del límite | Uso en el pico | Uso en la ráfaga | Aguanta hasta |
|-----------|--------|-----------------|----------------|------------------|---------------|
| **Proveedor de listas** (reconsulta) | 5 consultas/s | Referencia [F37] | **3,24** | — (no depende del juego) | 108.000 cuentas en 6 h |
| **Base única: transacciones de saldo** | ~10.000 /s (~14.000) | Supuesto sobre referencia [F34][S36] | **0,30** | 0,40 | **~40.000 jugadores** |
| Servidor de aplicación | 16.000 peticiones/s | Supuesto [S32][S33] | 0,26 | 0,33 | ~46.000 jugadores |
| Red pública | 1 Gbit/s | Supuesto [S35] | 0,24 | ~0,30 | ~50.000 jugadores |
| Base del alterno | Igual a la principal | Supuesto sobre referencia [F34][S36] | 0,30 | 0,40 | ~40.000 jugadores |
| Enlace entre sitios para la copia | 6 Mbit/s de carga | Supuesto [S34] | Despreciable | Despreciable | — |
| Búsqueda del identificador y lecturas de fondo | Dentro del límite de la base | Supuesto [S36] | Incluido arriba | Incluido arriba | — |

**Lectura:**

1. **En la operación de saldo nada se satura con el pico de diseño.** El primero en saturarse cuando crece la carga es la base única, al 30 % en el pico y al 40 % en la ráfaga.
2. **Lo que ya está saturado es una dependencia externa:** el proveedor de listas, a 3,2 veces su límite, en la dimensión del retiro.
3. **La base y el servidor de aplicación están cerca** (40.000 frente a 46.000 jugadores). La diferencia es que el servidor de aplicación se multiplica sin tocar ningún ADR, y la base única no: todas las escrituras del saldo pasan por ella.

---

## 4. Cuello de botella

### Propuesta: la base única como cuello de la operación de saldo, y el proveedor de listas como límite ya superado en la otra dimensión

> **Operación de saldo.** La base de datos única es el primer componente que se satura. Aguanta entre ~10.000 y ~14.000 transacciones de saldo por segundo [F34 · referencia][S36 · supuesto]; en el pico de diseño recibe 3.000/s, un uso de 0,30 (0,40 en la ráfaga). Con la carga del pico **no se satura**: la línea base aguanta hasta ~40.000 jugadores conectados, 3,3 veces el pico.
>
> **Retiro.** El proveedor de listas restrictivas recomienda como máximo 5 consultas por segundo [F37 · referencia]. La reconsulta necesita 16,2/s [S19 · supuesto]: un uso de **3,24**. La reconsulta tarda ~19,4 h en lugar de 6 h.

**Por qué así:**
- La operación crítica es el saldo (fase 1). La sección 7 de la plantilla y la extensión X1 hablan de esa operación y de esa unidad. Con X1 (×5), el pico llega a 15.000/s y supera el límite de la base, así que ahí se ve su saturación.
- El material dice que, si nada se satura, se diga así y se busque el límite en otra dimensión, por ejemplo en una dependencia externa. El proveedor de listas es exactamente eso, y su número de uso (3,24) es el que pide la rúbrica.
- La base única no se multiplica sin cambiar las decisiones de ADR-02 y ADR-03, que viven en su paso de escritura. Es el cuello que importa para el Bloque 4.

### Variantes

| Variante | Qué dice la sección 4 de la plantilla | A favor | En contra |
|----------|----------------------------------------|---------|-----------|
| **A (recomendada)** | Componente: la base única, uso 0,30, no se satura en el pico. Se agrega el proveedor de listas como la dependencia que ya se satura (3,24) | Coherente con la operación crítica, con la sección 7 y con X1. Dice con honestidad que en el pico nada del saldo se satura | El campo "componente que se satura primero" nombra un componente que en el pico no se satura. Hay que explicarlo en la sustentación |
| **B** | Componente: el proveedor de listas, uso 3,24. La base única queda como el primero en saturarse dentro de la operación de saldo | Responde de forma literal a "¿qué se satura primero hoy?" | El campo "hasta cuánta carga aguanta" pide jugadores o tasa por segundo, y el proveedor se mide en cuentas por reconsulta. Separa la sección 4 de la operación crítica de las secciones 3, 7 y 8 |

**Decisión del equipo:** variante A o B. La recomendación es A.

### Riesgos que el equipo debe conocer para la sustentación

| Riesgo | Qué pasa si se cumple | Cómo se defiende |
|--------|------------------------|------------------|
| **El límite de la base viene de una prueba de otro sistema** (otra máquina, otros datos) | El límite real puede ser distinto | El descuento de S36 es explícito, los efectos a favor se ignoran y el resultado se da como un rango. Con el extremo alto (14.000/s), la conclusión no cambia: tampoco se satura en el pico |
| **Un proveedor abre una conexión por petición** | El servidor de aplicación pasa al 45 % y se satura antes que la base (~27.000 jugadores) | HTTP/1.1 usa conexiones persistentes por omisión [F38]. Si pasara, se arregla multiplicando el servidor de aplicación, sin tocar ningún ADR |
| **El proveedor de listas permite solo 1 consulta/s** | La reconsulta tarda ~97 h y supera el máximo legal de 72 h para el retiro [F1] | Se usa 5/s para no exagerar. Con cualquiera de los dos valores, la línea base incumple las 6 h. Es el número que ADR-01 debe mover en la fase 5 |
| **Un servidor de 4 núcleos sería "más simple"** | El límite bajaría por debajo del pico | La prueba da 4.676 a 6.242 transacciones/s con 4 núcleos y solo 100 GB [F34]. Con el descuento por los datos de la línea base (~1,1 TB), quedaría por debajo de las 4.000/s de la ráfaga. No "funcionaría", así que no es la línea base (S32) |

---

## 5. Hasta cuánta carga aguanta hoy la línea base

```
operación de saldo:  límite de la base ÷ operaciones por jugador
                     = 10.000 /s ÷ 0,249 /s por jugador ≈ 40.000 jugadores conectados
                     margen sobre el pico: 10.000 ÷ 3.000 = 3,3   (14.000 ÷ 3.000 = 4,7)

si además debe caber la ráfaga (4.000 ÷ 3.000 = 1,33 veces el sostenido):
                     10.000 ÷ 1,33 = 7.500 operaciones/s sostenidas ≈ 30.000 jugadores conectados

reconsulta:          5 consultas/s × 21.600 s = 108.000 cuentas en 6 h
                     la línea base no cumple la meta desde ~108.000 cuentas registradas; hoy hay 350.000
```

**Para la plantilla:** la línea base aguanta ~10.000 operaciones de saldo/s, **~40.000 jugadores conectados** (3,3 veces el pico de 12.000), o ~30.000 si la ráfaga debe caber también. La reconsulta no cumple las 6 h desde ~108.000 cuentas.

---

## 6. Qué ve el usuario cuando se supera el límite (tarea 4.4)

### 6.1 Base única por encima de ~10.000 transacciones/s

Por encima del límite las transacciones se acumulan en cola y cada respuesta tarda más. La medida de 250 ms se rompe primero; el problema visible empieza al pasar los ~3 s de espera del proveedor [S14].

| Quién | Qué ve |
|-------|--------|
| **Jugador** | El giro o la apuesta se queda esperando y termina en un mensaje de error del juego. Si era un descuento, el proveedor lo da por fallido y lo revierte [F24]: la jugada se anula. Si era un abono, el premio no aparece en el saldo hasta que el proveedor lo reenvía con éxito |
| **Proveedor de juego** | Operaciones sin respuesta en ~3 s. Reenvía los abonos y revierte los descuentos [F24][F26]. **Cada reenvío es una petición más**: la carga sube justo cuando la base ya no da abasto, y la saturación se agrava sola |
| **Regulador** | Más reversiones y operaciones repetidas en el historial que se extrae para el sistema de inspección. La extracción lee la misma base saturada y se atrasa |

### 6.2 Proveedor de listas por encima de 5 consultas/s

| Quién | Qué ve |
|-------|--------|
| **Jugador** | Su retiro queda "en revisión" durante la reconsulta: hasta ~19,4 h en lugar de 6 h. Unos **~2.430 retiros** retenidos por reconsulta, varias veces al mes. Un retiro pedido al empezar la reconsulta consume casi toda la meta de 24 h [S8] antes de evaluarse |
| **Proveedor de listas** | Si la reconsulta corre "sin ritmo máximo", como en la línea base (`02` §3.2), recibe más de 5 consultas/s y empieza a rechazar peticiones [F37]. La reconsulta tiene que reintentarlas y tarda todavía más |
| **Regulador** | Con 5 consultas/s, el retiro se sigue pagando dentro del máximo de 72 h [F1]. Con 1 consulta/s (~97 h), no |

---

## 7. Cifras nuevas

### 7.1 Fuentes

| ID | Fuente | Qué aporta |
|----|--------|-----------|
| F34 | ClickHouse, "PostgresBench: benchmarking NVMe managed Postgres", S. Srirampur (30 de julio de 2026). https://clickhouse.com/blog/benchmarking-nvme-managed-postgres-planetscale-vs-clickhouse | Transacciones por segundo y p99 de una carga tipo TPC-B en servidores de 4 y 16 núcleos con disco local, con 100 y 500 GB de datos y confirmación tras una réplica |
| F35 | PostgreSQL, documentación de *pgbench*, guion "tpcb-like". https://www.postgresql.org/docs/current/pgbench.html | Qué hace cada transacción de la prueba F34: actualiza tres saldos, lee uno y agrega una fila al historial |
| F36 | I. Ristić, *OpenSSL Cookbook*, Feisty Duck, capítulo "Performance". https://www.feistyduck.com/library/openssl-cookbook/online/openssl-command-line/performance.html | ~1.009 firmas por segundo con clave de 2.048 bits en un núcleo; cada conexión cifrada nueva cuesta una firma |
| F37 | Foro de desarrolladores de LSEG, respuestas del personal técnico sobre el servicio de consulta de listas World-Check One: "API call limit" y "Error 429 WC1 API". https://community.developers.lseg.com/discussion/114061/api-call-limit/p1 · https://community.developers.lseg.com/discussion/24958/error-429-wc1-api | Ritmo máximo recomendado de 5 consultas por segundo (y 1 por segundo en otra respuesta); por encima, las peticiones se rechazan |
| F38 | IETF, RFC 9112 "HTTP/1.1", §9.3 *Persistence* (2022). https://www.rfc-editor.org/rfc/rfc9112.html | HTTP/1.1 usa conexiones persistentes por omisión |

Los nombres de productos aparecen solo en esta lista, nunca en un mecanismo (`00` §6).

**Sobre F37.** No es documentación formal, sino respuestas del personal del proveedor en su foro público. La documentación formal dice que el servicio limita el ritmo y que el valor varía con la capacidad, sin dar un número. Es la mejor cifra pública que se encontró; el equipo debe poder decirlo en la sustentación.

### 7.2 Supuestos

| ID | Supuesto | Razón |
|----|----------|-------|
| S32 | Cada servidor de la línea base (S30), de aplicación y de base de datos, en los dos sitios, tiene **16 núcleos virtuales, 128 GB de memoria y disco de estado sólido local** | Es el tamaño de la prueba de referencia F34, lo que permite usar su resultado sin cambiar de máquina. Un servidor de 4 núcleos quedaría por debajo de la ráfaga (§4), así que no sería "la versión más simple que funcionaría". Agregado el 13 de noviembre de 2026. |
| S33 | Cada petición cuesta **~1 ms de cómputo** en el servidor de aplicación, sin contar el cifrado. El sitio de los jugadores genera **~0,1 peticiones/s por jugador conectado**. Los proveedores **reutilizan las conexiones** | El trabajo por petición en la línea base es corto (interpretar, comparar una clave, dos llamadas a la base, armar la respuesta). Durante el juego el saldo lo muestra el proveedor, así que el sitio del operador solo atiende navegación y consultas ocasionales. La reutilización de conexiones es el comportamiento por omisión de HTTP/1.1 [F38]. Es el supuesto más débil de la fase: no hay referencia pública del costo por petición. Agregado el 13 de noviembre de 2026. |
| S34 | Cada operación ocupa **~250 bytes** en el historial, contando su entrada en el índice del identificador | Identificador de transacción del proveedor, jugador, proveedor, tipo, monto, ronda, saldo resultante y marcas de tiempo, más la entrada en el índice. Orden de magnitud. Agregado el 13 de noviembre de 2026. |
| S35 | El sitio principal tiene un enlace a internet de **1 Gbit/s**. Una operación de saldo ocupa **~2 KB** en la red (petición y respuesta) y una petición del sitio de los jugadores **~20 KB**, sin el contenido de los juegos | Enlace estándar para un servidor en un centro de datos. Las peticiones de saldo son mensajes cortos con pocos campos. El contenido de los juegos lo sirve el proveedor, no el operador. Agregado el 13 de noviembre de 2026. |
| S36 | El límite de la base única de la línea base es el de la prueba F34 con **−30 %**: ~10.000 transacciones/s, con 14.000/s como extremo alto | En F34, cinco veces más datos bajaron el rendimiento entre 15 % y 17 %. Se descuenta un escalón igual (−15 %) porque la línea base tiene 2,2 veces más datos que la prueba, y otro −15 % por el índice del historial y las lecturas de fondo. Se ignoran dos efectos a favor: la línea base no espera a una réplica y escribe menos filas por transacción. Agregado el 13 de noviembre de 2026. |

---

## 8. Texto para la plantilla (sección 4)

*Para transcribir en la fase 10. Redactado con la variante A.*

**Componente que se satura primero:** la base de datos única del sitio principal, en el paso de escritura del saldo. Con la carga del pico no se satura. La dependencia que sí se satura hoy es el proveedor de listas restrictivas, en la reconsulta de cuentas (otra dimensión).

**Su límite:** ~10.000 transacciones de saldo por segundo (rango 10.000–14.000) | tipo: supuesto apoyado en una referencia | razón o fuente: prueba pública de una base de datos relacional con carga tipo TPC-B en un servidor de 16 núcleos, 128 GB y disco local, con 500 GB de datos: 14.046 transacciones/s (F34, F35). Se descuenta un 30 % porque nuestros datos son 2,2 veces más grandes y la búsqueda del identificador y las lecturas de fondo usan la misma base (S36). Servidor del mismo tamaño que la prueba (S32).
Proveedor de listas: 5 consultas por segundo | tipo: referencia | razón o fuente: ritmo máximo recomendado por el personal técnico de un proveedor de listas de alcance mundial en su foro de desarrolladores (F37).

**Uso con la carga del pico (carga / límite, con la cuenta):**
```
base única:          3.000 transacciones/s ÷ 10.000 /s = 0,30   (ráfaga: 4.000 ÷ 10.000 = 0,40)
servidor de app.:    (3.000 + 1.200 del sitio) peticiones/s ÷ 16.000 /s = 0,26
red pública:         240 Mbit/s ÷ 1.000 Mbit/s = 0,24
proveedor de listas: 350.000 cuentas ÷ 21.600 s = 16,2 consultas/s ÷ 5 /s = 3,24
                     → la reconsulta tarda 350.000 ÷ 5 = 70.000 s ≈ 19,4 h, no 6 h
```

**Hasta cuánta carga aguanta hoy la línea base (usuarios concurrentes o tasa por segundo):** ~10.000 operaciones de saldo/s, es decir, 10.000 ÷ 0,249 ≈ 40.000 jugadores conectados, 3,3 veces el pico de 12.000 (≈ 30.000 si la ráfaga también debe caber). La reconsulta cumple las 6 h solo hasta 5 × 21.600 = 108.000 cuentas; hoy hay 350.000.

**Qué le pasa al usuario cuando se supera ese límite:** por encima de ~10.000 transacciones/s las respuestas se encolan. Al pasar los ~3 s de espera del proveedor, el giro del jugador termina en error: si era una apuesta, el proveedor la revierte y la jugada se anula; si era un premio, no aparece en el saldo hasta que el proveedor lo reenvía. Cada reenvío suma carga a la base ya saturada. En la reconsulta, unos 2.430 retiros quedan "en revisión" hasta ~19,4 h, varias veces al mes, y un retiro pedido al empezar la reconsulta consume casi toda la meta de 24 h antes de evaluarse.

**Dónde está esto en la entrega:** presentación HTML, sección "Cuello de botella" (se integra en la fase 10), y `bloque-3/04-cuello-de-botella.md`.

---

## 9. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| Se revisaron todos los candidatos de `02` §5, incluidas la red y la dependencia externa | ✔ §2: siete candidatos |
| El componente que se satura primero tiene un límite con fuente | ✔ Base única: F34 y F35, con el descuento de S36. Proveedor de listas: F37 |
| Hay un número de uso con su cuenta | ✔ §3: 0,30 en la base (0,40 en la ráfaga) y 3,24 en el proveedor de listas |
| Se dice hasta cuánta carga aguanta hoy la línea base | ✔ §5: ~10.000/s, ~40.000 jugadores, margen 3,3 |
| Hay una consecuencia para el usuario | ✔ §6: jugador, proveedor y regulador |
| Las cuentas cuadran | ✔ 14.046 × 0,7 = 9.832; 10.000 ÷ 0,249 = 40.160; 16.000 ÷ 0,349 = 45.845; 350.000 ÷ 5 = 70.000 s = 19,4 h; 3.000 × 19,44 ÷ 24 = 2.430; 4.400 M × 250 B = 1,1 TB |
| Las cifras que llegan de la fase 3 no cambiaron | ✔ 3.000/s, 4.000/s, 0,249, 12.000, 16,2/s, 4.400 M y 8.850 M |
| Cada cifra nueva tiene tipo y origen | ✔ F34 a F38 con su enlace; S32 a S36 con su razón |
| Ningún mecanismo nombra tecnologías ni marcas | ✔ Los nombres de productos solo están en la lista de fuentes (§7.1) |
| No se proponen tácticas | ✔ Las salidas al cuello quedan para la fase 5; solo se nombra a qué ADR toca cada uno |

**Observación para la fase 5.** El descuento de S36 depende en parte de la búsqueda del identificador en el historial, que es la opción A de ADR-03. Si la fase 5 elige otra opción para ADR-03, la cuenta del límite de la base se rehace con el mismo método.

---

## 10. Punto de control (cerrado el 13 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las cinco recomendaciones.

1. **Cuello de botella en la plantilla: variante A.** La base única es el cuello de la operación de saldo y el proveedor de listas va como la dependencia que ya se satura. La variante B queda registrada (§4).
2. **Límite de diseño de la base: ~10.000/s**, con el descuento de S36. El valor de referencia de 14.000/s se conserva como extremo alto del rango.
3. **Tamaño de los servidores (S32): 16 núcleos y 128 GB**, el tamaño de la prueba de referencia.
4. **Límite del proveedor de listas: 5 consultas/s.** El valor de 1 consulta/s queda como riesgo registrado (§4).
5. **Fuentes F34 a F38 y supuestos S32 a S36 aprobados**, con sus razones. S33 queda señalado como el supuesto más débil de la fase.

**Para contrastar las cuentas:** en la calculadora del material, con los valores de la fase 3 (16.000 personas en una ventana de 60 min, 0,249 × 2.700 operaciones por persona) y un límite del componente de 10.000 operaciones/s, el uso da ~0,30. Con 14.000 da ~0,21.

La fase 5 (tácticas y efecto) puede arrancar.
