# Fase 8 — Extensión X1: crecimiento de la carga

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 8 (Extensión, solo X1: pregrado)
**Criterio de la rúbrica:** Extensión. Nivel "Logrado": la extensión del nivel aplicada a los números propios: qué se sostiene, qué se rompe y en qué valor, qué ve el usuario y si implica un ADR nuevo.

**Insumos:**
- `03-perfil-de-carga.md`: 3.000 operaciones de saldo/s (4.000 en la ráfaga), 12.000 jugadores, 0,249 operaciones/s por jugador (§1, §2.1, §4).
- `04-cuello-de-botella.md`: límites y uso de la línea base (§2, §3) y qué ve cada actor al superar el límite (§6.1).
- `05-tacticas-y-efecto.md`: fichas TAC-01 a TAC-10.
- `06-adrs.md`: "hasta dónde" y condiciones de revisión de los cuatro ADR.
- `07-interaccion-y-limites.md`: límites con todas las tácticas (§6), presupuestos compartidos (§3), I1 a I6 y la observación para esta fase (§8).
- `docs/class-material/Tácticas y sizing · Bloque 3.html`: texto de X1 (constante `GEN_EXT`) y campos de la sección 8 de la plantilla.

**Qué fija esta fase:**
1. La nueva tasa del pico, calculada con la cadena de la fase 3 (tarea 8.2).
2. Qué táctica se sostiene, cuál deja de alcanzar y en qué valor (tarea 8.3).
3. Qué ve cada actor cuando se rompe (tarea 8.4).
4. Qué se cambiaría y si implica un ADR nuevo o reemplazar uno (tarea 8.5).

**No se agregan fuentes ni supuestos.** Toda cifra viene de las fases 3 a 7 con su marca `F#` o `S#`; las cuentas nuevas son derivadas y se escriben completas. Lo que no cuadra en una fase cerrada **no se corrige aquí**: se anota en §10 como observación para la fase 9.

---

## 1. Resumen

| Pregunta | Respuesta |
|----------|-----------|
| ¿Cuál es la nueva carga? | 5 × 3.000 = **15.000 operaciones de saldo/s** durante 2 h (7.200 s); 20.000/s en la ráfaga |
| ¿Qué se satura? | La base única: **uso 1,5** (1,07 con el extremo alto de 14.000/s). El servidor de aplicación llega a 1,05 |
| ¿Qué táctica se rompe primero? | **TAC-03 (Reconfiguración)**: deja de alcanzar desde ~7.600 operaciones/s si cada operación llega 3 veces durante un cambio de servidor (~9.000 con 1 copia). A 15.000/s no se recupera nunca |
| ¿Qué se sostiene? | Las garantías que no dependen de la capacidad: 0 pérdidas (TAC-01), 0 duplicados (TAC-04), verificación de origen (TAC-07: 0,045 ms) |
| ¿Qué ve el usuario? | A los **6 s** la cola supera los ~3 s de espera del proveedor [S14]: desde ahí las jugadas fallan y se revierten, y los reenvíos agravan la saturación (`04` §6.1) |
| ¿Qué cambiaríamos? | **Admitir solo hasta 7.500 operaciones/s** y rechazar el resto de los descuentos de forma explícita (*Limitar la tasa de eventos* y *Priorizar eventos*). No es "más servidores": la base no se multiplica sin cambiar ADR-02 y ADR-03 |
| ¿Implica un ADR nuevo? | **Sí: ADR-05** (admisión de operaciones de saldo bajo sobrecarga). No reemplaza a ninguno; sí dispara la condición de revisión (a) de ADR-02 |
| Costo de la respuesta | **~27 M de jugadas rechazadas** en las 2 h (50 %) y un segundo servidor de aplicación (~USD 12.000 al año [F41]). Sin ella, desde el segundo 6 las jugadas fallan y se revierten (en el mejor caso, 1 de cada 3: 36 M de operaciones en las 2 h) |

**Lectura.** X1 no revela un defecto de las tácticas: revela que **las diez tácticas compran garantías, no capacidad** (`07` §6.2). El límite del sistema con todas ellas es el mismo de la línea base, ~10.000/s, y 15.000/s lo supera por 1,5 veces. Lo que cambia es qué se pierde cuando se supera: con las tácticas no se pierde dinero ni se duplica nada; se pierde la jugada.

---

## 2. Extensión recibida y su lectura (tarea 8.1)

> **X1 · Crecimiento de la carga.** Un evento, una campaña o una alianza multiplica por 5 la tasa del pico de su operación crítica durante 2 horas. El resto del perfil de carga se mantiene.

La operación crítica es la de saldo (fase 1). Se lee así: la **tasa** de operaciones de saldo se multiplica por 5; "el resto del perfil" (retiros, reconsulta, historial, presupuestos de tiempo) no cambia.

**Una precisión que el equipo debe conocer** (decisión 1, §11). Con los mismos 12.000 jugadores conectados, 15.000 operaciones/s serían 15.000 ÷ 12.000 = **1,25 operaciones/s por jugador**, y el juego más rápido da 0,4 (1 giro cada 5 s × 2 operaciones por jugada, `03` §2.1). La tasa solo puede multiplicarse por 5 con más jugadores a ritmo normal: 5 × 12.000 = **60.000 conectados**. Las dos lecturas llevan la base única al mismo uso (la base solo ve operaciones por segundo), pero **no al mismo uso del servidor de aplicación y de la red** (§4). La recomendación es responder con la lectura literal, que es la que dice el texto, y mostrar la segunda al lado.

---

## 3. Nueva carga (tarea 8.2)

Sin números nuevos: la tasa de la fase 3 por el factor de X1.

```
tasa del pico              5 × 3.000 = 15.000 operaciones de saldo/s        [S6][X1]
                           (5 × 2.992 = 14.960; se usa 15.000 como en la fase 7)
ráfaga                     5 × 4.000 = 20.000 operaciones/s                  (supuesto: la ráfaga escala igual)
duración                   2 h = 7.200 s
operaciones en las 2 h     15.000 × 7.200 = 108 M
                           108 ÷ 24 = 4,5 días normales; 108 ÷ 58 = 1,9 días pico, comprimidos en 2 h
jugadas por segundo        15.000 ÷ 2 = 7.500 (1 descuento + 1 abono por jugada [S6])
```

---

## 4. Uso por componente con la nueva carga

Límites de `04` §3 y de `07` §6.1. **Lectura literal** = 12.000 conectados; **lectura de 60.000** = 5 × 12.000 conectados al ritmo de siempre (decisión 1).

| Componente | Cuenta (lectura literal) | Uso literal | Uso con 60.000 | ¿Se sostiene? |
|-----------|--------------------------|-------------|----------------|---------------|
| **Base única: saldo** | 15.000 ÷ 10.000 [F34][S36] | **1,5** (1,07 con 14.000/s) · ráfaga 2,0 | 1,5 | **No** |
| Servidor de aplicación | (15.000 × 1,045 + 1.200) ÷ 16.000 [S33][F36] | **1,05** · ráfaga 1,38 | (15.000 × 1,045 + 6.000) ÷ 16.000 = **1,35** | **No** (en el límite con la lectura literal) |
| Red pública | 15.000 × 2 KB = 240 Mbit/s + 1.200 × 20 KB = 192 Mbit/s = 432 ÷ 1.000 [S35] | 0,43 | 240 + 960 = 1.200 ÷ 1.000 = **1,2** | Sí / **No** |
| Base del alterno | Recibe las mismas escrituras que la principal [S32] | 1,5 | 1,5 | **No** |
| Volumen de datos | 108 M × (250 + 100) bytes [S34][S41] = 37,8 GB; 1,4 + 0,04 = 1,44 TB; 2,5 ÷ 1,44 | margen 1,7 (antes 1,8) | igual | Sí |
| Retiros y reconsulta | No cambian (X1: "el resto se mantiene") | sin cambio | sin cambio | Sí |

```
tasa a la que se satura cada componente (literal):
  servidor de aplicación   (16.000 − 1.200) ÷ 1,045 = 14.162 /s      con 60.000: (16.000 − 6.000) ÷ 1,045 = 9.569 /s
  base única               ~10.000 /s (9.900 /s con la reconsulta corriendo, TAC-10)
```

**Lectura.** Con 15.000/s se superan **dos** límites a la vez: el de la base (que ninguna táctica movió) y, en el borde, el del servidor de aplicación. La diferencia entre ellos importa para la respuesta (§8): el servidor de aplicación se multiplica sin tocar ningún ADR; la base única no.

---

## 5. Escalera: a qué tasa deja de alcanzar cada cosa

Ordenada por tasa. Cuentas completas; los 300 s son la medida de ESC-02 (p99 ≤ 250 ms a los 5 min [S24]) y 30 s el cambio de servidor [S40].

| Tasa (operaciones/s) | Qué deja de alcanzar | Cuenta |
|----------------------|----------------------|--------|
| **~7.500** | La ráfaga de 4/3 deja de caber en la base (condición de revisión (a) de ADR-02) | 10.000 ÷ (4.000 ÷ 3.000) = 7.500 |
| **~7.600** | TAC-03 deja de volver a p99 ≤ 250 ms en 5 min si cada operación llega **3 veces** durante un cambio de servidor | 90 R ÷ (9.900 − R) ≤ 300 → R ≤ 2.970.000 ÷ 390 = 7.615 |
| **~9.000** | Lo mismo con **1 copia** por operación | 30 R ÷ (9.900 − R) ≤ 300 → R ≤ 2.970.000 ÷ 330 = 9.000 |
| **~9.900** | Se satura el único servidor que escribe: TAC-01 no le suma capacidad de escritura | 10.000 − 100 de la reconsulta |
| ~14.200 | Se satura el servidor de aplicación (lectura literal; 9.600 con 60.000) | (16.000 − 1.200) ÷ 1,045 = 14.162 |

La primera línea es la que ya anticipaba ADR-02: **X1 dispara su condición de revisión (a)** (pico sostenido > ~7.500/s). Las dos siguientes son consecuencia de la interacción I1 de la fase 7: los reenvíos que crea TAC-03 crecen con la tasa.

---

## 6. Qué se sostiene y qué se rompe, táctica por táctica (tarea 8.3)

Criterio: una táctica **se sostiene** si su garantía no depende de la capacidad libre; **deja de alcanzar** si su métrica sale de la medida del ADR; **se rompe** si su mecanismo deja de funcionar.

| Táctica | Veredicto con ×5 | Valor |
|---------|------------------|-------|
| **TAC-01** Redundancia activa | **Se sostiene la garantía**, no la capacidad | 0 operaciones confirmadas perdidas (sigue en 2 de 3 servidores). El servidor que atiende se satura a 9.900/s: TAC-01 no suma escritura |
| **TAC-02** Latido | **Sostenida en cómputo, expuesta en la práctica** | Mensajes de pocos bytes. Riesgo razonado, sin medida: si el servidor saturado retrasa su señal más de 300 ms, los otros dos lo declaran caído sin estarlo; cada cambio cuesta ≤ 30 s (0,5 min) y 86 cambios agotan los 43,2 min del mes (43,2 ÷ 0,5) |
| **TAC-03** Reconfiguración | **Se rompe** | A 15.000/s un cambio deja 15.000 × 30 = 450.000 operaciones sin respuesta y la capacidad libre (9.900/s) es **menor** que la llegada: la cola no se drena mientras dure el evento. Dejaba de alcanzar desde ~7.600/s (§5) |
| **TAC-04** Transacciones | **Se sostiene** | 0 duplicados: la segunda inserción de la misma clave falla dentro de su transacción, a cualquier tasa. Paga: +15.000 inserciones/s dentro del mismo límite de la base y ~10,8 GB de registro (15.000 × 7.200 × 100 bytes) |
| **TAC-05** Validar entradas | **Se sostiene** | 100 % de misma clave con datos distintos rechazadas; no depende de la tasa |
| **TAC-06** Usar un intermediario | **Se sostiene** | Llamada dentro del proceso, < 0,1 ms |
| **TAC-07** Verificar integridad | **Se sostiene** | 15.000 × 0,045 ms = 675 ms/s = 0,675 núcleos = 4,2 % de 16 núcleos; 0,045 ms de los 20 ms de S26 |
| **TAC-08** Informar a los actores | **Se sostiene** | El plazo de ≤ 5 min no depende de la tasa; el tope de 240 alertas/día (S27) tampoco |
| **TAC-09** Múltiples copias de datos | **Se sostiene, sin garantía de tiempo** | La lista no depende del saldo, pero comparte servidor con la puerta de saldo (uso 1,05): su evaluación de ≤ 10 ms [S42] no se puede asegurar mientras dure la saturación |
| **TAC-10** Limitar la tasa de eventos | **Se sostiene, pero compite** | +100/s sobre una base al 1,5. Se pausa durante el evento (§8) |

**La que se rompe primero es TAC-03**, y se rompe por una razón que no es de la táctica: depende de que sobre capacidad después de un cambio de servidor, y X1 se la quita. Es la lectura de I1 llevada al límite.

**Por qué el efecto no es "más lento" sino "se acabó".** Con 15.000 entrando y ~10.000 saliendo, la cola crece 5.000/s. El proveedor espera ~3 s [S14], es decir, una cola de 3 × 10.000 = 30.000 operaciones, que se alcanza a los 30.000 ÷ 5.000 = **6 s** (5,8 s con la reconsulta corriendo). Desde ahí toda operación nueva espera más de lo que el proveedor tolera.

---

## 7. Qué ve cada actor (tarea 8.4)

Sobre `04` §6.1, con los números de X1.

| Actor | Qué ve |
|-------|--------|
| **Jugador** | A partir del segundo 6 el giro o la apuesta se queda esperando y termina en un error del juego; el proveedor revierte el descuento [F24] y la jugada se anula. Si era un abono, el premio no aparece hasta que el proveedor lo reenvía con éxito. En el mejor caso (se atiende lo que cabe) falla 1 de cada 3 operaciones: (15.000 − 10.000) ÷ 15.000 = 33 %, **36 M de operaciones** en las 2 h (5.000 × 7.200) |
| **Proveedor de juego** | Operaciones sin respuesta a los ~3 s: revierte los descuentos y reenvía los abonos [F24][F26]. **Cada reenvío suma carga** a una base que ya no da abasto. TAC-04 evita que el reenvío abone dos veces, pero no que la base lo procese |
| **Regulador** | Más reversiones y repeticiones en el historial que se extrae para el sistema de inspección; la extracción lee la misma base saturada y se atrasa. Las tácticas mantienen 0 duplicados y 0 operaciones confirmadas perdidas: **lo que se registra es correcto, pero incompleto en el tiempo** |
| **Equipo de operaciones** | Si TAC-03 corre durante la saturación: la medida de ESC-02 (volver a p99 ≤ 250 ms en 5 min) no se cumple mientras dure el evento (2 h ≫ 5 min) |

---

## 8. Qué cambiaríamos (tarea 8.5)

### 8.1 Por qué no basta con más servidores

| Componente | ¿Se multiplica sin tocar un ADR? | Por qué |
|-----------|----------------------------------|---------|
| Servidor de aplicación | **Sí**, con el costo de I6 | N copias de la lista de ~500 MB [S42] y N cargas por actualización |
| Red pública | **Sí** (solo con 60.000) | Un enlace mayor que el de S35 |
| **Base única** | **No** | Todas las escrituras del saldo pasan por un solo servidor; los otros dos de TAC-01 no suman capacidad de escritura. Repartir las cuentas entre grupos de servidores cambia ADR-02 y ADR-03 (lecturas entre cuentas de grupos distintos dejan de ser una sola transacción): es la pista de `07` §6.3, y es del Bloque 4 |

Comprar servidores resuelve el 14.200/s del servidor de aplicación y deja el 10.000/s de la base donde está.

### 8.2 Propuesta: admitir hasta un tope y rechazar el resto a tiempo

Dos tácticas del catálogo (Rendimiento → Controlar la demanda):

- **Limitar la tasa de eventos:** la puerta de saldo (paso 4 de `02` §3.1) admite como máximo **7.500 operaciones/s** hacia la base.
- **Priorizar eventos:** cuando se llega al tope, **se rechazan descuentos nuevos y se siguen aceptando abonos y reversiones**. Cada descuento admitido crea la obligación de un abono; rechazar descuentos acota la carga futura y no deja ningún premio sin pagar.

```
tope                         7.500 operaciones/s = 3.750 jugadas/s admitidas  (demanda: 7.500 jugadas/s)
jugadas rechazadas           (7.500 − 3.750) ÷ 7.500 = 50 %  →  3.750 /s × 7.200 s = 27 M en las 2 h
uso de la base               7.500 ÷ 10.000 = 0,75 (0,76 con la reconsulta)
cambio de servidor (3 copias) 90 × 7.500 ÷ (9.900 − 7.500) = 675.000 ÷ 2.400 = 281 s ≤ 300 s  (margen 1,07)
```

**Por qué 7.500 y no 9.900.** Con el tope en 9.900/s no queda capacidad libre: un cambio de servidor no drena nunca. 7.500 es además el umbral que ADR-02 ya adoptó (condición (a)) y el valor más alto que cumple la medida de TAC-03 con 3 copias por operación (7.615). El margen es **estrecho (1,07)** y depende de cuántas copias envía cada proveedor, cifra que no está en las fuentes (observación 4 de `07` §8).

**Qué más cambia.**
1. **Segundo servidor de aplicación.** Rechazar también cuesta cómputo. Con el tope, el servidor lleva 7.500 admitidas + 7.500 rechazadas: (7.500 × 1,045 + 7.500 × c + 1.200) ÷ 16.000 es 0,80 si rechazar cuesta c = 0,5 ms y **1,03 si cuesta 1 ms** (el máximo que cabe es 0,93 ms). Con la lectura de 60.000 no cabe ni con 0,5 ms (1,10). Dos servidores bajan el uso a 0,52 (c = 1 ms). Costo: ~USD 1.002 × 12 = **~USD 12.000 al año** [F41] y 1.000 MB de lista en lugar de 500 (I6).
2. **Pausar la reconsulta durante el evento.** Pausa de 2 h y luego 350.000 ÷ 50 /s = 7.000 s = 1,94 h: 2 + 1,94 = **3,9 h ≤ 6 h** [S19]. Quita los +100/s a una base que ya está cargada.
3. **Mantener fuera de este tope las operaciones críticas de otras funciones** (retiros y depósitos): son 0,5 depósitos/s (`03` §5.2) y no compiten.

### 8.3 Costo y punto débil

| Qué se paga | Valor |
|-------------|-------|
| Jugadas rechazadas | 27 M en 2 h (50 %): hasta 3.750 por segundo |
| Servidor de aplicación adicional | ~USD 12.000 al año [F41] |
| Memoria de la lista | 1.000 MB (0,8 % de 128 GB) |
| Ingreso | Sin estimar: el proyecto no tiene modelo de ingresos por jugada |

**Punto débil, que el equipo debe poder decir.** Las fuentes dicen que un descuento **sin respuesta** se da por fallido y se revierte [F24]; **no documentan** un código de rechazo por carga. El argumento es que un rechazo explícito en milisegundos no puede ser peor que un silencio de ~3 s, pero hay que confirmarlo con cada proveedor (los 5 de S42 (a)). Si un proveedor lo tratara como error grave, el tope tendría que aplicarse a nivel de sesiones y no de operaciones.

**Un matiz con ESC-05.** El límite de ≤ 0,001 % de legítimas rechazadas [S27] mide los rechazos indebidos de la **verificación de origen**. Un rechazo por admisión es otra clase y no debe contarse en esa medida; se anota para que las dos no se mezclen en la sustentación.

### 8.4 ¿ADR nuevo o reemplazo?

**Nuevo: ADR-05.** No reemplaza a ADR-01 a ADR-04:

| ADR | Qué le pasa con X1 |
|-----|--------------------|
| ADR-02 | **No se reemplaza.** Se dispara su condición de revisión (a) (> ~7.500/s). ADR-05 es la respuesta a ese disparo; si el ×5 fuera permanente, aplicaría la pista del Bloque 4 |
| ADR-03 | Se sostiene (TAC-04) |
| ADR-04 | Se sostiene (TAC-07) |
| ADR-01 | Se sostiene; solo se pausa la reconsulta |

**Aviso sobre la regla del proyecto.** Un ADR nace de un escenario crítico: atributo dominante alto y un costo sobre **otro atributo alto**. ADR-05 sacrifica jugadas, es decir, Adecuación funcional (2, no es alta). Con el criterio estricto del Bloque 2 sería un escenario no crítico. Se responde "nuevo" porque la plantilla pregunta por una decisión de arquitectura, y se deja a la decisión del equipo si se formaliza (§11, decisión 4).

---

## 9. Texto para la plantilla (sección 8)

*Para transcribir en la fase 10. Redactado con la recomendación de cada decisión del §11.*

**8. Extensión**

**X1 · Crecimiento de la carga**
- Extensión recibida: Un evento, una campaña o una alianza multiplica por 5 la tasa del pico de su operación crítica durante 2 horas. El resto del perfil de carga se mantiene.
- Nueva carga del pico (con la cuenta): 5 × 3.000 operaciones de saldo/s = 15.000 operaciones de saldo/s durante 2 h (7.200 s), con ráfagas de 5 × 4.000 = 20.000/s. Son 15.000 × 7.200 = 108 M de operaciones, 4,5 veces un día normal, en 2 horas.
- Táctica que se sostiene y por qué: TAC-01 (Redundancia activa), TAC-04 (Transacciones), TAC-05 (Validar entradas), TAC-06 (Usar un intermediario) y TAC-07 (Verificar integridad del mensaje), porque su garantía no depende de la capacidad libre: siguen en 0 operaciones perdidas y 0 duplicados, y la verificación suma 15.000 × 0,045 ms = 0,675 núcleos (4,2 % de 16). TAC-04 sigue evitando abonos dobles aunque los reenvíos se multipliquen.
- Táctica que se rompe o deja de alcanzar, y en qué valor: TAC-03 (Reconfiguración, ADR-02) se rompe. Desde ~7.600 operaciones/s deja de volver a p99 ≤ 250 ms en 5 min si cada operación llega 3 veces durante un cambio de servidor (90 R ÷ (9.900 − R) ≤ 300), y a 15.000/s un cambio deja 450.000 operaciones sin respuesta con menos capacidad libre (9.900/s) que llegadas: la cola no se drena. Además la base única queda en uso 15.000 ÷ 10.000 = 1,5 y se pasa su límite de ~10.000/s; el servidor de aplicación llega a 1,05. TAC-01 no suma capacidad de escritura. A los 6 s la cola supera los ~3 s del proveedor y las jugadas se revierten; el jugador ve un error del juego y el proveedor reenvía, lo que agrava la saturación.
- Qué cambiaríamos: admitir como máximo 7.500 operaciones/s en la puerta de saldo (Limitar la tasa de eventos) y, al llegar al tope, rechazar descuentos nuevos y seguir aceptando abonos y reversiones (Priorizar eventos). Con el tope, la base queda en 0,75 y un cambio de servidor drena en 281 s ≤ 300 s. Costo: 50 % de las jugadas rechazadas, 27 M en las 2 h (3.750 por segundo × 7.200 s). Además, un segundo servidor de aplicación (~USD 12.000 al año) y pausar la reconsulta durante el evento (2 h + 1,94 h = 3,9 h ≤ 6 h). Más servidores no basta: la base única no se multiplica sin cambiar ADR-02 y ADR-03.
- ¿Implica un ADR nuevo o reemplazar uno existente? (ninguno | nuevo | reemplaza ADR-…): nuevo (ADR-05, admisión de operaciones de saldo bajo sobrecarga); ADR-02 no se reemplaza, pero se dispara su condición de revisión (a).

**Dónde está esto en la entrega:** presentación HTML, sección "Extensión" (se integra en la fase 10), y `bloque-3/08-extension.md`.

---

## 10. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| El texto de X1 está copiado tal cual | ✔ §2, igual a `GEN_EXT` del material |
| La nueva tasa sale de la cadena de la fase 3, sin números nuevos | ✔ §3: 5 × 3.000 |
| Cada componente tiene su uso con la nueva carga | ✔ §4 |
| Qué táctica se sostiene, cuál se rompe y en qué valor | ✔ §5 y §6: las diez tácticas |
| Qué ve el usuario (y el proveedor, el regulador, operaciones) | ✔ §7 |
| La respuesta no se reduce a "más servidores" | ✔ §8.1: la base única no se multiplica |
| Dice si implica un ADR nuevo o reemplazar uno | ✔ §8.4: nuevo, ADR-05 |
| Cuentas recalculadas | ✔ 5 × 3.000 = 15.000; 15.000 × 7.200 = 108 M; 108 ÷ 24 = 4,5; 15.000 ÷ 10.000 = 1,5; 15.000 ÷ 14.000 = 1,07; 20.000 ÷ 10.000 = 2,0; (15.000 × 1,045 + 1.200) ÷ 16.000 = 1,05; (20.000 × 1,045 + 1.200) ÷ 16.000 = 1,38; (15.000 × 1,045 + 6.000) ÷ 16.000 = 1,35; 432 ÷ 1.000 = 0,43; 1.200 ÷ 1.000 = 1,2; 108 M × 350 bytes = 37,8 GB; 2,5 ÷ 1,44 = 1,7; 2.970.000 ÷ 390 = 7.615; 2.970.000 ÷ 330 = 9.000; (16.000 − 1.200) ÷ 1,045 = 14.162; (16.000 − 6.000) ÷ 1,045 = 9.569; 30.000 ÷ 5.000 = 6 s; 5.000 × 7.200 = 36 M; 15.000 × 30 = 450.000; 15.000 × 0,045 = 675 ms/s = 0,675 núcleos = 4,2 %; 3.750 × 7.200 = 27 M; 675.000 ÷ 2.400 = 281; 300 ÷ 281 = 1,07; 2 + 350.000 ÷ 50 ÷ 3.600 = 3,94 h; (7.500 × 1,045 + 7.500 × 1 + 1.200) ÷ 16.000 = 1,03 y ÷ 32.000 = 0,52; 43,2 ÷ 0,5 = 86; 1.002 × 12 = 12.024 |
| No se agregaron fuentes ni supuestos | ✔ Solo F24, F26, F34, F36, F41, S6, S14, S19, S24, S26, S27, S32 a S36, S40 a S42 (ya registradas); ningún `F#` ni `S#` nuevo |
| Las cifras de las fases 3 a 7 no cambiaron | ✔ Se reutilizan sin modificar |
| Solo nombres de táctica del catálogo y ningún mecanismo con marcas | ✔ Limitar la tasa de eventos y Priorizar eventos están en el catálogo (Rendimiento → Controlar la demanda) |
| Ninguna fase cerrada se modificó | ✔ Solo se agregó este archivo |

**Observaciones para la fase 9 (candidatas a `H#`; no se corrigen aquí):**
1. **La lectura literal de X1 exige 1,25 operaciones/s por jugador**, 3 veces el juego más rápido (0,4). Una tasa ×5 solo es consistente con S6 si crecen los jugadores (60.000). Por eso §4 muestra las dos lecturas.
2. **El rechazo explícito de un descuento por carga no tiene fuente.** [F24] documenta solo el descuento sin respuesta. Es la afirmación más débil de la fase (§8.3).
3. **El margen de 1,07 del tope de 7.500** (281 s de 300 s) hereda la cifra frágil de las copias por operación (`07` §8, observación 4).
4. **El costo de rechazar una petición (c)** no tiene fuente: S33 mide 1 ms por petición atendida. Con c = 1 ms el servidor de aplicación queda en 1,03; el máximo que cabe es 0,93 ms.
5. **Latido bajo saturación (TAC-02)** es un riesgo razonado, no medido: no hay cifra de cuánto retrasa la señal un servidor al 1,5.
6. **Ingreso perdido por jugada rechazada:** el proyecto no tiene modelo de ingresos; el costo se expresa solo en jugadas.
7. **ADR-05 y el criterio de escenario crítico** (§8.4): sacrifica Adecuación funcional (2). Si se formaliza, hay que propagar ESC-06 por la cadena del Bloque 2.

---

## 11. Punto de control (cerrado el 15 de noviembre de 2026)

**Decisiones del equipo:** se aprueban las cuatro recomendaciones. Las variantes quedan registradas.

| # | Decisión | Aprobado | Variante |
|---|----------|----------|----------|
| 1 | Lectura de "el resto del perfil se mantiene" | **Literal**: solo crece la tasa del saldo, 12.000 conectados, y se muestra al lado la lectura de 60.000 (§4). Es la que dice el texto y no agrega números | **60.000 conectados**: es la consistente con S6 (1,25 frente a 0,4 operaciones/s por jugador); suma la saturación de la red (1,2) y del servidor de aplicación (1,35). La base se satura igual en ambas |
| 2 | Tope de admisión | **7.500/s**: cumple TAC-03 con 3 copias (281 s ≤ 300 s, margen 1,07), coincide con la condición (a) de ADR-02 y rechaza 50 % de las jugadas | **9.900/s**: rechaza 34 % (18,4 M jugadas), pero la base queda en 0,99 y un cambio de servidor no drena nunca |
| 3 | Qué táctica se declara "la que se rompe" en la plantilla | **TAC-03**, desde ~7.600/s, y se menciona que la base se satura a ~9.900/s (§6) | **Solo la base y TAC-01 a 9.900/s**: más simple e incondicional, pero no muestra la interacción I1 llevada al límite |
| 4 | Alcance de ADR-05 | **Declararlo y esbozarlo aquí**, sin tocar el Bloque 2: la plantilla solo pide decir si es nuevo | **Formalizarlo ahora** (ESC-06 y ADR-05 con su Context): propaga la cadena del Bloque 2 y choca con el criterio de escenario crítico (§8.4) |

**Para llevar a la sustentación:** "Con la extensión, ¿qué táctica se rompe primero y qué ve el usuario?" Respuesta corta: TAC-03, desde ~7.600/s; a 15.000/s la cola alcanza los 3 s del proveedor en 6 s y las jugadas se revierten. Y la pregunta que sigue: "¿por qué no más servidores?", porque la base única recibe todas las escrituras (§8.1).

La fase 9 (revisión integral) puede arrancar.
