# Fase 8 — Revisión integral

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Documentos revisados:** `00` a `07`
**Fecha:** 26 de octubre de 2026

**Método:**
1. **Autoevaluación (8.1):** cada criterio de la rúbrica, con la evidencia que lo sostiene y el riesgo de que un evaluador lo lea distinto.
2. **Coherencia en cadena (8.2):**
   - se recalcularon con un script las 31 cifras derivadas;
   - cada cifra de los escenarios se rastreó hasta la ficha;
   - las citas clave se contrastaron otra vez con sus fuentes;
   - se verificó, también con un script, que el Context de cada ADR sea idéntico a su escenario.
3. **Revisión en frío (8.3):** queda pendiente para alguien que no haya redactado (sección 5).

---

## Resumen

- **9 hallazgos, todos corregidos.** Uno es de severidad alta (H1) y obliga a cambiar el texto de ESC-03, que el equipo había aprobado. Ese cambio necesita confirmación.
- **Cifras:** las 31 cifras derivadas cuadran. Tres cifras de los escenarios no tenían origen en la ficha y ahora lo tienen (S23–S25).
- **Context de los ADR:** idéntico a su escenario en los tres casos, verificado después de las correcciones.
- **Vocabulario:** ninguna respuesta ni artefacto de los escenarios nombra tácticas ni tecnologías.
- **Autoevaluación:** los 9 criterios se defienden en nivel Logrado. Quedan 4 riesgos de interpretación, con su defensa escrita (sección 4).

---

## 1. Autoevaluación contra la rúbrica

| # | Criterio | Nivel que reclamamos | Evidencia | Riesgo de lectura |
|---|----------|:--------------------:|-----------|-------------------|
| 1 | Ficha de contexto | Logrado | `02` §1 (ficha resumen con cifras de carga, disponibilidad, regulación, presupuesto y usuarios), §4 (requisitos cuantificados), §6 (25 supuestos con razón) | Bajo. Las cifras sin fuente pública están declaradas como supuestos |
| 2 | Priorización | Logrado | `04` §2 (justificación de cada alto y cada bajo), §3 ("Lo que nos costó"), §6 (verificación aritmética) | Bajo |
| 3 | Estímulo y entorno | Logrado | `06`: estímulos observables por el operador; entornos con franja, carga y modo de operación | Bajo, tras corregir ESC-04 (H2) |
| 4 | Respuesta | Logrado | `06`: cada respuesta distingue 2 o 3 casos ("Casos que distingue la respuesta") | Bajo |
| 5 | Medida de la respuesta | Logrado | `06`: cifras con unidad y percentil, y el método de verificación (conciliación, cruce por marca de tiempo) | Bajo, tras rastrear S23–S25 (H3) |
| 6 | Coherencia con la priorización | Logrado | `06` "Resumen" y lista de chequeo: dominante con puntaje y atributo que paga | **Medio:** ESC-03 tiene dominante con puntaje 3 (ver riesgo R2) |
| 7 | Selección de escenarios críticos | Logrado | `05` §2 (prueba de criticidad a 10 candidatos), §3 (justificación en una línea); `07` "Selección de los ADR" | Bajo: 4 ADRs, uno por cada tensión fuerte entre atributos altos, y ESC-04 sin ADR por criterio (sección 7) |
| 8 | ADR: contexto y estado | Logrado | `07`: Context idéntico al escenario (verificado con script), título numerado que nombra la decisión, estado Propuesto y "Pendiente para el Bloque 3" | Bajo |
| 9 | ADR: opciones y trade-offs | Logrado | `07`: tres opciones por ADR, descritas como mecanismos, con lo que gana, lo que paga, el atributo sacrificado y si cumple la medida | Bajo |

---

## 2. Coherencia en cadena

### 2.1 Cifras derivadas

Se recalcularon **31** cifras: concurrencia y minutos-jugador (S4), operaciones por segundo por juego (S6), operaciones diarias y anuales, depósitos y retiros (S7, S8), GGR y presupuesto (S9), cuota de mercado (S1), minutos de caída mensual (S13), ritmo de reconsulta (S19), retiros con diligencia reforzada (S21), anotaciones de ADR-03 y suma de la priorización. **Todas cuadran.**

### 2.2 Cifras de los escenarios rastreadas a la ficha

| Escenario | Cifra | Origen |
|-----------|-------|--------|
| ESC-01 | p95 ≤ 1 s; ≤ 15 min; p99 ≤ 250 ms; ≤ 6 h; ≤ 24 h; 12.000 jugadores; ~1.800 depósitos/h | S20, S16, S14, S19, S8, S4, S7 |
| ESC-02 | ≤ 15 min; ≤ 30 min; p99 ≤ 250 ms; 2.400 apuestas | S13, S22, S14, S5 |
| ESC-02 | **Vuelta a p99 ≤ 250 ms en 5 min** | **No tenía origen → S24** |
| ESC-03 | ~3 s; p99 ≤ 250 ms; 5 proveedores | S14, ficha §1 |
| ESC-03 | **Alerta en ≤ 5 min** | **No tenía origen → S25** |
| ESC-04 | 720 retiros; 2.400 jugadores; p99 ≤ 250 ms | S5, S6, S14 |
| ESC-04 | **Saldo visible en p95 ≤ 2 s** | **No tenía origen → S23** |

Montos como COP 3.000.000 (ESC-01) o COP 50.000 (ESC-03) son ilustrativos del caso, no requisitos.

### 2.3 Contraste con las fuentes

Se volvieron a contrastar las citas de la investigación que alimentan cifras o restricciones (F1, F3, F7, F14, F17, F18, F20–F24, F26–F28, F32). Dos no decían lo que escribimos: H1 (F26) y H4 (F17).

---

## 3. Hallazgos y correcciones

| # | Severidad | Hallazgo | Corrección aplicada | Archivos |
|---|-----------|----------|---------------------|----------|
| **H1** | **Alta** | ESC-03 rechazaba una petición con "el mismo identificador pero otro jugador". Según la fuente F26, en algunos protocolos el identificador es único solo **por jugador**, y dos jugadores pueden compartirlo. El escenario rechazaba operaciones legítimas, y ADR-03 citaba mal la fuente ("único por proveedor"). | Una petición es "la misma" si repite **proveedor, jugador e identificador**. Se ajustaron la respuesta y la medida de ESC-03, se agregó un caso ("mismo identificador para otro jugador → operación nueva") y se corrigieron el Context, la restricción y la opción B de ADR-03. **Cambia texto aprobado: requiere confirmación del equipo.** | `06`, `07`, `01`, `02`, `03` |
| **H2** | Media | En ESC-04 la fuente eran "720 jugadores" y el estímulo, "se retiran y el proveedor envía…": dos eventos, y el primero ocurre en el sistema del proveedor, donde el operador no lo observa. | Fuente: el proveedor del *crash*. Estímulo: envía 720 peticiones de abono en el mismo segundo. ESC-04 no tiene ADR, así que no afecta `07`. | `06` |
| **H3** | Media | Tres cifras de escenarios sin origen en la ficha (tabla 2.2). | Se agregaron los supuestos S23, S24 y S25, sus filas en §4.2 y §4.3 de la ficha y la trazabilidad en cada escenario. | `02`, `06` |
| **H4** | Media | La investigación atribuía el +39,38 % al recaudo online. Recalculado (276.099 / 701.121), es el crecimiento del recaudo **total** de Coljuegos. | Se corrigió la tabla de cifras; se aclara que lo que creció 84,24 % fueron las apuestas deportivas en línea. Ninguna cifra de la ficha dependía de este dato. | `01` |
| **H5** | Baja | La priorización mandaba a "escribir 18 en el presupuesto de puntos" del widget, pero la versión compartida del widget no tiene ese campo. | Nota corregida: se cargan los deslizadores y la suma se verifica a mano. | `04` |
| **H6** | Baja | No quedaba escrito por qué un 3 cuenta como "alto". | Párrafo nuevo: los 10 puntos libres están todos en los cuatro altos; los demás quedaron en el mínimo. | `04` |
| **H7** | Baja | La verificación de la selección decía "entre 6 y 8 candidatos ✔ 9". | Redacción corregida. | `05` |
| **H8** | Baja | La sección de supuestos de los escenarios decía que S22 "debe agregarse a la ficha", cuando ya estaba agregado. | Actualizado. | `06` |
| **H9** | Baja | El plan de trabajo describía un alcance sin el *crash*. | Actualizado. | `00` |

---

## 4. Riesgos que quedan

No son errores, pero un evaluador podría leerlos distinto. Para cada uno dejamos la defensa.

| # | Riesgo | Defensa |
|---|--------|---------|
| R1 | La fuente de ESC-02 es "una falla de hardware". Alguien podría decir que no es un actor. | En escenarios de fiabilidad, la fuente típica es una falla interna. No es una condición del entorno: la carga y la franja horaria sí están en el entorno. |
| R2 | El dominante de ESC-03 (Compatibilidad) tiene 3, no 4. ¿Está "entre los más altos"? | Es alto un atributo con 3 puntos o más: 9 de los 10 puntos libres se concentran en los cuatro altos (`04` §1, párrafo "Qué cuenta como alto"). Además, cumplirlo cuesta Fiabilidad, que también es alta. *Defensa actualizada el 13 de noviembre de 2026 (sección 8).* |
| R3 | Varias cifras clave son estimaciones (S10, S14, S15, S23–S25). | Están declaradas con su razón, como pide el plan. Las respuestas "Depende" de los ADR muestran que lo que falta se resuelve midiendo. |
| R4 | *Resuelto el 1 de noviembre:* la entrega pasa a 4 ADRs (sección 7). El riesgo nuevo es que pregunten por qué T7 no era fuerte desde el comienzo. | `03`, T7, apartado "Por qué se promovió de leve a fuerte": al revisar la primera presentación se vio que la interfaz de saldo es la puerta por la que entra el dinero, y su protección depende de cómo autentica cada proveedor. |

**Vigencia normativa.** Los requerimientos técnicos de Coljuegos están en revisión en 2026, y el Decreto 0240 de 2026 tiene un estado jurídico incierto. Si cambian antes de la entrega, hay que revisar la ficha §4 y las restricciones de los ADR.

---

## 5. Revisión en frío (pendiente)

El plan pide una lectura de alguien que no haya participado en la redacción. Propuesta: un compañero del equipo lee solo la **ficha resumen** (`02` §1), los **escenarios** (`06`) y los **ADR** (`07`), y responde:

1. Con la ficha resumen, ¿puede puntuar los 8 atributos sin preguntar nada? ¿Su puntaje se parece al nuestro?
2. ¿Entiende cada escenario sin leer las notas de apoyo?
3. ¿Puede decir, en cada escenario, qué tendría que medir para saber si se cumple?
4. En cada ADR, ¿entiende por qué la opción descartada no cumple la medida?
5. ¿Encuentra alguna palabra que suene a solución técnica dentro de un escenario?

---

## 6. Punto de control (26 de octubre de 2026)

1. **H1 confirmado por el equipo:** queda la nueva redacción de ESC-03 y del Context de ADR-03.
2. **Revisión en frío pendiente:** el equipo designará más adelante a un compañero que no haya redactado (sección 5).
3. **Estado:** fases 1 a 8 cerradas, salvo la revisión en frío. La **fase 9 (integración** en HTML interactivo o en presentación) arranca cuando el equipo lo decida.

---

## 7. Cambio posterior a la revisión (1 de noviembre de 2026)

**Decisión del equipo:** subir L4 (Seguridad × Compatibilidad) a tensión fuerte **T7** y agregar su escenario y su ADR. La decisión salió de revisar la primera presentación del equipo, que ya trataba la suplantación de proveedores como un conflicto central.

**Qué se agregó:**
- **ESC-05 · Petición de abono falsificada a nombre de un proveedor.** Seguridad (4), con una restricción de compatibilidad (3). Tiene una sola fuente, un solo estímulo y una medida con las dos caras de la tensión: 100 % de lo falsificado rechazado y ≤ 0,001 % de lo legítimo rechazado.
- **ADR-04 · Verificación del origen de las peticiones de saldo de los proveedores.** Tres opciones: clave fija con direcciones permitidas (no cumple), firma por petición con una pieza por proveedor (depende) y esquema único del operador (no cumple, porque deja fuera a proveedores).

**Documentos actualizados:**

| Archivo | Cambio |
|---------|--------|
| `01` | Obligación E11 (autenticar todos los accesos, también los de otros sistemas) y fila en las tensiones preliminares |
| `02` | Requisitos de autenticación de proveedores en §1, §3, §4.2 y §4.4; supuestos S26–S29 |
| `03` | T7 con su situación, quién pierde y por qué se promovió; matriz, conteo (7 fuertes, 12 leves), tabla de atributos y verificación |
| `04` | T7 en la tabla de tensiones críticas, en la justificación de Seguridad y de Compatibilidad, y en las variantes. Los puntajes no cambian |
| `05` | Candidato K10, prueba de criticidad, selección de 5 escenarios con 4 críticos y nueva justificación en una línea |
| `06` | ESC-05, fila en el resumen, columna en la lista de chequeo y nota sobre su relación con ESC-03 |
| `07` | ADR-04, selección y tabla de los ADR, relación ADR-03 ↔ ADR-04 y verificación |

**Verificaciones repetidas:**
- El Context de los **cuatro** ADR es idéntico a su escenario (script).
- La matriz del mapa tiene 7 pares fuertes (14 celdas) y la tabla de leves tiene 12 filas.
- Ninguna respuesta ni artefacto de los cinco escenarios nombra tácticas ni tecnologías.
- Cada cifra de ESC-05 tiene origen en la ficha: S4, S6, S14 y S26–S29.

**Pendiente:** la revisión en frío (sección 5) debe incluir ESC-05 y ADR-04.

---

## 8. Cambio posterior a la exposición (13 de noviembre de 2026)

**Origen:** retroalimentación del profesor a la exposición del Bloque 2. Señaló que la Adecuación funcional podría ser más alta, sin fijar un puntaje.

**Decisión del equipo:** subir la Adecuación funcional de 1 a 2 y bajar la Fiabilidad de 4 a 3. El reparto vigente es Seguridad 4, Fiabilidad 3, Desempeño 3, Compatibilidad 3, Adecuación funcional 2, Usabilidad 1, Mantenibilidad 1 y Portabilidad 1.

**Por qué el profesor tiene razón.** La justificación del 1 decía que las funciones "las dicta el reglamento" y que la lógica de juego es del proveedor. El widget de ISO 25010 del curso contradice el primer argumento: su ejemplo de completitud funcional es un sistema que cubre todos los casos que exige la ley. El segundo argumento solo cubre el resultado del juego: el saldo nunca negativo (C6), los límites de depósito (C9), la autoexclusión (C5), el plazo de la orden de retiro (C7) y las reglas fiscales (L8) los calcula el operador.

**Por qué el punto sale de Fiabilidad.** La sinergia S2 del mapa ya decía que el saldo nunca negativo es a la vez regla funcional y garantía de fiabilidad, pero ese peso estaba atribuido entero a Fiabilidad. Las alternativas están en `04` §5.2: sacar el punto de Seguridad contradice la regla de fallo cerrado, y sacarlo de Desempeño o de Compatibilidad elimina dos ADR.

**Qué no cambia:**
- Las tensiones críticas siguen siendo T1, T2, T3 y T7. Los cuatro atributos altos son los mismos, ahora con 4, 3, 3 y 3.
- ESC-04 sigue sin ADR: con 2, la Adecuación funcional no está entre los altos.
- El texto de los cinco escenarios y de los cuatro ADR. Solo cambian los puntajes que se citan alrededor.

**Documentos actualizados:**

| Archivo | Cambio |
|---------|--------|
| `04` | Reparto vigente, justificación de Fiabilidad (3) y de Adecuación funcional (2), "Lo que nos costó", tensiones críticas, variantes del ajuste (§5.2), verificación aritmética y punto de control |
| `05` | Puntajes de la prueba de criticidad (K3 a K7), justificación en una línea y punto de control |
| `06` | Puntajes del resumen, de las notas de criticidad de ESC-02, ESC-03 y ESC-04, y de la lista de chequeo; punto de control |
| `07` | Puntajes de la sección "Selección de los ADR" y nota de cambio. Ningún ADR se edita |
| `08` | Defensa del riesgo R2 y esta sección |
| `Casino Andino.html` | Constante `ATTRS` (puntajes y textos de Fiabilidad, Adecuación funcional, Seguridad y Compatibilidad), radar, "Qué cuenta como alto", "Lo que nos costó", ESC-04 y preguntas de sustentación |
| `CLAUDE.md` | Reparto adoptado en la tabla de reglas |

**Riesgo nuevo:**

| # | Riesgo | Defensa |
|---|--------|---------|
| R5 | Con Fiabilidad en 3, dos de los cuatro escenarios críticos (ESC-02 y ESC-03) tienen dominante en 3, y T2 y T3 quedan entre dos atributos de 3. Alguien podría decir que ya no son "los más altos". | El corte de atributo alto es 3 y está escrito en `04` §1. Esos tres atributos concentran 6 de los 10 puntos libres, y el siguiente atributo tiene 2. Si el profesor pide más para la Adecuación funcional, la alternativa es la variante E de `04` §5.2, con un quinto ADR. |

**Verificaciones repetidas:**
- El reparto vigente suma 18 (4 + 3 + 3 + 3 + 2 + 1 + 1 + 1), con todos los atributos entre 1 y 5 y un solo atributo en 4 o más.
- El Context de los cuatro ADR sigue idéntico a su escenario: el cambio no toca ninguna fila de los escenarios ni el cuerpo de los ADR.
- No se agregó ninguna cifra nueva a escenarios ni ADR, así que no hay supuestos nuevos.
- La presentación HTML muestra el mismo reparto y los mismos cuatro escenarios críticos.
