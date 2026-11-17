---
equipo: Equipo 8
integrantes: Fabio Guillermo Rodriguez Díaz, Sara Isabel Torres Isaza, Yeison Andres Sanchez Rodas, Juan Esteban Velez Loaiza
nivel: pregrado
caso: proyecto final (pregrado)
nombre_del_sistema: Casino Andino
formato_de_entrega: widget
enlace_o_archivo_de_la_entrega: bloque-3/Casino Andino.html (presentación HTML autocontenida)
---

# Entrega Bloque 3: tácticas, ADR completos y sizing (texto plano)

---

## 1. Punto de partida (lo que viene del Bloque 2)

**Tensión dominante, en una línea (la del Bloque 2):**
En este sistema, garantizar que ninguna operación de saldo confirmada se pierda choca con responder cada operación en p99 ≤ 250 ms con ~3.000 operaciones por segundo en el pico, porque asegurar cada operación, también en el sistema alterno, antes de confirmarla alarga cada respuesta y reduce cuántas caben por segundo.
Cifras: pérdida cero [S13 · supuesto][F1 · referencia], p99 ≤ 250 ms [S14 · supuesto], ~3.000 operaciones por segundo [S6 · supuesto], sistema alterno al día de inmediato [F1 · referencia]. (Tensión T2 del Bloque 2, Fiabilidad × Eficiencia de desempeño.)

**Escenarios críticos que llegan del Bloque 2:** E1, E2, E3 y E5, con sus cuatro ADR en estado Propuesto.
- E1 (ESC-01) · Retiro de un jugador incluido en una lista recién recibida → ADR-1 (ADR-01).
- E2 (ESC-02) · Caída del servidor de saldo en pleno pico → ADR-2 (ADR-02).
- E3 (ESC-03) · El proveedor del *crash* envía tres veces el mismo abono → ADR-3 (ADR-03).
- E5 (ESC-05) · Petición de abono falsificada a nombre de un proveedor → ADR-4 (ADR-04). ADR-4 nace de E5, no de E4.
- E4 (ESC-04, ráfaga de retiros del *crash*) sigue sin ADR: su costo cae en la Adecuación funcional, que con 2 puntos no está entre los atributos altos.
- En los documentos del proyecto los identificadores son ESC-NN, ADR-NN y TAC-NN; aquí se usan E1, ADR-1 y T1, T2… dentro de cada ADR, con el TAC-NN del proyecto entre paréntesis.

**Qué cambió desde el Bloque 2 y por qué:**
- 13 de noviembre de 2026: Adecuación funcional sube de 1 a 2 y Fiabilidad baja de 4 a 3. Reparto vigente: Seguridad 4, Fiabilidad 3, Compatibilidad 3, Desempeño 3, Adecuación funcional 2 y el resto en 1. Razón: retroalimentación del profesor a la exposición: el operador calcula reglas que exige el reglamento (saldo nunca negativo, límites de depósito, autoexclusión, plazo del retiro, reglas fiscales). El punto sale de Fiabilidad porque el saldo nunca negativo ya estaba atribuido entero a ella. No cambian las tensiones críticas, los cuatro atributos altos, el texto de los cinco escenarios ni de los cuatro ADR.
- 15 de noviembre de 2026: se agrega a la ficha el supuesto S44: en una ronda de *crash*, hasta el 30 % de los 2.400 jugadores se retira en el mismo segundo (720 retiros). Razón: la revisión del Bloque 3 encontró que la cifra estaba en la ficha y en ESC-04 sin supuesto que la definiera. No cambian el texto y la medida de ESC-04, los ADR ni los puntajes.
- Ninguna medida de un escenario cambió.

---

## 2. Línea base: la solución actual

**Cómo funciona hoy, en un párrafo:**
El operador tiene un sitio principal en un centro de datos Tier III con un servidor de aplicación y un servidor de base de datos, y un sistema alterno en un sitio separado con la misma composición. Cuando un proveedor de juego pide descontar, abonar o revertir saldo, la petición llega cifrada por internet, se acepta solo si viene de una dirección de red registrada para ese proveedor y trae su clave fija, y la aplicación la interpreta con el código propio del protocolo de ese proveedor. Luego busca el identificador de la transacción en el historial de operaciones del jugador: si ya está, reconstruye la respuesta; si no, en una sola transacción de la base de datos comprueba que el saldo alcanza, cambia el saldo y anota la operación en el historial, y responde al proveedor en cuanto la escritura queda grabada en el disco del sitio principal. Después, un proceso copia las operaciones al sistema alterno, segundos más tarde, y otro extrae periódicamente las transacciones para el sistema de inspección de Coljuegos. Los retiros se evalúan contra el perfil de riesgo y la marca de listas restrictivas guardada en cada cuenta; cuando llega una lista nueva, una reconsulta recorre todas las cuentas consultando al proveedor de listas, y mientras dura, todos los retiros quedan retenidos.
Composición del sitio: 1 servidor de aplicación y 1 servidor de base de datos [S30 · supuesto]. Centro Tier III y sistema alterno [F1 · referencia].

**Componentes que intervienen en la operación crítica, en orden:**
1. Proveedor de juego (externo): al resolverse una jugada, envía la petición de descuento, abono o reversión con su propio protocolo y su identificador de transacción. Si no recibe respuesta en ~3 s, la da por fallida, la reenvía o la revierte.
2. Red pública hasta el sitio principal: lleva la petición por un canal cifrado, autenticado en ambos sentidos.
3. Entrada del sitio principal: descarta toda petición que no venga de una dirección de red registrada para algún proveedor.
4. Servidor de aplicación, interfaz de saldo: (a) compara la clave fija de la petición con la del proveedor; (b) interpreta el protocolo de ese proveedor con un código propio dentro de la misma aplicación; (c) busca el identificador en el historial de operaciones del jugador, con proveedor y jugador. Si lo encuentra, reconstruye la respuesta original y salta al paso 6.
5. Servidor de base de datos, base única: en una transacción comprueba que el saldo alcanza, cambia el saldo y agrega la operación al historial. Confirma cuando la escritura queda grabada en el disco local. La misma base guarda cuentas, perfiles de riesgo, depósitos y retiros.
6. Servidor de aplicación, respuesta: devuelve al proveedor el resultado y el saldo nuevo.
7. Copia al sistema alterno (después de responder): un proceso lee las operaciones ya confirmadas y las graba en el alterno, segundos después.
8. Extracción para el sistema de inspección (después de responder): un proceso lee del historial las transacciones nuevas y las envía en el formato de Coljuegos, con la periodicidad del modelo de datos.
Los pasos 1 a 6 están en el camino de la respuesta y consumen los 250 ms. Los pasos 7 y 8 no la retrasan, pero leen la misma base de datos.

**Dónde está esto en la entrega:** presentación HTML, pestaña "2 · Línea base", y `bloque-3/02-linea-base.md` §2 y §3.

---

## 3. Perfil de carga

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

**Otra dimensión que importa en este sistema (si la hay):**
Volumen de identificadores de transacción por conservar: ~3.000 millones (24 M/día × 4 meses; supuestos S4 y S6, referencia F24). Reconsulta de listas restrictivas: ≥ 16,2 consultas/s durante ≤ 6 h, varias veces al mes, contra un proveedor externo (supuestos S3 y S19). Presupuesto de tiempo de la operación de saldo: p99 ≤ 250 ms, de los cuales ≤ 20 ms son para verificar el origen de la petición (supuestos S14 y S26).

**Proporción de lecturas y escrituras en la operación crítica:**
1 : 1 en el camino de la respuesta (2 lecturas y 2 escrituras por operación); 2 : 1 contando la copia al sistema alterno y la extracción para Coljuegos. Toda operación escribe: 6.000 escrituras/s en 3.000 transacciones/s en el pico.

**Dónde está esto en la entrega:** presentación HTML, pestaña "3 · Perfil de carga", y `bloque-3/03-perfil-de-carga.md`.

---

## 4. Capacidad actual y cuello de botella

**Componente que se satura primero:**
La base de datos única del sitio principal, en el paso de escritura del saldo. Con la carga del pico no se satura. La dependencia que sí se satura hoy es el proveedor de listas restrictivas, en la reconsulta de cuentas (otra dimensión).

**Su límite:** ~10.000 transacciones de saldo por segundo (rango 10.000–14.000) | tipo: supuesto (toma el tipo de su insumo más débil, S36; se apoya en la referencia F34) | razón o fuente: prueba pública de una base de datos relacional con carga tipo TPC-B en un servidor de 16 núcleos, 128 GB y disco local, con 500 GB de datos: 14.046 transacciones/s (F34, F35). Se descuenta un 30 % (14.046 × 0,7 = 9.832 ≈ 10.000) porque nuestros datos son 2,2 veces más grandes y la búsqueda del identificador y las lecturas de fondo usan la misma base (S36). Servidor del mismo tamaño que la prueba (S32).
Proveedor de listas: 5 consultas por segundo | tipo: referencia | razón o fuente: ritmo máximo recomendado por el personal técnico de un proveedor de listas de alcance mundial en su foro de desarrolladores (F37).

**Uso con la carga del pico (carga / límite, con la cuenta):**
```
base única:          3.000 transacciones/s ÷ 10.000 /s = 0,30   (ráfaga: 4.000 ÷ 10.000 = 0,40)
servidor de app.:    (3.000 + 1.200 del sitio) peticiones/s ÷ 16.000 /s = 0,26
red pública:         240 Mbit/s ÷ 1.000 Mbit/s = 0,24
proveedor de listas: 350.000 cuentas ÷ 21.600 s = 16,2 consultas/s ÷ 5 /s = 3,24
                     → la reconsulta tarda 350.000 ÷ 5 = 70.000 s ≈ 19,4 h, no 6 h
```

**Hasta cuánta carga aguanta hoy la línea base (usuarios concurrentes o tasa por segundo):**
~10.000 operaciones de saldo/s, es decir, 10.000 ÷ 0,249 ≈ 40.000 jugadores conectados, 3,3 veces el pico de 12.000 (≈ 30.000 si la ráfaga también debe caber). La reconsulta cumple las 6 h solo hasta 5 × 21.600 = 108.000 cuentas; hoy hay 350.000.

**Qué le pasa al usuario cuando se supera ese límite:**
Por encima de ~10.000 transacciones/s las respuestas se encolan. Al pasar los ~3 s de espera del proveedor, el giro del jugador termina en error: si era una apuesta, el proveedor la revierte y la jugada se anula; si era un premio, no aparece en el saldo hasta que el proveedor lo reenvía. Cada reenvío suma carga a la base ya saturada. En la reconsulta, unos 2.430 retiros quedan "en revisión" hasta ~19,4 h, varias veces al mes, y un retiro pedido al empezar la reconsulta consume casi toda la meta de 24 h antes de evaluarse.

**Dónde está esto en la entrega:** presentación HTML, pestaña "4 · Cuello de botella", y `bloque-3/04-cuello-de-botella.md`.

---

## 5. ADR completados (uno por cada escenario crítico)

### ADR-1
- Título: Evaluación de los retiros contra listas restrictivas recién recibidas
- Estado: Aceptado
- Escenario que lo origina: E1 (ESC-01)
- Medida de la respuesta del escenario (copiada literal): **0 retiros pagados** a personas incluidas en una lista ya recibida, verificado cruzando pagos y listas por marca de tiempo. Evaluación del retiro en **p95 ≤ 1 s**, también durante la reconsulta. Alerta al oficial de cumplimiento en **≤ 15 min**. Durante la reconsulta, las operaciones de saldo del juego se mantienen en **p99 ≤ 250 ms**. Reconsulta completa en **≤ 6 h**. Los retiros de riesgo normal mantienen la orden de transferencia en **≤ 24 h**
- Opciones del Bloque 2, una línea cada una:
  - A: Retener los retiros hasta que termine la reconsulta.
  - B: Consultar al proveedor de listas en cada retiro.
  - C: Lista vigente consultable por el operador, independiente de la reconsulta.
- Opción elegida: C.

**Decision, en voz activa:**
Usaremos una lista vigente propia en la memoria del servidor de aplicación (T1) para evaluar cada retiro desde el minuto en que llega la actualización del proveedor, sin esperar a la reconsulta. Limitaremos la reconsulta de las 350.000 cuentas a 50 cuentas por segundo (T2), comparando cada cuenta con esa lista y no con el proveedor. La lista nueva solo reemplaza a la anterior cuando termina de cargarse; mientras carga, los retiros esperan (fallo cerrado).

**Por qué se descartan las demás opciones (frente a la medida de la respuesta):**
- A · Retener hasta terminar la reconsulta: incumple la evaluación en p95 ≤ 1 s. El retiro espera hasta ~19,4 h (la reconsulta corre al ritmo del proveedor, 350.000 ÷ 5 /s = 70.000 s) y se retienen ~2.430 retiros por reconsulta. Un retiro pedido al empezar consume casi toda la meta de 24 h.
- B · Consultar al proveedor en cada retiro: incumple p95 ≤ 1 s durante la reconsulta y la disponibilidad de los retiros. El proveedor admite 5 consultas/s [F37] y la reconsulta ya le pide 16,2/s (uso 3,24): cada retiro haría fila detrás de ella o sería rechazado. Con fallo cerrado, una caída del proveedor detiene todos los retiros.

**T1 (TAC-09)**
- Táctica (nombre del catálogo): Múltiples copias de datos (Rendimiento → Gestionar recursos)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): cuando llega la actualización del proveedor (paso 4 del retiro), la evaluación de riesgo carga la lista completa en la memoria del servidor de aplicación, indexada por número de documento, y guarda el archivo recibido para la auditoría. Solo cuando la carga termina, la lista nueva reemplaza a la anterior. Desde ese momento, cada retiro (paso 3) se evalúa contra la lista nueva, haya o no terminado la reconsulta. Mientras carga, los retiros esperan (fallo cerrado).
- Responsabilidad: gestiona recursos y previene (el pago a una persona listada)
- Métrica que mueve: tiempo de evaluación de un retiro mientras corre la reconsulta
- Valor en la línea base: hasta ~19,4 h (el retiro espera a que termine la reconsulta al ritmo del proveedor); ~2.430 retiros retenidos por reconsulta
- Valor estimado con la táctica: ≤ 10 ms por retiro. Solo esperan los retiros que llegan durante la carga: ≤ 1 min, ~2 retiros (125 por hora ÷ 60)
- Cómo se estimó (cálculo o razonamiento, con unidades): evaluar es una búsqueda en un índice en memoria más la lectura del perfil de riesgo en la base única, del mismo orden que una operación de saldo (p99 ≤ 40 ms saturada [F34]); se usa ≤ 10 ms con la base al 30 % [S42]. Carga: 1 M de registros × 500 bytes = 500 MB, en ≤ 1 min [S42]. Retenidos: 3.000 retiros al día ÷ 24 h = 125 por hora × 1 min ÷ 60 min ≈ 2.
- Tipo de la estimación: supuesto [S42]; se apoya en la referencia [F34] | razón o fuente: S42 (b) la lista vinculante tiene 877 registros [F42] y 1 M es una cota para listas comerciales más amplias; construir un índice de 1 M de entradas en memoria toma segundos y 1 min es margen; S42 (c) una búsqueda en memoria y una lectura en la base, del orden de la operación de saldo [F34].
- Costo cuantificado (en dinero, latencia, frescura de datos, memoria, complejidad de operación u otra unidad): memoria: ~500 MB de 128 GB (0,4 %) [S32][S42]. Disco: una copia del archivo por actualización, varias veces al mes. Operación: dos procesos que mantener coherentes (lista vigente y reconsulta); si el servidor de aplicación se reinicia, la lista se recarga y los retiros esperan ≤ 1 min.
- Atributo que paga: Mantenibilidad y Eficiencia de desempeño (utilización de memoria)

**T2 (TAC-10)**
- Táctica (nombre del catálogo): Limitar la tasa de eventos (Rendimiento → Controlar la demanda)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): la reconsulta (paso 5 del retiro) recorre las 350.000 cuentas, compara el documento de cada una con la lista en memoria de T1 y guarda la marca solo si cambió. Lo hace con un ritmo máximo de 50 cuentas/s, en lugar de "sin ritmo máximo" como en la línea base.
- Responsabilidad: controla la demanda
- Métrica que mueve: uso de la base única durante la reconsulta. La duración de la reconsulta es un efecto conjunto con T1.
- Valor en la línea base: sin la táctica (opción C sin ritmo máximo): uso de ~1,0. Con la lista propia y sin ritmo máximo, la reconsulta leería tan rápido como responda la base: tomaría la capacidad libre, 10.000 − 3.000 = 7.000 lecturas/s, y llevaría el uso de 0,30 a ~1,0. La línea base no sirve de comparación para esta métrica: en ella la reconsulta va al ritmo del proveedor (5 consultas/s) y el uso de la base se queda en 0,30.
- Valor estimado con la táctica: uso de 0,31 (+0,01 sobre el 0,30 del saldo). Efecto conjunto con T1: la reconsulta pasa de ~19,4 h en la línea base a ~1,9 h, frente a ≤ 6 h.
- Cómo se estimó (cálculo o razonamiento, con unidades): carga: 50 lecturas/s + hasta 50 escrituras/s = 100 ÷ 10.000 = +0,01 de uso, frente a 7.000 ÷ 10.000 = +0,70 sin el tope. Duración: 350.000 cuentas ÷ 50 /s = 7.000 s ÷ 3.600 s/h = 1,94 h. La baja desde ~19,4 h la produce T1 (TAC-09), que saca al proveedor de la reconsulta; el tope de T2 la hace más lenta que sin tope, pero la deja dentro de las 6 h. El ritmo de 50/s es ~3 veces el mínimo de 16,2/s [S19].
- Tipo de la estimación: supuesto [S42] con el límite de la base de la fase 4 [F34][S36] | razón o fuente: S42 (d) ~3 veces el mínimo de 16,2/s [S19], con +0,01 de uso de la base.
- Costo cuantificado: tiempo: la reconsulta dura ~1,9 h en lugar de los minutos que tardaría sin ritmo máximo. Los retiros no esperan por eso (T1), pero una marca de riesgo que cambie llega a su cuenta hasta 1,9 h después de recibir la lista.
- Atributo que paga: Eficiencia de desempeño (comportamiento temporal de la reconsulta)

**Hasta dónde llegan estas tácticas:**
- Nuevo límite del sistema con las tácticas de este ADR (usuarios concurrentes o tasa por segundo): la reconsulta cumple las 6 h hasta 1.080.000 cuentas, y el proveedor de listas queda a 0,002 de su límite. La evaluación del retiro ya no depende de la reconsulta.
- Cómo se calculó:
```
reconsulta a 50 cuentas/s en 6 h     50 /s × 21.600 s = 1.080.000 cuentas
margen sobre las cuentas de hoy      1.080.000 ÷ 350.000 = 3,1
duración de la reconsulta de hoy     350.000 ÷ 50 /s = 7.000 s = 1,94 h        ≤ 6 h
uso del proveedor de listas          ~1.000 registros al día ÷ 86.400 s = 0,012 /s ÷ 5 /s = 0,002
uso de la base única                 0,30 + (50 lecturas + 50 escrituras) ÷ 10.000 = 0,31
```
- Siguiente componente que se satura: la base única. La reconsulta le suma 0,01 de uso por cada 50 cuentas/s, así que subir el ritmo (la variante de 100/s suma 0,02) compite con el saldo.
- ¿Se cumple la medida de la respuesta con la carga del pico? (sí | no | depende) y por qué: sí.
  - 0 retiros pagados a personas de una lista ya recibida: la evaluación usa la lista nueva desde que termina de cargarse (≤ 1 min); solo esperan ~2 retiros por actualización (125 por hora ÷ 60) y no se pagan. Cumple.
  - Evaluación p95 ≤ 1 s, también durante la reconsulta: ≤ 10 ms [S42]. Cumple.
  - Alerta al oficial de cumplimiento ≤ 15 min: no cambia; la alerta nace de la evaluación, que ahora tarda ≤ 10 ms. Cumple.
  - Saldo p99 ≤ 250 ms durante la reconsulta: la base sube de 0,30 a 0,31 de uso. Cumple.
  - Reconsulta completa ≤ 6 h: ~1,9 h. Cumple.
  - Orden de transferencia de retiros normales ≤ 24 h: sin espera por la reconsulta. Cumple.

**Consequences:**
- Gana: la evaluación del retiro pasa de hasta ~19,4 h a ≤ 10 ms; la reconsulta, de ~19,4 h a ~1,9 h; el uso del proveedor de listas, de 3,24 a 0,002, y los retiros dejan de depender de un tercero en el camino crítico.
- Paga: ~500 MB de memoria de 128 GB (0,4 %) [S42]; dos procesos que mantener coherentes (lista vigente y reconsulta); una marca de riesgo que cambie llega a la cuenta hasta ~1,9 h después de recibir la lista; si el servidor de aplicación se reinicia, la lista se recarga y los retiros esperan ≤ 1 min.
- No resuelve: (1) las coincidencias parciales de nombre (homonimia): la lista se indexa por número de documento; (2) la opción C supone que el proveedor entrega la lista completa y no solo respuestas a consultas por documento; la lista vinculante de Naciones Unidas tiene 877 registros [F42] y el millón de S42 es una cota; (3) la lista vive en memoria: no sobrevive a un reinicio sin recargarse.
- Condición de revisión (umbral medible que obliga a revisar esta decisión): revisar esta decisión si (a) las cuentas registradas superan ~1.000.000 (la reconsulta a 50/s llegaría a 5,6 h, cerca de las 6 h), (b) la lista del proveedor supera 1 M de registros o su carga tarda más de 1 min [S42], o (c) el proveedor deja de entregar la lista completa.

**Dónde está este ADR en la entrega:** presentación HTML, pestaña "6 · ADR" (ADR-01), y `bloque-3/06-adrs.md` §2; las fichas de las tácticas en la pestaña "5 · Tácticas" y en `bloque-3/05-tacticas-y-efecto.md` §4.

### ADR-2
- Título: Permanencia de las operaciones de saldo confirmadas ante la caída de un servidor
- Estado: Aceptado
- Escenario que lo origina: E2 (ESC-02)
- Medida de la respuesta del escenario (copiada literal): **0 operaciones confirmadas perdidas**, verificado por la conciliación operador–proveedores posterior al evento. **0 saldos negativos.** Juego y saldo restablecidos en **≤ 15 min**. **100 %** de las jugadas incompletas con su resultado visible al reconectarse. Aviso a Coljuegos en **≤ 30 min**. A los **5 min** del restablecimiento, las operaciones de saldo vuelven a **p99 ≤ 250 ms**
- Opciones del Bloque 2, una línea cada una:
  - A: Confirmar solo después de escribir en dos sitios físicos.
  - B: Confirmar tras la grabación local y copiar después.
  - C: Libreta de operaciones de solo agregar, confirmada por mayoría.
- Opción elegida: C.

**Decision, en voz activa:**
Usaremos una libreta de operaciones de solo agregar, mantenida por tres servidores de base de datos del sitio principal (T1), y responderemos al proveedor cuando al menos dos de los tres la tengan grabada en disco. Detectaremos la caída del servidor que atiende con una señal de vida entre los tres, con un intervalo aleatorio de 150 a 300 ms (T2), y los dos que quedan elegirán al que tiene la libreta completa para atender (T3). El sistema alterno seguirá recibiendo la copia leyendo la libreta, fuera del camino de cada jugada.

**Por qué se descartan las demás opciones (frente a la medida de la respuesta):**
- A · Dos sitios antes de confirmar: cumple la pérdida cero y la latencia, pero incumple la disponibilidad de juego y saldo. La ida y vuelta al alterno cuesta ~10 ms de 250 ms [F39][S38] y 30 operaciones esperando a la vez: cabe. Pero con un solo enlace a 99,9 % [S39] hay 0,001 × 43.200 min = 43,2 min al mes sin enlace, exactamente el margen de 99,9 % de S13. En esos minutos se detiene el juego o se vuelve a la línea base.
- B · Grabación local y copia posterior: incumple 0 operaciones confirmadas perdidas. Es la línea base: ~3.000 operaciones perdidas (1 s de atraso × 3.000 /s; 4.000 en la ráfaga; hasta 15.000 con 5 s) [S37] y 10 a 30 min para restablecer [S40], contra ≤ 15 min de la medida.

**T1 (TAC-01)**
- Táctica (nombre del catálogo): Redundancia activa (Disponibilidad → Recuperarse)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): el sitio principal pasa de 1 a 3 servidores de base de datos, del tamaño de S32. Cada transacción de saldo (paso 5) se anota en orden en una libreta de solo agregar que los tres reciben. El servidor que atiende responde al proveedor (paso 6) cuando al menos 2 de los 3 tienen la operación grabada en disco. El saldo de cada jugador se calcula desde esa libreta. La copia al sistema alterno (paso 7) sigue igual, leyendo la libreta.
- Responsabilidad: previene (la pérdida de lo confirmado)
- Métrica que mueve: operaciones de saldo confirmadas que se pierden cuando falla el servidor de base de datos
- Valor en la línea base: ~3.000 (hasta 4.000 en la ráfaga; hasta 15.000 si la copia se atrasa 5 s)
- Valor estimado con la táctica: 0, mientras sobreviva 1 de los otros 2 servidores
- Cómo se estimó (cálculo o razonamiento, con unidades): línea base: lo confirmado y todavía no copiado al alterno = retraso de la copia × tasa = 1 s × 3.000 operaciones/s = 3.000 (4.000/s × 1 s en la ráfaga; 5 s × 3.000/s = 15.000). Con la táctica: ninguna operación se confirma sin estar grabada en un segundo servidor; si cae uno, la otra copia la tiene.
- Tipo de la estimación: supuesto [S37] sobre el retraso de la copia; la cuenta con la tasa es de la fase 3 [S6] | razón o fuente: S37, la línea base copia "segundos después"; el enlace usa 6 Mbit/s de su capacidad, así que el atraso lo fija el proceso de copia, no la red; 1 s es el extremo optimista y 5 s el pesimista.
- Costo cuantificado: dinero: 2 servidores más ≈ 2 × USD 1.002 al mes = ~USD 24.000 al año [F41], ≈ COP 96 M al año a COP 4.000 por dólar [S9], 1,3 % del presupuesto de tecnología (COP 7.300 M). Latencia: +~1 ms por operación [S40]. Red interna: 3.000/s × 350 bytes × 2 copias ≈ 2,1 MB/s ≈ 17 Mbit/s [S34][S41].
- Atributo que paga: Eficiencia de desempeño (utilización de recursos) y Mantenibilidad (tres servidores que operar y recertificar en lugar de uno)

**T2 (TAC-02)**
- Táctica (nombre del catálogo): Latido (heartbeat) (Disponibilidad → Detectar fallas)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): el servidor de base de datos que atiende envía una señal a los otros dos varias veces por segundo. Si un servidor deja de recibirla durante un intervalo aleatorio de 150 a 300 ms, declara caído al que atendía.
- Responsabilidad: detecta
- Métrica que mueve: tiempo desde la caída del servidor de saldo hasta que el sistema la detecta
- Valor en la línea base: sin detección automática: la caída la notan personas, como primer paso del cambio manual de 10 a 30 min [S40]. S40 no separa cuánto de ese tiempo es la detección, así que los 10 a 30 min son una cota superior
- Valor estimado con la táctica: ≤ 0,3 s
- Cómo se estimó (cálculo o razonamiento, con unidades): el intervalo de 150–300 ms es el que recomienda la referencia del algoritmo de consenso por mayoría [F40]. La línea base es la primera parte del cambio manual de S40, sin cifra propia.
- Tipo de la estimación: referencia [F40] para la táctica; supuesto [S40] para la línea base | razón o fuente: F40, referencia del algoritmo de consenso por mayoría; S40, en la línea base pasar al alterno a mano toma 10 a 30 min: alguien debe notar la caída, decidir el cambio, activar el alterno y redirigir a los proveedores.
- Costo cuantificado: caídas falsas: una pausa de red de más de 300 ms dispara un cambio de servidor innecesario, con el costo de T3 (~30 s sin saldo). Tráfico: despreciable (mensajes de pocos bytes).
- Atributo que paga: Fiabilidad (disponibilidad, por las caídas falsas)

**T3 (TAC-03)**
- Táctica (nombre del catálogo): Reconfiguración (Disponibilidad → Recuperarse)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): cuando T2 declara caído al servidor que atendía, los dos que quedan eligen al que tiene la libreta completa, y ese pasa a atender. El servidor de aplicación (paso 4) se reconecta al nuevo y retoma las operaciones. Las que estaban en curso sin confirmar no tienen efecto; el proveedor las revierte o las reenvía [F24].
- Responsabilidad: recupera
- Métrica que mueve: tiempo con el juego y el saldo detenidos tras la caída del servidor de base de datos
- Valor en la línea base: 10 a 30 min: pasar a mano al sistema alterno en el otro sitio [S40]
- Valor estimado con la táctica: ≤ 30 s
- Cómo se estimó (cálculo o razonamiento, con unidades): detección ≤ 0,3 s (T2) + elección en menos de 1 s [F40] + reconexión del servidor de aplicación y vuelta a atender ≤ 30 s [S40]. La medida de E2 es ≤ 15 min: margen de 15 min ÷ 0,5 min = 30 veces.
- Tipo de la estimación: supuesto [S40]; se apoya en la referencia [F40] | razón o fuente: S40, los 30 s cubren la detección y la elección [F40] con amplio margen para reconectar.
- Costo cuantificado: operaciones afectadas en cada cambio: 3.000/s × 30 s = hasta ~90.000 operaciones sin respuesta, que los proveedores revierten (descuentos) o reenvían (abonos) [F24]. Capacidad: mientras no se reemplace el servidor caído, quedan 2 de 3 y una segunda falla detiene el juego.
- Atributo que paga: Mantenibilidad (procedimiento de cambio que hay que probar y recertificar) y, durante 30 s, Eficiencia de desempeño

**Hasta dónde llegan estas tácticas:**
- Nuevo límite del sistema con las tácticas de este ADR (usuarios concurrentes o tasa por segundo): ~10.000 operaciones de saldo/s, sin cambio. T1 no mueve el límite: la prueba de referencia ya confirma cada transacción después de que una réplica la recibe [F34], y la espera local suma ~1 ms [S40].
- Cómo se calculó:
```
margen sobre el pico            10.000 ÷ 3.000 = 3,3
margen sobre la ráfaga          10.000 ÷ 4.000 = 2,5
tolerancia a fallas             1 servidor de 3 (mayoría de 2)
cambio de servidor              3.000 /s × 30 s = ~90.000 operaciones sin respuesta
vuelta a p99 ≤ 250 ms           ~90.000 reenvíos ÷ (10.000 − 3.000) /s ≈ 13 s     (medida: ≤ 5 min; ~39 s si cada operación llega 3 veces)
```
- Siguiente componente que se satura: la base única, ahora el servidor de base de datos que atiende: todas las escrituras del saldo pasan por él, y los otros dos reciben la libreta pero no suman capacidad de escritura.
- ¿Se cumple la medida de la respuesta con la carga del pico? (sí | no | depende) y por qué: sí, para la caída de un servidor.
  - 0 operaciones confirmadas perdidas: 0 mientras sobreviva 1 de los otros 2 servidores. Cumple.
  - 0 saldos negativos: no cambia: la regla de saldo no negativo sigue en la transacción (C6). Cumple.
  - Juego y saldo restablecidos en ≤ 15 min: ≤ 30 s; margen 15 min ÷ 0,5 min = 30 veces. Cumple.
  - 100 % de jugadas incompletas con resultado visible: las operaciones en curso sin confirmar no tienen efecto y el proveedor las revierte o reenvía [F24]. Cumple.
  - Aviso a Coljuegos ≤ 30 min: no cambia; la detección en ≤ 0,3 s adelanta el registro del inicio del evento. Cumple.
  - p99 ≤ 250 ms a los 5 min del restablecimiento: ≈ 13 s para absorber los reenvíos (~39 s con 3 copias por operación). Cumple.

**Consequences:**
- Gana: la pérdida de operaciones confirmadas pasa de ~3.000 a 0, y el restablecimiento de 10–30 min a ≤ 30 s.
- Paga: ~USD 24.000 al año por 2 servidores más (~COP 96 M, 1,3 % del presupuesto de tecnología) [F41][S9]; +~1 ms por operación; ~17 Mbit/s de red interna; hasta ~90.000 operaciones revertidas o reenviadas en cada cambio de servidor; una pausa de red de más de 300 ms dispara un cambio innecesario (~30 s sin saldo); tres servidores que operar y recertificar.
- No resuelve: (1) la pérdida del sitio completo sigue dejando ~3.000 operaciones en riesgo, porque el alterno recibe la copia con atraso (S37); (2) mientras no se reponga el servidor caído quedan 2 de 3 y una segunda falla detiene el juego; (3) el límite de escritura sigue siendo el de un solo servidor.
- Condición de revisión (umbral medible que obliga a revisar esta decisión): revisar si (a) el pico sostenido supera ~7.500 operaciones/s (~30.000 jugadores: es el punto donde la ráfaga deja de caber), (b) hay más de 20 cambios de servidor al mes (20 × 0,5 min = 10 min, casi una cuarta parte de los 43,2 min de S13), o (c) el atraso de la copia al alterno supera 5 s (15.000 operaciones en riesgo).

**Dónde está este ADR en la entrega:** presentación HTML, pestaña "6 · ADR" (ADR-02), y `bloque-3/06-adrs.md` §3; las fichas de las tácticas en la pestaña "5 · Tácticas" y en `bloque-3/05-tacticas-y-efecto.md` §2.1.

### ADR-3
- Título: Tratamiento de las peticiones de saldo repetidas de los proveedores de juego
- Estado: Aceptado
- Escenario que lo origina: E3 (ESC-03)
- Medida de la respuesta del escenario (copiada literal): **0 abonos duplicados.** **100 %** de las peticiones repetidas reciben una respuesta idéntica a la original, en **p99 ≤ 250 ms**. **0 diferencias** en la conciliación diaria de saldos operador–proveedor. **0 transacciones duplicadas** en el reporte a Coljuegos. **100 %** de las peticiones que repiten proveedor, jugador e identificador con datos distintos rechazadas, con alerta en **≤ 5 min**
- Opciones del Bloque 2, una línea cada una:
  - A: Buscar el identificador en el historial de operaciones.
  - B: Registro de identificadores procesados, guardado junto con el cambio de saldo.
  - C: Una pieza de traducción por proveedor que descarta repetidos antes de llegar al saldo.
- Opción elegida: una combinación, B con la traducción de C. El Bloque 2 dejó esa combinación por evaluar ("Pendiente para el Bloque 3").

**Decision, en voz activa:**
Usaremos un registro de identificadores procesados dentro de la misma transacción que cambia el saldo (T1), con clave única de proveedor, jugador e identificador. Si la clave ya existe, devolveremos la respuesta guardada sin tocar el saldo; si existe con datos distintos, rechazaremos la petición y alertaremos (T2). Del traductor por proveedor tomaremos solo la traducción del protocolo: una pieza por proveedor entrega al saldo una petición común (T3). Su memoria de repetidos de 24 h no se usa.

**Por qué se descartan las demás opciones (frente a la medida de la respuesta):**
- A · Buscar en el historial: incumple 0 abonos duplicados. Es la línea base: dos copias que llegan a la vez pasan ambas la búsqueda antes de que alguna quede grabada y se aplican dos veces. Ocurre cuando la primera copia tarda más que los ~3 s del proveedor: con la base saturada o tras un cambio de servidor (hasta ~90.000 reenvíos, ADR-2). Además la búsqueda recorre ~4.400 M de registros del historial.
- C · Traductor que filtra repetidos: parcial: no garantiza 0 duplicados. Su memoria de 24 h no ve repeticiones tardías: un proveedor puede reenviar una reversión días después, dentro de los 4 meses de conciliación [F24], y una caída del traductor borra la memoria. Su parte de traducción sí se toma (T3).

**T1 (TAC-04)**
- Táctica (nombre del catálogo): Transacciones (Disponibilidad → Prevenir fallas). En la línea base la táctica ya existe por el piso de C6; aquí se refuerza: entra un registro más al paso indivisible.
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): un registro de identificadores procesados en la base única, con clave única proveedor + jugador + identificador [F26]. En el paso 5, la misma transacción que cambia el saldo inserta la clave, un resumen de los datos de la petición (tipo y monto) y la respuesta dada (resultado y saldo). Si la clave ya existe, la transacción no toca el saldo y devuelve la respuesta guardada. El registro se guarda por día y se borra por días completos a los 4 meses [F24]. Reemplaza la búsqueda del paso 4c.
- Responsabilidad: previene (el doble abono)
- Métrica que mueve: abonos aplicados dos veces cuando dos copias de la misma petición se procesan a la vez
- Valor en la línea base: 1 de más por cada par de copias que se solapan: las dos pasan la búsqueda antes de que alguna quede grabada. Ocurre cuando la primera copia tarda más que los ~3 s del proveedor: con la base saturada o tras un cambio de servidor (T3 de ADR-2, hasta ~90.000 reenvíos)
- Valor estimado con la táctica: 0: la segunda inserción de la misma clave falla dentro de su transacción y no cambia el saldo
- Cómo se estimó (cálculo o razonamiento, con unidades): razonamiento sobre la unicidad: dos transacciones no pueden insertar la misma clave única; la que llega segunda ve la primera y devuelve su respuesta. No depende de la carga ni del momento en que llegan las copias.
- Tipo de la estimación: referencia para la regla de unicidad del identificador [F26]; el costo es supuesto [S41] | razón o fuente: S41, los campos son cortos y el resumen tiene tamaño fijo; una inserción en un índice que ya está en memoria es del orden de microsegundos; ≤ 1 ms es una cota amplia.
- Costo cuantificado: disco: ~300 GB [S41]. Escrituras: +3.000 inserciones/s. Latencia: ≤ 1 ms por operación [S41]. Límite de la base: sin cambio, ~10.000/s.
- Atributo que paga: Eficiencia de desempeño (utilización de recursos)

**T2 (TAC-05)**
- Táctica (nombre del catálogo): Validar entradas (Seguridad → Resistir ataques)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): cuando la clave ya existe en el registro de T1, la transacción compara el resumen guardado (tipo y monto) con el de la petición nueva. Iguales: devuelve la respuesta guardada. Distintos: rechaza la petición sin tocar el saldo y deja un evento de alerta en el mismo canal de Informar a los actores (TAC-08, ADR-4).
- Responsabilidad: detecta y resiste
- Métrica que mueve: porcentaje de peticiones con la misma clave y datos distintos que se rechazan, y tiempo hasta la alerta
- Valor en la línea base: 0 %: la búsqueda del paso 4c encuentra el identificador y devuelve la respuesta original sin comparar los datos. Sin alerta
- Valor estimado con la táctica: 100 % rechazadas; alerta en ≤ 1 min (ventana de agrupación de TAC-08), frente a ≤ 5 min de la medida [S25]
- Cómo se estimó (cálculo o razonamiento, con unidades): la comparación es exacta sobre un resumen de 32 bytes: no hay casos grises. La clave incluye al jugador, así que dos jugadores con el mismo identificador no se confunden [F26].
- Tipo de la estimación: razonamiento sobre el mecanismo; ventana de alerta de TAC-08 [S43] | razón o fuente: S43, con el primer rechazo de un origen se abre el incidente; agrupar por minuto evita una alerta por petición y queda 5 veces por debajo del plazo de ≤ 5 min [S25][S28].
- Costo cuantificado: disco: 32 bytes por registro × 3.000 M = ~96 GB, ya contados en los ~300 GB de T1. Cómputo: un resumen por petición, microsegundos. Riesgo: si un proveedor reutiliza un identificador para otra operación del mismo jugador, una petición legítima se rechaza; cuenta dentro del ≤ 0,001 % de S27.
- Atributo que paga: Compatibilidad (un proveedor que no respete la unicidad por jugador ve rechazos)

**T3 (TAC-06)**
- Táctica (nombre del catálogo): Usar un intermediario (Modificabilidad → Reducir el acoplamiento). Hace lo que el Bloque 2 llamó "Gestionar interfaces".
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): en el servidor de aplicación, el paso 4b se separa en 5 piezas, una por proveedor. Cada pieza recibe el protocolo de su proveedor y entrega a la interfaz de saldo una petición común: proveedor, jugador, identificador, tipo, monto y resultado de la verificación de origen (TAC-07). Las piezas no guardan memoria de repetidos: eso lo hace T1. Se ejecutan dentro del mismo proceso, sin salto de red. La comparten ADR-3 y ADR-4.
- Responsabilidad: desacopla
- Métrica que mueve: partes del sistema que cambian al integrar un proveedor nuevo
- Valor en la línea base: la interfaz de saldo completa: el código de cada protocolo está mezclado con el del saldo
- Valor estimado con la táctica: 1 pieza nueva y 0 cambios en la interfaz de saldo, para cada uno de los 1 o 2 proveedores nuevos al año [S11]
- Cómo se estimó (cálculo o razonamiento, con unidades): por construcción: la interfaz de saldo solo conoce la petición común.
- Tipo de la estimación: razonamiento sobre el mecanismo; frecuencia de proveedores nuevos [S11 · supuesto] | razón o fuente: S11, supuesto de la ficha del Bloque 2.
- Costo cuantificado: mantenimiento: 5 piezas que mantener y recertificar; cada cambio en una pieza pasa por el laboratorio, 2 a 4 semanas [S10]. Latencia: una llamada dentro del proceso, < 0,1 ms.
- Atributo que paga: Mantenibilidad (más piezas, cada una con su validación)

**Hasta dónde llegan estas tácticas:**
- Nuevo límite del sistema con las tácticas de este ADR (usuarios concurrentes o tasa por segundo): ~10.000 operaciones de saldo/s, sin cambio. Por transacción, la línea base con el registro hace 1 cambio de saldo, 1 lectura y 2 inserciones; la prueba de referencia hace 3 cambios de saldo, 1 lectura y 1 inserción [F35]: sigue siendo menos trabajo que la prueba.
- Cómo se calculó:
```
escrituras adicionales          1 inserción × 3.000 /s = 3.000 inserciones/s          [S6]
tiempo adicional                ≤ 1 ms de 250 ms                                      [S41]
registro                        3.000 M identificadores × ~100 bytes = ~300 GB       [S41]
datos de la base                1,1 TB de historial + 0,3 TB = 1,4 TB = 2,8 × los 500 GB de F34
tope de datos del descuento     S36 ya descontó 5 × 500 GB = 2,5 TB
margen de datos                 2,5 TB ÷ 1,4 TB = 1,8
margen sobre el pico            10.000 ÷ 3.000 = 3,3   (ráfaga: 10.000 ÷ 4.000 = 2,5)
```
- Siguiente componente que se satura: la base única, por dos vías: la tasa (~10.000/s) y el volumen: a 2,5 TB se llega al escalón de datos que S36 descontó completo; con el registro y el historial creciendo en proporción, eso equivale a ~1,8 veces las operaciones diarias de hoy (~43 M al día).
- ¿Se cumple la medida de la respuesta con la carga del pico? (sí | no | depende) y por qué: sí.
  - 0 abonos duplicados: 0: la segunda inserción de la misma clave falla dentro de su transacción. Cumple.
  - 100 % de repetidas con respuesta idéntica, p99 ≤ 250 ms: se devuelve la respuesta guardada; +≤ 1 ms. Cumple.
  - 0 diferencias en la conciliación diaria: el registro guarda cada petición con su respuesta. Cumple.
  - 0 duplicados en el reporte a Coljuegos: una sola aplicación por clave. Cumple.
  - 100 % de misma clave con datos distintos rechazadas, alerta ≤ 5 min: 100 %, alerta ≤ 1 min. Cumple.

**Consequences:**
- Gana: 0 duplicados sin importar el momento en que lleguen las copias (incluidos los ~90.000 reenvíos de un cambio de servidor), alerta en ≤ 1 min en lugar de sin alerta, y un proveedor nuevo es 1 pieza nueva con 0 cambios en la interfaz de saldo [S11].
- Paga: ~300 GB de disco [S41]; +3.000 inserciones/s y ≤ 1 ms por operación; 5 piezas que mantener y recertificar, cada cambio entre 2 y 4 semanas de laboratorio [S10]; hasta 240 alertas falsas al día entre ADR-3 y ADR-4 si los rechazos legítimos llegan al tope de S27, que es común a los dos.
- No resuelve: (1) un proveedor que reutilice un identificador para otra operación del mismo jugador ve rechazada una petición legítima; (2) las repeticiones posteriores a los 4 meses de conservación [F24]; (3) el registro suma volumen a la misma base que es el cuello del saldo.
- Condición de revisión (umbral medible que obliga a revisar esta decisión): revisar si (a) los datos de la base superan 2,5 TB o el promedio supera ~43 M de operaciones al día, (b) los rechazos legítimos de ADR-3 y ADR-4, sumados, superan 0,001 % de las peticiones (240 al día) [S27], o (c) un proveedor reutiliza identificadores entre operaciones distintas del mismo jugador.

**Dónde está este ADR en la entrega:** presentación HTML, pestaña "6 · ADR" (ADR-03), y `bloque-3/06-adrs.md` §4; las fichas de las tácticas en la pestaña "5 · Tácticas" y en `bloque-3/05-tacticas-y-efecto.md` §2.2.

### ADR-4
- Título: Verificación del origen de las peticiones de saldo de los proveedores
- Estado: Aceptado
- Escenario que lo origina: E5 (ESC-05)
- Medida de la respuesta del escenario (copiada literal): **100 %** de las peticiones de una batería de falsificación rechazadas (sin credencial, con la credencial de otro proveedor y copia de una petición legítima con el monto modificado), con **0 saldos modificados**, verificado en la prueba de carga y en las pruebas de penetración del laboratorio. **≤ 0,001 %** de las peticiones legítimas rechazadas por la verificación. La verificación añade **≤ 20 ms (p99)** a cada operación, y las operaciones de saldo siguen en **p99 ≤ 250 ms**. Alerta al equipo de seguridad en **≤ 5 min**
- Opciones del Bloque 2, una línea cada una:
  - A: Clave fija y lista de direcciones de red permitidas por proveedor.
  - B: Firma de cada petición con un secreto por proveedor, verificada por una pieza de traducción por proveedor.
  - C: Un esquema único, definido por el operador, para todos los proveedores.
- Opción elegida: B. La firma cubre el contenido de la petición con el esquema de cada proveedor; la hora firmada queda fuera (ver "No resuelve").

**Decision, en voz activa:**
Usaremos una firma por petición verificada en la pieza de cada proveedor (T1), con el esquema y la clave de ese proveedor (T2), antes de que la petición llegue al saldo. Conservaremos la clave fija y la lista de direcciones de la línea base como primera barrera. Cada rechazo de firma dejará un evento que un proceso aparte agrupa por origen en ventanas de 1 minuto y alerta al equipo de seguridad (T3). No integraremos a un proveedor que no firme el contenido de sus peticiones.

**Por qué se descartan las demás opciones (frente a la medida de la respuesta):**
- A · Clave fija y direcciones permitidas: incumple el 100 % de la batería. Es la línea base: rechaza 2 de 3 tipos (~67 %); pasa la copia legítima con el monto modificado porque la clave no depende del contenido.
- C · Esquema único del operador: incumple la aceptación de las peticiones legítimas de los 5 proveedores. Cada proveedor trae su esquema y no acepta cambiarlo [S29]; con ese supuesto, quizá ningún proveedor la acepte. Un certificado vencido cortaría a un proveedor completo.

**T1 (TAC-06)**
- Táctica (nombre del catálogo): Usar un intermediario (Modificabilidad → Reducir el acoplamiento). Hace lo que el Bloque 2 llamó "Gestionar interfaces". Es la misma táctica de T3 en ADR-3.
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): en el servidor de aplicación, el paso 4b se separa en 5 piezas, una por proveedor. Cada pieza recibe el protocolo de su proveedor y entrega a la interfaz de saldo una petición común: proveedor, jugador, identificador, tipo, monto y resultado de la verificación de origen (T2). Las piezas no guardan memoria de repetidos: eso lo hace TAC-04 (ADR-3). Se ejecutan dentro del mismo proceso, sin salto de red. La comparten ADR-3 y ADR-4.
- Responsabilidad: desacopla
- Métrica que mueve: partes del sistema que cambian al integrar un proveedor nuevo
- Valor en la línea base: la interfaz de saldo completa: el código de cada protocolo está mezclado con el del saldo
- Valor estimado con la táctica: 1 pieza nueva y 0 cambios en la interfaz de saldo, para cada uno de los 1 o 2 proveedores nuevos al año [S11]
- Cómo se estimó (cálculo o razonamiento, con unidades): por construcción: la interfaz de saldo solo conoce la petición común.
- Tipo de la estimación: razonamiento sobre el mecanismo; frecuencia de proveedores nuevos [S11 · supuesto] | razón o fuente: S11, supuesto de la ficha del Bloque 2.
- Costo cuantificado: mantenimiento: 5 piezas que mantener y recertificar; cada cambio en una pieza pasa por el laboratorio, 2 a 4 semanas [S10]. Latencia: una llamada dentro del proceso, < 0,1 ms.
- Atributo que paga: Mantenibilidad (más piezas, cada una con su validación)

**T2 (TAC-07)**
- Táctica (nombre del catálogo): Verificar integridad del mensaje (Seguridad → Detectar ataques)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): en la pieza de su proveedor (T1), antes de que la petición llegue al saldo (paso 4a), se verifica la firma del contenido con el esquema y la clave de ese proveedor: clave pública para quien firma con clave privada, secreto compartido para quien firma con resumen. Si la firma no corresponde, la petición se rechaza sin tocar el saldo y se registra con su origen. La clave fija y la lista de direcciones de la línea base se conservan como primera barrera.
- Responsabilidad: detecta y resiste
- Métrica que mueve: porcentaje de la batería de falsificación de E5 que se rechaza
- Valor en la línea base: 2 de 3 tipos (~67 %): se rechaza la petición sin credencial y la que trae la credencial de otro proveedor; pasa la copia legítima con el monto modificado, porque la clave fija no depende del contenido
- Valor estimado con la táctica: 3 de 3 tipos (100 %): cambiar el monto invalida la firma; sin la clave del proveedor no se puede firmar; la firma de otro proveedor no se verifica con la clave de este
- Cómo se estimó (cálculo o razonamiento, con unidades): recorrido de los tres tipos de la batería contra el mecanismo. Tiempo: 1 ÷ 22.220 verificaciones/s = 0,045 ms por petición [F36].
- Tipo de la estimación: referencia [F24][F26][F36]; que los 5 proveedores firmen es supuesto [S42] | razón o fuente: S42 (a), las dos integraciones documentadas lo hacen [F24][F26] y S29 dice que cada proveedor trae su esquema.
- Costo cuantificado: latencia: +0,05 ms de los 20 ms de S26. Cómputo: 3.000/s × 0,045 ms = 135 ms por segundo = 0,135 núcleos, 0,8 % del servidor de aplicación (16 núcleos). Operación: 5 claves que guardar y rotar; cada rotación o cambio de esquema es un cambio en la seguridad de las comunicaciones y pasa por el laboratorio (B3) [F1], 2 a 4 semanas [S10].
- Atributo que paga: Mantenibilidad (claves y validaciones) y, en mínima parte, Eficiencia de desempeño

**T3 (TAC-08)**
- Táctica (nombre del catálogo): Informar a los actores (Seguridad → Reaccionar)
- Mecanismo concreto en este sistema (qué componente, qué dato, en qué paso de la operación crítica): cada rechazo de T2 (firma inválida) y de Validar entradas (TAC-05, misma clave, datos distintos, ADR-3) deja un evento con el origen de red, el proveedor que dice ser y el motivo. Un proceso aparte agrupa los eventos por origen en ventanas de 1 minuto [S43]: el primer evento de un origen abre un incidente y alerta al equipo de seguridad; los siguientes del mismo origen se suman a ese incidente.
- Responsabilidad: hace observable y reacciona
- Métrica que mueve: tiempo desde la primera petición falsificada hasta la alerta al equipo de seguridad
- Valor en la línea base: sin alerta: la línea base rechaza lo que no trae la clave, pero nadie se entera hasta revisar el registro
- Valor estimado con la táctica: ≤ 1 min, frente a ≤ 5 min de la medida [S28] y de E3 [S25]
- Cómo se estimó (cálculo o razonamiento, con unidades): el incidente se abre con el primer evento; la ventana de agrupación acota el resto a 1 min [S43].
- Tipo de la estimación: supuesto [S43] | razón o fuente: con el primer rechazo de un origen se abre el incidente; agrupar por minuto evita una alerta por petición y queda 5 veces por debajo del plazo de ≤ 5 min [S25][S28].
- Costo cuantificado: alertas falsas: hasta 240 al día si las peticiones legítimas rechazadas llegan al tope de S27 (24 M × 0,001 %). Operación: un proceso más que vigilar; si falla, los rechazos siguen pero sin alerta.
- Atributo que paga: Mantenibilidad (proceso y guardia del equipo de seguridad)

**Hasta dónde llegan estas tácticas:**
- Nuevo límite del sistema con las tácticas de este ADR (usuarios concurrentes o tasa por segundo): el servidor de aplicación pasa de ~45.800 a ~44.400 jugadores conectados. La verificación suma ~0,045 ms de cómputo a cada operación de saldo [F36].
- Cómo se calculó:
```
costo de verificar              1 ÷ 22.220 verificaciones/s = 0,045 ms                [F36]
cómputo en el pico              3.000 /s × 0,045 ms = 135 ms/s = 0,135 núcleos = 0,8 % de 16 núcleos
uso del servidor de aplicación  (3.000 + 1.200 + 135) ÷ 16.000 = 0,27                 (antes 0,26)
petición-equivalente por jugador 0,249 × (1 + 0,045) + 0,1 = 0,360 /s
límite en jugadores             16.000 ÷ 0,360 = ~44.400                              (antes 16.000 ÷ 0,349 = ~45.800)
margen sobre el pico            44.400 ÷ 12.000 = 3,7
presupuesto de verificación     20 ms ÷ 0,05 ms = 400 veces
```
- Siguiente componente que se satura: la base única (~40.000 jugadores). Sigue siendo anterior al servidor de aplicación, que además se multiplica sin tocar ningún ADR de la operación de saldo; solo obliga a replicar la lista vigente de ADR-1.
- ¿Se cumple la medida de la respuesta con la carga del pico? (sí | no | depende) y por qué: depende de S42 (a): sí, si los 5 proveedores firman el contenido con su propio esquema. Con un proveedor que no firme el contenido no se cumpliría, y por eso no se integra (Decision).
  - 100 % de la batería rechazada, 0 saldos modificados: 3 de 3 tipos (sin credencial, credencial de otro proveedor, monto modificado). Depende de S42 (a).
  - ≤ 0,001 % de legítimas rechazadas: firma válida de un proveedor legítimo no se rechaza; un reloj desfasado no interviene porque no se verifica la hora. Cumple.
  - Verificación ≤ 20 ms (p99): +0,05 ms. Cumple.
  - Saldo p99 ≤ 250 ms: +0,05 ms (más ≤ 1 ms de ADR-3 y ~1 ms de ADR-2: suma completa en la sección 6). Cumple.
  - Alerta ≤ 5 min: ≤ 1 min. Cumple.

**Consequences:**
- Gana: la batería de falsificación pasa de ~67 % a 100 % con +0,05 ms de 20 ms; cada proveedor conserva su esquema; el tráfico alterado se detecta y se alerta en ≤ 1 min.
- Paga: +0,05 ms por operación; 0,8 % de CPU del servidor de aplicación; 5 claves que guardar y rotar, y cada rotación o cambio de esquema pasa por el laboratorio (B3) [F1] en 2 a 4 semanas [S10]; hasta 240 alertas falsas al día entre ADR-3 y ADR-4 (tope común de S27).
- No resuelve: (1) la filtración del secreto de un proveedor: quien lo tenga firma como ese proveedor; (2) un proveedor que no firme el contenido (S42 (a) falla): para él la copia con el monto modificado pasaría, por eso no se integra; (3) no hay rechazo por petición vieja, porque los esquemas documentados no firman la hora [F24][F26]; las copias exactas las cubre TAC-04 (ADR-3), no este ADR; (4) un certificado propio por proveedor (opción C) queda como variante, no incluida.
- Condición de revisión (umbral medible que obliga a revisar esta decisión): revisar si (a) un proveedor actual o nuevo no firma el contenido de sus peticiones, (b) las peticiones legítimas rechazadas por ADR-3 y ADR-4, sumadas, superan 0,001 % (240 al día) [S27], o (c) la verificación supera 20 ms (p99), por ejemplo con un esquema más costoso que la firma de 2.048 bits.

**Dónde está este ADR en la entrega:** presentación HTML, pestaña "6 · ADR" (ADR-04), y `bloque-3/06-adrs.md` §5; las fichas de las tácticas en la pestaña "5 · Tácticas" y en `bloque-3/05-tacticas-y-efecto.md` §3.

---

## 6. Interacción entre las tácticas

**¿Alguna táctica de un ADR empeora la medida de otro escenario?** (sí | no; si sí, cuál, cuál y cuánto)
Sí. TAC-03 (Reconfiguración, ADR-2) empeora la medida de E3 (ADR-3): cada cambio de servidor de base de datos deja hasta ~90.000 operaciones sin respuesta (3.000/s × 30 s) que los proveedores reenvían, que es exactamente el estímulo de E3. Las repetidas tardan más de 250 ms durante ~13 s (90.000 ÷ 6.900 /s) y hasta ~39 s si cada operación llega 3 veces. También empeoran, en menor medida: TAC-02 (Latido, ADR-2) por los cambios falsos de servidor (~30 s sin saldo, ~1 retiro en espera, 0,5 min de los 43,2 min mensuales); TAC-01, TAC-04, TAC-06 y TAC-07 suman +2,15 ms al tiempo del saldo (de ~41 a ~43 ms de 250 ms); y TAC-10 (ADR-1) sube el uso de la base única de 0,30 a 0,31.

**Cómo lo resolvemos o por qué lo aceptamos:**
Los duplicados se resuelven con TAC-04 (ADR-3): el registro de identificadores va dentro de la misma transacción del saldo, así que los reenvíos devuelven la respuesta guardada y no abonan dos veces (0 duplicados). El tiempo se acepta: E3 se escribió con la base operando, y en un cambio de servidor rige E2 (p99 ≤ 250 ms a los 5 min, 300 s frente a 39 s: margen 7,7). Los cambios falsos de TAC-02 se aceptan y su frecuencia se vigila con la condición de revisión de ADR-2 (más de 20 cambios al mes). Los +2,15 ms se aceptan con un margen de 5,8 veces sobre los 250 ms. Una interacción más: TAC-05 y TAC-07 comparten el canal de alertas de TAC-08; se acepta con un tope común de 240 alertas falsas al día (S27).

---

## 7. Resumen de límites

- Carga del pico (sección 3): 3.000 operaciones de saldo por segundo (4.000 en la ráfaga), con 12.000 jugadores conectados.
- Límite de la línea base (sección 4): ~10.000 operaciones de saldo/s, ~40.000 jugadores conectados.
- Límite con todas las tácticas aplicadas: ~10.000 operaciones de saldo/s, ~40.000 jugadores conectados (sin cambio); la reconsulta de cuentas pasa de 108.000 a 1.080.000 cuentas en 6 h.
- Margen sobre el pico (límite con tácticas / carga del pico): 10.000 ÷ 3.000 = 3,3 (2,5 en la ráfaga). El margen de volumen de datos es 1,8.
- Primer componente que se satura después: la base de datos única, el servidor que atiende todas las escrituras del saldo.
- Qué haría falta para pasar ese límite (una línea; no es una decisión, es una pista para el Bloque 4): que las escrituras del saldo no pasen todas por un solo servidor de base de datos, por ejemplo repartiendo las cuentas de los jugadores entre varios grupos de servidores; pista para el Bloque 4, no una decisión.

---

## 8. Extensión

### X1 · Crecimiento de la carga
- Extensión recibida: Un evento, una campaña o una alianza multiplica por 5 la tasa del pico de su operación crítica durante 2 horas. El resto del perfil de carga se mantiene.
- Nueva carga del pico (con la cuenta): 5 × 3.000 operaciones de saldo/s = 15.000 operaciones de saldo/s durante 2 h (7.200 s), con ráfagas de 5 × 4.000 = 20.000/s. Son 15.000 × 7.200 = 108 M de operaciones, 4,5 veces un día normal, en 2 horas.
- Táctica que se sostiene y por qué: TAC-01 (Redundancia activa), TAC-04 (Transacciones), TAC-05 (Validar entradas), TAC-06 (Usar un intermediario) y TAC-07 (Verificar integridad del mensaje), porque su garantía no depende de la capacidad libre: siguen en 0 operaciones perdidas y 0 duplicados, y la verificación suma 15.000 × 0,045 ms = 0,675 núcleos (4,2 % de 16). TAC-04 sigue evitando abonos dobles aunque los reenvíos se multipliquen.
- Táctica que se rompe o deja de alcanzar, y en qué valor: TAC-03 (Reconfiguración, ADR-2) se rompe. Desde ~7.600 operaciones/s deja de volver a p99 ≤ 250 ms en 5 min si cada operación llega 3 veces durante un cambio de servidor (90 R ÷ (9.900 − R) ≤ 300), y a 15.000/s un cambio deja 450.000 operaciones sin respuesta con menos capacidad libre (9.900/s) que llegadas: la cola no se drena. Además la base única queda en uso 15.000 ÷ 10.000 = 1,5 y se pasa su límite de ~10.000/s; el servidor de aplicación llega a 1,05. TAC-01 no suma capacidad de escritura. A los 6 s la cola supera los ~3 s del proveedor y las jugadas se revierten; el jugador ve un error del juego y el proveedor reenvía, lo que agrava la saturación.
- Qué cambiaríamos: admitir como máximo 7.500 operaciones/s en la puerta de saldo (Limitar la tasa de eventos) y, al llegar al tope, rechazar descuentos nuevos y seguir aceptando abonos y reversiones (Priorizar eventos). Con el tope, la base queda en 0,75 y un cambio de servidor drena en 281 s ≤ 300 s. Costo: 50 % de las jugadas rechazadas, 27 M en las 2 h (3.750 por segundo × 7.200 s). Además, un segundo servidor de aplicación (~USD 12.000 al año) y pausar la reconsulta durante el evento (2 h + 1,94 h = 3,9 h ≤ 6 h). Más servidores no basta: la base única no se multiplica sin cambiar ADR-2 y ADR-3.
- ¿Implica un ADR nuevo o reemplazar uno existente? (ninguno | nuevo | reemplaza ADR-…): nuevo (ADR-5, admisión de operaciones de saldo bajo sobrecarga); ADR-2 no se reemplaza, pero se dispara su condición de revisión (a).

**Dónde está esto en la entrega:** presentación HTML, pestaña "8 · Extensión X1", y `bloque-3/08-extension.md`.

---

## 10. Lo que el equipo reconoce

**Números de los que menos estamos seguros, y por qué:**
1. El límite de la base de datos, ~10.000 operaciones de saldo por segundo (F34 y S36). Sale de una prueba pública hecha en otra máquina y con otros datos. Le restamos un 30 % por criterio propio, sin medirlo. El valor real puede estar entre 10.000 y 14.000/s, o por debajo si la prueba no se parece tanto a nuestra operación. De ese número dependen el margen de 3,3 sobre el pico y la conclusión de la extensión.
2. Cuántas veces reenvía cada proveedor una operación durante un cambio de servidor (supusimos 3). Depende de la política de reintentos de cada proveedor, que no está publicada. Con 3 copias, el sistema se recupera en ~39 s y el tope de 7.500/s de la extensión queda con un margen de apenas 1,07. Con más copias, ese margen desaparece.
3. El límite del proveedor de listas, 5 consultas por segundo (F37). Lo dijo el personal técnico del proveedor en su foro, no en la documentación formal, y en otra respuesta del mismo foro aparece 1/s. Con 1/s, la reconsulta en la línea base tardaría ~97 h y superaría el máximo legal de 72 h para pagar un retiro.

**Lo que más discutimos como equipo y no resolvimos:**
1. Cómo leer la extensión X1. El texto dice que el resto del perfil de carga se mantiene. Con los mismos 12.000 jugadores, cada uno tendría que hacer 1,25 operaciones por segundo, tres veces más que el juego más rápido. La lectura coherente con nuestros números es que lleguen 60.000 jugadores. Elegimos la lectura literal porque es lo que dice el enunciado, y mostramos la otra al lado, pero no sabemos cuál espera el profesor.
2. Si la respuesta a X1 merece un ADR propio. ADR-5 (admitir solo hasta 7.500 operaciones/s) sacrifica jugadas, es decir, Adecuación funcional, que tiene 2 puntos y no está entre los atributos altos. Con nuestro propio criterio del Bloque 2 no sería un escenario crítico. Lo declaramos como ADR nuevo sin formalizarlo.

**Autochequeo del equipo (marque lo que se cumple):**
- [x] Cada número del perfil de carga dice si es dato, supuesto o referencia, y tiene razón o fuente.
- [x] La tasa del pico sale de una cuenta que se puede seguir, con unidades.
- [x] Identificamos el componente que se satura primero en la línea base y con qué carga.
- [x] Cada ADR tiene Decision en voz activa y explica por qué descarta las demás opciones.
- [x] Cada táctica se nombra con el catálogo y se describe como un mecanismo concreto de nuestro sistema.
- [x] Cada táctica dice qué métrica mueve, de cuánto a cuánto y cómo lo estimamos.
- [x] Cada táctica tiene un costo cuantificado, no solo un adjetivo.
- [x] Cada ADR dice hasta dónde llegan sus tácticas y cuál es el siguiente cuello de botella.
- [x] Cada ADR tiene una condición de revisión medible.
- [x] Aplicamos la extensión a nuestros propios números.

---

## 11. Inconsistencias detectadas por la IA (solo la IA diligencia; no corrija lo anterior)

- Ninguna detectada
