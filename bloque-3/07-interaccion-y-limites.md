# Fase 7 — Interacción entre tácticas y resumen de límites

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 6 (interacción entre las tácticas) y 7 (resumen de límites)
**Criterios de la rúbrica:** Interacción entre tácticas · Hasta dónde. Nivel "Logrado": se identifica qué táctica afecta qué escenario, en cuánto, y cómo se resuelve o por qué se acepta; nuevo límite calculado, margen sobre el pico, siguiente componente que se satura y condición de revisión medible.

**Insumos:**
- `bloque-2/06-escenarios.md`: la medida de la respuesta de ESC-01 a ESC-05, incluido ESC-04 (sin ADR).
- `03-perfil-de-carga.md`: 3.000 operaciones de saldo/s (4.000 en la ráfaga), 12.000 jugadores, 0,249 operaciones/s por jugador, presupuestos de tiempo (§5.3).
- `04-cuello-de-botella.md`: límites de la línea base y uso en el pico (§3 y §5).
- `05-tacticas-y-efecto.md`: fichas TAC-01 a TAC-10 y efecto conjunto (§5).
- `06-adrs.md`: "hasta dónde" de cada ADR y condiciones de revisión.

**Qué fija esta fase:**
1. Una matriz de las diez tácticas contra los cinco escenarios (tarea 7.1).
2. Lo que las tácticas suman a los mismos recursos, contrastado con la medida de cada uno (tarea 7.2).
3. Cómo se resuelve o por qué se acepta cada interacción (tarea 7.3).
4. El resumen de límites del sistema completo (tarea 7.4).

**No se agregan fuentes ni supuestos.** Toda cifra viene de las fases 3 a 6, con su marca `F#` o `S#`; las cuentas nuevas son derivadas y se escriben completas. Lo que no cuadra en una fase ya cerrada **no se corrige aquí**: se anota en §8 como observación para la fase 9.

---

## 1. Resumen

| # | Táctica → escenario afectado | Magnitud | Resolución |
|---|------------------------------|----------|------------|
| I1 | TAC-03 (ADR-02) → ESC-03 | Cada cambio de servidor deja hasta ~90.000 operaciones sin respuesta, que los proveedores reenvían: es el estímulo de ESC-03. Las repetidas tardan más de 250 ms durante ~13 s (hasta ~39 s si cada operación llega 3 veces) | **Resuelta** en los duplicados (0, por TAC-04). **Aceptada** en el tiempo: la medida vigente en ese momento es la de ESC-02 (p99 ≤ 250 ms a los 5 min) |
| I2 | TAC-02 (ADR-02) → ESC-01, 03, 04 y 05 | Una pausa de red de más de 300 ms provoca un cambio de servidor innecesario: ~30 s sin saldo, ~1 retiro en espera, 0,5 min de los 43,2 min mensuales de S13 | **Aceptada**, acotada por la condición de revisión (b) de ADR-02 |
| I3 | TAC-01, TAC-04, TAC-06 y TAC-07 → tiempo del saldo en ESC-01, 03, 04 y 05 | +2,15 ms sobre ~41 ms: ~43 ms de 250 ms | **Aceptada**: margen de 5,8 veces |
| I4 | TAC-10 (ADR-01) → saldo de ESC-02 a ESC-05 | Uso de la base única de 0,30 a 0,31 (0,40 a 0,41 en la ráfaga) | **Resuelta** por el propio tope de 50 cuentas/s |
| I5 | TAC-05 (ADR-03) y TAC-07 (ADR-04) → TAC-08 y la carga del equipo de seguridad | Hasta 240 alertas falsas al día si el tope de S27 es común; hasta 480 si es por fuente | **Decisión del equipo** (§9, decisión 2) |
| I6 | TAC-09 (ADR-01) → multiplicar el servidor de aplicación | N copias de ~500 MB y N cargas de lista que coordinar; hoy N = 1: sin efecto en el pico | **Aceptada hoy**; pista para el Bloque 4 |

**Lectura:** ninguna interacción rompe una medida con la carga del pico. Las tres que sí empeoran un número de otro escenario (I1, I2, I3) lo hacen en una ventana corta o por menos del 1 % del presupuesto. La que más importa para la sustentación es I1: es la única donde una táctica **crea** el estímulo de otro escenario.

---

## 2. Matriz táctica × escenario (tarea 7.1)

**▲** mejora la medida del escenario · **▼** la empeora (con la magnitud) · **·** sin efecto. Las medidas que se contrastan: ESC-01 (evaluación del retiro p95 ≤ 1 s; saldo p99 ≤ 250 ms durante la reconsulta), ESC-02 (0 perdidas; restablecer ≤ 15 min; p99 ≤ 250 ms a los 5 min), ESC-03 (0 duplicados; repetidas p99 ≤ 250 ms), ESC-04 (720 abonos p99 ≤ 250 ms; saldo visible p95 ≤ 2 s) y ESC-05 (batería 100 %; ≤ 0,001 % legítimas rechazadas; verificación ≤ 20 ms; alerta ≤ 5 min).

| Táctica | ESC-01 | ESC-02 | ESC-03 | ESC-04 | ESC-05 |
|---------|--------|--------|--------|--------|--------|
| TAC-01 Redundancia activa | ▼ +1 ms en el saldo | ▲ perdidas ~3.000 → 0 | ▼ +1 ms | ▼ +1 ms | ▼ +1 ms |
| TAC-02 Latido | ▼ cambio falso (I2) | ▲ detección 10–30 min → ≤ 0,3 s | ▼ cambio falso (I2) | ▼ cambio falso (I2) | ▼ cambio falso (I2) |
| TAC-03 Reconfiguración | ▼ ~1 retiro espera ≤ 30 s | ▲ restablecer 10–30 min → ≤ 30 s | ▼ hasta ~90.000 reenvíos (I1) | ▼ 720 abonos sin respuesta ≤ 30 s, si el cambio cae en la ráfaga | ▼ ≤ 30 s sin saldo |
| TAC-04 Transacciones | ▼ +≤ 1 ms | · | ▲ duplicados 1 por par → 0 | ▼ +≤ 1 ms; 720 inserciones | ▲ cubre la copia exacta · ▼ +≤ 1 ms |
| TAC-05 Validar entradas | · | · | ▲ 0 % → 100 % de misma clave con datos distintos | · | ▼ alertas falsas (I5) |
| TAC-06 Usar un intermediario | ▼ < 0,1 ms | · | ▲ proveedor nuevo: 1 pieza | ▼ < 0,1 ms | ▲ 1 pieza por esquema · ▼ < 0,1 ms |
| TAC-07 Verificar integridad | ▼ +0,05 ms | · | ▼ +0,05 ms | ▼ +0,05 ms; 720 × 0,045 ms = 32 ms de cómputo ese segundo | ▲ batería 67 % → 100 % |
| TAC-08 Informar a los actores | · | · | ▲ alerta de la clave repetida ≤ 1 min | · | ▲ alerta sin alerta → ≤ 1 min · ▼ alertas falsas (I5) |
| TAC-09 Múltiples copias de datos | ▲ evaluación hasta ~19,4 h → ≤ 10 ms | · | · | · | · |
| TAC-10 Limitar la tasa de eventos | ▲ reconsulta ~19,4 h → ~1,9 h | ▼ +0,01 de uso (I4) | ▼ +0,01 de uso | ▼ +0,01 de uso | ▼ +0,01 de uso |

**ESC-04 (sin ADR).** Es el escenario que no eligió una táctica, pero las usa: sus 720 abonos pasan por TAC-06, TAC-07, TAC-04 y TAC-01. Ninguna lo rompe (ver §3.1 y §3.5).

**Qué falta en la matriz, a propósito.** La fila de TAC-09 y TAC-10 en ESC-02 a ESC-05 no incluye la memoria de ~500 MB de TAC-09 (0,4 % de 128 GB [S42]) porque ninguna medida de esos escenarios depende de memoria.

---

## 3. Presupuestos compartidos (tarea 7.2)

### 3.1 Tiempo de la operación de saldo (ADR-02, ADR-03 y ADR-04)

Los tres ADR de la operación de saldo suman al mismo camino de la respuesta: ADR-02 y ADR-03 en el paso 5 de `02` §3.1, y ADR-04 y la pieza compartida (TAC-06) en el paso 4. Contra la medida de ESC-02 a ESC-05: **p99 ≤ 250 ms** [S14].

| Componente del tiempo | Valor | Origen |
|-----------------------|-------|--------|
| Base única, saturada (cota) | < 40 ms | [F34] (p99 de 31 a 39 ms), `04` §2.1 |
| Servidor de aplicación | ~1 ms | [S33] |
| TAC-01: grabación en una segunda copia | ~1 ms | [S40] |
| TAC-04: una inserción más en el índice | ≤ 1 ms | [S41] |
| TAC-07: verificar la firma | 0,045 ms | [F36] |
| TAC-06: llamada dentro del proceso | < 0,1 ms | `05` TAC-06 |

```
tiempo con las tácticas   40 + 1 + 1 + 1 + 0,045 + 0,1 = 43,1 ms
lo que suman las tácticas 1 + 1 + 0,045 + 0,1 = 2,15 ms   (2,15 ÷ 250 = 0,9 %)
margen sobre la medida    250 ÷ 43,1 = 5,8
verificación (ESC-05)     0,045 ms de los 20 ms de S26 = 0,2 %   (margen 20 ÷ 0,05 = 400)
```

**Dos precisiones.** (1) Sumar los p99 de cada paso es una cota prudente: es raro que todos los pasos estén en su peor 1 % en la misma petición. (2) F34 ya mide con la espera de una réplica, así que el ~1 ms de TAC-01 puede estar contado dos veces; se deja sumado porque la cota es más segura. **La medida se cumple con margen de 5,8.**

### 3.2 Capacidad de la base única (ADR-01, ADR-02, ADR-03)

```
carga del saldo                 3.000 transacciones/s                         (ráfaga: 4.000)
reconsulta (TAC-10)             50 lecturas/s + 50 escrituras/s = 100 /s
uso con las tácticas            (3.000 + 100) ÷ 10.000 = 0,31                 (ráfaga: 4.100 ÷ 10.000 = 0,41)
capacidad libre para el saldo   10.000 − 100 = 9.900 /s                      → 9.900 ÷ 3.000 = 3,3
```

TAC-01 no suma capacidad: solo un servidor atiende las escrituras (`06` ADR-02). TAC-04 suma 3.000 inserciones/s, pero dentro de la misma transacción de saldo y por debajo del trabajo de la transacción de referencia [F35], así que el límite no cambia (`05` §2.2). **El uso sube de 0,30 a 0,31 y el margen sobre el pico se mantiene en 3,3.**

### 3.3 Minutos de caída al mes (ADR-02, I2)

```
presupuesto mensual (S13)       0,001 × 43.200 min = 43,2 min al mes
costo de un cambio de servidor  ≤ 30 s = 0,5 min                   [S40]
cambios que agotan el presupuesto  43,2 ÷ 0,5 = 86 al mes
umbral de revisión de ADR-02    20 cambios × 0,5 min = 10 min      (10 ÷ 43,2 = 23 %)
```

### 3.4 Rechazos legítimos y alertas (ADR-03 y ADR-04)

```
tope de rechazos legítimos      0,001 % × 24 M = 240 al día        [S27]
si el tope es común             240 alertas/día = 1 cada 86.400 ÷ 240 = 6 min
si el tope es por fuente        240 + 240 = 480 al día = 1 cada 86.400 ÷ 480 = 3 min
```

El plazo de la alerta (≤ 5 min [S25][S28]) no cambia con ninguna de las dos lecturas: el incidente se abre con el primer evento (TAC-08). Lo que crece es la carga del equipo de seguridad.

### 3.5 Servidor de aplicación (ADR-04, ESC-04)

```
TAC-07 en el pico        3.000 /s × 0,045 ms = 135 ms/s = 0,135 núcleos = 0,8 % de 16 núcleos
uso en el pico           (3.000 × 1,045 + 1.200) ÷ 16.000 = 0,27      (antes 0,26)
uso en la ráfaga (cota)  (4.000 × 1,045 + 1.200) ÷ 16.000 = 0,34      (antes 0,33)
ráfaga de ESC-04         (3.560 × 1,045 + 1.200) ÷ 16.000 = 0,31      (3.560 reconstruidas, `03` §4)
```

---

## 4. Interacciones en detalle (tarea 7.3)

### I1 · TAC-03 Reconfiguración (ADR-02) → ESC-03

- **Qué pasa.** Cuando cae el servidor de base de datos que atiende, el cambio de servidor tarda hasta 30 s [S40]. En ese tiempo las operaciones de saldo no se responden y, pasados los ~3 s de espera del proveedor, este las reenvía o las revierte [F24].
- **Magnitud.**

```
operaciones sin respuesta en el cambio   3.000 /s × 30 s = 90.000          (ráfaga: 4.000 × 30 = 120.000)
tiempo en drenar 1 copia por operación   90.000 ÷ (10.000 − 3.000 − 100) /s = 13 s    (ráfaga: 120.000 ÷ 5.900 = 20 s)
con 3 copias por operación               270.000 ÷ 6.900 = 39 s            (3 envíos: el estímulo de ESC-03)
medida vigente                           p99 ≤ 250 ms a los 5 min = 300 s  [S24]  → margen 300 ÷ 39 = 7,7
copias por operación que agotan los 5 min  300 s × 6.900 /s ÷ 90.000 = 23   (ráfaga: 14)
```

- **Qué empeora.** La medida de ESC-03 "repetidas en p99 ≤ 250 ms" no se cumple durante esos ~13 a ~39 s. Los duplicados **no** pasan: TAC-04 devuelve la respuesta guardada sin tocar el saldo.
- **Resolución.** (1) Los 0 duplicados se mantienen porque TAC-04 es indivisible con el cambio de saldo (`05` §2.1, interacción anotada para esta fase). (2) El tiempo se acepta: ESC-03 se escribió con la base operando (`bloque-2/06-escenarios.md`, entorno de ESC-03) y no con un cambio de servidor; en ese momento rige ESC-02, que da 5 min para volver a p99 ≤ 250 ms.
- **Cifra frágil.** Cuántas copias envía cada proveedor durante 30 s no está en las fuentes: depende de su política de reintentos. El número de 23 copias es la holgura, no una estimación. Se anota para la fase 9.

### I2 · TAC-02 Latido (ADR-02) → ESC-01, 03, 04 y 05

- **Qué pasa.** Una pausa de red de más de 300 ms hace que los otros dos servidores declaren caído al que atiende, aunque no lo esté. El cambio cuesta lo mismo que uno verdadero (`05` TAC-02 y TAC-03).
- **Magnitud.** ~30 s sin saldo [S40]; ~90.000 operaciones sin respuesta (I1); retiros en espera: 125 por hora ÷ 3.600 s × 30 s = **~1** (1 de 125 = 0,8 %, bajo el 5 % que tolera el p95 de ESC-01); 0,5 min de los 43,2 min mensuales (§3.3).
- **Resolución.** Se acepta. El peor caso mensual lo acota la condición de revisión (b) de ADR-02: más de 20 cambios al mes (10 min, 23 % del presupuesto). Si las pausas de red de más de 300 ms son frecuentes, el intervalo de 150 a 300 ms [F40] es lo primero que se revisa.

### I3 · TAC-01, TAC-04, TAC-06 y TAC-07 → tiempo del saldo (ESC-01, 03, 04 y 05)

- **Magnitud.** +2,15 ms sobre ~41 ms (§3.1): ~43 ms de 250 ms, 0,9 % del presupuesto.
- **Resolución.** Se acepta: margen de 5,8 veces. La suma de ADR-02, ADR-03 y ADR-04 sobre el mismo paso, que la fase 6 dejó pendiente, queda calculada.
- **ESC-04.** Los 720 abonos del mismo segundo pasan por las cuatro tácticas y suman 720 × 0,045 ms = 32 ms de cómputo ese segundo; el uso del servidor de aplicación llega a 0,31 (§3.5). La medida de ESC-04 (p99 ≤ 250 ms) se cumple.

### I4 · TAC-10 Limitar la tasa de eventos (ADR-01) → saldo de ESC-02 a ESC-05

- **Magnitud.** +0,01 de uso de la base única (§3.2). En un cambio de servidor, la capacidad libre baja de 7.000 a 6.900 /s y el drenaje pasa de 12,9 s (`06` ADR-02) a 13,0 s.
- **Resolución.** Resuelta por el diseño: el tope de 50 cuentas/s existe para que la reconsulta no tome la capacidad libre (`05` TAC-10: sin tope, el uso iría de 0,30 a ~1,0). Subir a 100 cuentas/s, la variante de ADR-01, duplicaría este efecto a +0,02.

### I5 · TAC-05 (ADR-03) y TAC-07 (ADR-04) → TAC-08 y el equipo de seguridad

- **Qué pasa.** Los dos rechazos (misma clave con datos distintos, firma inválida) dejan eventos en el mismo canal de TAC-08 (`05` TAC-08). Las fases 5 y 6 escriben "hasta 240 alertas falsas al día [S27]" en cada ADR, sin decir si los 240 son comunes o por fuente.
- **Magnitud.** 240 al día si el tope es común (1 alerta cada 6 min); 480 si es por fuente (1 cada 3 min). Ninguna lectura rompe el plazo de ≤ 5 min.
- **Resolución.** **Decisión del equipo** (§9, decisión 2). Recomendación: tope común, porque S27 mide los rechazos indebidos de la verificación y es el número más estricto; con él, las dos frases de la fase 6 son consistentes.

### I6 · TAC-09 Múltiples copias de datos (ADR-01) → el servidor de aplicación multiplicado

- **Qué pasa.** La lista vigente vive en la memoria del servidor de aplicación. Con N servidores hay N copias de ~500 MB [S42] y cada actualización se carga N veces; mientras cada uno carga, sus retiros esperan ≤ 1 min.
- **Magnitud.** Hoy N = 1: sin efecto en el pico. Con N = 2, 1.000 MB y 2 cargas coordinadas.
- **Resolución.** Se acepta hoy. Es una pista para el Bloque 4: la lista tendría que consultarse desde un lugar común o repartirse con un único responsable de la carga. Es la misma advertencia del ejemplo del material ("cada instancia tiene su propio índice").

---

## 5. Sinergias (tarea 7.1)

Las tácticas también se ayudan entre sí. Se anotan porque son lo que justifica el conjunto:

| Pareja | Qué se gana |
|--------|-------------|
| TAC-04 con TAC-03 | Los ~90.000 reenvíos no duplican abonos (I1) |
| TAC-04 con TAC-07 | La copia exacta de una petición legítima la cubre TAC-04; la copia con el monto modificado la cubre TAC-07. Juntas dejan sin hueco la batería de ESC-05 (`05` §3) |
| TAC-06 con TAC-07 y TAC-04 | Una sola pieza por proveedor sirve a ADR-03 y a ADR-04: 5 piezas, no 10 |
| TAC-08 con TAC-05 y TAC-07 | Un solo canal de alertas para dos escenarios (con el costo de I5) |
| TAC-09 con TAC-10 | La reconsulta compara contra la lista propia: el proveedor baja de uso 3,24 a 0,002 |

---

## 6. Resumen de límites (tarea 7.4)

### 6.1 Por componente

| Componente | Línea base | Con todas las tácticas | Qué lo mueve |
|-----------|-----------|------------------------|--------------|
| **Base única: saldo** | ~10.000/s · uso 0,30 · ~40.000 jugadores | **~10.000/s · uso 0,31 · ~40.000 jugadores** (9.900 ÷ 0,249 = 39.800) | TAC-10 suma 0,01; TAC-01 y TAC-04 no mueven el límite |
| Servidor de aplicación | 16.000 peticiones/s · ~45.800 jugadores | 16.000 peticiones/s · **~44.400 jugadores** | TAC-07: 0,249 × 1,045 + 0,1 = 0,360 /s por jugador |
| Base única: volumen | 1,1 TB de historial | **1,4 TB**; tope de 2,5 TB (margen **1,8**) | TAC-04: ~300 GB de registro |
| Proveedor de listas | 5 consultas/s · uso 3,24 · 108.000 cuentas en 6 h | Uso **0,002** | TAC-09: la reconsulta deja de consultarlo |
| Reconsulta | ~19,4 h | ~1,9 h · hasta **1.080.000 cuentas** en 6 h | TAC-10: 50 × 21.600 = 1.080.000 |
| Red pública | 1 Gbit/s · 0,24 | Sin cambio | Ninguna táctica agrega bytes públicos |
| Copia al alterno | ~3.000 operaciones en riesgo (S37) | **Sin cambio**: la pérdida del sitio completo sigue sin cubrirse | TAC-01 cubre la caída de un servidor, no la del sitio |

### 6.2 Los números para la plantilla

```
carga del pico                  3.000 operaciones de saldo/s    (ráfaga: 4.000)         [S6]
límite de la línea base         ~10.000 operaciones de saldo/s  (~40.000 jugadores)     [F34][S36]
límite con las tácticas         ~10.000 operaciones de saldo/s  (~40.000 jugadores)
margen sobre el pico            10.000 ÷ 3.000 = 3,3            (ráfaga: 10.000 ÷ 4.000 = 2,5)
siguiente componente            la base única, el servidor que atiende todas las escrituras
margen más estrecho             volumen de datos: 2,5 TB ÷ 1,4 TB = 1,8
```

**Lectura.** Las diez tácticas compran garantías (0 pérdidas, 0 duplicados, origen verificado, retiro evaluado al instante), no capacidad: el límite del sistema es el mismo de la línea base. Dos matices que conviene decir en la sustentación:

1. **El margen de 3,3 es de tasa. El de volumen es 1,8**, más estrecho: a ~43 M de operaciones al día los datos llegan al escalón que S36 ya descontó completo (`06` ADR-03). Crecer más que eso no rompe la base, pero el límite de 10.000/s deja de tener sustento en F34.
2. **El servidor de aplicación se acerca a la base:** la distancia entre los dos baja de ~5.800 a ~4.400 jugadores por TAC-07, y ahora lleva una lista en memoria (I6).

### 6.3 Qué haría falta para pasar el límite (pista para el Bloque 4, no una decisión)

Que las escrituras del saldo no pasen todas por un solo servidor de base de datos: por ejemplo, repartir las cuentas de los jugadores entre varios grupos de tres servidores. Cada grupo repetiría el costo de TAC-01 (2 servidores más) y las lecturas entre cuentas de grupos distintos ya no serían una sola transacción. Cuál estilo lo resuelve es del Bloque 4.

---

## 7. Texto para la plantilla (secciones 6 y 7)

*Para transcribir en la fase 10. Redactado con la recomendación de cada decisión del §9.*

**6. Interacción entre las tácticas**

**¿Alguna táctica de un ADR empeora la medida de otro escenario?** Sí. TAC-03 (Reconfiguración, ADR-02) empeora la medida de ESC-03 (ADR-03): cada cambio de servidor de base de datos deja hasta ~90.000 operaciones sin respuesta (3.000/s × 30 s) que los proveedores reenvían, que es exactamente el estímulo de ESC-03. Las repetidas tardan más de 250 ms durante ~13 s (90.000 ÷ 6.900 /s) y hasta ~39 s si cada operación llega 3 veces. También empeoran, en menor medida: TAC-02 (Latido, ADR-02) por los cambios falsos de servidor (~30 s sin saldo, ~1 retiro en espera, 0,5 min de los 43,2 min mensuales); TAC-01, TAC-04, TAC-06 y TAC-07 suman +2,15 ms al tiempo del saldo (de ~41 a ~43 ms de 250 ms); y TAC-10 (ADR-01) sube el uso de la base única de 0,30 a 0,31.

**Cómo lo resolvemos o por qué lo aceptamos:** los duplicados se resuelven con TAC-04 (ADR-03): el registro de identificadores va dentro de la misma transacción del saldo, así que los reenvíos devuelven la respuesta guardada y no abonan dos veces (0 duplicados). El tiempo se acepta: ESC-03 se escribió con la base operando, y en un cambio de servidor rige ESC-02 (p99 ≤ 250 ms a los 5 min, 300 s frente a 39 s: margen 7,7). Los cambios falsos de TAC-02 se aceptan y su frecuencia se vigila con la condición de revisión de ADR-02 (más de 20 cambios al mes). Los +2,15 ms se aceptan con un margen de 5,8 veces sobre los 250 ms. Una interacción más: TAC-05 y TAC-07 comparten el canal de alertas de TAC-08; se acepta con un tope común de 240 alertas falsas al día (S27).

**7. Resumen de límites**

- Carga del pico (sección 3): 3.000 operaciones de saldo por segundo (4.000 en la ráfaga), con 12.000 jugadores conectados.
- Límite de la línea base (sección 4): ~10.000 operaciones de saldo/s, ~40.000 jugadores conectados.
- Límite con todas las tácticas aplicadas: ~10.000 operaciones de saldo/s, ~40.000 jugadores conectados (sin cambio); la reconsulta de cuentas pasa de 108.000 a 1.080.000 cuentas en 6 h.
- Margen sobre el pico (límite con tácticas / carga del pico): 10.000 ÷ 3.000 = 3,3 (2,5 en la ráfaga). El margen de volumen de datos es 1,8.
- Primer componente que se satura después: la base de datos única, el servidor que atiende todas las escrituras del saldo.
- Qué haría falta para pasar ese límite: que las escrituras del saldo no pasen todas por un solo servidor de base de datos, por ejemplo repartiendo las cuentas de los jugadores entre varios grupos de servidores; pista para el Bloque 4, no una decisión.

**Dónde está esto en la entrega:** presentación HTML, secciones "Interacción" y "Límites" (se integran en la fase 10), y `bloque-3/07-interaccion-y-limites.md`.

---

## 8. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| La matriz cubre las 10 tácticas y los 5 escenarios, incluido ESC-04 | ✔ §2 |
| Cada interacción tiene táctica, escenario afectado, magnitud en número y resolución | ✔ §1 y §4, I1 a I6 |
| Suma de lo que ADR-02, ADR-03 y ADR-04 agregan al paso de escritura, contra la medida de tiempo | ✔ §3.1: 43,1 ms de 250 ms, margen 5,8 |
| Margen sobre el pico calculado y ningún margen menor que 1 marcado como cumplido | ✔ 3,3 · 2,5 · 1,8 · 5,8 · 7,7 |
| Siguiente componente que se satura y pista para el Bloque 4 | ✔ §6.3, marcada como no decisión |
| Cuentas recalculadas | ✔ 3.000 × 30 = 90.000; 90.000 ÷ 6.900 = 13,0 s; 120.000 ÷ 5.900 = 20,3 s; 270.000 ÷ 6.900 = 39,1 s; 300 ÷ 39,1 = 7,7; 300 × 6.900 ÷ 90.000 = 23; (3.000 + 100) ÷ 10.000 = 0,31; 125 ÷ 3.600 × 30 = 1,04; 43,2 ÷ 0,5 = 86; 10 ÷ 43,2 = 23 %; 40 + 1 + 1 + 1 + 0,045 + 0,1 = 43,1 y 250 ÷ 43,1 = 5,8; 720 × 0,045 = 32,4; (3.560 × 1,045 + 1.200) ÷ 16.000 = 0,31; 86.400 ÷ 240 = 360 s = 6 min; 9.900 ÷ 0,249 = 39.759; 2,5 ÷ 1,4 = 1,8 |
| No se agregaron fuentes ni supuestos | ✔ Solo se usan F24, F34 a F36, S6, S13, S14, S24 a S28, S33, S36, S37, S40 a S42 y F40, ya registradas; ningún `F#` ni `S#` nuevo |
| Las cifras de las fases 3 a 6 no cambiaron | ✔ Se reutilizan sin modificar |
| Solo nombres de táctica del catálogo y ningún mecanismo con marcas | ✔ |
| Ninguna fase cerrada se modificó | ✔ Solo se agregó este archivo |

**Observaciones para la fase 9 (candidatas a `H#`; no se corrigen aquí):**
1. **"El servidor de aplicación se multiplica sin tocar ningún ADR"** (`04` §3, lectura 3, y `06` ADR-04, siguiente componente). Con TAC-09 la lista vive en ese servidor, así que multiplicarlo sí toca ADR-01 (I6). Es un matiz de redacción, no de cifras.
2. **El tope de 240 al día de S27** aparece en ADR-03 y en ADR-04 como si fuera de cada uno (I5). Hay que decir que es común.
3. **El "≈ 13 s" de `06` ADR-02** cuenta 1 copia por operación y 7.000/s libres. Con TAC-10 son 6.900/s (13,0 s) y, con 3 copias, ~39 s (I1). No cambia la conclusión.
4. **La política de reintentos de los proveedores** (cuántas copias por operación durante 30 s) no tiene fuente. Es el número menos seguro de esta fase.
5. **`05` §5** suma +~2 ms sin la pieza de TAC-06 (< 0,1 ms); con ella son 2,15 ms (§3.1).

**Observaciones para la fase 8 (extensión X1):** a 5 × 3.000 = 15.000 operaciones/s la base única (~10.000/s) queda en uso 1,5. TAC-01 y TAC-04 no mueven ese límite y los reenvíos de I1 crecen con la tasa. La respuesta no puede ser "más servidores" (§6.3).

---

## 9. Punto de control (cerrado el 15 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las tres recomendaciones.

| # | Decisión | Aprobado | Variantes registradas |
|---|----------|----------|-----------------------|
| 1 | Interacción principal de la sección 6 de la plantilla | **I1** (TAC-03 → ESC-03): es la única donde una táctica crea el estímulo de otro escenario, y las demás se listan en una frase | I3 (tiempo del saldo, +2,15 ms): más fácil de defender, pero no muestra una interacción real entre ADR |
| 2 | Alcance del tope de 240 alertas falsas al día (S27) | **Común** a TAC-05 y TAC-07: 240 al día, 1 cada 6 min | Por fuente: 480 al día, 1 cada 3 min; las frases de la fase 6 seguirían diciendo "hasta 240" por ADR |
| 3 | Redacción de la pista para el Bloque 4 | **§6.3 con el ejemplo** (repartir las cuentas entre varios grupos de servidores) | Sin ejemplo: "que las escrituras del saldo no pasen todas por un solo servidor"; no adelanta el estilo del Bloque 4 |

**Para llevar a la sustentación:** I1 y el margen de volumen (1,8 frente a 3,3) son las dos preguntas más probables. El número más débil de la fase es el de copias por operación en un cambio de servidor (observación 4).

La fase 8 (extensión X1) puede arrancar.
