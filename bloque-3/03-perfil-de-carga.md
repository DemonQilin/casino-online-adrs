# Fase 3 — Perfil de carga

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 3 (Perfil de carga)
**Criterio de la rúbrica:** Perfil de carga. Nivel "Logrado": cadena completa de usuarios a operaciones por segundo en el pico, con cuentas y unidades, cada número marcado como dato, supuesto o referencia.

**Insumos:**
- `01-punto-de-partida.md`: operación crítica (saldo), retiro como otra dimensión y tipos de cifra.
- `02-linea-base.md` §3: los pasos de la operación de saldo, de donde salen las lecturas y escrituras.
- `bloque-2/02-ficha-contexto.md` §4 y §6: supuestos S2 a S8, S13, S14, S19, S20 y S23 a S28.
- `docs/class-material/Tácticas y sizing · Bloque 3.html`: sección 3 de la plantilla y calculadora de la cadena (ley de Little).

**Qué fija esta fase:**
1. La cadena de jugadores registrados a operaciones de saldo por segundo en el pico.
2. El promedio del día frente al pico, lado a lado.
3. Las dimensiones que no son operaciones por segundo.
4. La proporción de lecturas y escrituras en la operación de saldo.

No entran límites de componentes: esos nacen en la fase 4. Esta fase no cambia ninguna cifra del Bloque 2; la recalcula y agrega un solo supuesto (S31).

---

## 1. Cadena de carga

Cada eslabón sale del anterior. La cuenta va completa en la columna "Cómo sale".

| # | Eslabón | Valor | Cómo sale | Tipo y origen |
|---|---------|-------|-----------|---------------|
| 1 | Cuentas registradas (año 3) | **350.000** | ~3 cuentas por cada jugador activo al mes | [S3 · supuesto] |
| 2 | Jugadores activos al mes | **120.000** | 350.000 / 3 ≈ 117.000, redondeado a la cifra de S2 | [S2 · supuesto] |
| 3 | Jugadores activos en un día promedio | **36.000** | 120.000 × 30 % | [S4 · supuesto] |
| 4 | Jugadores activos en el día pico de diseño | **~86.000** | 36.000 × 2,4 = 86.400. El día pico (quincena, fin de semana y promoción) es el día promedio multiplicado por el mismo factor ×2,4 de S4 | [S31 · supuesto] |
| 5 | Llegadas en la hora pico | **~16.000 jugadores/h** | 86.400 × 18,5 % = 16.000. La hora pico concentra la misma proporción de llegadas que en un día normal (cuenta en §2.2) | [S4][S31 · supuesto] |
| 6 | Concurrentes en el pico | **12.000** | Ley de Little: 16.000 / 3.600 s × 2.700 s = 4,44 llegadas/s × 2.700 s = 12.000 | [S4 · supuesto] |
| 7 | Operaciones de saldo por jugador conectado | **~0,25 por segundo** (~15 por minuto) | Promedio ponderado de la mezcla de juegos (§2.1): 2.992 / 12.000 = 0,249 | [S5][S6 · supuesto] |
| 8 | **Tasa del pico de la operación crítica** | **~3.000 operaciones de saldo/s** sostenidas | 12.000 × 0,249 /s = 2.992 /s | [S6 · supuesto] |
| 9 | Ráfaga dentro del pico | **~4.000 operaciones/s** durante segundos | Cota de S6 cuando coinciden las peticiones del *crash* (§4) | [S6 · supuesto] |

Todas las cifras de la cadena son supuestos: el proyecto no tiene cliente ni mediciones, y no hay datos públicos de concurrencia en el mercado colombiano (`bloque-2/01-investigacion-dominio.md` §6). Por la regla de la fase 1 (§5), toda cifra derivada toma el tipo de su insumo más débil.

---

## 2. Cómo se calculó la tasa del pico

### 2.1 De jugadores conectados a operaciones por segundo

Mezcla del pico [S5] y ritmo de cada juego [S6]. Cada jugada es 1 descuento + 1 abono = 2 operaciones de saldo [S6].

| Juego | Jugadores | Ritmo por jugador | Jugadas/s | Operaciones de saldo/s (promedio) |
|-------|-----------|-------------------|-----------|-----------------------------------|
| Tragamonedas | 12.000 × 55 % = 6.600 | 1 giro cada 5 s | 6.600 / 5 s = 1.320 | 1.320 × 2 = **2.640** |
| Ruleta | 12.000 × 15 % = 1.800 | 1 jugada cada 30 s | 1.800 / 30 s = 60 | 60 × 2 = **120** |
| *Crash* | 12.000 × 20 % = 2.400 | 1 ronda cada 25 s | 2.400 / 25 s = 96 | 96 × 2 = **192** |
| Blackjack en vivo | 12.000 × 10 % = 1.200 | 1 ronda cada 60 s | 1.200 / 60 s = 20 | 20 × 2 = **40** |
| **Total** | **12.000** | | **1.496** | **2.992 ≈ 3.000** |

```
operaciones por jugador = 2.992 operaciones/s ÷ 12.000 jugadores = 0,249 operaciones/s por jugador
                        ≈ 15 por minuto ≈ 670 por sesión de 45 min (0,249 × 2.700 s)
```

**Nota sobre S6.** La ficha llega a 3.300/s sumando el promedio de tragamonedas y ruleta con las ventanas de apuesta del *crash* (480/s) y del blackjack (80/s), y lo redondea a ~3.000. El promedio de los cuatro juegos da 2.992/s: **la cifra sostenida de S6 se confirma**. Las ventanas de apuesta no son el promedio, sino ráfagas; se tratan en §4.

### 2.2 Concurrentes: la ley de Little

```
concurrentes = llegadas por segundo × segundos de permanencia
```

La permanencia es la sesión promedio de 45 min = 2.700 s [S4].

**Día normal (contraste con S4):**
```
llegadas en la hora pico = 5.000 concurrentes ÷ 2.700 s = 1,85 llegadas/s
                         = 1,85 × 3.600 s ≈ 6.670 llegadas en la hora
proporción del día       = 6.670 ÷ 36.000 activos ≈ 18,5 %
```

**Día pico de diseño:**
```
llegadas en la hora pico = 86.400 activos × 18,5 % ≈ 16.000 llegadas en la hora
                         = 16.000 ÷ 3.600 s = 4,44 llegadas/s
concurrentes             = 4,44 llegadas/s × 2.700 s = 12.000 jugadores   ✔ coincide con S4
```

La cadena cierra: 86.400 activos en el día pico, con la misma forma del día normal, producen los 12.000 concurrentes del pico de diseño.

---

## 3. Promedio frente al pico

*Se diseña para el pico, no para el promedio* (plan, principio 4).

| Momento | Concurrentes | Cuenta | Operaciones de saldo/s |
|---------|--------------|--------|------------------------|
| Promedio de un día normal (24 h) | 36.000 × 2.700 s ÷ 86.400 s = **1.125** | 1.125 × 0,249 | **~280** |
| Promedio del día pico (24 h) | 86.400 × 2.700 s ÷ 86.400 s = **2.700** | 2.700 × 0,249 | **~670** |
| Pico normal (viernes o sábado, 20:00–24:00) | **5.000** [S4] | 5.000 × 0,249 | **~1.250** |
| **Pico de diseño** (quincena + promoción) | **12.000** [S4] | 12.000 × 0,249 | **~3.000** |
| Ráfaga dentro del pico | — | §4 | **~4.000** |

Volumen diario, misma cuenta:
```
día normal: 36.000 jugadores × 2.700 s × 0,249 /s ≈ 24 millones de operaciones   ✔ coincide con la ficha §4.5
día pico:   86.400 jugadores × 2.700 s × 0,249 /s ≈ 58 millones de operaciones
```

**Lectura:** el pico de diseño es **~11 veces** el promedio de un día normal (3.000 / 280) y **2,4 veces** el pico normal. Dimensionar con los 24 millones diarios repartidos en 24 h daría ~280/s: el sistema quedaría corto en un factor de 11 justo el día que más dinero mueve.

---

## 4. La ráfaga

S6 da ~4.000/s "cuando coinciden los retiros del *crash*". Recalculada segundo a segundo:

| Segundo | Cuenta | Operaciones/s |
|---------|--------|---------------|
| Ventana de apuesta del *crash* | 2.760 (tragamonedas y ruleta) + 480 (2.400 apuestas en 5 s) + 80 (ventana del blackjack) | ~3.320 |
| Retiro masivo del *crash* (ESC-04) | 2.760 + 720 abonos en el mismo segundo + 80 | ~3.560 |
| Cota de S6 | 3.300 + 720 | ~4.000 |

La cota de S6 suma en el mismo segundo las apuestas y los retiros del *crash*, que en una ronda compartida no coinciden: las apuestas cierran antes de que la ronda arranque. La ráfaga que se puede reconstruir llega a ~3.560/s, unos 440/s (≈ 12 %) por debajo de la cota.

**Propuesta: conservar ~4.000/s como cota de la ráfaga.** El exceso cubre coincidencias que S6 no contó, como el abono simultáneo de las 10 mesas de blackjack al cerrar una ronda (hasta 1.200 abonos). Diseñar para una ráfaga algo mayor no rompe nada, y cambiar S6 obligaría a propagar el cambio a la ficha, la priorización y la selección de escenarios. El equipo aprobó conservarla (§10).

---

## 5. Otras dimensiones que importan

No todo se mide en operaciones de saldo por segundo. Estas son las que usan las fases 4 a 8.

### 5.1 Volumen que se acumula

| Dimensión | Cuenta | Valor | Tipo y origen | Quién lo usa |
|-----------|--------|-------|---------------|--------------|
| Identificadores de transacción por conservar para conciliación (≥ 4 meses) | 24 M/día × 122 días | **~3.000 millones** | [S4][S6 · supuesto][F24 · referencia] | ADR-03: la búsqueda del identificador recorre este volumen |
| Historial de operaciones disponible (≥ 6 meses) | 24 M/día × 183 días | **~4.400 millones** | [S4][S6 · supuesto][F1 · referencia] | Línea base: el historial es donde se busca el identificador |
| Historial en un año | 24,2 M/día × 365 días | **~8.830 millones** (24,2 M × 365 = 8.834 M; la ficha §4.5 lo redondea a ~8.900) | [S4][S6 · supuesto] | Fase 4: crecimiento de la base única |

Se calcula con el día promedio de S4, que ya promedia los días pico del mes. El tamaño en bytes depende de cómo se guarde cada registro: es una cifra sobre la solución y nace en la fase donde se proponga el mecanismo.

### 5.2 Retiro y listas restrictivas (ADR-01)

| Dimensión | Cuenta | Valor | Tipo y origen |
|-----------|--------|-------|---------------|
| Ritmo mínimo de la reconsulta | 350.000 cuentas ÷ 21.600 s | **≥ 16,2 consultas/s** al proveedor de listas, durante ≤ 6 h | [S3][S19 · supuesto] |
| Carga de la reconsulta sobre la base única (línea base) | 1 lectura de la cuenta + 1 escritura de la marca, por cuenta | **≥ 16 lecturas/s y ≤ 16 escrituras/s** | [S3][S19 · supuesto] |
| Frecuencia de la reconsulta | — | Varias veces al mes | [S19 · supuesto] |
| Retiros al día | 12.000 depósitos ÷ 4 | **~3.000** (promedio ~125 por hora) | [S7][S8 · supuesto] |
| Retiros retenidos por una reconsulta (opción A de la línea base) | 3.000 × 6 h ÷ 24 h | **hasta ~750** | [S8][S19 · supuesto] |
| Depósitos en la hora pico | 12.000 × 15 % = 1.800 ÷ 3.600 s | **0,5 depósitos/s**, cada uno con su evaluación de riesgo | [S7 · supuesto] |

Frente a las ~3.000 operaciones de saldo por segundo, los depósitos (0,5/s) y la reconsulta (~16/s) son poco volumen. Su peso está en otra parte: la reconsulta depende de un proveedor externo y lee la misma base que el juego (`02` §5).

### 5.3 Presupuestos de tiempo

| Presupuesto | Valor | Tipo y origen |
|-------------|-------|---------------|
| Operación de saldo completa, medida en el operador | p99 ≤ 250 ms | [S14 · supuesto] |
| Espera del proveedor antes de dar la operación por fallida | ~3 s | [S14 · supuesto][F24 · referencia] |
| Verificación del origen de la petición (ADR-04) | ≤ 20 ms (p99) de los 250 ms; quedan 230 ms para el resto | [S26 · supuesto] |
| Saldo visible al jugador después de un abono | p95 ≤ 2 s | [S23 · supuesto] |
| Evaluación de riesgo de un retiro o depósito | p95 ≤ 1 s | [S20 · supuesto] |
| Alerta técnica o de seguridad por una petición anómala | ≤ 5 min | [S25][S28 · supuesto] |
| Vuelta a p99 ≤ 250 ms después de restablecer el servicio | ≤ 5 min | [S24 · supuesto] |
| Recuperación de juego y saldo | ≤ 15 min | [S13 · supuesto] |

### 5.4 Errores tolerados en el pico

```
peticiones legítimas rechazadas: ≤ 0,001 % [S27]
  en el pico:  3.000 /s × 0,00001 = 0,03 rechazos/s ≈ 108 por hora
  en el día:   24 M × 0,00001 ≈ 240 por día   ✔ coincide con S27
```

---

## 6. Lecturas y escrituras en la operación crítica

Accesos a la base única por cada operación de saldo, según los pasos de la línea base (`02` §3.1):

| Paso de la línea base | Lecturas | Escrituras | ¿En el camino de la respuesta? |
|-----------------------|----------|------------|--------------------------------|
| 4c · Buscar el identificador en el historial | 1 | — | Sí |
| 5 · Comprobar que el saldo alcanza | 1 | — | Sí |
| 5 · Cambiar el saldo | — | 1 | Sí |
| 5 · Anotar la operación en el historial | — | 1 | Sí |
| 7 · Copiar la operación al sistema alterno | 1 | — | No |
| 8 · Extraerla para el sistema de inspección | 1 | — | No |
| **Total por operación** | **4** | **2** | |

```
en el camino de la respuesta: 2 lecturas : 2 escrituras  →  1 : 1
contando los procesos de fondo: 4 lecturas : 2 escrituras  →  2 : 1

en el pico (× 3.000 operaciones/s):
  lecturas:   4 × 3.000 = 12.000 /s   (6.000 en el camino de la respuesta)
  escrituras: 2 × 3.000 =  6.000 /s   en 3.000 transacciones/s, cada una grabada en disco antes de responder
```

**Lectura:** la operación crítica **escribe siempre**. A diferencia de los casos del material, donde predominan las lecturas, aquí cada una de las ~3.000 operaciones por segundo termina en una transacción que se graba en disco antes de responder. Las lecturas se pueden repartir; las escrituras del saldo no se pueden evitar sin cambiar lo que significa "confirmada". Esto orienta la fase 4: el primer candidato es la escritura en la base única.

Una petición repetida (ESC-03) solo lee: encuentra el identificador y reconstruye la respuesta. No hay una cifra de cuántas peticiones llegan repetidas; con cualquier proporción pequeña, la cuenta no cambia de forma apreciable.

---

## 7. Cifra nueva

| ID | Supuesto | Razón |
|----|----------|-------|
| S31 | El día pico de diseño (quincena, fin de semana y promoción) tiene la forma del día promedio multiplicada por **×2,4**: **~86.000 jugadores activos** (86.400) y sesiones de 45 min | Es el mismo factor de S4, aplicado al día completo y no solo a la hora pico; así la cadena llega a los 12.000 concurrentes sin cifras adicionales. Equivale al 72 % de los activos del mes jugando ese día, alto pero coherente con una promoción en día de pago. **No cambia la tasa del pico**, que depende de los 12.000 concurrentes de S4; solo fija el volumen del día pico (~58 M de operaciones). Agregado el 13 de noviembre de 2026. |

**Variante de S31.** El día pico tiene ~60.000 activos (50 % del mes) con sesiones más largas, de ~65 min (60.000 × 18,5 % ÷ 3.600 s × 3.900 s ≈ 12.000). Llega a los mismos 12.000 concurrentes y casi al mismo volumen diario (60.000 × 3.900 s × 0,249 ≈ 58 M), pero necesita una segunda cifra nueva: la duración de la sesión en el día pico.

---

## 8. Texto para la plantilla (sección 3)

*Para transcribir en la fase 10.*

**Cadena de carga:**
- Usuarios registrados: 350.000 cuentas | tipo: supuesto | razón o fuente: ~3 cuentas por jugador activo al mes por el abandono típico de cuentas (S3).
- Usuarios activos en el día pico: ~86.000 | tipo: supuesto | razón o fuente: 120.000 activos al mes (S2) × 30 % que juega cada día (S4) = 36.000 en un día promedio; el día pico de diseño (quincena + promoción) lo multiplica por 2,4 (S4, S31).
- Usuarios concurrentes en el pico: 12.000 | tipo: supuesto | razón o fuente: ley de Little con 16.000 llegadas en la hora pico (18,5 % del día, la misma proporción de un día normal) y sesiones de 45 min (S4).
- Operaciones críticas por usuario durante el pico: ~0,25 operaciones de saldo por segundo (~15 por minuto) | tipo: supuesto | razón o fuente: mezcla de juegos (S5) y ritmo por juego, con 1 descuento + 1 abono por jugada (S6).
- Tasa en el pico de la operación crítica (por segundo): ~3.000 operaciones de saldo/s sostenidas, ráfagas de ~4.000/s | tipo: supuesto | razón o fuente: 12.000 × 0,249 /s = 2.992 /s (S6).

**Cómo se calculó la tasa en el pico (operaciones, con unidades):**
```
llegadas en la hora pico = 86.400 activos × 18,5 % ≈ 16.000 llegadas/h = 4,44 llegadas/s
concurrentes             = 4,44 llegadas/s × 2.700 s de sesión = 12.000 jugadores
tasa                     = tragamonedas 6.600 / 5 s × 2 + ruleta 1.800 / 30 s × 2
                           + crash 2.400 / 25 s × 2 + blackjack 1.200 / 60 s × 2
                         = 2.640 + 120 + 192 + 40 = 2.992 ≈ 3.000 operaciones de saldo/s
promedio de un día normal = 24 M operaciones ÷ 86.400 s ≈ 280 /s  →  el pico es ~11 veces el promedio
```

**Otra dimensión que importa en este sistema:** volumen de identificadores de transacción por conservar: ~3.000 millones (24 M/día × 4 meses; supuestos S4 y S6, referencia F24). Reconsulta de listas restrictivas: ≥ 16,2 consultas/s durante ≤ 6 h, varias veces al mes, contra un proveedor externo (supuestos S3 y S19). Presupuesto de tiempo de la operación de saldo: p99 ≤ 250 ms, de los cuales ≤ 20 ms son para verificar el origen de la petición (supuestos S14 y S26).

**Proporción de lecturas y escrituras en la operación crítica:** 1 : 1 en el camino de la respuesta (2 lecturas y 2 escrituras por operación); 2 : 1 contando la copia al sistema alterno y la extracción para Coljuegos. Toda operación escribe: 6.000 escrituras/s en 3.000 transacciones/s en el pico.

**Dónde está esto en la entrega:** presentación HTML, sección "Perfil de carga" (se integra en la fase 10), y `bloque-3/03-perfil-de-carga.md`.

---

## 9. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| La tasa del pico sale de una cuenta con unidades | ✔ §2.1 y §2.2: jugadores → llegadas/s → concurrentes → operaciones/s |
| Cada número de la cadena tiene tipo y origen | ✔ §1: S2 a S6 y S31, todos supuestos; F1 y F24 en §5 |
| Se distingue el promedio del pico | ✔ §3: 280/s, 1.250/s, 3.000/s y 4.000/s lado a lado |
| Las cifras derivadas cuadran con la ficha | ✔ 2.992 ≈ 3.000/s (S6); 24 M al día (ficha §4.5); 16,2 consultas/s (S19); 240 rechazos al día (S27); 12.000 concurrentes (S4) |
| La ráfaga se revisó | ✔ §4: reconstruible hasta ~3.560/s; la cota de S6 (~4.000/s) se conserva por decisión del equipo |
| Hay otras dimensiones, con tipo | ✔ §5: volumen, reconsulta, retiros, presupuestos de tiempo y errores tolerados |
| Proporción de lecturas y escrituras | ✔ §6 |
| No se nombran tácticas, tecnologías ni marcas | ✔ Solo los pasos de la línea base |
| Cifras nuevas registradas con razón | ✔ S31 en §7 |
| No hay límites de componentes | ✔ Quedan para la fase 4 |

**Observación para la fase 9.** Los 720 retiros del *crash* en un mismo segundo son el 30 % de los 2.400 jugadores del *crash* (S5). Ese 30 % no está escrito en ningún supuesto; `bloque-2/08-revision.md` traza la cifra a S5, S6 y S14. Se anota como candidato a hallazgo, sin cambiarlo aquí. *Resuelto en la fase 9: es el supuesto S44 de la ficha (`09-revision.md`, H10).*

---

## 10. Punto de control (cerrado el 13 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las tres recomendaciones.

1. **Activos en el día pico: S31 aprobado**, ~86.000 (el día promedio × 2,4). La variante de ~60.000 con sesiones de ~65 min queda registrada (§7).
2. **Ráfaga: se conserva la cota de ~4.000/s de S6** como cifra de diseño. No se cambia la ficha; la reconstrucción de ~3.560/s queda como sustento de que la cota es conservadora (§4).
3. **Unidad del componente crítico para la fase 4:** transacciones de saldo por segundo sobre la base única (3.000/s en el pico, 6.000 escrituras/s), con las lecturas de fondo como carga aparte (§6).

Las cuentas de §2 y §3 se pueden contrastar con la calculadora del material: 16.000 personas en una ventana de 60 min, 0,249 × 2.700 operaciones por persona, da ~2.990 operaciones/s.

La fase 4 (cuello de botella) puede arrancar.
