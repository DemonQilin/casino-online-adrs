# Plan de trabajo — Bloque 3: tácticas, ADR completos y sizing

**Materia:** Arquitectura de Software · Bloque 3 (tácticas y sizing)
**Sistema:** Casino Andino, casino online regulado en Colombia
**Nivel:** pregrado · **Caso:** proyecto final (pregrado)
**Material de autoridad:** `docs/class-material/Tácticas y sizing · Bloque 3.html` (entrega, plantilla y rúbrica) y `docs/class-material/PedalVa Tácticas.html` (catálogo de tácticas del curso)
**Insumos del Bloque 2:** `bloque-2/02-ficha-contexto.md`, `bloque-2/06-escenarios.md`, `bloque-2/07-adrs.md` y `bloque-2/08-revision.md`
**Estado del plan:** validado por el equipo el 13 de noviembre de 2026.

---

## 1. Qué hay que entregar (según el material del profesor)

| # | Pieza | Regla clave |
|---|-------|-------------|
| 1 | Línea base | Cómo funciona hoy la operación crítica y qué componentes intervienen, en orden. Como el sistema no existe, es **la versión más simple que funcionaría, sin ninguna de las tácticas que se van a proponer**. |
| 2 | Perfil de carga | Una cadena en la que cada número sale del anterior, de usuarios registrados a operaciones por segundo en el pico. Cada número dice si es dato, supuesto o referencia. |
| 3 | Cuello de botella | El componente que se satura primero en la línea base, su límite con fuente, el uso con la carga del pico y lo que ve el usuario al superarlo. |
| 4 | ADR completados | Los ADR-01 a ADR-04 pasan a **Aceptado**: opción elegida, Decision en voz activa, por qué se descartan las demás y Consequences con números. |
| 5 | Tácticas | Por cada táctica: nombre del catálogo, mecanismo concreto, responsabilidad, métrica que mueve, valor antes y después, cómo se estimó, costo cuantificado y atributo que paga. |
| 6 | Hasta dónde | Nuevo límite con las tácticas, margen sobre el pico, siguiente componente que se satura y condición de revisión medible. |
| 7 | Interacción entre tácticas | Qué táctica de un ADR empeora la medida de otro escenario, en cuánto, y cómo se resuelve o por qué se acepta. |
| 8 | Extensión X1 | "Un evento, una campaña o una alianza multiplica por 5 la tasa del pico de su operación crítica durante 2 horas. El resto del perfil de carga se mantiene." Se responde con los números propios. |
| 9 | Resumen en texto plano | La plantilla del material, diligenciada por transcripción literal de la entrega. |

**Formato:** la entrega en presentación HTML, como en el Bloque 2, más la plantilla en un archivo `.md`.

**Fuera del alcance por ser de posgrado:** la extensión X2 (degradación de una dependencia), la sensibilidad a los supuestos (sección 9 de la plantilla) y lo que se mediría en producción.

---

## 2. Principios de trabajo

1. **Un archivo markdown por fase.** Cada archivo declara sus insumos y cierra con su verificación y su punto de control. La presentación y la plantilla se integran al final.
2. **Punto de control al cierre de cada fase.** Ninguna fase arranca sobre una anterior sin validar.
3. **La línea base no tiene tácticas.** Si la línea base ya resuelve el escenario, no hay contra qué comparar.
4. **Se diseña para el pico, no para el promedio.** Toda tasa distingue el promedio del pico.
5. **Toda cifra tiene origen y tipo.** Se conservan las marcas `F#` y `S#` del proyecto y se agrega el tipo que pide la plantilla: dato, supuesto o referencia. La equivalencia se define en la fase 1.
6. **Cada cuenta se escribe completa**, con sus operaciones y sus unidades, para que cualquiera pueda seguirla y discutirla.
7. **Un costo es un número.** "Más complejo" no es un costo; "un servidor más" o "hasta 5 s de retraso" sí.
8. **No se inventa precisión.** Se prefieren rangos atados a un supuesto antes que cifras con decimales.
9. **Tácticas con el nombre del catálogo; mecanismos sin marcas.** A partir de este bloque las tácticas se nombran, pero solo con los nombres del catálogo del curso. Los mecanismos se siguen describiendo sin tecnologías ni productos.
10. **Los ADR no se reescriben.** `bloque-2/07-adrs.md` conserva su texto y recibe una nota fechada con el cambio de estado y el enlace al Bloque 3.
11. **La rúbrica es el criterio de terminado.** Cada fase indica qué criterio alimenta y debe alcanzar el nivel "Logrado".

---

## 3. Mapa de fases

| Fase | Entregable | Sección de la plantilla | Criterio de la rúbrica | Punto de control |
|------|-----------|-------------------------|------------------------|------------------|
| 1. Punto de partida | `01-punto-de-partida.md` | 1 | Base de todos | **Decisión del equipo:** operación crítica y tipos de cifra |
| 2. Línea base | `02-linea-base.md` | 2 | Línea base y cuello de botella | **Decisión del equipo:** qué entra en la línea base |
| 3. Perfil de carga | `03-perfil-de-carga.md` | 3 | Perfil de carga | Validar la cadena y las cuentas |
| 4. Cuello de botella | `04-cuello-de-botella.md` | 4 | Línea base y cuello de botella | Validar límites y fuentes nuevas |
| 5. Tácticas y efecto | `05-tacticas-y-efecto.md` | 5 (bloques de táctica) | Tácticas como mecanismos · Efecto estimado · Costo cuantificado | **Decisión del equipo:** opción elegida por ADR |
| 6. ADR completados | `06-adrs.md` | 5 (ADR) | Decision · Hasta dónde | Validar Decision y Consequences |
| 7. Interacción y límites | `07-interaccion-y-limites.md` | 6 y 7 | Interacción entre tácticas · Hasta dónde | Validar |
| 8. Extensión | `08-extension.md` | 8 (X1) | Extensión | Validar |
| 9. Revisión integral | `09-revision.md` | 10 | Los 9 criterios | Aprobación final |
| 10. Integración | Presentación HTML y `plantilla-entrega.md` | Todas | — | Entrega |

---

## 4. Fases en detalle

### Fase 1 — Punto de partida

**Objetivo:** fijar qué llega del Bloque 2 y las convenciones que usa todo el Bloque 3.

| Tarea | Qué se hace |
|-------|------------|
| 1.1 Tensión dominante | Escribirla en una línea, con la forma de la plantilla: "En este sistema, ___ choca con ___ porque ___". Sale de las tensiones fuertes del Bloque 2. |
| 1.2 Escenarios críticos | Listar ESC-01, ESC-02, ESC-03 y ESC-05 con su ADR. |
| 1.3 Qué cambió desde el Bloque 2 | Declarar los cambios posteriores a la entrega y su razón, a partir de `bloque-2/08-revision.md` §7 y §8 (entre ellos, el cambio de puntajes del 13 de noviembre de 2026). |
| 1.4 Operación crítica | Definir la operación sobre la que se hace el sizing. Propuesta: la operación de saldo, donde viven ADR-02, ADR-03 y ADR-04. ADR-01 vive en el camino del retiro y su límite se mide en otra dimensión (el ritmo de la reconsulta). |
| 1.5 Tipos de cifra | Definir la equivalencia entre las marcas del proyecto (`F#`, `S#`) y los tres tipos de la plantilla (dato, supuesto, referencia). |

**Entregable:** `01-punto-de-partida.md`

**Terminado cuando:** los ADR de esta entrega son los mismos del Bloque 2 y todo lo que cambió está declarado.

**Punto de control:** **decisión del equipo** sobre la tensión dominante, la operación crítica y la equivalencia de tipos.

---

### Fase 2 — Línea base

**Objetivo:** describir la solución contra la que se mide todo lo demás.

| Tarea | Qué se hace |
|-------|------------|
| 2.1 Descripción | Un párrafo que cuenta cómo funcionaría la operación crítica en la versión más simple. |
| 2.2 Componentes en orden | La lista de componentes que intervienen en la operación de saldo y en el retiro, de punta a punta. |
| 2.3 Frontera con la regulación | Separar lo que la línea base incluye porque el reglamento lo exige (por ejemplo, el sistema alterno) de lo que sería una táctica elegida por el equipo. |
| 2.4 Chequeo | Verificar que ninguna táctica de la fase 5 aparece en la línea base. |

**Entregable:** `02-linea-base.md`

**Terminado cuando:** cualquiera puede seguir la operación crítica de punta a punta y ver qué piezas se pueden saturar.

**Punto de control:** **decisión del equipo.** La línea base no viene dada; se proponen variantes a partir de las opciones más simples de cada ADR.

---

### Fase 3 — Perfil de carga

**Objetivo:** llegar del número de jugadores a la tasa del pico con una cuenta que se pueda seguir.

| Tarea | Qué se hace |
|-------|------------|
| 3.1 Cadena de carga | Jugadores registrados → activos en el día pico → concurrentes en el pico → operaciones por jugador → operaciones de saldo por segundo. Cada número con su tipo y su `F#` o `S#`. |
| 3.2 Cuenta de la tasa | Escribir la operación completa con unidades y contrastar el promedio del día con el pico. Si aplica, usar la relación entre llegadas y permanencia para los concurrentes. |
| 3.3 Otras dimensiones | Lo que no se mide en operaciones por segundo: volumen de identificadores por conservar, ritmo de la reconsulta de cuentas, retiros al día y presupuesto de tiempo de cada verificación. |
| 3.4 Lecturas y escrituras | La proporción en la operación crítica. |
| 3.5 Cifras nuevas | Registrar cada cifra nueva con identificador, tipo y razón (ver la convención del §5). |

**Entregable:** `03-perfil-de-carga.md`

**Terminado cuando:** la tasa del pico sale de una cuenta con unidades y cada número de la cadena tiene tipo y origen.

---

### Fase 4 — Capacidad actual y cuello de botella

**Objetivo:** identificar qué se satura primero en la línea base y con cuánta carga.

| Tarea | Qué se hace |
|-------|------------|
| 4.1 Candidatos | Revisar cada componente de la línea base, incluidas la red y las dependencias externas, no solo el servidor. |
| 4.2 Límites con fuente | Investigar el límite de cada candidato en documentación del componente o en pruebas públicas que se puedan citar. Cada fuente nueva recibe su `F#`. |
| 4.3 Uso en el pico | Carga ÷ límite, con la cuenta, y la carga máxima que aguanta hoy la línea base. |
| 4.4 Consecuencia | Qué ve el jugador, el proveedor o el regulador cuando se supera el límite. |

**Entregable:** `04-cuello-de-botella.md`

**Terminado cuando:** hay un componente que se satura primero, su límite con fuente, un número de uso y una consecuencia para el usuario.

**Nota:** si con la carga del pico nada se satura, se dice así y el límite se busca en otra dimensión (disponibilidad, costo o una dependencia externa). El material lo permite expresamente.

---

### Fase 5 — Tácticas y efecto

**Objetivo:** cerrar con números las opciones que el Bloque 2 dejó en "Depende" y proponer la opción de cada ADR.

| Tarea | Qué se hace |
|-------|------------|
| 5.1 Opciones y catálogo | Relacionar cada opción del Bloque 2 con tácticas del catálogo. El catálogo no tiene una familia de Compatibilidad: los mecanismos que ADR-03 y ADR-04 anotaron como "Gestionar interfaces" se ubican en las familias que sí existen. |
| 5.2 Ficha por táctica | De 2 a 3 tácticas por ADR, con: nombre del catálogo, mecanismo concreto (qué componente, qué dato, en qué paso), responsabilidad, métrica que mueve, valor en la línea base, valor estimado, cómo se estimó, tipo de la estimación, costo cuantificado y atributo que paga. |
| 5.3 Cierre de los "Depende" | Resolver con números lo que quedó por medir: opción B de ADR-01, opciones A y C de ADR-02, opción B de ADR-03 y opción B de ADR-04. |
| 5.4 Propuesta de opción | Una recomendación por ADR, con sus variantes, justificada frente a la medida de la respuesta. |

**Orden de trabajo:** ADR-02 y ADR-03 se deciden juntos, porque afectan el mismo paso de escritura del saldo (así quedó anotado en el Bloque 2). Siguen ADR-04, que vive en la misma interfaz, y ADR-01.

**Entregable:** `05-tacticas-y-efecto.md`

**Terminado cuando:** cada táctica mueve una métrica de un valor a otro, con la estimación explicada, y tiene un costo en número.

**Punto de control:** **decisión del equipo.** Es la decisión central del bloque: qué opción se elige en cada ADR.

---

### Fase 6 — ADR completados

**Objetivo:** llevar los cuatro ADR a estado Aceptado.

| Tarea | Qué se hace |
|-------|------------|
| 6.1 Encabezado | Título, estado Aceptado, escenario de origen, Context copiado tal cual y medida de la respuesta copiada literal. |
| 6.2 Opciones | Las opciones del Bloque 2 resumidas en una línea cada una, sin reescribirlas, y la opción elegida. |
| 6.3 Decision | En voz activa ("Usaremos… para…"), con las tácticas, y por qué se descarta cada una de las demás opciones frente a la medida. |
| 6.4 Hasta dónde | Nuevo límite con las tácticas, cómo se calculó, siguiente componente que se satura y si la medida se cumple con la carga del pico (sí, no o depende). |
| 6.5 Consequences | Qué se gana, qué se paga, qué no se resuelve y la condición de revisión: un umbral medible que obliga a revisar la decisión. |
| 6.6 Enlace con el Bloque 2 | Agregar a `bloque-2/07-adrs.md` la nota fechada del cambio de estado, con el enlace a este archivo. |

**Entregable:** `06-adrs.md`, con una sección por ADR.

**Terminado cuando:** los cuatro ADR tienen Decision justificada frente a cada opción descartada, límite calculado y condición de revisión medible.

---

### Fase 7 — Interacción y resumen de límites

**Objetivo:** revisar las tácticas en conjunto.

| Tarea | Qué se hace |
|-------|------------|
| 7.1 Matriz táctica × escenario | Cruzar cada táctica con la medida de los cinco escenarios, incluido ESC-04, y anotar a cuál empeora y en cuánto. |
| 7.2 Presupuesto compartido | Sumar lo que las tácticas de ADR-02, ADR-03 y ADR-04 agregan al mismo paso de escritura y contrastarlo con la medida de tiempo de la operación de saldo. |
| 7.3 Resolución | Para cada interacción: cómo se resuelve o por qué se acepta. |
| 7.4 Resumen de límites | Carga del pico, límite de la línea base, límite con todas las tácticas, margen sobre el pico, siguiente componente que se satura y qué haría falta para pasarlo (pista para el Bloque 4, no una decisión). |

**Entregable:** `07-interaccion-y-limites.md`

**Terminado cuando:** cada interacción tiene táctica, escenario afectado, magnitud y resolución, y el margen sobre el pico está calculado.

---

### Fase 8 — Extensión X1

**Objetivo:** aplicar la extensión general a los números propios.

| Tarea | Qué se hace |
|-------|------------|
| 8.1 Texto recibido | Copiar la extensión X1 tal cual. |
| 8.2 Nueva carga | Calcular la nueva tasa del pico a partir de la cadena de la fase 3, sin números nuevos. |
| 8.3 Tácticas | Cuál se sostiene y por qué, cuál se rompe o deja de alcanzar, y en qué valor. |
| 8.4 Efecto visible | Qué ve el usuario cuando la táctica se rompe. |
| 8.5 Respuesta | Qué se cambiaría y si implica un ADR nuevo o reemplazar uno existente. |

**Entregable:** `08-extension.md`

**Terminado cuando:** la respuesta usa los números de la fase 3 y no se reduce a "más servidores".

---

### Fase 9 — Revisión integral

**Objetivo:** verificar que la entrega alcanza el nivel Logrado antes de integrarla.

| Tarea | Qué se hace |
|-------|------------|
| 9.1 Autoevaluación | Recorrer los 9 criterios de la rúbrica de pregrado y señalar la evidencia de cada uno. |
| 9.2 Coherencia en cadena | Recalcular las cifras derivadas, rastrear cada cifra hasta su `F#` o `S#` y comparar el Context de cada ADR con su escenario. |
| 9.3 Autochequeo de la plantilla | Marcar los 10 puntos del autochequeo de la sección 10. |
| 9.4 Lo que el equipo reconoce | Los números de los que el equipo está menos seguro y lo que discutió sin resolver. **Lo aporta el equipo**: no se redacta por él. |

**Entregable:** `09-revision.md`, con hallazgos (`H#`) y correcciones aplicadas.

---

### Fase 10 — Integración

| Tarea | Qué se hace |
|-------|------------|
| 10.1 Presentación HTML | Un archivo autocontenido con el contenido en constantes, como `bloque-2/Casino Andino.html`. Se integra desde los markdown y nunca es la fuente. |
| 10.2 Plantilla | `plantilla-entrega.md`: la plantilla del material diligenciada por **transcripción literal**. No se rehacen cuentas, se borra la sección 0 y las partes de posgrado, y la sección 11 registra las inconsistencias encontradas sin corregirlas. |
| 10.3 Índices | Actualizar `README.md` y el mapa del curso de `CLAUDE.md`. |

---

## 5. Convenciones

- **Nombres de archivo:** `NN-nombre.md`, numerados por fase.
- **Identificadores que se conservan:** `ESC-NN`, `ADR-NN`, `S#`, `F#`, `H#`. En la plantilla aparecen como E1, ADR-1, etc.; la correspondencia se anota una vez.
- **Identificador de táctica:** `TAC-NN`. La plantilla numera las tácticas como T1, T2…, pero en el proyecto `T#` ya nombra las tensiones fuertes, así que en los markdown se usa `TAC-NN`.
- **Cifras nuevas:** las cifras de requisito siguen en la ficha del Bloque 2. Las cifras sobre la solución (límites de la línea base, efecto y costo de las tácticas) continúan la numeración `S#` y `F#` y se registran en la fase del Bloque 3 donde nacen. Esta convención ajusta la regla anterior del proyecto, que enviaba toda cifra nueva a la ficha; se separa porque la ficha no debe nombrar soluciones.
- **Vocabulario:** el catálogo de tácticas agrupa por atributos con nombres distintos a los de ISO 25010. Equivalencia propuesta, por validar en la fase 5:

  | Atributo del catálogo | Característica ISO 25010 |
  |-----------------------|--------------------------|
  | Disponibilidad | Fiabilidad |
  | Rendimiento | Eficiencia de desempeño |
  | Seguridad | Seguridad |
  | Modificabilidad | Mantenibilidad |
  | Testabilidad | Mantenibilidad |
  | Despliegue | Portabilidad |

---

## 6. Riesgos a vigilar

| Riesgo | Cómo lo evitamos |
|--------|------------------|
| Describir como línea base una solución que ya tiene tácticas | Chequeo 2.4 y punto de control de la fase 2. |
| Diseñar para el promedio del día | La fase 3 escribe el promedio y el pico lado a lado. |
| Dar un límite sin fuente | Cada límite de la fase 4 lleva su `F#`; sin fuente se declara como supuesto, con su razón. |
| Que entren marcas por la puerta de las fuentes | El nombre de un producto solo aparece en la lista de fuentes, nunca en un mecanismo ni en una Decision. |
| Nombrar tácticas que no están en el catálogo | Tarea 5.1: toda táctica se busca en el catálogo antes de usarla. |
| Costos en adjetivos | Cada ficha de táctica exige un número y el atributo que paga. |
| Decir que las tácticas "escalan" sin límite, o marcar cumplida una medida con margen menor que 1 | La fase 6 calcula el límite y la fase 7 el margen. |
| Responder la extensión con "más servidores" | La fase 8 revisa si el cuello está en algo que no se escala. |
| Reescribir los ADR del Bloque 2 | Solo se agrega la nota de cambio de estado (tarea 6.6). |
| Cuentas que el equipo no pueda defender en la sustentación | Cada cuenta va completa y el equipo la rehace en el punto de control; la calculadora del material sirve para contrastar. |

---

## 7. Preguntas de la sustentación (pregrado)

El material anuncia estas preguntas; cada fase debe dejar su respuesta a la vista.

| Pregunta | Fase que la responde |
|----------|----------------------|
| ¿De dónde sale este número? ¿Es dato, supuesto o referencia? | 1, 3 y 4 |
| ¿Qué componente se satura primero hoy y con cuánta carga? | 4 |
| ¿Cuánto mueve su táctica esa métrica y cómo lo estimaron? | 5 |
| Con la extensión, ¿qué táctica se rompe primero y qué ve el usuario? | 8 |
| ¿Qué haría falta para pasar el siguiente cuello de botella? | 7 |

---

## 8. Punto de control del plan (cerrado el 13 de noviembre de 2026)

1. **Mapa de fases aprobado**, con sus puntos de control.
2. **Convención de cifras nuevas aprobada** (§5). La regla correspondiente de `CLAUDE.md` se actualizó en el mismo cambio.
3. **Identificador `TAC-NN` aprobado** para las tácticas.
4. **Campo `caso` de la plantilla: "proyecto final (pregrado)"**, definido por el equipo el 15 de noviembre de 2026. Casino Andino es el proyecto final de la materia. Hasta esa fecha el plan usaba "proyecto propio".
