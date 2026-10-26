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
| 7 | Selección de escenarios críticos | Logrado | `05` §2 (prueba de criticidad a 9 candidatos), §3 (justificación en una línea); `07` "Selección de los ADR" | **Medio:** 3 ADRs y no 4 (ver riesgo R4) |
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
| R2 | El dominante de ESC-03 (Compatibilidad) tiene 3, no 4. ¿Está "entre los más altos"? | Los 10 puntos libres se concentran solo en los cuatro altos; los demás quedaron en 1 (`04` §1, párrafo "Qué cuenta como alto"). Además, cumplirlo cuesta Fiabilidad, que tiene 4. |
| R3 | Varias cifras clave son estimaciones (S10, S14, S15, S23–S25). | Están declaradas con su razón, como pide el plan. Las respuestas "Depende" de los ADR muestran que lo que falta se resuelve midiendo. |
| R4 | Al comienzo se habló de 4 ADRs y la entrega trae 3. | El material pide un ADR por escenario crítico, y solo hay tres tensiones fuertes entre atributos altos. ESC-04 sin ADR demuestra el criterio; escribir un ADR para todos es el nivel Inicial de la rúbrica. |

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
