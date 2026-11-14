# Fase 1 — Punto de partida

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 1 (Punto de partida)
**Criterio de la rúbrica:** base de todos. Esta fase no tiene criterio propio, pero las convenciones que fija (operación crítica y tipos de cifra) son las que evalúa el criterio "Perfil de carga".

**Insumos:**
- `bloque-2/03-mapa-tensiones.md`: tensiones fuertes T1 a T7.
- `bloque-2/04-priorizacion.md` y `bloque-2/05-seleccion-escenarios.md`: reparto de puntos y escenarios críticos.
- `bloque-2/06-escenarios.md` y `bloque-2/07-adrs.md`: escenarios y ADR en estado Propuesto.
- `bloque-2/08-revision.md` §7 y §8: cambios posteriores a la revisión.
- `bloque-2/02-ficha-contexto.md` §4 y §6: cifras y supuestos citados aquí.

**Qué fija esta fase:**
1. La tensión dominante que encabeza la entrega.
2. Los escenarios críticos y sus ADR, sin cambios respecto al Bloque 2.
3. Lo que cambió desde la entrega del Bloque 2.
4. La operación crítica sobre la que se hace el sizing.
5. Cómo se marca cada cifra: dato, supuesto o referencia.

Las cifras se marcan con la convención de la sección 5 de este archivo.

---

## 1. Tensión dominante

La plantilla pide una sola tensión, en la forma "En este sistema, ___ choca con ___ porque ___". El Bloque 2 tiene cuatro tensiones fuertes entre atributos altos (T1, T2, T3 y T7). Hay que escoger la que mejor resume la entrega del Bloque 3.

### Propuesta: T2, Fiabilidad × Eficiencia de desempeño

> En este sistema, **garantizar que ninguna operación de saldo confirmada se pierda** choca con **responder cada operación en p99 ≤ 250 ms con ~3.000 operaciones por segundo en el pico**, porque **asegurar cada operación, también en el sistema alterno, antes de confirmarla alarga cada respuesta y reduce cuántas caben por segundo.**

Cifras: pérdida cero [S13 · supuesto][F1 · referencia], p99 ≤ 250 ms [S14 · supuesto], ~3.000 operaciones por segundo [S6 · supuesto], sistema alterno al día de inmediato [F1 · referencia].

**Por qué esta:**
- Vive en la operación de saldo, la misma sobre la que se hace el sizing (sección 4). La tensión dominante y la cadena de carga hablan del mismo número.
- Es la tensión que más se tensa con la carga: a más operaciones por segundo, más cuesta asegurar cada una. Las otras tres no dependen tanto del volumen.
- La tocan dos ADR: ADR-02 la resuelve de frente y ADR-03 paga en Fiabilidad.

**En contra:** no incluye la Seguridad, el único atributo con 4 puntos.

### Variantes

| Variante | Frase | A favor | En contra |
|----------|-------|---------|-----------|
| **T1**, Seguridad × Desempeño | En este sistema, **evaluar cada depósito y retiro contra el perfil de riesgo y las listas restrictivas** choca con **confirmar a tiempo y sostener el pico de juego**, porque **cada control añade tiempo a la operación y la reconsulta de 350.000 cuentas en ≤ 6 h compite por recursos con ~3.000 operaciones de saldo por segundo.** [S3, S6, S19 · supuestos] | Une el atributo más alto (Seguridad, 4) con Desempeño. El mapa la señala como el centro del sistema (`03` §5.2). | Su ADR (ADR-01) vive en el camino del retiro, no en la operación de saldo. El sizing quedaría desalineado de la tensión que lo encabeza. |
| **T7**, Seguridad × Compatibilidad | En este sistema, **autenticar cada petición de saldo** choca con **aceptar el esquema propio de cada uno de los 5 proveedores**, porque **la interfaz por la que entra el dinero queda tan protegida como el proveedor menos exigente.** [dato: 5 proveedores; S29 · supuesto] | Está en la misma interfaz de saldo y tiene a Seguridad (4). | Es un choque de integración, no de carga: con más operaciones por segundo no se agrava. Además es la tensión más reciente (promovida el 1 de noviembre). |

**Decisión del equipo:** cuál de las tres encabeza la entrega. La recomendación es T2.

---

## 2. Escenarios críticos que llegan del Bloque 2

Llegan los mismos cuatro escenarios críticos y los mismos cuatro ADR, en estado Propuesto.

| Escenario | ADR | Tensión | Atributo dominante | Atributo que paga |
|-----------|-----|---------|--------------------|-------------------|
| ESC-01 · Retiro de un jugador incluido en una lista recién recibida | ADR-01: Evaluación de los retiros contra listas restrictivas recién recibidas | T1 | Seguridad (4) | Desempeño (3) |
| ESC-02 · Caída del servidor de saldo en pleno pico | ADR-02: Permanencia de las operaciones de saldo confirmadas ante la caída de un servidor | T2 | Fiabilidad (3) | Desempeño (3) |
| ESC-03 · El proveedor del *crash* envía tres veces el mismo abono | ADR-03: Tratamiento de las peticiones de saldo repetidas de los proveedores de juego | T3 | Compatibilidad (3) | Fiabilidad (3) |
| ESC-05 · Petición de abono falsificada a nombre de un proveedor | ADR-04: Verificación del origen de las peticiones de saldo de los proveedores | T7 | Seguridad (4) | Compatibilidad (3) |

**ESC-04 (ráfaga de retiros del *crash*) sigue sin ADR.** Su costo cae en la Adecuación funcional, que con 2 puntos no está entre los atributos altos. En la fase 7 se revisa igual: una táctica de otro ADR puede empeorar su medida.

**Correspondencia con la plantilla.** La plantilla numera los escenarios como E1, E2… y los ADR como ADR-1, ADR-2… En la plantilla se usa esta correspondencia; en los markdown se conservan los identificadores del proyecto.

| Proyecto | Plantilla |
|----------|-----------|
| ESC-01, ESC-02, ESC-03, ESC-04, ESC-05 | E1, E2, E3, E4, E5 |
| ADR-01, ADR-02, ADR-03, ADR-04 | ADR-1, ADR-2, ADR-3, ADR-4 |

ADR-4 nace de E5, no de E4. Se dice así en la plantilla para que no parezca un error.

---

## 3. Qué cambió desde el Bloque 2 y por qué

### 3.1 Cambios en el contenido

| Fecha | Cambio | Razón | Qué no cambia |
|-------|--------|-------|---------------|
| 13 de noviembre de 2026 | Adecuación funcional sube de 1 a 2 y Fiabilidad baja de 4 a 3. Reparto vigente: Seguridad 4, Fiabilidad 3, Compatibilidad 3, Desempeño 3, Adecuación funcional 2 y el resto en 1. | Retroalimentación del profesor a la exposición: el operador calcula reglas que exige el reglamento (saldo nunca negativo, límites de depósito, autoexclusión, plazo del retiro, reglas fiscales). El punto sale de Fiabilidad porque el saldo nunca negativo ya estaba atribuido entero a ella (`08` §8). | Las tensiones críticas, los cuatro atributos altos, el texto de los cinco escenarios y de los cuatro ADR. |

Es el único cambio. T7, ESC-05 y ADR-04 (agregados el 1 de noviembre, `08` §7) ya estaban en la versión expuesta al profesor, así que no cuentan como cambio desde el Bloque 2.

**Ningún supuesto ni medida de un escenario cambió.** Los supuestos S23 a S29 se agregaron antes de la exposición.

**Riesgo que trae el cambio de puntajes (R5).** Con Fiabilidad en 3, ESC-02 y ESC-03 tienen dominante en 3. La defensa está en `08` §8: el corte de atributo alto es 3, esos tres atributos concentran 6 de los 10 puntos libres y el siguiente atributo tiene 2.

### 3.2 Cambios en las reglas de trabajo

- **Cifras nuevas.** Las cifras sobre la solución (límites, efecto y costo de las tácticas) continúan la numeración `S#` y `F#` y se registran en la fase del Bloque 3 donde nacen, no en la ficha (`00` §5).
- **Tácticas.** A partir de este bloque se nombran, con los nombres del catálogo del curso. Los mecanismos siguen sin marcas ni tecnologías.
- **ADR.** Pasan de Propuesto a Aceptado en la fase 6. `bloque-2/07-adrs.md` no se reescribe: recibe una nota fechada con el cambio de estado.

---

## 4. Operación crítica

### Propuesta: la operación de saldo pedida por un proveedor

Cada jugada hace que el proveedor del juego pida al operador descontar o abonar el saldo del jugador, o revertir una operación anterior.

| Dato de la operación | Valor | Origen |
|----------------------|-------|--------|
| Tasa sostenida en el pico de diseño | ~3.000 operaciones por segundo | [S6 · supuesto] |
| Ráfaga | ~4.000 operaciones por segundo | [S6 · supuesto] |
| Jugadores conectados en el pico de diseño | 12.000 | [S4 · supuesto] |
| Tiempo de respuesta medido en el operador | p99 ≤ 250 ms | [S14 · supuesto] |
| Tiempo de espera del proveedor | ~3 s | [S14 · supuesto] |
| Pérdida tolerada de operaciones confirmadas | Ninguna | [S13 · supuesto][F1 · referencia] |

**Por qué esta:**
- Es la de mayor volumen del sistema y la que más cuesta perder: cada operación mueve dinero.
- En ella viven tres de los cuatro ADR: ADR-02 (que la operación confirmada no se pierda), ADR-03 (que una petición repetida no se aplique dos veces) y ADR-04 (que la petición venga de verdad del proveedor). Los tres agregan trabajo al mismo paso de escritura del saldo, así que comparten el presupuesto de 250 ms.
- Coincide con la tensión dominante propuesta (T2).

### ADR-01 se mide en otra dimensión

ADR-01 no vive en la operación de saldo, sino en el camino del retiro. Su carga no se cuenta en operaciones de saldo por segundo:

| Dimensión | Valor | Origen |
|-----------|-------|--------|
| Ritmo mínimo de la reconsulta | 350.000 cuentas / 6 h ≈ 16 consultas por segundo | [S3 · supuesto][S19 · supuesto] |
| Retiros al día | ~3.000 | [S8 · supuesto] |
| Evaluación de riesgo de un retiro | p95 ≤ 1 s | [S20 · supuesto] |

Comprobación de la cuenta: 6 h × 3.600 s/h = 21.600 s; 350.000 / 21.600 s ≈ 16,2 consultas por segundo.

Estas cifras entran en el perfil de carga como "otra dimensión que importa" (fase 3, tarea 3.3), que es el campo que la plantilla prevé para lo que no son solicitudes por segundo.

### Variante

Dimensionar dos operaciones críticas: el saldo y el retiro, cada una con su cadena completa.

- **A favor:** ADR-01 tendría su propio cuello de botella y su propio margen.
- **En contra:** la plantilla pide una sola cadena hacia una operación crítica, y la extensión X1 multiplica "la tasa del pico de su operación crítica". Con dos, la extensión se aplicaría dos veces y la comparación de límites de la sección 7 de la plantilla se partiría en dos.

**Decisión del equipo:** una operación crítica (saldo) con el retiro como otra dimensión, o dos operaciones. La recomendación es la primera.

---

## 5. Tipos de cifra

La plantilla pide marcar cada número con uno de tres tipos. El proyecto ya usa `F#` y `S#`; esta tabla define cómo se traducen.

| Tipo de la plantilla | Qué dice el material | Qué es en este proyecto |
|----------------------|----------------------|-------------------------|
| **Referencia** | Sale de una fuente externa que se puede citar | Toda cifra con `F#`, definidas en `bloque-2/01-investigacion-dominio.md`. Incluye las normas de Coljuegos, como el máximo de 72 h para el retiro, el bloqueo a los 5 intentos fallidos o el cierre a los 20 min de inactividad [F1]. Las fuentes nuevas del Bloque 3 (límites de componentes) continúan la numeración. |
| **Supuesto** | Lo decide el equipo, con una razón | Toda cifra con `S#`, definidas en `bloque-2/02-ficha-contexto.md` §6, y las nuevas del Bloque 3, que se registran con su razón en la fase donde nacen. |
| **Dato** | Lo dio el cliente o la semilla | Casino Andino es un proyecto propio, sin cliente. Se marcan como dato las **definiciones del caso** que el equipo fijó como enunciado y que no se derivan de nada: 5 proveedores de juego, 4 tipos de juego, 5 medios de pago y el horizonte de diseño en el año 3 (ficha §1 y §2). El equipo hace de cliente. |

**Cómo se escribe la marca en los markdown del Bloque 3:** `[S6 · supuesto]`, `[F27 · referencia]`, `[dato]`. Una cifra que combina varias fuentes lleva todas sus marcas, como `[S13 · supuesto][F1 · referencia]`.

**Una cifra derivada** (por ejemplo, 16 consultas por segundo a partir de 350.000 cuentas y 6 h) toma el tipo de su insumo más débil: si un insumo es supuesto, la cifra es supuesto. La cuenta se escribe completa para que se vea de dónde sale.

### Variante

No usar el tipo "dato" y marcar las definiciones del caso como supuestos, con la razón "definición del caso".

- **A favor:** evita la pregunta "¿quién es el cliente?".
- **En contra:** mezcla dos cosas distintas. Un supuesto como S6 se puede discutir y cambiar con una razón; que haya 5 proveedores es el enunciado del caso. La rúbrica premia que se distinga el tipo de cada número.

**Decisión del equipo:** usar o no el tipo "dato". La recomendación es usarlo, solo para las definiciones del caso.

---

## 6. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| Los ADR son los mismos del Bloque 2: cuatro, con los mismos títulos y en estado Propuesto | ✔ Títulos copiados de `bloque-2/07-adrs.md` |
| Todo lo que cambió desde el Bloque 2 está declarado, con fecha y razón | ✔ §3: un solo cambio, el de puntajes del 13 de noviembre |
| Cada cifra citada tiene `F#` o `S#` y su tipo | ✔ S3, S4, S6, S8, S13, S14, S19, S20, S29 y F1, todas en la ficha del Bloque 2 |
| Las cuentas cuadran | ✔ 350.000 / 21.600 s ≈ 16,2 consultas por segundo, igual que S19 |
| No se nombran tácticas, tecnologías ni marcas | ✔ |
| No hay cifras nuevas | ✔ Esta fase no agrega `S#` ni `F#` |

---

## 7. Punto de control (cerrado el 13 de noviembre de 2026)

**Decisiones del equipo:**

1. **Tensión dominante: T2**, Fiabilidad × Desempeño (§1). T1 y T7 quedan como variantes registradas.
2. **Operación crítica: la operación de saldo pedida por un proveedor**, con el camino del retiro (ADR-01) como otra dimensión del perfil de carga (§4).
3. **Tipos de cifra:** `F#` es referencia, `S#` es supuesto y "dato" se usa solo para las definiciones del caso (§5).
4. **Versión expuesta del Bloque 2:** ya incluía ESC-05 y ADR-04. El único cambio declarado es el de puntajes del 13 de noviembre (§3.1).

La fase 2 (línea base) puede arrancar.
