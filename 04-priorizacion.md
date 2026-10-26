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

## 1. Propuesta recomendada: "dinero íntegro, integrado y a tiempo"

| Atributo | Puntos | Nivel | En una línea |
|----------|:------:|-------|--------------|
| **Seguridad** | **4** | Alto | El dinero de terceros y la concesión dependen de que no se pague a quien no se debe |
| **Fiabilidad** | **4** | Alto | Ninguna transacción confirmada puede perderse, y la continuidad la exige el regulador |
| **Eficiencia de desempeño** | **3** | Medio-alto | Unas 3.000 operaciones de saldo por segundo, con respuestas que el proveedor espera en tiempo |
| **Compatibilidad** | **3** | Medio-alto | Cada jugada pasa por un proveedor externo, y el regulador recibe cada transacción |
| Adecuación funcional | 1 | Bajo | Las funciones las dicta el reglamento y la lógica de juego es del proveedor |
| Usabilidad | 1 | Bajo | La fricción la fija la regulación y la interfaz del juego es del proveedor |
| Mantenibilidad | 1 | Bajo | El ritmo de cambio lo marca el laboratorio, no el código |
| Portabilidad | 1 | Bajo | El entorno lo fija la regulación; no hay ninguna tensión fuerte |
| **Total** | **18** | | |

**Qué cuenta como "alto".** Los 10 puntos libres se reparten **solo** entre estos cuatro atributos (3 + 3 + 2 + 2); los otros cuatro se quedan en el mínimo. Por eso Desempeño y Compatibilidad, con 3, se consideran altos para la prueba de criticidad: concentran puntos que se les quitaron a los demás.

**Para el widget:** la versión del widget de ISO 25010 que compartió el profesor no trae el campo de presupuesto de puntos que menciona el material. En la pestaña "Ronda en vivo" se cargan estos valores en los deslizadores y la suma de 18 se verifica a mano (sección 6). El radar queda cargado hacia el cuadrante Seguridad–Fiabilidad–Desempeño–Compatibilidad.

---

## 2. Justificación de cada atributo

### 2.1 Atributos altos

**Seguridad (4).**
- El operador custodia el dinero de los jugadores, y cada depósito y retiro pasa por SARLAFT. Eso implica perfil de riesgo, listas restrictivas, reconsulta de 350.000 cuentas en ≤ 6 h y diligencia reforzada (ficha §4.4).
- El equipo ya tomó una postura explícita: **fallo cerrado** en los retiros, porque legalmente es más grave pagar dinero de lavado que demorar un retiro (ficha, restricción 9).
- El reglamento exige bloqueo a los 5 intentos, detección de suplantación, cifrado de ≥ 2048 bits, ISO 27001 como guía y pruebas de penetración (C3, C4, E6–E8).
- Está en tres tensiones fuertes: T1, T4 y T5.

**Fiabilidad (4).**
- **Ninguna transacción de saldo confirmada puede perderse.** El saldo nunca puede ser negativo (C6), y toda pérdida de datos se reporta de inmediato a Coljuegos (E5).
- La continuidad es una obligación regulatoria, no una aspiración: hay que operar en un centro Tier III, mantener un sistema alterno certificado al día, cumplir los escenarios mínimos del plan de continuidad, y Coljuegos puede exigir tiempos de recuperación menores (A1, E1–E4).
- Metas de la ficha: 99,9 % mensual para juego y saldo, con recuperación en ≤ 15 min (§4.3).
- Cada minuto caído empuja jugadores hacia los sitios ilegales [F31].
- Está en dos tensiones fuertes: T2 y T3.

**¿Por qué Seguridad y Fiabilidad empatan en 4 y no hay un 5?** Subir cualquiera de las dos a 5 obliga a bajar Desempeño o Compatibilidad a 2. Eso sacaría del foco una tensión fuerte (ver la variante B en la sección 5). Cuando Seguridad y Fiabilidad chocan (L1), el desempate **ya está resuelto como regla explícita**: fallo cerrado, gana Seguridad. No hace falta gastar un punto en expresarlo.

**Eficiencia de desempeño (3).**
- En el pico de diseño hay unas **3.000 operaciones de saldo por segundo**, con ráfagas de 4.000. La meta es **p99 ≤ 250 ms** (ficha §4.1, §4.2).
- El *crash* concentra hasta 720 retiros en el mismo segundo.
- La reconsulta masiva de listas convive con el juego en el pico.
- Está en tres tensiones fuertes: T1, T2 y T6.

*¿Por qué no 4?* La meta interna de 250 ms deja un margen amplio frente al tiempo de espera del proveedor, unos 3 s (S14). Un problema de desempeño casi siempre se traduce en lentitud, no en dinero perdido. Además, subirlo a 4 bajaría la Compatibilidad a 2.

**Compatibilidad (3).**
- El catálogo completo llega por integración: **5 proveedores de juego con protocolos distintos**, que piden descontar, abonar o revertir el saldo en **cada jugada**, con mensajes repetidos y reversiones automáticas (ficha §3).
- A eso se suman 5 medios de pago, la verificación de identidad y las listas restrictivas.
- El **sistema de inspección** es interoperabilidad obligatoria: formato XML fijado por Coljuegos, sin huecos (A2, A3).
- Es el reto de interoperabilidad que motivó la elección del tema.
- Está en una tensión fuerte, T3, contra un atributo alto (Fiabilidad).

*¿Por qué no 4?* Los protocolos los definen los proveedores y el regulador; el operador se adapta a ellos. Sus fallas se pagan en Fiabilidad, que ya está en 4.

### 2.2 Atributos bajos

**Adecuación funcional (1).**
- Lo que el sistema debe hacer está **enumerado por el reglamento**: campos de registro, autoexclusión, historial de 30 días, límites, información en pantalla, entre otros (sección C de la investigación). Se cumple o no se cumple; no da margen para decisiones de arquitectura.
- La lógica que determina resultados y el retorno al jugador son **del proveedor** y están certificados por un laboratorio (D1–D3).
- **Costo aceptado:** la tensión T6 (corrección del *crash* frente a desempeño) deja de ser crítica. Es defendible porque el premio del *crash* lo calcula el proveedor. Al operador le corresponde responder a tiempo, y eso ya está cubierto por Desempeño.

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
| Seguridad y Fiabilidad a 4 | Usabilidad a 1 | Más fricción que los competidores, incluidos los ilegales. Los retiros pueden detenerse por fallo cerrado |
| Compatibilidad a 3 | Mantenibilidad a 1 | Cada cambio regulatorio y cada proveedor nuevo cuesta más; no invertimos en acelerar el cambio |
| Desempeño a 3 | Adecuación funcional a 1 | La corrección del *crash* queda delegada al proveedor; no la tratamos como decisión propia |
| Que Desempeño y Compatibilidad sigan en 3 | Que Seguridad no llegue a 5 | La preferencia por la seguridad se expresa como regla (fallo cerrado), no como puntaje |
| Todo lo anterior | Portabilidad a 1 | Mover el sistema a otro entorno será caro y obligará a recertificar |

---

## 4. Qué tensiones quedan críticas con esta priorización

Un escenario es crítico si su atributo dominante está entre los más altos **y** cumplirlo cuesta otro atributo alto. Los altos aquí son Seguridad (4), Fiabilidad (4), Desempeño (3) y Compatibilidad (3).

| Tensión | Atributos | ¿Ambos altos? | ¿Candidata a escenario crítico? |
|---------|-----------|:-------------:|:-------------------------------:|
| T1 | Seguridad (4) × Desempeño (3) | ✔ | **Sí** |
| T2 | Fiabilidad (4) × Desempeño (3) | ✔ | **Sí** |
| T3 | Compatibilidad (3) × Fiabilidad (4) | ✔ | **Sí** |
| L1 | Seguridad (4) × Fiabilidad (4) | ✔ | Posible: es leve, pero está entre dos altos y tiene la regla de fallo cerrado |
| T4 | Seguridad (4) × Mantenibilidad (1) | ✘ | No: el costo cae en un atributo bajo |
| T5 | Seguridad (4) × Usabilidad (1) | ✘ | No |
| T6 | Desempeño (3) × Adecuación funcional (1) | ✘ | No |

**Lectura para la fase 5:** tres tensiones fuertes quedan entre atributos altos. Eso apunta a **3 o 4 escenarios críticos**, y por lo tanto a 3 o 4 ADRs. T4, T5 o T6 pueden aportar un escenario **no crítico** que muestre el criterio de selección.

---

## 5. Variantes alternativas

Las dos variantes también suman 18. Cambian qué tensiones quedan críticas.

| Atributo | **Recomendada** | B · "Seguridad absoluta" | C · "Ritmo regulatorio" |
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
| Tensiones críticas | T1, T2, T3 | T1, T2 | T1, T2, T4 |
| Qué se pierde frente a la recomendada | — | T3 sale del foco: la interoperabilidad con proveedores, el tema que motivó el proyecto, deja de tener ADR | T3 sale del foco y la Compatibilidad queda en el mínimo, aunque el sistema de inspección es obligatorio |
| Qué se gana | — | La postura de fallo cerrado se refleja en el puntaje | La volatilidad regulatoria (impuestos, premio inmediato, revisión técnica) gana un ADR propio |

**Por qué recomendamos la primera:** cubre los cuatro retos que motivaron el tema (tiempo real, seguridad, interoperabilidad y regulación). Además deja tres tensiones fuertes entre atributos altos, lo que da material para ADRs bien fundados, y conserva la decisión de fallo cerrado como regla explícita.

---

## 6. Verificación aritmética

| Comprobación | Recomendada | B | C |
|--------------|:-----------:|:-:|:-:|
| Suma = 18 | 4+4+3+3+1+1+1+1 = **18** ✔ | 5+4+3+2+1+1+1+1 = **18** ✔ | 4+4+3+1+3+1+1+1 = **18** ✔ |
| Todos ≥ 1 y ≤ 5 | ✔ | ✔ | ✔ |
| Como máximo tres atributos ≥ 4 | 2 ✔ | 2 ✔ | 2 ✔ |
| Obligó a bajar algún atributo (puerta 4) | ✔ Cuatro en el mínimo | ✔ | ✔ |

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

1. **El equipo adopta la propuesta recomendada:** Seguridad 4, Fiabilidad 4, Desempeño 3, Compatibilidad 3, Adecuación funcional 1, Usabilidad 1, Mantenibilidad 1, Portabilidad 1.
2. **Mantenibilidad en 1 se mantiene, con el riesgo asumido explícitamente.** El equipo acepta el costo de la regulación cambiante y se apoya en una seguridad fuerte: un sistema con controles sólidos tiene más probabilidad de seguir cumpliendo cuando cambian las reglas, aunque cambiarlo cueste más.
