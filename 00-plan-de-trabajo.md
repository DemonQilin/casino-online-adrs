# Plan de trabajo — Casino online regulado en Colombia

> [Índice](./README.md) · [Siguiente →](./01-investigacion-dominio.md)

**Materia:** Arquitectura de Software · Bloque 2 (atributos de calidad, escenarios y ADR)
**Sistema hipotético:** casino online de juegos de azar (tragamonedas, ruleta con generador de números aleatorios, blackjack en vivo y un juego *crash*), operado bajo la regulación colombiana. **No incluye apuestas deportivas.**
**Entrega:** en equipo. Juan comparte con el equipo lo que se define en cada fase.

---

## 1. Qué hay que entregar (según el material del profesor)

| # | Pieza | Regla clave |
|---|-------|-------------|
| 1 | Ficha de contexto | Requisitos no funcionales **con cifras**. Alguien externo debe poder puntuar los atributos sin inventar nada. |
| 2 | Priorización ISO 25010 | Puntaje de 1 a 5 para cada uno de los 8 atributos, con un total de **18 puntos**. Se justifica cada atributo alto y cada atributo bajo. |
| 3 | Escenarios de calidad | **De 3 a 5**. Cada uno lleva su atributo y seis partes. Va uno por tensión, no uno por atributo. |
| 4 | ADRs | **Uno por cada escenario crítico**, escrito solo hasta las opciones consideradas. Decision y Consequences quedan para el Bloque 3. |

El número de ADRs no está fijado: es igual al número de escenarios que resulten críticos (fase 5).

---

## 2. Principios de trabajo

1. **Un archivo markdown por fase.** Cada archivo es el insumo de la siguiente fase. La integración en HTML o presentación se hace al final y queda fuera de este plan.
2. **Punto de control al cierre de cada fase.** Juan revisa y comparte con el equipo antes de avanzar. Ninguna fase arranca sobre una anterior sin validar.
3. **Toda cifra tiene origen.** Cada número se marca como **[Fuente]**, con enlace, o como **[Supuesto]**, con una razón en una línea.
4. **No hay soluciones antes de la fase 7.** Ni en la ficha ni en los escenarios aparecen tácticas, tecnologías o marcas. Las opciones de los ADR se describen como mecanismos del sistema.
5. **La rúbrica es el criterio de terminado.** Cada fase indica qué criterio de la rúbrica alimenta y qué nivel ("Logrado") debe alcanzar.

---

## 3. Mapa de fases

| Fase | Entregable | Alimenta (rúbrica / puerta de revisión) | Punto de control |
|------|-----------|------------------------------------------|------------------|
| 1. Investigación del dominio | [`01-investigacion-dominio.md`](./01-investigacion-dominio.md) | Base factual de la ficha | Confirmar alcance de juegos |
| 2. Ficha de contexto | [`02-ficha-contexto.md`](./02-ficha-contexto.md) | Ficha de contexto · puerta 1 | Validar cifras y supuestos |
| 3. Mapa de tensiones | [`03-mapa-tensiones.md`](./03-mapa-tensiones.md) | Puerta 2 y 3 | Validar pares en tensión |
| 4. Priorización (18 puntos) | [`04-priorizacion.md`](./04-priorizacion.md) | Priorización · puerta 4 | **Decisión del equipo** |
| 5. Selección de escenarios | [`05-seleccion-escenarios.md`](./05-seleccion-escenarios.md) | Selección de escenarios críticos | **Decisión del equipo** |
| 6. Redacción de escenarios | [`06-escenarios.md`](./06-escenarios.md) | Estímulo y entorno · Respuesta · Medida · Coherencia | Validar escenarios |
| 7. ADRs hasta opciones | [`07-adrs.md`](./07-adrs.md) | ADR: contexto y estado · ADR: opciones y trade-offs | Validar opciones |
| 8. Revisión integral | [`08-revision.md`](./08-revision.md) | Los 9 criterios | Aprobación final |

---

## 4. Fases en detalle

### Fase 1 — Investigación del dominio

**Objetivo:** reunir hechos verificables para que la ficha de contexto no invente nada.

| Tarea | Qué se hace | Resultado |
|-------|------------|-----------|
| 1.1 Marco regulatorio | Identificar la autoridad (Coljuegos) y los requisitos para operar juegos de suerte y azar por internet: habilitación, certificación de juegos y del generador aleatorio, reporte al regulador, prevención de lavado de activos, protección de datos (Ley 1581 de 2012), verificación de edad y juego responsable. | Tabla: obligación → fuente → atributo de calidad que toca |
| 1.2 Modelo operativo | Describir actores (jugador, operador, proveedores de juegos, agregadores, pasarelas de pago, regulador, laboratorio certificador) y flujos principales (registro y verificación de identidad, depósito, jugada y resultado, retiro). Se describen **flujos, no arquitectura**. | Lista de actores y flujos |
| 1.3 Cifras de referencia | Buscar datos para dimensionar: tamaño del mercado online colombiano, usuarios, patrones de pico (noches, fines de semana, quincenas), exigencias del juego en vivo. | Tabla de cifras, cada una con fuente o marcada como estimación |

**Entregable:** [`01-investigacion-dominio.md`](./01-investigacion-dominio.md), con lista de fuentes.

**Terminado cuando:**
- cada obligación regulatoria tiene fuente;
- cada cifra tiene fuente o está marcada como estimación.

**Punto de control:** el equipo confirma qué tipos de juego entran en el alcance.

---

### Fase 2 — Ficha de contexto

**Objetivo:** describir el casino con requisitos no funcionales cuantificados, en el formato de las fichas de los 10 escenarios del widget de ISO 25010 (descripción breve más viñetas con cifras).

| Tarea | Qué se hace |
|-------|------------|
| 2.1 Descripción y alcance | Qué es el producto, para quién, qué incluye y qué excluye explícitamente (apuestas deportivas). |
| 2.2 Actores e integraciones | Sistemas externos con los que se intercambia información y por qué. |
| 2.3 Requisitos no funcionales | Tabla con cifras de usuarios registrados y concurrentes, picos, disponibilidad esperada, tiempos de respuesta del juego, volumen de transacciones, retención de datos, regulación, presupuesto, tamaño del equipo y restricciones. |
| 2.4 Supuestos | Lista de los números que no vienen de una fuente, con la razón de cada uno. |

**Entregable:** [`02-ficha-contexto.md`](./02-ficha-contexto.md)

**Terminado cuando:**
- pasa la prueba "alguien externo puede puntuar sin inventar";
- se cumple la puerta de revisión 1 (cifras, no adjetivos);
- no se nombran soluciones ni tecnologías;
- alcanza el nivel Logrado en el criterio *Ficha de contexto*.

---

### Fase 3 — Mapa de tensiones

**Objetivo:** identificar dónde chocan los atributos de calidad en este casino concreto.

| Tarea | Qué se hace |
|-------|------------|
| 3.1 Recorrido de pares | Partir de la pestaña Tensiones del widget de ISO 25010 y revisar los 28 pares posibles. Se conservan solo los que tienen una situación concreta en el casino. |
| 3.2 Ficha por par | Para cada par conservado: la situación del casino, el tipo (tensión fuerte, tensión leve o sinergia) y quién pierde si se prioriza el otro atributo. |

**Entregable:** [`03-mapa-tensiones.md`](./03-mapa-tensiones.md), con una matriz resumen y una ficha por par.

**Terminado cuando:**
- hay al menos tres atributos en tensión real (puerta 2);
- cada par dice quién pierde (puerta 3).

---

### Fase 4 — Priorización con presupuesto de 18 puntos

**Objetivo:** decidir qué atributos importan más, haciendo visible lo que se sacrifica.

| Tarea | Qué se hace |
|-------|------------|
| 4.1 Propuesta de puntajes | Asignar de 1 a 5 a los 8 atributos. Restricciones: suma exacta de 18, mínimo 1 por atributo, máximo tres atributos en 4 o más. |
| 4.2 Justificación | Justificar cada atributo alto y cada atributo bajo, citando la ficha de contexto. |
| 4.3 Costo explícito | Una sección "Lo que nos costó": qué se bajó para poder subir qué. |
| 4.4 Verificación | Revisión aritmética de la suma y los límites. |

**Entregable:** [`04-priorizacion.md`](./04-priorizacion.md)

**Terminado cuando:**
- se cumple la puerta 4 (respeta los 18 puntos y obligó a bajar algún atributo);
- alcanza el nivel Logrado en *Priorización*.

**Punto de control:** **decisión del equipo.** Es la pieza más discutible de toda la entrega. Se puede llevar con una o dos variantes alternativas para que el equipo elija.

---

### Fase 5 — Selección de escenarios

**Objetivo:** escoger qué escenarios escribir y cuáles de ellos son críticos.

| Tarea | Qué se hace |
|-------|------------|
| 5.1 Candidatos | Listar de 6 a 8 candidatos en una línea cada uno (fuente, estímulo y entorno), derivados de las tensiones entre atributos altos. |
| 5.2 Prueba de criticidad | Hacer a cada candidato las dos preguntas del material: ¿su atributo dominante está entre los más altos? ¿Cumplirlo cuesta otro atributo también alto? |
| 5.3 Selección | Elegir de 3 a 5 escenarios y marcar cuáles son críticos. Escribir una línea que justifique la selección. |

**Entregable:** [`05-seleccion-escenarios.md`](./05-seleccion-escenarios.md)

**Decisión del equipo:** cuántos escenarios y cuántos críticos. Orientación: 4 o 5 escenarios, de los cuales 3 o 4 son críticos. Incluir algún escenario no crítico ayuda a demostrar el criterio de selección, porque escribir un ADR para todos es el nivel Inicial de la rúbrica.

**Terminado cuando:** alcanza el nivel Logrado en *Selección de escenarios críticos*.

---

### Fase 6 — Redacción de escenarios

**Objetivo:** escribir los escenarios elegidos con las seis partes, con calidad de rúbrica.

**Tarea 6.1 — Redacción.** Para cada escenario (ESC-01, ESC-02, …), completar la tabla: atributo, fuente, estímulo, artefacto, entorno, respuesta y medida de la respuesta.

**Tarea 6.2 — Lista de chequeo por escenario** (basada en el Constructor y la Clínica del taller GPSMetro):
- La fuente es un actor, no una condición.
- Hay un solo estímulo, y es observable.
- El artefacto se nombra por su función.
- El entorno tiene cifras.
- La respuesta es observable y no menciona tácticas.
- La medida tiene número, unidad y percentil o umbral.
- El atributo declarado es el correcto (por ejemplo, no confundir disponibilidad con corrección del dato).

**Tarea 6.3 — Trazabilidad.** Para cada escenario, indicar de qué puntaje de la priorización sale y qué atributo paga el costo.

**Entregable:** [`06-escenarios.md`](./06-escenarios.md)

**Terminado cuando:**
- todos los escenarios pasan la lista de chequeo;
- alcanzan el nivel Logrado en *Estímulo y entorno*, *Respuesta*, *Medida de la respuesta* y *Coherencia con la priorización*.

---

### Fase 7 — ADRs hasta las opciones consideradas

**Objetivo:** abrir el registro de decisiones del proyecto con un ADR por cada escenario crítico.

| Tarea | Qué se hace |
|-------|------------|
| 7.1 Tres preguntas | Para cada escenario crítico: ¿cuál es la brecha?, ¿qué mecanismo tendría que cambiar o agregarse para cerrarla?, ¿qué atributo paga el costo? |
| 7.2 Opciones | Plantear al menos 2 opciones (idealmente 3), descritas como mecanismos y sin nombres de tácticas ni marcas. Tabla por opción: qué gana, qué paga, atributo sacrificado y si cumple la medida de la respuesta. |
| 7.3 Armado del ADR | Título numerado que nombra la decisión, estado **Propuesto**, Context igual al escenario copiado tal cual, opciones y nota de pendientes. |

**Entregable:** [`07-adrs.md`](./07-adrs.md), con una sección por ADR (ADR-01, ADR-02, …).

**Terminado cuando:**
- no hay Decision ni Consequences escritas;
- alcanza el nivel Logrado en *ADR: contexto y estado* y en *ADR: opciones y trade-offs*.

---

### Fase 8 — Revisión integral

**Objetivo:** verificar que la entrega completa alcanza el nivel Logrado antes de integrarla.

| Tarea | Qué se hace |
|-------|------------|
| 8.1 Autoevaluación | Recorrer los 9 criterios de la rúbrica y señalar dónde está la evidencia de cada uno. |
| 8.2 Coherencia en cadena | Verificar que las cifras y los nombres son consistentes de la ficha a la priorización, a los escenarios y a los ADRs. |
| 8.3 Revisión en frío | Pedir una lectura a alguien que no haya participado en la redacción: un compañero del equipo o una revisión independiente. |

**Entregable:** [`08-revision.md`](./08-revision.md), con hallazgos y correcciones aplicadas.

---

### Fase 9 — Integración final (fuera de este plan)

Convertir los archivos markdown en el formato final (HTML interactivo o presentación). Se planea cuando terminen las fases 1 a 8.

---

## 5. Convenciones

- **Nombres de archivo:** `NN-nombre.md`, numerados por fase.
- **Identificadores:** escenarios `ESC-01`…; decisiones `ADR-01`….
- **Atributos:** se usan las 8 características de ISO 25010 del widget del curso. Algunos términos del taller GPSMetro se traducen así:

  | Término de GPSMetro | Característica ISO 25010 |
  |---------------------|--------------------------|
  | Disponibilidad | Fiabilidad |
  | Escalabilidad | Eficiencia de desempeño (capacidad) |
  | Rendimiento | Eficiencia de desempeño |
  | Interoperabilidad | Compatibilidad |

- **Cifras:** cada una marcada como **[Fuente]** o **[Supuesto]**.

---

## 6. Riesgos a vigilar

| Riesgo | Cómo lo evitamos |
|--------|------------------|
| Sesgo del dominio: que todo termine siendo Seguridad | La priorización obliga a elegir, y los escenarios se derivan de tensiones, no de un solo atributo. |
| Cifras de mercado difíciles de verificar | Supuestos declarados con su razón. |
| Colar soluciones antes de tiempo (fases 2 a 6) | Lista de chequeo de la fase 6 y regla de oro del taller. |
| Ampliar el alcance hacia apuestas deportivas | Exclusión explícita en la ficha. |
| Inconsistencias entre archivos | Revisión en cadena de la fase 8. |

---

> [Índice](./README.md) · [Siguiente →](./01-investigacion-dominio.md)
