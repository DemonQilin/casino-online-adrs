# Fase 9 — Revisión integral

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 10 (Lo que el equipo reconoce y autochequeo)
**Criterio de la rúbrica:** los nueve criterios de pregrado. Esta fase no tiene criterio propio: comprueba que los otros alcanzan el nivel "Logrado".
**Documentos revisados:** `00` a `08` del Bloque 3, y `bloque-2/02-ficha-contexto.md`, `bloque-2/06-escenarios.md` y `bloque-2/07-adrs.md` como insumos.
**Fecha:** 15 de noviembre de 2026

**Método:**
1. **Autoevaluación (9.1):** cada criterio de la rúbrica de pregrado, con la evidencia que lo sostiene y el riesgo de que un evaluador lo lea distinto.
2. **Coherencia en cadena (9.2):**
   - se recalcularon con un script las 154 cifras derivadas de las fases 1 a 8;
   - cada cifra de los ADR se rastreó hasta su `F#` o `S#`;
   - se verificó, también con un script, que el Context de cada ADR sea idéntico a su escenario;
   - se cotejaron los nombres de las tácticas y sus familias con el catálogo del curso (`PedalVa Tácticas.html`, constante de tácticas).
3. **Autochequeo de la plantilla (9.3):** los 10 puntos de la sección 10.
4. **Observaciones pendientes:** las que dejaron las fases 3, 7 y 8 para esta fase se resolvieron todas: se corrigieron o pasaron a la lista de riesgos.

---

## Resumen

- **12 hallazgos (H10 a H21), ninguno de severidad alta.** La numeración sigue la del Bloque 2, que llegó a H9. Los doce están corregidos. H10 y H11 cambiaban texto aprobado y se aplicaron con la decisión del equipo (sección 8).
- **Cifras:** las 154 cifras derivadas cuadran después de las correcciones. Antes no cuadraban dos: el uso de la red en la ráfaga (H15) y el redondeo del historial anual (H14). Ninguna conclusión ni ningún margen cambia.
- **Cifras sin origen:** dos no tenían `S#`. Una es del Bloque 2, los 720 retiros del *crash* (H10, ahora S44 en la ficha); la otra es de la fase 8, el costo de rechazar una petición (H12, ahora S45).
- **Context de los ADR:** las 7 filas de cada uno de los 4 ADR aparecen textuales en `bloque-2/06-escenarios.md`.
- **Catálogo:** los diez nombres de táctica coinciden con el catálogo. La familia de dos tácticas estaba mal escrita (H16).
- **Autoevaluación:** los 9 criterios se defienden en nivel Logrado. "Efecto estimado" quedó en riesgo bajo después de corregir TAC-10 (H11). Quedan 8 riesgos de interpretación con su defensa (sección 4).

---

## 1. Autoevaluación contra la rúbrica (tarea 9.1)

| # | Criterio | Nivel que reclamamos | Evidencia | Riesgo de lectura |
|---|----------|:--------------------:|-----------|-------------------|
| 1 | Línea base y cuello de botella | Logrado | `02` §2–§3 (línea base sin tácticas, pasos en orden) y §4 (piso regulatorio separado de las tácticas); `04` §2 (límite de cada candidato con fuente: F34–F38, S32–S36), §3 (uso 0,30 en la base y 3,24 en el proveedor de listas), §5 (aguanta ~40.000 jugadores) y §6 (qué ven jugador, proveedor y regulador) | **Medio:** el componente "que se satura primero" no se satura en el pico (R10) |
| 2 | Perfil de carga | Logrado | `03` §1 (cadena de 350.000 cuentas a 3.000 operaciones/s, cada eslabón con tipo y `S#`), §2 (cuenta con unidades y ley de Little), §3 (promedio y pico lado a lado: 280/s frente a 3.000/s), §5 (otras dimensiones) y §6 (lecturas y escrituras) | Bajo, después de H10 |
| 3 | Decision | Logrado | `06` §2–§5: "Usaremos…" en los cuatro ADR; cada opción descartada se contrasta con la medida y con un número | Bajo |
| 4 | Tácticas como mecanismos | Logrado | `05`: diez fichas con nombre del catálogo, componente, dato y paso de `02` §3.1 o §3.2 | Bajo, después de H16 |
| 5 | Efecto estimado | Logrado | `05` §6: cada táctica con métrica, valor antes y después, cuenta y tipo de la estimación | Bajo, después de H11: TAC-10 se compara con "sin la táctica" y la duración se atribuye a TAC-09 y TAC-10 juntas |
| 6 | Costo cuantificado | Logrado | `05`: USD al año, ms por operación, GB, núcleos, semanas de laboratorio, alertas al día; cada costo con su atributo | Bajo |
| 7 | Hasta dónde | Logrado | `06` (límite calculado, siguiente componente, ¿se cumple?, condición de revisión con umbral en los cuatro ADR) y `07` §6 (margen 3,3 en tasa, 2,5 en la ráfaga y 1,8 en volumen) | **Medio:** en ADR-02 y ADR-03 el "siguiente" componente es la misma base (R9) |
| 8 | Interacción entre tácticas | Logrado | `07` §2 (matriz de 10 tácticas × 5 escenarios, con ESC-04), §3 (presupuestos compartidos) y §4 (I1 a I6, con magnitud y resolución) | Bajo |
| 9 | Extensión | Logrado | `08`: X1 copiada tal cual; 15.000/s con la cadena de la fase 3; qué se sostiene, qué se rompe y en qué valor (TAC-03 desde ~7.600/s); qué ve cada actor; respuesta que no es "más servidores"; ADR-05 nuevo | **Medio:** la lectura literal exige 1,25 operaciones/s por jugador (R11) y el rechazo por carga no tiene fuente (R12) |

La sensibilidad a los supuestos es solo de posgrado y no se evalúa.

---

## 2. Coherencia en cadena (tarea 9.2)

### 2.1 Cifras derivadas

Se recalcularon **154** cifras: la cadena de carga y la ley de Little (fase 3), los límites, usos y jugadores que aguanta cada candidato (fase 4), el efecto y el costo de las diez tácticas (fase 5), los márgenes y las condiciones de revisión (fase 6), los presupuestos compartidos y las interacciones (fase 7) y la extensión X1 completa (fase 8). **Todas cuadran** después de corregir H14 y H15.

### 2.2 Cifras de los ADR rastreadas a su origen

| ADR | Cifras | Origen |
|-----|--------|--------|
| ADR-01 | 50 cuentas/s; 1.080.000 cuentas en 6 h; ≤ 10 ms; ≤ 1 min; 500 MB; 5 consultas/s; 19,4 h; 2.430 retiros | S42, S19, S3, F37, `04` §2.7 |
| ADR-02 | ~10 ms al alterno; 43,2 min al mes; ~3.000 perdidas; 10–30 min; ≤ 30 s; ~1 ms; USD 24.000 | S38, F39, S39, S13, S37, S40, F40, F41, S9 |
| ADR-03 | 3.000 inserciones/s; ≤ 1 ms; ~300 GB; 2,5 TB; 4 meses; 240 al día | S6, S41, S36, F24, S27 |
| ADR-04 | 0,045 ms; 20 ms; 67 % → 100 %; ≤ 1 min; 2–4 semanas; 240 al día | F36, S26, S42, S43, S10, S27 |

Todas tienen origen. Fuera de los ADR había dos cifras sin `S#`: los 720 retiros del *crash* (H10) y el costo de rechazar una petición en la fase 8 (H12).

### 2.3 Context de los ADR frente a su escenario

Un script tomó las 7 filas del Context de cada ADR en `06` (atributo, fuente, estímulo, artefacto, entorno, respuesta y medida) y buscó cada una, textual, en `bloque-2/06-escenarios.md`. **Las 28 filas aparecen sin cambios.**

### 2.4 Catálogo y marcas

- **Nombres de táctica:** los diez de `05` y los dos de la respuesta de X1 (Limitar la tasa de eventos, Priorizar eventos) están en el catálogo con ese nombre exacto.
- **Familias:** coinciden, salvo "Recuperarse de fallas" en TAC-01 y TAC-03; en el catálogo la familia se llama "Recuperarse" (H16).
- **Marcas:** los nombres de productos solo aparecen en las listas de fuentes (`04` §7.1 y `05` §7.1). En el análisis de la fase 4 aparecen "TPC-B" (nombre de una carga de prueba estándar) y "HTTP/1.1" (el protocolo que fija las conexiones persistentes). No están en ningún mecanismo ni en ninguna Decision, así que se dejan.

---

## 3. Hallazgos y correcciones

| # | Severidad | Hallazgo | Corrección | Archivos |
|---|-----------|----------|-----------|----------|
| **H10** | Media | Los **720 retiros del *crash* en un mismo segundo** (30 % de los 2.400 jugadores del *crash*) aparecen en la ficha (§1 y §3) y en ESC-04, pero ningún supuesto los define. `bloque-2/08-revision.md` los trazó a S5 y S6, que no derivan esa cifra. Lo anotó la fase 3 (`03` §9). | **Aplicada (decisión D1):** se agregó el supuesto S44 a la ficha §6 ("hasta el 30 % de los jugadores de una ronda de *crash* se retira en el mismo segundo, en un multiplicador redondo como 2,00x"), se citó en la trazabilidad de ESC-04 y en la constante `SUPS` de la presentación del Bloque 2, y se registró con fecha en `bloque-2/08-revision.md` §9 y en `01` §3.1 (cambios desde el Bloque 2). La presentación del Bloque 2 pasa de 29 a 30 supuestos. | `bloque-2/02`, `bloque-2/06`, `bloque-2/08`, `bloque-2/Casino Andino.html`, `01`, `03` |
| **H11** | Media | En TAC-10, el valor "en la línea base" no es la línea base. (1) La reconsulta pasa de ~19,4 h a ~1,9 h sobre todo por TAC-09: con la lista propia la reconsulta deja de esperar al proveedor, y el tope de TAC-10 la hace **más lenta** que sin tope. (2) El uso de ~1,0 es el de la opción C sin tope; frente a la línea base real, la reconsulta sube el uso de la base de 0,30 a 0,31. | **Aplicada (decisión D2):** el valor de comparación pasa a ser "sin la táctica (opción C sin ritmo máximo)". La métrica principal de TAC-10 pasa a ser el uso de la base durante la reconsulta (~1,0 → 0,31). La duración (~19,4 h → ~1,9 h) se declara como efecto conjunto de TAC-09 y TAC-10. Las tablas resumen y la matriz se ajustan igual. | `05` TAC-10 y §6, `06` ADR-01, `07` §2 |
| **H12** | Media | En `08` §8.2, el costo de rechazar una petición (c = 0,5 a 1 ms) no tenía origen. Además, el número S44 estaba mencionado como hipótesis en `06` §9, y "S42 a S45" en `05` §7.2, para variantes que no se adoptaron. | Se registra **S45** en `08` §12 (0,5 ms, con 1 ms como extremo pesimista, y su razón). Se cita en `08` §8.2 y en la verificación. Las menciones hipotéticas se reescriben como "un supuesto nuevo" y "cuatro supuestos", para que S44 y S45 tengan un solo significado. | `08`, `06` §9, `05` §7.2 |
| **H13** | Media | La fase 7 decidió que el tope de 240 alertas falsas al día (S27) es **común** a TAC-05 y TAC-07 (`07` §9, decisión 2), pero ADR-03 y ADR-04 seguían diciendo "hasta 240" cada uno. | En el "Paga" y en la condición de revisión (b) de los dos ADR: "240 al día entre ADR-03 y ADR-04, tope común de S27". | `06` ADR-03 y ADR-04 |
| **H14** | Baja | El historial de un año decía ~8.850 M. La cuenta da 36.000 × 2.700 s × 0,249 × 365 = 8.834 M. | Se cambia a ~8.830 M. Los ~2,2 TB por año no cambian. | `03` §5.1, `04` §2.2 y §9 |
| **H15** | Baja | En la tabla de usos, la red pública en la ráfaga figuraba con ~0,30. La cuenta da (4.000 × 2 KB + 1.200 × 20 KB) × 8 = 256 Mbit/s, es decir **0,26**. | Corregido. La red sigue lejos de saturarse. | `04` §3 |
| **H16** | Baja | La familia de TAC-01 y TAC-03 aparecía como "Recuperarse de fallas"; en el catálogo se llama "Recuperarse". | Corregido. | `05` |
| **H17** | Baja | Los cuatro ADR decían "Aceptado (13 de noviembre de 2026)", pero el punto de control de la fase 6 y la nota en `bloque-2/07-adrs.md` son del 15 de noviembre. El 13 se eligieron las opciones; el 15 se aceptaron los ADR. | Se cambia la fecha a 15 de noviembre de 2026. | `06` |
| **H18** | Baja | "El servidor de aplicación se multiplica sin tocar ningún ADR" deja de ser cierto con TAC-09: la lista vigente de ADR-01 vive en ese servidor (`07` I6). En `04` §3 la frase es correcta, porque describe la línea base. | En `06` ADR-04 y `08` §4 se precisa que multiplicarlo no toca los ADR de la operación de saldo, pero obliga a replicar la lista de ADR-01. | `06`, `08` |
| **H19** | Baja | La vuelta a p99 ≤ 250 ms de ADR-02 ("≈ 13 s") cuenta una copia por operación. Con 3 copias, el estímulo de ESC-03, son ~39 s (`07` I1). | Se agrega el caso de 3 copias. La medida (≤ 5 min) se sigue cumpliendo. | `06` ADR-02 |
| **H20** | Baja | `05` §5 sumaba "+~2 ms" al tiempo del saldo sin la pieza de TAC-06 (< 0,1 ms); `07` §3.1 da 2,15 ms. | Se cambia a +~2,15 ms. | `05` §5 |
| **H21** | Baja | `08` §3 escribía "(supuesto: la ráfaga escala igual)" sin `S#`. No es un supuesto nuevo, sino la lectura de X1: la ráfaga es parte del pico. | Se reescribe como lectura de X1. | `08` §3 |

**Fases cerradas que se tocan:** `01`, `03`, `04`, `05`, `06`, `07` y `08` del Bloque 3, y `02`, `06`, `08` y la presentación del Bloque 2. Ninguna corrección cambia una decisión, un límite ni un margen.

---

## 4. Riesgos para la sustentación

No son errores, pero un evaluador podría leerlos distinto. Continúan la numeración del Bloque 2 (R1 a R4) y de la fase 1 (R5).

| # | Riesgo | Defensa |
|---|--------|---------|
| R6 | **Las copias por operación durante un cambio de servidor** dependen de la política de reintentos de cada proveedor, que no está en las fuentes. De ese número dependen los ~39 s de I1, el ~7.600/s de la escalera de X1 y el margen de 1,07 del tope de 7.500/s. | Se usa 3, el estímulo de ESC-03, y se muestra la holgura: hacen falta 23 copias por operación para agotar los 5 min (`07` I1). Es el número que habría que pedir a cada proveedor. |
| R7 | **F37 (5 consultas/s)** sale del foro de desarrolladores del proveedor de listas, no de su documentación formal, y otra respuesta del mismo foro dice 1/s. | Es la mejor cifra pública. Con cualquiera de los dos valores, la línea base incumple las 6 h, y con la opción C el proveedor deja de estar en el camino (uso 0,002). |
| R8 | **S33 (1 ms por petición)** no tiene referencia pública. | Con los cuatro componentes del cálculo, el servidor de aplicación aguanta ~44.400 jugadores y la base ~40.000: aunque S33 fuera 10 % mayor, la base seguiría saturándose primero. |
| R9 | En ADR-02 y ADR-03, **el "siguiente componente que se satura" es la misma base única**. Alguien podría esperar otro componente. | Es el resultado: las tácticas de esos ADR compran garantías, no capacidad (`07` §6.2). Detrás de la base está el servidor de aplicación (~44.400 jugadores) y, en volumen, el tope de 2,5 TB (margen 1,8). |
| R10 | En la sección 4 de la plantilla (variante A), **el componente "que se satura primero" no se satura en el pico** (uso 0,30). | El material permite decirlo y buscar el límite en otra dimensión. Esa dimensión es el proveedor de listas, con uso 3,24 (`04` §4). |
| R11 | **La lectura literal de X1** exige 1,25 operaciones/s por jugador, tres veces el juego más rápido. | Es lo que dice el texto. `08` §4 muestra al lado la lectura de 60.000 jugadores, y la base se satura igual en las dos. |
| R12 | **El rechazo explícito por carga** que propone `08` §8 no está documentado por los proveedores: F24 solo documenta el descuento sin respuesta. | Un rechazo en milisegundos no puede ser peor que un silencio de ~3 s, pero hay que confirmarlo con cada proveedor (`08` §8.3). |
| R13 | **ADR-05 sacrifica Adecuación funcional (2)**, que no es alta. Con el criterio del Bloque 2, su escenario no sería crítico. | La plantilla pregunta por la decisión, no por la criticidad. El equipo decidió declararlo y esbozarlo sin formalizarlo (`08` §11, decisión 4). |

---

## 5. Autochequeo de la plantilla (tarea 9.3)

| Punto del autochequeo | ¿Se cumple? | Evidencia |
|-----------------------|:-----------:|-----------|
| Cada número del perfil de carga dice si es dato, supuesto o referencia, y tiene razón o fuente | ✔ | `03` §1 y §8; `01` §5 define los tipos. Con H10, los 720 retiros también |
| La tasa del pico sale de una cuenta que se puede seguir, con unidades | ✔ | `03` §2: 4,44 llegadas/s × 2.700 s = 12.000; 12.000 × 0,249 /s = 2.992 /s |
| Identificamos el componente que se satura primero en la línea base y con qué carga | ✔ | `04` §3–§5: base única, uso 0,30, ~10.000/s (~40.000 jugadores); proveedor de listas, uso 3,24 |
| Cada ADR tiene Decision en voz activa y explica por qué descarta las demás opciones | ✔ | `06` §2–§5 |
| Cada táctica se nombra con el catálogo y se describe como un mecanismo concreto de nuestro sistema | ✔ | `05` (fichas) y §2.4 de esta fase |
| Cada táctica dice qué métrica mueve, de cuánto a cuánto y cómo lo estimamos | ✔ | `05` §6, con TAC-10 corregido (H11) |
| Cada táctica tiene un costo cuantificado, no solo un adjetivo | ✔ | `05`, campo "Costo cuantificado" |
| Cada ADR dice hasta dónde llegan sus tácticas y cuál es el siguiente cuello de botella | ✔ | `06`, "Hasta dónde llegan estas tácticas" en los cuatro ADR |
| Cada ADR tiene una condición de revisión medible | ✔ | `06`, Consequences: umbrales en cuentas, operaciones/s, cambios al mes, TB, % y ms |
| Aplicamos la extensión a nuestros propios números | ✔ | `08` §3: 5 × 3.000 = 15.000/s, con los límites de las fases 4 y 7 |

---

## 6. Lo que el equipo reconoce (tarea 9.4)

*Esta sección la escribe el equipo (plan, tarea 9.4). No se redacta por él. Abajo están los insumos que dejaron las fases, para que el equipo elija y escriba con sus palabras.*

**Números de los que menos estamos seguros, y por qué:**
[Pendiente del equipo]

**Lo que más discutimos como equipo y no resolvimos:**
[Pendiente del equipo]

**Insumos (números que las propias fases señalaron como débiles):**

| Número | Dónde | Por qué es débil |
|--------|-------|------------------|
| Copias por operación en un cambio de servidor | `07` I1, `08` §5 y §8 | Depende de la política de reintentos de cada proveedor; sin fuente (R6) |
| S33 · 1 ms por petición | `04` §2.4 | Sin referencia pública; la fase 4 lo llamó el supuesto más débil |
| S36 · −30 % sobre la prueba de referencia | `04` §2.1 | El límite viene de otro sistema; el descuento es un juicio, no una medida |
| F37 · 5 consultas/s | `04` §2.7 | Foro, no documentación formal; otra respuesta dice 1/s (R7) |
| S37 · 1 s de atraso de la copia | `05` §7.2 | De él depende "~3.000 operaciones perdidas" en la línea base |
| S40 · 10–30 min del cambio manual | `05` §7.2 | Estimación del procedimiento de la línea base |
| S45 · costo de rechazar una petición | `08` §8.2 y §12 | Sin referencia; el servidor cabe hasta 0,93 ms |
| Rechazo explícito por carga | `08` §8.3 | Ningún proveedor documenta cómo lo trata (R12) |

Decisiones que el equipo tomó con variantes registradas y que podrían salir en la discusión: la tensión dominante (T2 frente a T1 y T7, `01` §1), la variante A del cuello de botella (`04` §4), la lectura literal de X1 (`08` §11) y el alcance de ADR-05 (`08` §11).

---

## 7. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| Los 9 criterios de pregrado tienen nivel y evidencia | ✔ §1 |
| Las cifras derivadas se recalcularon y cuadran | ✔ 154 cifras, 0 diferencias después de H14 y H15 |
| Cada cifra de un ADR se rastrea hasta una `F#` o un `S#` | ✔ §2.2; las dos cifras sin origen fuera de los ADR son H10 y H12 |
| El Context de cada ADR coincide con su escenario | ✔ §2.3: 28 de 28 filas |
| Nombres de táctica y familias del catálogo | ✔ §2.4, después de H16 |
| Ninguna marca en un mecanismo, una Decision o un escenario | ✔ §2.4 |
| La priorización suma 18 y respeta los límites | ✔ No cambió desde el 13 de noviembre (`01` §3.1): 4 + 3 + 3 + 3 + 2 + 1 + 1 + 1 = 18, un solo atributo en 4 |
| Los 10 puntos del autochequeo están marcados con evidencia | ✔ §5 |
| Las observaciones que las fases 3, 7 y 8 dejaron para la fase 9 están resueltas | ✔ `03` §9 → H10; `07` §8 → H13, H18, H19, H20 y R6; `08` §10 → H12, R6, R11, R12 y R13 (la 5 y la 6, latido bajo saturación e ingreso por jugada, se mantienen como riesgos razonados en `08`) |
| Cada `S#` y `H#` nuevo es único | ✔ S44 definido solo en la ficha del Bloque 2 y S45 solo en `08` §12; H10 a H21 sin repetir los H1 a H9 del Bloque 2 |
| La presentación HTML dice lo mismo que los markdown | Bloque 2: ✔ la presentación incluye S44 y cuenta 30 supuestos. Bloque 3: pendiente, se integra en la fase 10 |

---

## 8. Punto de control (cerrado el 15 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las cuatro recomendaciones.

| # | Decisión | Aprobado | Variantes registradas |
|---|----------|---------------|-----------|
| D1 | Origen de los 720 retiros del *crash* (H10) | **S44 en la ficha del Bloque 2**, con la trazabilidad de ESC-04, la constante `SUPS` de la presentación del Bloque 2 y una nota fechada en `bloque-2/08-revision.md`. Es lo que pide la regla del proyecto: una cifra de requisito de un escenario va a la ficha | Registrarlo en `03` §7 como supuesto del Bloque 3 (no toca el Bloque 2, pero pone una cifra de requisito fuera de la ficha); o dejarlo como riesgo, con la defensa de que la ráfaga de ~4.000/s ya lo cubre |
| D2 | Cómo se presenta el efecto de TAC-10 (H11) | **Cambiar el nombre del valor de comparación** a "sin la táctica (opción C sin ritmo máximo)": métrica principal, uso de la base ~1,0 → 0,31; la duración como efecto conjunto de TAC-09 y TAC-10 | Dejar el texto como está y defenderlo en la sustentación |
| D3 | Sección 6, "Lo que el equipo reconoce" | **El equipo la escribe** con los insumos de §6, antes de la fase 10 | — |
| D4 | Campo `caso` de la plantilla (pendiente desde `00` §8) | **"Proyecto final (pregrado)"**, definido por el equipo el 15 de noviembre de 2026: Casino Andino es el proyecto final de la materia | "Proyecto propio" |

D1 y D2 están aplicadas (sección 3). El campo `caso` quedó definido (D4). Antes de la fase 10, el equipo escribe la sección 6 (D3).

La fase 10 (integración) puede arrancar.
