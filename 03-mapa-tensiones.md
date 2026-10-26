# Fase 3 — Mapa de tensiones

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Insumos:**
- `02-ficha-contexto.md`: las cifras citadas salen de ahí; [S#] son sus supuestos.
- `01-investigacion-dominio.md`: las referencias [F#] y los códigos de obligación (A1, B3, C6…) remiten a su sección 1.2.
- La pestaña **Tensiones** del widget de ISO 25010 del curso, como punto de partida.

**Método:**
1. Se recorrieron los **28 pares** posibles entre las 8 características de ISO 25010.
2. Cada par se clasificó según cómo se manifiesta **en este casino, con sus cifras**, no en abstracto.
3. Para cada tensión se indica **quién pierde** si se prioriza uno u otro atributo.

**Leyenda:**
- **F** = tensión fuerte
- **L** = tensión leve
- **S** = sinergia
- **·** = sin relación relevante en este proyecto

Como en la fase anterior, aquí se describen necesidades en conflicto, no soluciones.

---

## 1. Matriz resumen

|  | Func. | Desemp. | Compat. | Usab. | Fiab. | Segur. | Manten. | Portab. |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Adecuación funcional** | — | **F** (T6) | S | L | S | L | L | · |
| **Eficiencia de desempeño** | **F** (T6) | — | L | L | **F** (T2) | **F** (T1) | L | L |
| **Compatibilidad** | S | L | — | S | **F** (T3) | L | L | S |
| **Usabilidad** | L | L | S | — | S | **F** (T5) | · | L |
| **Fiabilidad** | S | **F** (T2) | **F** (T3) | S | — | L | L | S |
| **Seguridad** | L | **F** (T1) | L | **F** (T5) | L | — | **F** (T4) | L |
| **Mantenibilidad** | L | L | L | · | L | **F** (T4) | — | S |
| **Portabilidad** | · | L | S | L | S | L | S | — |

**Conteo de los 28 pares:**

| Tipo | Cantidad |
|------|---------:|
| Tensión fuerte | 6 |
| Tensión leve | 13 |
| Sinergia | 7 |
| Sin relación relevante | 2 |

---

## 2. Tensiones fuertes

Estas son las candidatas naturales a escenarios críticos.

### T1 · Seguridad × Eficiencia de desempeño

**Situación en Casino Andino.** Cada depósito y cada retiro deben pasar por varios controles:
- evaluación de riesgo SARLAFT que actualiza el perfil del jugador (p95 ≤ 1 s);
- detección de suplantación por cambios bruscos de comportamiento (C4);
- verificación contra listas restrictivas, con la regla de que **ningún retiro se paga a alguien que ya esté en una lista recibida**.

Además, la **reconsulta de las 350.000 cuentas en ≤ 6 h** debe convivir con el pico de ~3.000 operaciones de saldo por segundo sin degradar el juego. Y las comunicaciones transaccionales van cifradas con claves de ≥ 2048 bits (E7).

**Si se prioriza Seguridad**, pierde Desempeño:
- cada control añade tiempo a la confirmación del depósito (≤ 10 s) y a la orden del retiro;
- la reconsulta masiva compite por recursos con el juego en el pico.

**Si se prioriza Desempeño**, pierde Seguridad:
- controles más livianos o diferidos dejan ventanas en las que se paga a quien no se debería;
- se pueden pasar por alto señales de lavado o de suplantación.

**Evidencia:** ficha §4.2, §4.4, S18–S20; F1 (C4, E7); F9.

### T2 · Fiabilidad × Eficiencia de desempeño

**Situación en Casino Andino.** **Ninguna transacción de saldo confirmada puede perderse.** El sistema alterno certificado debe quedar al día **de inmediato** (E1), y al recuperarse de una falla hay que resolver las jugadas incompletas (D6). Al mismo tiempo, el operador debe responder cada operación pedida por un proveedor en **p99 ≤ 250 ms**, con ~3.000 operaciones por segundo y ráfagas de 4.000.

**Si se prioriza Fiabilidad**, pierde Desempeño. Garantizar que cada operación quedó a salvo, también en el sistema alterno, antes de confirmarla al proveedor alarga cada respuesta y reduce cuántas caben por segundo.

**Si se prioriza Desempeño**, pierde Fiabilidad. Confirmar antes de asegurar la operación abre la posibilidad de perder transacciones ya confirmadas ante una falla. Eso viola directamente el requisito de pérdida cero y la regla de saldo nunca negativo (C6).

**Evidencia:** ficha §4.2, §4.3, S6, S13, S14; F1 (C6, D6, E1, E5).

### T3 · Compatibilidad × Fiabilidad

**Situación en Casino Andino.** Los **5 proveedores** piden descontar, abonar o revertir el saldo **en cada jugada**, cada uno con su propio protocolo. Cada operación trae un identificador (en algunos protocolos, único solo por jugador) y puede llegar **repetida**. Si el operador no responde en ~3 s, el proveedor **revierte por su cuenta** [F24][F26]. El sistema de inspección debe capturar **cada** transacción en el formato de Coljuegos, sin huecos.

**Si se prioriza Compatibilidad** (aceptar la variedad de protocolos, sumar proveedores rápido, adaptarse a cada uno), pierde Fiabilidad:
- cada variante es una nueva forma de descontar dos veces, de abonar sin descuento previo o de que el saldo del operador no coincida con el del proveedor;
- una caída del proveedor de listas o de pagos se convierte en una falla del operador.

**Si se prioriza Fiabilidad** (exigir garantías estrictas de cada operación), pierde Compatibilidad. Se rechazan o retrasan operaciones que el proveedor considera válidas, el proveedor revierte jugadas y la integración de proveedores nuevos se vuelve más lenta.

**Evidencia:** ficha §3, S11, S14; F1 (A3, C6); F24; F26.

### T4 · Seguridad × Mantenibilidad

**Situación en Casino Andino.** Todo cambio en un **elemento crítico** (saldo, registro, verificación de identidad, comunicaciones, integridad del juego, reporte) debe validarlo el **laboratorio antes de producción**, lo que tarda **de 2 a 4 semanas** (B3, S10). A esto se suman:
- recertificación integral anual (B2);
- ambientes separados y gestión formal de cambios (B7);
- registros de auditoría protegidos (E6).

Mientras tanto, el negocio pide **entregas cada 2 semanas**, **~20 juegos nuevos al mes** y **3 o 4 cambios regulatorios al año** (impuestos, premio inmediato, revisión de los requerimientos técnicos).

**Si se prioriza Seguridad**, pierde Mantenibilidad. Cada cambio arrastra la validación del laboratorio y el proceso formal, y el ritmo real de entrega cae muy por debajo del deseado.

**Si se prioriza Mantenibilidad**, pierde Seguridad. Acelerar los cambios presiona para tocar elementos críticos sin la validación completa. Eso pone en riesgo la integridad del saldo y del juego, y la propia concesión.

**Nota:** esta tensión es **regulatoria**, no solo técnica. El regulador decide cuánto cuesta cambiar.

**Evidencia:** ficha §4.6, S10, S11; F1 (B2, B3, B5, B7); F13; F14; F15.

### T5 · Seguridad × Usabilidad

**Situación en Casino Andino.** Los controles que ve el jugador son varios:
- bloqueo de la cuenta a los **5 intentos fallidos** (C3);
- cierre de sesión a los **20 minutos** de inactividad (C12);
- verificación de identidad y consulta de listas en el registro;
- **diligencia reforzada** que retiene retiros hasta **48 h** (~60 al día);
- reactivación de cuentas con verificación reforzada.

El usuario es mayoritariamente móvil, con gama media y 4G variable, y compite con **118 marcas**, muchas ilegales y sin estos controles [F31].

**Si se prioriza Seguridad**, pierde Usabilidad. Más fricción en el registro, en el acceso y en los retiros, y jugadores legítimos bloqueados o esperando. Cada fricción empuja jugadores hacia sitios ilegales.

**Si se prioriza Usabilidad**, pierde Seguridad. Menos verificaciones abren la puerta a la suplantación, al lavado de activos y al juego de menores o autoexcluidos.

**Margen de maniobra:** los 5 intentos y los 20 minutos son **piso regulatorio fijo**. La negociación real está en la diligencia reforzada y en la experiencia de registro y reactivación.

**Evidencia:** ficha §1, §4.4, §4.8, S18, S21; F1 (C2, C3, C12); F31.

### T6 · Adecuación funcional (corrección) × Eficiencia de desempeño

**Situación en Casino Andino.** En el *crash*, el premio se calcula con el multiplicador vigente **en el instante del retiro** [F13]. Hasta **720 jugadores pueden retirarse en el mismo segundo**, y el multiplicador liquidado no puede superar el mostrado ni desfasarse más de **0,05x** (S15). En el blackjack en vivo, toda apuesta debe registrarse **antes** de que el servidor cierre la ventana. El reglamento además exige informar al jugador si el tiempo de respuesta afecta su probabilidad de ganar (D5).

**Si se prioriza la corrección funcional** (cada retiro liquidado con el multiplicador exacto, en orden, verificado), pierde Desempeño. Procesar con esa precisión una ráfaga de retiros simultáneos limita el volumen y alarga la respuesta justo en el momento de mayor carga.

**Si se prioriza Desempeño** (absorber la ráfaga como venga), pierde la corrección funcional. Los retiros que se demoran se liquidan con un multiplicador distinto al que vio el jugador, y aparecen reclamos, disputas y posibles incumplimientos de equidad.

**Evidencia:** ficha §2.2, §3, §4.2, S5, S6, S15; F1 (D5); F13; F32.

---

## 3. Tensiones leves

| ID | Par | Situación en Casino Andino | Si se prioriza el primero, pierde el segundo | Si se prioriza el segundo, pierde el primero |
|----|-----|----------------------------|---------------------------------------------|---------------------------------------------|
| L1 | Fiabilidad × Seguridad | **¿Fallar abierto o cerrado?** Si el proveedor de listas o la evaluación de riesgo no responden, ¿se pagan los retiros para cumplir la meta de 24 h, o se detienen? El reglamento ya inclina la balanza en las fallas graves: suspender todo (E3). | Seguridad: pagar sin evaluar abre una ventana a pagos indebidos | Fiabilidad: retiros detenidos mientras dura la falla externa |
| L2 | Fiabilidad × Mantenibilidad | **Contraria a la sinergia típica del widget.** El sistema alterno con la misma certificación exige replicar **de inmediato** cada cambio (E1), así que todo cambio cuesta el doble. | Mantenibilidad: cada entrega se hace y verifica en dos entornos | Fiabilidad: un alterno desfasado no sirve en la contingencia |
| L3 | Compatibilidad × Mantenibilidad | **Contraria a la sinergia típica del widget.** No hay un estándar común entre proveedores de juego: cada uno trae su protocolo. Un juego sobre otra plataforma o versión obliga a **recertificar la integración** (B4). | Mantenibilidad: más variantes que mantener y recertificar | Compatibilidad: menos proveedores y juegos, catálogo más lento |
| L4 | Compatibilidad × Seguridad | Hay 5 proveedores de juego, 5 medios de pago, listas, identidad y el acceso remoto de Coljuegos (A5). Cada integración es una superficie de ataque. | Seguridad: más puntos de entrada que proteger | Compatibilidad: integraciones más lentas y restringidas |
| L5 | Desempeño × Compatibilidad | Cada una de las ~24 M de operaciones diarias debe traducirse al formato XML de Coljuegos (A3), y cada protocolo de proveedor debe traducirse al modelo del operador. | Compatibilidad: presión para simplificar o diferir la captura regulatoria | Desempeño: la traducción consume recursos en el pico |
| L6 | Usabilidad × Desempeño | El multiplicador del *crash*, el video del blackjack y el *jackpot* deben verse actualizados en **todos** los dispositivos (D4), en celulares de gama media y 4G variable, con carga de pantalla p95 ≤ 3 s. | Desempeño: más actualizaciones visuales en tiempo real para miles de dispositivos | Usabilidad: información desactualizada o pantallas más pobres |
| L7 | Adecuación funcional × Usabilidad | El registro obligatorio tiene muchos campos (documento, fecha y lugar de expedición, municipio DIVIPOLA…). Además, la pantalla debe mostrar siempre el tiempo de sesión, el aviso de juego responsable, el saldo y el sello (C13). Todo en pantallas móviles pequeñas. | Usabilidad: interfaz cargada y registro largo | Adecuación funcional: no es opcional, es exigencia regulatoria |
| L8 | Adecuación funcional × Mantenibilidad | Reglas fiscales y regulatorias que cambian por decreto (GGR bimestral, impuesto del 16 %, mecánicas de premio inmediato). Agregarlas sin reestructurar acumula complejidad. | Mantenibilidad: el sistema se vuelve difícil de cambiar | Adecuación funcional: las reglas nuevas tardan en llegar |
| L9 | Adecuación funcional × Seguridad | Historial descargable de 30 días, repetición de las últimas 10 partidas y acceso de Coljuegos: cada función expone datos sensibles. | Seguridad: más superficie de fuga de datos | Adecuación funcional: funciones exigidas que no se pueden quitar |
| L10 | Desempeño × Mantenibilidad | Sostener ~3.000 operaciones por segundo con p99 ≤ 250 ms empuja a optimizaciones especializadas que restan claridad, y cada optimización en un elemento crítico pasa por el laboratorio. | Mantenibilidad: código más difícil de cambiar y de recertificar | Desempeño: no se alcanza la meta en el pico |
| L11 | Desempeño × Portabilidad | Ajustar el sistema al entorno donde corre lo ata a ese entorno. A la vez, el sistema alterno exige poder correr igual en otro sitio (E1). | Portabilidad: más difícil llevarlo al alterno o a otro centro de datos | Desempeño: menos margen para aprovechar el entorno |
| L12 | Seguridad × Portabilidad | La certificación es del entorno concreto. Mover el sistema (otro centro de datos, otro proveedor de infraestructura) reabre pruebas de seguridad y certificación (B1, B3). El sistema de inspección debe estar en Colombia (A2). | Portabilidad: cambiar de entorno es caro y lento | Seguridad: se exponen controles no validados en el entorno nuevo |
| L13 | Usabilidad × Portabilidad | Web de escritorio, web móvil y app Android, con gama media. Las diferencias de funcionalidad entre dispositivos deben documentarse y aceptarse (E10). | Portabilidad: adaptar la experiencia a cada canal impide reutilizar lo mismo en todos | Usabilidad: una experiencia única se siente mal en algún canal |

---

## 4. Sinergias

| ID | Par | Por qué se refuerzan en Casino Andino |
|----|-----|---------------------------------------|
| S1 | Adecuación funcional × Compatibilidad | La oferta de juegos y medios de pago **llega por integración**: sin compatibilidad no hay catálogo ni depósitos. |
| S2 | Adecuación funcional × Fiabilidad | El saldo nunca negativo y la reconstrucción de jugadas son a la vez reglas funcionales y garantías de fiabilidad. |
| S3 | Compatibilidad × Usabilidad | Más medios de pago locales (PSE, Nequi, efectivo) facilitan depositar y retirar al usuario colombiano. |
| S4 | Compatibilidad × Portabilidad | El formato fijado por Coljuegos y los contratos con proveedores son independientes del entorno donde corre el operador. |
| S5 | Usabilidad × Fiabilidad | Que el jugador vea el resultado de su jugada al reconectarse (D6) y reciba el aviso inmediato de pérdida de comunicación es fiabilidad que se percibe como buena experiencia. |
| S6 | Fiabilidad × Portabilidad | El sistema alterno obligatorio exige poder instalar y operar el mismo sistema en dos entornos: la capacidad de instalación es condición de la continuidad. |
| S7 | Mantenibilidad × Portabilidad | Aislar lo que depende del entorno facilita a la vez cambiar el sistema y llevarlo al alterno. |

**Sin relación relevante:**
- **Adecuación funcional × Portabilidad:** no hay funciones atadas a una plataforma.
- **Usabilidad × Mantenibilidad:** no aparece un conflicto concreto con las cifras del proyecto.

---

## 5. Lectura del mapa

### 5.1 Atributos en tensión real

Esta tabla responde a la puerta de revisión 2.

| Atributo | Tensiones fuertes | Leves | Sinergias |
|----------|-------------------|------:|----------:|
| Seguridad | T1, T4, T5 | 4 | 0 |
| Eficiencia de desempeño | T1, T2, T6 | 4 | 0 |
| Fiabilidad | T2, T3 | 2 | 3 |
| Compatibilidad | T3 | 3 | 3 |
| Mantenibilidad | T4 | 4 | 1 |
| Usabilidad | T5 | 3 | 2 |
| Adecuación funcional | T6 | 3 | 2 |
| Portabilidad | — | 3 | 3 |

**Siete de los ocho atributos** participan en al menos una tensión fuerte; solo Portabilidad queda fuera. La puerta pide al menos tres.

### 5.2 Hallazgos para presentar

1. **Seguridad y Desempeño son el centro del mapa.** Cada uno está en tres tensiones fuertes. Casi toda decisión del casino pasa por uno de ellos.
2. **La Mantenibilidad la limita el regulador** (T4), no solo el código. Eso la vuelve costosa de subir en la priorización, pero también peligrosa de bajar demasiado, por la volatilidad regulatoria.
3. **La Portabilidad no tiene tensiones fuertes.** Además está acotada por restricciones fijas (inspección en Colombia, entorno certificado). Es la candidata natural a puntaje bajo.
4. **El *crash* aporta la única tensión fuerte que involucra a la Adecuación funcional** (T6). Justifica haberlo incluido en el alcance.

### 5.3 Insumo para la fase 4

La priorización con 18 puntos deberá decidir cuáles de **Seguridad, Desempeño y Fiabilidad** van arriba (solo caben tres atributos en 4 o más), y qué pasa con **Compatibilidad** y **Mantenibilidad**, que tienen tensiones fuertes pero compiten por los mismos puntos.

---

## 6. Verificación de la fase

| Criterio | Estado |
|----------|--------|
| Se recorrieron los 28 pares | ✔ 6 fuertes, 13 leves, 7 sinergias y 2 sin relación |
| Puerta 2: al menos tres atributos en tensión y cada par nombrado | ✔ Siete atributos en tensiones fuertes, pares nombrados T1–T6 |
| Puerta 3: cada par dice quién pierde | ✔ Cada tensión fuerte y leve indica qué se pierde en ambas direcciones |
| Cada tensión anclada en cifras de la ficha o en obligaciones | ✔ Con referencias a §, S# y F# |
| Sin soluciones | ✔ Se describen necesidades en conflicto, no mecanismos |

---

## 7. Punto de control (cerrado el 25 de octubre de 2026)

1. **El equipo confirma las seis tensiones fuertes.** L1 se mantiene como leve.
   - **Criterio del equipo para L1: fallo cerrado.** Si la evaluación de riesgo o la consulta de listas no están disponibles, los retiros se detienen.
   - Razón: legalmente es más grave pagar dinero proveniente de lavado de activos que demorar un retiro.
   - El criterio quedó registrado en la ficha (§4.4 y restricción 9).
2. **La presentación se enfocará en las tensiones fuertes.** La inversión de sinergias (L2, L3) queda solo como dato del mapa y no se presenta como hallazgo.
