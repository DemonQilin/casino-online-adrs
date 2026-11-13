# Fase 4 — Priorización con presupuesto de 18 puntos

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Insumos:**
- `02-ficha-contexto.md`: cifras y supuestos [S#].
- `03-mapa-tensiones.md`: tensiones T1–T6 y L1–L13.
- `01-investigacion-dominio.md`: obligaciones [F#] y códigos A1, B3, C6…

**Reglas del presupuesto (del material del curso):**
- Cada uno de los 8 atributos de ISO 25010 recibe de 1 a 5 puntos.
- La suma es exactamente **18**.
- Como cada atributo vale mínimo 1, quedan **10 puntos libres** para repartir.
- Consecuencia: caben como máximo **tres atributos en 4 o más**, y es imposible tener tres en 5.
- Se justifica **cada atributo alto y cada atributo bajo**, y debe verse **qué se bajó para poder subir otro**.

---

## 1. Reparto adoptado: "dinero íntegro, integrado y a tiempo"

> **Reparto vigente desde el 13 de noviembre de 2026.** Tras la exposición, el profesor señaló que la Adecuación funcional podría ser más alta. El equipo la subió de 1 a 2 y bajó la Fiabilidad de 4 a 3. El reparto inicial se conserva en la sección 5.1, las variantes del ajuste están en la sección 5.2 y el cambio queda registrado en la sección 8 y en `08-revision.md` §8.

| Atributo | Puntos | Nivel | En una línea |
|----------|:------:|-------|--------------|
| **Seguridad** | **4** | Alto | El dinero de terceros y la concesión dependen de que no se pague a quien no se debe |
| **Fiabilidad** | **3** | Medio-alto | Ninguna transacción confirmada puede perderse, y la continuidad la exige el regulador |
| **Eficiencia de desempeño** | **3** | Medio-alto | Unas 3.000 operaciones de saldo por segundo, con respuestas que el proveedor espera en tiempo |
| **Compatibilidad** | **3** | Medio-alto | Cada jugada pasa por un proveedor externo, y el regulador recibe cada transacción |
| Adecuación funcional | 2 | Medio-bajo | El reglamento enumera las funciones, pero cumplirlas completas y calcular bien el dinero le corresponde al operador |
| Usabilidad | 1 | Bajo | La fricción la fija la regulación y la interfaz del juego es del proveedor |
| Mantenibilidad | 1 | Bajo | El ritmo de cambio lo marca el laboratorio, no el código |
| Portabilidad | 1 | Bajo | El entorno lo fija la regulación; no hay ninguna tensión fuerte |
| **Total** | **18** | | |

**Qué cuenta como "alto".** Es alto un atributo con 3 puntos o más. De los 10 puntos libres, 9 se concentran en cuatro atributos (3 + 2 + 2 + 2). El punto restante saca a la Adecuación funcional del mínimo, pero no la lleva al grupo de los altos. Por eso Fiabilidad, Desempeño y Compatibilidad, con 3, se consideran altos para la prueba de criticidad: concentran puntos que se les quitaron a los demás.

**Para el widget:** la versión del widget de ISO 25010 que compartió el profesor no trae el campo de presupuesto de puntos que menciona el material. En la pestaña "Ronda en vivo" se cargan estos valores en los deslizadores y la suma de 18 se verifica a mano (sección 6). El radar queda cargado hacia el cuadrante Seguridad–Fiabilidad–Desempeño–Compatibilidad.

---

## 2. Justificación de cada atributo

### 2.1 Atributos altos

**Seguridad (4).**
- El operador custodia el dinero de los jugadores, y cada depósito y retiro pasa por SARLAFT. Eso implica perfil de riesgo, listas restrictivas, reconsulta de 350.000 cuentas en ≤ 6 h y diligencia reforzada (ficha §4.4).
- El equipo ya tomó una postura explícita: **fallo cerrado** en los retiros, porque legalmente es más grave pagar dinero de lavado que demorar un retiro (ficha, restricción 9).
- El reglamento exige bloqueo a los 5 intentos, detección de suplantación, cifrado de ≥ 2048 bits, ISO 27001 como guía y pruebas de penetración (C3, C4, E6–E8).
- Está en cuatro tensiones fuertes: T1, T4, T5 y T7.

**Fiabilidad (3).**
- **Ninguna transacción de saldo confirmada puede perderse.** El saldo nunca puede ser negativo (C6), y toda pérdida de datos se reporta de inmediato a Coljuegos (E5).
- La continuidad es una obligación regulatoria, no una aspiración: hay que operar en un centro Tier III, mantener un sistema alterno certificado al día, cumplir los escenarios mínimos del plan de continuidad, y Coljuegos puede exigir tiempos de recuperación menores (A1, E1–E4).
- Metas de la ficha: 99,9 % mensual para juego y saldo, con recuperación en ≤ 15 min (§4.3).
- Cada minuto caído empuja jugadores hacia los sitios ilegales [F31].
- Está en dos tensiones fuertes: T2 y T3.

*¿Por qué no 4?* Parte de lo que antes se le cargaba a Fiabilidad es corrección funcional, según el widget de ISO 25010: que el saldo quede bien calculado y nunca sea negativo es una regla del negocio, además de una garantía (sinergia S2). Ese punto pasó a la Adecuación funcional. Lo que queda en Fiabilidad sigue siendo alto: no perder lo confirmado, recuperarse a tiempo y mantener el sistema alterno al día. Sus dos tensiones fuertes, T2 y T3, siguen siendo críticas.

**¿Por qué Seguridad es el único 4 y no hay un 5?** Subir Seguridad a 5 obliga a bajar Fiabilidad, Desempeño o Compatibilidad a 2, y eso sacaría del foco dos tensiones fuertes (ver la variante B en la sección 5.1). La otra salida sería devolver la Adecuación funcional al mínimo. Cuando Seguridad y Fiabilidad chocan (L1), el desempate **ya está resuelto como regla explícita**: fallo cerrado, gana Seguridad. Ahora el puntaje dice lo mismo que la regla.

**Eficiencia de desempeño (3).**
- En el pico de diseño hay unas **3.000 operaciones de saldo por segundo**, con ráfagas de 4.000. La meta es **p99 ≤ 250 ms** (ficha §4.1, §4.2).
- El *crash* concentra hasta 720 retiros en el mismo segundo.
- La reconsulta masiva de listas convive con el juego en el pico.
- Está en tres tensiones fuertes: T1, T2 y T6.

*¿Por qué no 4?* La meta interna de 250 ms deja un margen amplio frente al tiempo de espera del proveedor, unos 3 s (S14). Un problema de desempeño casi siempre se traduce en lentitud, no en dinero perdido. Además, subirlo a 4 obligaría a bajar otro atributo alto a 2 o a devolver la Adecuación funcional al mínimo.

**Compatibilidad (3).**
- El catálogo completo llega por integración: **5 proveedores de juego con protocolos distintos**, que piden descontar, abonar o revertir el saldo en **cada jugada**, con mensajes repetidos y reversiones automáticas (ficha §3).
- A eso se suman 5 medios de pago, la verificación de identidad y las listas restrictivas.
- El **sistema de inspección** es interoperabilidad obligatoria: formato XML fijado por Coljuegos, sin huecos (A2, A3).
- Es el reto de interoperabilidad que motivó la elección del tema.
- Está en dos tensiones fuertes contra atributos altos: T3 (contra Fiabilidad) y T7 (contra Seguridad).

*¿Por qué no 4?* Los protocolos los definen los proveedores y el regulador; el operador se adapta a ellos. Sus fallas se pagan en Fiabilidad, que ya está entre los altos.

### 2.2 Atributo intermedio

**Adecuación funcional (2).** *Subió de 1 a 2 el 13 de noviembre de 2026.*
- **Completitud.** Lo que el sistema debe hacer está **enumerado por el reglamento**: campos de registro, autoexclusión, historial de 30 días, límites, información en pantalla, entre otros (sección C de la investigación). Que la lista venga de la norma no le quita peso: dejar un caso sin cubrir es un incumplimiento ante Coljuegos. El widget de ISO 25010 ilustra la completitud funcional justamente con un sistema que cubre todos los casos que exige la ley.
- **Corrección propia del operador.** Hay reglas del dinero que calcula el operador y no el proveedor: el saldo nunca negativo (C6), los límites de depósito (C9), la autoexclusión que bloquea apuestas pero permite retirar (C5), el plazo de la orden de retiro (C7) y las reglas fiscales que cambian por decreto (L8).
- *¿Por qué no 3?* La lógica que determina resultados y el retorno al jugador son **del proveedor** y están certificados por un laboratorio (D1–D3). Además, el reglamento deja poco margen de decisión: la mayoría de estas funciones se cumple o no se cumple. Con 3 sería un atributo alto: T6 pasaría a crítica, haría falta un quinto ADR y el punto saldría de Seguridad, que perdería su 4 (variante E, sección 5.2).
- **Costo aceptado:** la tensión T6 (corrección del *crash* frente a desempeño) sigue sin ser crítica. Es defendible porque el premio del *crash* lo calcula el proveedor. Al operador le corresponde responder a tiempo, y eso ya está cubierto por Desempeño.

### 2.3 Atributos bajos

**Usabilidad (1).**
- La fricción principal está **fijada por la regulación** y no se puede negociar: 5 intentos, 20 minutos, registro con muchos campos, información obligatoria en pantalla (C1, C3, C12, C13).
- La interfaz de los juegos, que es donde el jugador pasa su tiempo, **la hace el proveedor**.
- El equipo ya eligió seguridad sobre experiencia (fallo cerrado).
- **Costo aceptado:** competimos con sitios ilegales sin fricción [F31]. La tensión T5 deja de ser crítica, y la experiencia del lobby y de la cuenta no recibe inversión preferente.

**Mantenibilidad (1).**
- El ritmo real de cambio lo marca el **laboratorio**: de 2 a 4 semanas por cada cambio crítico, más la recertificación anual (B2, B3, S10). Invertir en un código muy fácil de cambiar no acelera ese cuello de botella.
- **Costo aceptado:** 3 o 4 cambios regulatorios al año y 20 juegos nuevos al mes costarán más de lo deseable, y la tensión T4 deja de ser crítica.
- **Es el bajo más arriesgado** de la propuesta (ver la variante C).

**Portabilidad (1).**
- Es el único atributo **sin ninguna tensión fuerte** (mapa §5.1).
- El entorno está fijado por la regulación: inspección en Colombia, centro Tier III, entorno certificado (A1, A2).
- Lo único parecido a portar, el sistema alterno, ya está cubierto por Fiabilidad.
- **Costo aceptado:** cambiar de centro de datos o de proveedor de infraestructura será caro, y además exige recertificar (L12).

---

## 3. Lo que nos costó

| Para subir… | …bajamos | Consecuencia que aceptamos |
|-------------|----------|---------------------------|
| Seguridad a 4 y Fiabilidad a 3 | Usabilidad a 1 | Más fricción que los competidores, incluidos los ilegales. Los retiros pueden detenerse por fallo cerrado |
| Compatibilidad a 3 | Mantenibilidad a 1 | Cada cambio regulatorio y cada proveedor nuevo cuesta más; no invertimos en acelerar el cambio |
| Adecuación funcional a 2 | Fiabilidad de 4 a 3 | No buscamos más continuidad que la que exigen el regulador y las metas de la ficha (§4.3); ese punto va a que las reglas del dinero se cumplan completas y exactas |
| Desempeño a 3 | Que Adecuación funcional no pase de 2 | La corrección del *crash* queda delegada al proveedor; no la tratamos como decisión propia |
| Que Fiabilidad, Desempeño y Compatibilidad sigan en 3 | Que Seguridad no llegue a 5 | La preferencia por la seguridad se expresa con el puntaje más alto y con la regla de fallo cerrado, no con un 5 |
| Todo lo anterior | Portabilidad a 1 | Mover el sistema a otro entorno será caro y obligará a recertificar |

---

## 4. Qué tensiones quedan críticas con esta priorización

Un escenario es crítico si su atributo dominante está entre los más altos **y** cumplirlo cuesta otro atributo alto. Los altos aquí son Seguridad (4), Fiabilidad (3), Desempeño (3) y Compatibilidad (3).

| Tensión | Atributos | ¿Ambos altos? | ¿Candidata a escenario crítico? |
|---------|-----------|:-------------:|:-------------------------------:|
| T1 | Seguridad (4) × Desempeño (3) | ✔ | **Sí** |
| T2 | Fiabilidad (3) × Desempeño (3) | ✔ | **Sí** |
| T3 | Compatibilidad (3) × Fiabilidad (3) | ✔ | **Sí** |
| T7 | Seguridad (4) × Compatibilidad (3) | ✔ | **Sí** (promovida a fuerte el 1 de noviembre de 2026) |
| L1 | Seguridad (4) × Fiabilidad (3) | ✔ | Posible: es leve, pero está entre dos altos y tiene la regla de fallo cerrado |
| T4 | Seguridad (4) × Mantenibilidad (1) | ✘ | No: el costo cae en un atributo bajo |
| T5 | Seguridad (4) × Usabilidad (1) | ✘ | No |
| T6 | Desempeño (3) × Adecuación funcional (2) | ✘ | No: con 2, la Adecuación funcional no está entre los altos |

**Lectura para la fase 5:** cuatro tensiones fuertes quedan entre atributos altos (T1, T2, T3 y T7). Eso apunta a **4 escenarios críticos**, y por lo tanto a 4 ADRs. T4, T5 o T6 pueden aportar un escenario **no crítico** que muestre el criterio de selección.

---

## 5. Variantes alternativas

Todas las variantes suman 18. Cambian qué tensiones quedan críticas.

### 5.1 Variantes del punto de control inicial (25 de octubre de 2026)

Se compararon contra el reparto inicial, que tenía Fiabilidad en 4 y Adecuación funcional en 1.

| Atributo | **Inicial** | B · "Seguridad absoluta" | C · "Ritmo regulatorio" |
|----------|:---------------:|:------------------------:|:-----------------------:|
| Seguridad | 4 | **5** | 4 |
| Fiabilidad | 4 | 4 | 4 |
| Desempeño | 3 | 3 | 3 |
| Compatibilidad | 3 | **2** | **1** |
| Mantenibilidad | 1 | 1 | **3** |
| Adecuación funcional | 1 | 1 | 1 |
| Usabilidad | 1 | 1 | 1 |
| Portabilidad | 1 | 1 | 1 |
| **Total** | **18** | **18** | **18** |
| Tensiones críticas | T1, T2, T3, T7 | T1, T2 | T1, T2, T4 |
| Qué se pierde frente a la inicial | — | T3 y T7 salen del foco: la interoperabilidad con proveedores, el tema que motivó el proyecto, deja de tener ADR | T3 y T7 salen del foco y la Compatibilidad queda en el mínimo, aunque el sistema de inspección es obligatorio |
| Qué se gana | — | La postura de fallo cerrado se refleja en el puntaje | La volatilidad regulatoria (impuestos, premio inmediato, revisión técnica) gana un ADR propio |

**Por qué se adoptó la primera:** cubre los cuatro retos que motivaron el tema (tiempo real, seguridad, interoperabilidad y regulación). Además deja cuatro tensiones fuertes entre atributos altos, lo que da material para ADRs bien fundados, y conserva la decisión de fallo cerrado como regla explícita.

### 5.2 Variantes del ajuste por retroalimentación (13 de noviembre de 2026)

El profesor indicó que la Adecuación funcional podría ser más alta, sin fijar un puntaje. Como los otros tres bajos ya están en el mínimo, cada punto que sube tiene que salir de un atributo alto.

| Atributo | **Vigente** · sale de Fiabilidad | D · sale de Seguridad | E · "Funcional alta" |
|----------|:--------------------------------:|:---------------------:|:--------------------:|
| Seguridad | 4 | **3** | **3** |
| Fiabilidad | **3** | 4 | **3** |
| Desempeño | 3 | 3 | 3 |
| Compatibilidad | 3 | 3 | 3 |
| Adecuación funcional | **2** | **2** | **3** |
| Usabilidad | 1 | 1 | 1 |
| Mantenibilidad | 1 | 1 | 1 |
| Portabilidad | 1 | 1 | 1 |
| **Total** | **18** | **18** | **18** |
| Tensiones críticas | T1, T2, T3, T7 | T1, T2, T3, T7 | T1, T2, T3, T7 y T6 |
| Qué se pierde | Fiabilidad deja de estar en 4, aunque la continuidad es obligación regulatoria | Fiabilidad queda por encima de Seguridad, y eso contradice la regla de fallo cerrado, donde gana Seguridad | Ningún atributo queda en 4: el reparto pierde jerarquía entre los altos. ESC-04 pasa a crítico y exige un quinto ADR |
| Qué se gana | Los cuatro ADR no cambian y el puntaje coincide con la regla de fallo cerrado | Los cuatro ADR no cambian | La corrección del *crash* gana un ADR propio |

**Variante descartada sin tabla:** sacar el punto de Desempeño o de Compatibilidad. El atributo que baja a 2 deja de ser alto y arrastra dos tensiones críticas con sus ADR (T1 y T2, o T3 y T7).

**Por qué se adoptó la primera:** el punto que sale de Fiabilidad es el que la sinergia S2 ya reconocía como compartido con la Adecuación funcional, así que el cambio reubica un peso que estaba mal atribuido. Además conserva los cuatro ADR y deja el puntaje alineado con la regla de fallo cerrado. La variante E queda como alternativa si el profesor espera que la corrección funcional tenga una decisión de arquitectura propia.

---

## 6. Verificación aritmética

| Comprobación | Vigente | D | E | Inicial | B | C |
|--------------|:-------:|:-:|:-:|:-------:|:-:|:-:|
| Suma = 18 | 4+3+3+3+2+1+1+1 = **18** ✔ | 3+4+3+3+2+1+1+1 = **18** ✔ | 3+3+3+3+3+1+1+1 = **18** ✔ | 4+4+3+3+1+1+1+1 = **18** ✔ | 5+4+3+2+1+1+1+1 = **18** ✔ | 4+4+3+1+3+1+1+1 = **18** ✔ |
| Todos ≥ 1 y ≤ 5 | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |
| Como máximo tres atributos ≥ 4 | 1 ✔ | 1 ✔ | 0 ✔ | 2 ✔ | 2 ✔ | 2 ✔ |
| Obligó a bajar algún atributo (puerta 4) | ✔ Tres en el mínimo | ✔ | ✔ | ✔ Cuatro en el mínimo | ✔ | ✔ |

---

## 7. Verificación de la fase

| Criterio | Estado |
|----------|--------|
| Respeta los 18 puntos y obligó a bajar atributos (puerta 4) | ✔ |
| Cada atributo alto y bajo justificado con el contexto | ✔ Sección 2, con referencias a la ficha y al mapa |
| Se ve el costo de subir uno en lo que bajó | ✔ Sección 3 |
| Rúbrica, criterio *Priorización*, nivel "Logrado" | Justificación de altos y bajos más costo explícito |
| Puente hacia la *Coherencia con la priorización* | ✔ La sección 4 anticipa qué tensiones pueden ser críticas |

---

## 8. Punto de control (cerrado el 25 de octubre de 2026)

1. **El equipo adopta la propuesta recomendada:** Seguridad 4, Fiabilidad 4, Desempeño 3, Compatibilidad 3, Adecuación funcional 1, Usabilidad 1, Mantenibilidad 1, Portabilidad 1. *Actualizado por el punto 4.*
2. **Mantenibilidad en 1 se mantiene, con el riesgo asumido explícitamente.** El equipo acepta el costo de la regulación cambiante y se apoya en una seguridad fuerte: un sistema con controles sólidos tiene más probabilidad de seguir cumpliendo cuando cambian las reglas, aunque cambiarlo cueste más.
3. **1 de noviembre de 2026:** con la promoción de T7 (Seguridad × Compatibilidad) a tensión fuerte, la misma priorización deja cuatro tensiones críticas. Los puntajes no cambian.
4. **13 de noviembre de 2026:** tras la exposición, el profesor señaló que la Adecuación funcional podría ser más alta. **El equipo adopta el reparto vigente:** Seguridad 4, Fiabilidad 3, Desempeño 3, Compatibilidad 3, Adecuación funcional 2, Usabilidad 1, Mantenibilidad 1, Portabilidad 1. Las tensiones críticas no cambian (T1, T2, T3 y T7). La variante E (sección 5.2) queda como alternativa si el profesor pide más.