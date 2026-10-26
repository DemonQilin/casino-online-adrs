# Fase 2 — Ficha de contexto

**Proyecto:** casino online de juegos de azar regulado en Colombia
**Nombre de trabajo:** *Casino Andino* (operador ficticio)
**Insumo:** `01-investigacion-dominio.md` (las referencias [F#] remiten a sus fuentes)
**Horizonte de diseño:** año 3 de operación

**Convenciones:**
- **[F#]** es un dato verificado en la investigación.
- **[S#]** es un supuesto declarado. La sección 6 da la razón de cada uno.
- Esta ficha describe **necesidades y restricciones**. No describe cómo se construye el sistema.

---

## 1. Ficha resumen

> Este es el formato de las fichas de los 10 escenarios del widget del curso. Es lo que alguien externo lee para puntuar los atributos.

**Casino Andino** es un nuevo concesionario mediano de Coljuegos que ofrece **solo juegos de casino**, sin apuestas deportivas: tragamonedas, ruleta con generador aleatorio, blackjack en vivo con crupier real y un juego *crash* de multiplicador (estilo "el avión"). Todos los juegos los suministran **5 proveedores externos certificados**. El operador gestiona las cuentas, el dinero de los jugadores, los pagos y el cumplimiento regulatorio.

- **Carga:** 120.000 jugadores activos al mes y 350.000 cuentas registradas en el año 3. Pico normal de **5.000 jugadores simultáneos** y pico de diseño de **12.000** (viernes o sábado de quincena con promoción).
- **Dinero en movimiento:** en el pico de diseño, cerca de **3.000 operaciones de saldo por segundo**, con ráfagas de hasta 4.000/s. Cada jugada descuenta y abona el saldo a pedido del proveedor del juego. Además, unos **12.000 depósitos y 3.000 retiros al día**.
- **Tiempo real:** en el *crash*, el premio depende del instante exacto en que el jugador se retira, y hasta 720 jugadores pueden retirarse en el mismo segundo. En el blackjack en vivo, la ventana de apuesta dura entre 10 y 20 s y la cierra el servidor.
- **Disponibilidad:** juego y saldo al **99,9 % mensual** (≤ 43 min de caída al mes), con recuperación en ≤ 15 min. **Ninguna transacción de saldo confirmada puede perderse.** Hay un sistema alterno certificado obligatorio.
- **Regulación:** concesión de Coljuegos, con certificación integral por laboratorio, **recertificación anual** y **validación previa de cada cambio crítico**. Sistema de inspección en Colombia que reporta cada transacción al regulador. Antilavado con **SARLAFT** (basado en riesgo, cumpliendo los mínimos del SIPLAFT), Ley 1581 (datos personales) y juego responsable.
- **Seguridad:** bloqueo a los 5 intentos fallidos, cierre de sesión a los 20 minutos de inactividad y detección de suplantación por cambios bruscos de comportamiento. Además:
  - un **perfil de riesgo por jugador** que se actualiza en cada depósito y retiro;
  - consulta de **listas restrictivas** en el registro;
  - **reconsulta de las 350.000 cuentas en ≤ 6 h** cada vez que cambian las listas.
- **Diligencia reforzada:** ~2 % de los retiros (~60 al día), más todas las personas expuestas políticamente, se retienen para revisión y se resuelven en ≤ 48 h. Los demás retiros mantienen la meta de 24 h.
- **Integraciones:** 5 proveedores de juego, 5 medios de pago (PSE, Nequi, Daviplata, tarjetas y efectivo), verificación de identidad, consulta de listas restrictivas, Coljuegos y UIAF.
- **Cambio:** unos 20 juegos nuevos al mes y 3 o 4 cambios regulatorios al año. El equipo quiere entregar mejoras cada 2 semanas, pero cada cambio crítico pasa por el laboratorio, que tarda de 2 a 4 semanas.
- **Presupuesto y equipo:** 14 personas de ingeniería. Tecnología con un tope de ~15 % del ingreso bruto del juego (≈ COP 7.300 millones al año). Las cargas al Estado suman ~31 % de ese ingreso.
- **Usuarios:** 80 % juega desde el celular, con dispositivos de gama media y conexión 4G variable. Juegan por montos pequeños (~COP 25.000 por depósito).

---

## 2. Descripción y alcance

### 2.1 Qué es

Casino Andino es una plataforma web y móvil donde adultos residentes en Colombia se registran, depositan dinero, juegan casino online y retiran sus ganancias, bajo concesión de Coljuegos. El operador es dueño de la relación con el jugador y del dinero en su cuenta de juego. Los juegos son de terceros.

### 2.2 Incluye

| Juego | Proveedor | Cómo se determina el resultado |
|-------|-----------|-------------------------------|
| Tragamonedas (~450 títulos) | 3 proveedores con generador aleatorio | Generador aleatorio del proveedor, en servidor |
| Ruleta con generador aleatorio (~10 variantes) | Los mismos 3 proveedores | Generador aleatorio del proveedor, en servidor |
| Blackjack en vivo (10 mesas, formato de asientos ilimitados) | 1 proveedor de casino en vivo | Crupier real en un estudio del proveedor |
| Juego *crash* de multiplicador (1 título, ronda compartida) | 1 proveedor especializado | Generador aleatorio; el jugador decide cuándo retirarse |

Además incluye: registro y verificación, cuenta y saldo, depósitos y retiros, límites y autoexclusión, historial, soporte, reporte regulatorio y administración del riesgo de lavado de activos y financiación del terrorismo (**SARLAFT**).

### 2.3 Excluye

- Apuestas deportivas y apuestas en eventos.
- Juegos propios: el operador no desarrolla juegos.
- Póker entre jugadores, bingo y venta de otros juegos novedosos (Baloto, SUPER astro).
- Operación en salas físicas.

---

## 3. Actores e integraciones externas

| Sistema externo | Qué se intercambia | Frecuencia en el pico de diseño | Restricción conocida |
|----------------|-------------------|---------------------------------|---------------------|
| 3 proveedores de tragamonedas y ruleta | Lanzamiento de juego; peticiones de descuento, abono y reversión del saldo | ~2.760 operaciones de saldo/s en conjunto | Cada operación trae un identificador (en algunos protocolos, único solo por jugador [F26]) y puede llegar repetida. Si el operador no responde a tiempo, el proveedor revierte la operación [F24][F26]. Tiempo de espera del proveedor: ~3 s **[S14]** |
| 1 proveedor de casino en vivo | Video, apuestas por ronda, resultados y liquidación | 80 apuestas/s en ráfagas de ventana | La ventana la cierra el servidor del proveedor [F32] |
| 1 proveedor de *crash* | Apuestas al inicio de ronda, retiros durante la ronda y liquidación | 480 apuestas/s y hasta 720 retiros en un mismo segundo | El premio se calcula con el multiplicador vigente en el instante del retiro [F13] |
| 5 medios de pago (PSE, Nequi, Daviplata, tarjetas, efectivo) | Depósitos, retiros y códigos de autorización | ~1.800 depósitos en la hora pico | PSE concentra ~60 % de las transacciones [F22]. El medio debe estar a nombre del jugador [F27] |
| Servicio de verificación de identidad | Validación de documento y edad | ~1.000 registros al día **[S3]** | Documentos aceptados: CC, CE, PPT y PEP [F1] |
| Coljuegos | Datos transaccionales para inspección, acceso remoto y certificaciones | Continuo, según la periodicidad del modelo de datos | Formato fijado por Coljuegos (XML y XSD); sistema en Colombia en un centro Tier III [F1] |
| Proveedor de consulta de listas restrictivas | Listas de sanciones y de personas expuestas políticamente; consulta individual y reconsulta masiva | ~1.000 consultas al día en el registro; reconsulta de 350.000 cuentas en ≤ 6 h cuando cambian las listas | Las listas cambian varias veces al mes **[S19]** |
| UIAF | Reportes de operaciones sospechosas y de ganadores | Periódico y por evento | Según el SARLAFT del operador, que cumple los mínimos del SIPLAFT [F9] |
| Laboratorio certificador | Código, documentación, pruebas de seguridad y de carga | Anual y en cada cambio crítico | De 2 a 4 semanas por validación **[S10]** |

---

## 4. Requisitos no funcionales cuantificados

### 4.1 Carga y usuarios

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Jugadores activos al mes | 40.000 (año 1) → **120.000 (año 3)** | [S2] |
| Cuentas registradas | **350.000** (año 3) | [S3] |
| Jugadores activos al día | ~36.000 (año 3) | [S4] |
| Concurrencia en el pico normal | **5.000** jugadores simultáneos (viernes y sábado, 20:00–24:00) | [S4] |
| Concurrencia en el pico de diseño | **12.000** jugadores simultáneos (quincena + promoción) | [S4] |
| Mezcla de juego en el pico | 55 % tragamonedas · 15 % ruleta · 20 % *crash* · 10 % blackjack en vivo | [S5] |
| Operaciones de saldo en el pico de diseño | **~3.000/s** sostenidas, ráfagas de **~4.000/s** | [S6] |
| Depósitos | **~12.000 al día**; ~1.800 en la hora pico; ticket promedio de COP 25.000 | [S7] |
| Retiros | **~3.000 al día** | [S8] |
| Crecimiento | ×3 entre el año 1 y el año 3 | [S2] |

### 4.2 Tiempos de respuesta

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Operación de saldo pedida por un proveedor, medida en el operador | **p99 ≤ 250 ms** (muy por debajo del tiempo de espera del proveedor, ~3 s) | [S14] |
| Retiro en el *crash*: diferencia entre el multiplicador que ve el jugador y el que se liquida | Ninguna liquidación por encima del multiplicador mostrado; desfase máximo de **0,05x** | [S15] |
| Confirmación de un depósito al jugador | **p95 ≤ 10 s** para medios inmediatos | [F27][S7] |
| Carga de la pantalla del juego en 4G | **p95 ≤ 3 s** | [S12] |
| Orden de transferencia de un retiro normal | **≤ 24 h** como meta del operador; el máximo regulatorio es 72 h | [F1][S8] |
| Retiro con diligencia reforzada | Resolución en **≤ 48 h**, siempre dentro de las 72 h regulatorias. Su revisión **no retrasa** los retiros normales | [F1][S21] |
| Consulta de listas restrictivas en el registro | **p95 ≤ 5 s**. Ninguna cuenta se activa sin esta consulta | [S18] |
| Evaluación de riesgo de un depósito o retiro | **p95 ≤ 1 s**, dentro del presupuesto de confirmación del depósito (≤ 10 s) | [S20] |
| Aviso de pérdida de comunicación con el juego | Inmediato al detectarse | [F1] |
| Saldo actualizado visible al jugador después de un abono | **p95 ≤ 2 s** | [S23] |
| Alerta técnica por una petición de proveedor que repite proveedor, jugador e identificador con datos distintos | **≤ 5 min** | [S25] |

### 4.3 Disponibilidad y recuperación

| Servicio | Disponibilidad mensual | Recuperación máxima | Pérdida de datos tolerada | Origen |
|----------|------------------------|---------------------|---------------------------|--------|
| Juego y saldo | **99,9 %** (≤ 43 min/mes) | **≤ 15 min** | **Ninguna** transacción confirmada | [S13][F1] |
| Registro y depósitos | 99,5 % (≤ 3,6 h/mes) | ≤ 1 h | Ninguna transacción confirmada | [S13] |
| Consulta de saldo y solicitud de retiro | 99,5 % | ≤ 1 h | Ninguna | [S13][F1] |
| Sistema de inspección | Captura completa de toda transacción; ningún hueco en lo reportado | ≤ 4 h | Ninguna | [F1][S13] |
| Aviso a Coljuegos ante una falla grave | — | Aviso en **≤ 30 min** | — | [F1][S22] |
| Vuelta al desempeño normal después de restablecer juego y saldo | — | Operaciones de saldo de nuevo en p99 ≤ 250 ms a los **≤ 5 min** del restablecimiento | — | [S24] |

Restricciones fijas de esta área:
- Centro de datos Tier III como mínimo (99,982 %) [F1].
- Sistema alterno certificado, con los cambios replicados de inmediato [F1].
- Copias en un sitio físicamente separado [F1].
- Ante una falla grave se suspenden partidas, apuestas y pagos, y se informa a Coljuegos de inmediato [F1].
- Al reconectarse, el jugador ve el resultado de su jugada incompleta [F1].

### 4.4 Seguridad y cumplimiento

| Requisito | Cifra o regla | Origen |
|-----------|---------------|--------|
| Bloqueo por intentos fallidos | **5 intentos** sucesivos | [F1] |
| Cierre de sesión por inactividad | **≤ 20 min** | [F1] |
| Cifrado de datos personales y transaccionales | Clave de **≥ 2048 bits** | [F1] |
| Detección de suplantación | Por cambios bruscos en montos de depósito o retiro | [F1] |
| Sistema antilavado | **SARLAFT**, con segmentación por jugador, tipo de juego, medio de pago y municipio. Cumple los mínimos del SIPLAFT | [F9][F30][F33] |
| Perfil de riesgo del jugador | Se calcula en el registro y se **actualiza en cada depósito y retiro** | [S20] |
| Señales de alerta | Alerta al oficial de cumplimiento en **≤ 15 min** | [S16] |
| Listas restrictivas | Consulta en el registro (p95 ≤ 5 s). **Reconsulta de toda la base (350.000 cuentas) en ≤ 6 h** desde que el operador recibe una actualización. **Ningún retiro se paga** a una persona incluida en una lista ya recibida | [S18][S19] |
| Diligencia reforzada | Obligatoria para personas expuestas políticamente y jugadores de riesgo alto (~2 % de los retiros, ~60 al día) | [S21] |
| Falla de la evaluación de riesgo o del proveedor de listas | **Fallo cerrado:** los retiros se detienen hasta que la evaluación vuelva a estar disponible, siempre dentro de las 72 h regulatorias. El juego y los depósitos no se detienen por esta causa | Decisión del equipo (fase 3) [F1] |
| Saldo | **Nunca negativo**; sin transferencias entre jugadores | [F1] |
| Gestión de seguridad | Guiada por ISO 27001, 27002 y 27005; pruebas de penetración por el laboratorio | [F1] |
| Datos personales | Ley 1581: consentimiento previo y expreso, evaluación de impacto permanente, privacidad por diseño | [F1] |
| Datos de tarjeta | PCI DSS si se almacenan | [F1] |
| Relojes | Sincronizados con la hora legal colombiana; marcas de tiempo protegidas contra alteración | [F1] |

### 4.5 Datos y retención

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Registros de operaciones del jugador | ≥ 6 meses disponibles; el resto en almacenamiento externo durante la concesión | [F1] |
| Sistema de inspección | **2 años en línea**; el resto en respaldo durante toda la concesión (**5 años**) | [F1][S17] |
| Reconstrucción de jugadas | Toda jugada se puede reconstruir; el jugador ve sus **últimas 10 partidas** | [F1] |
| Historial descargable por el jugador | Últimos **30 días** | [F1] |
| Identificadores de transacción con proveedores | Conservados **≥ 4 meses** para conciliación | [F24] |
| Volumen de operaciones de saldo | ~24 millones al día (~8.900 millones al año) **[Derivado de S4 y S6]** | [S4][S6] |

### 4.6 Cambio y operación

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Juegos nuevos | **~20 al mes** sobre proveedores ya integrados | [S11] |
| Proveedores nuevos | 1 o 2 al año | [S11] |
| Cambios regulatorios | **3 o 4 al año**; en 2026 hubo al menos tres (premio inmediato, impuesto, revisión técnica) | [F13][F14][F15] |
| Ritmo deseado de entrega | Cada **2 semanas** | [S10] |
| Validación de un cambio crítico por el laboratorio | **De 2 a 4 semanas** | [S10] |
| Recertificación integral | Cada 12 meses; se presenta en ≤ 20 días hábiles | [F1] |
| Cambio de emergencia de seguridad | Inmediato, documentado en ≤ 24 h | [F1] |

### 4.7 Presupuesto y equipo

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Ingreso bruto del juego (GGR), año 3 | ~**COP 48.600 millones** al año | [S9] |
| Cargas al Estado sobre el GGR | 15 % de derechos de explotación + 16 % de impuesto al consumo ≈ **31 %** | [F3][F15] |
| Presupuesto de tecnología (equipo, infraestructura, licencias y certificación) | ~**15 % del GGR** ≈ COP 7.300 millones al año | [S9] |
| Equipo de ingeniería | **14 personas** | [S9] |

### 4.8 Canales, dispositivos y usuarios

| Requisito | Cifra | Origen |
|-----------|-------|--------|
| Canales | Web (escritorio y móvil) y app Android | [S12] |
| Tráfico móvil | **~80 %** | [S12] |
| Dispositivos y red | Gama media; 4G con calidad variable | [S12] |
| Perfil de gasto | ~30 % gasta entre COP 20.000 y 50.000 al mes | [F22] |
| Información obligatoria en pantalla | Tiempo de sesión, aviso de juego responsable, saldo, sello de Coljuegos | [F1] |
| Diferencias entre dispositivos | Se documentan y el jugador las acepta | [F1] |

---

## 5. Restricciones fijas

Estas condiciones no son negociables y valen para cualquier diseño.

1. Concesión de Coljuegos; dominio ".co"; tráfico de Colombia identificado [F1].
2. Toda la lógica que determina resultados se ejecuta del lado del proveedor o del operador, nunca en el dispositivo del jugador [F1].
3. Sistema de inspección en Colombia, en un centro Tier III, con el formato de datos de Coljuegos [F1].
4. Validación previa por el laboratorio de todo cambio en elementos críticos [F1].
5. Sistema alterno con la misma certificación [F1].
6. Retorno al jugador ≥ 83 % en todos los juegos [F3].
7. Juegos solo de proveedores externos certificados (decisión de la fase 1).
8. Antilavado con **SARLAFT**, cumpliendo como mínimo los criterios del SIPLAFT (decisión de la fase 1) [F9].
9. **Fallo cerrado en los retiros:** ante la indisponibilidad de los controles antilavado, ningún retiro se paga sin evaluar. Legalmente es más grave pagar dinero de lavado que demorar un retiro (decisión de la fase 3).

---

## 6. Supuestos declarados

| # | Supuesto | Razón |
|---|----------|-------|
| S1 | El operador apunta a **~3 %** del ingreso del segmento casino online en el año 3 | El casino representa ~34 % de un mercado online de ~USD 1.290 M [F21], es decir, ~USD 440 M. El top 5 concentra ~88 % [F20], así que un nuevo mediano compite por la porción restante. |
| S2 | 40.000 jugadores activos al mes en el año 1 y **120.000** en el año 3 | ~6 % de los ~2 M de jugadores reales proyectados [F21]. Como un jugador suele tener cuenta en varios operadores, la cuota en jugadores es mayor que la cuota en ingresos. |
| S3 | 350.000 cuentas registradas en el año 3 (~1.000 registros al día) | Relación de ~3 cuentas registradas por cada jugador activo al mes, por el abandono típico de las cuentas. |
| S4 | 30 % de los activos mensuales juega cada día (36.000), en sesiones promedio de 45 min. En el pico normal, 14 % de ellos está conectado a la vez (**5.000**). El pico de diseño es **×2,4 (12.000)** | No hay datos públicos de concurrencia (ver sección 6 de la investigación). Contraste de coherencia: 36.000 × 45 min ≈ 1,6 M de minutos-jugador al día; la franja de 20:00 a 24:00, con un promedio de ~3.500 conectados, concentra ~52 % de ese total. El factor ×2,4 cubre la coincidencia de quincena, fin de semana y promoción. Como referencia, el Mundial elevó las apuestas deportivas un 84 % [F17]. |
| S5 | Mezcla en el pico: 55 % tragamonedas, 15 % ruleta, 20 % *crash*, 10 % blackjack en vivo | Las tragamonedas dominan el catálogo y la ruleta es de las favoritas [F22]. El *crash* es muy popular en Colombia. |
| S6 | Ritmo por jugador: un giro de tragamonedas cada 5 s, una jugada de ruleta cada 30 s, una ronda de *crash* cada 25 s, una ronda de blackjack cada 60 s. Cada jugada = 1 descuento + 1 abono | Con 12.000 jugadores: tragamonedas 6.600 × 0,2/s × 2 = 2.640/s; ruleta 1.800 / 30 × 2 = 120/s; *crash* 2.400 apuestas en 5 s = 480/s; blackjack 1.200 apuestas en ventanas de 15 s = 80/s. **Total ≈ 3.300/s**, redondeado a ~3.000 sostenidas y ráfagas de ~4.000 cuando coinciden los retiros del *crash*. Promedio ponderado: ~0,25 operaciones de saldo por segundo por jugador conectado; con 36.000 × 45 min al día salen ~24 M de operaciones diarias. |
| S7 | 3 depósitos al mes por jugador activo, de COP 25.000 en promedio. El 15 % de los depósitos del día cae en la hora pico | Coherente con el gasto típico de COP 20.000–50.000 [F22]. 120.000 × 3 = 360.000 al mes ≈ 12.000 al día. |
| S8 | 1 retiro por cada 4 depósitos; meta de 24 h para la orden de transferencia | Los operadores del mercado retiran en 24–72 h [F27]. Pagar en 24 h compite contra los sitios ilegales [F31]. |
| S9 | GGR = 45 % de los depósitos → 120.000 × COP 75.000 × 0,45 × 12 ≈ **COP 48.600 M al año**. Tecnología al 15 % del GGR. 14 ingenieros | Contraste: a COP 4.000 por dólar son ~USD 12 M, cerca del 3 % del segmento (S1). |
| S10 | El laboratorio tarda **de 2 a 4 semanas** por validación. El equipo quiere entregar cada 2 semanas | No hay un plazo público. Es una estimación conservadora que hace visible la tensión con la mantenibilidad. |
| S11 | ~20 juegos nuevos al mes y 1 o 2 proveedores nuevos al año | Los catálogos del mercado se renuevan constantemente. |
| S12 | 80 % del tráfico es móvil, con dispositivos de gama media y 4G | Patrón típico del juego online en Colombia **[Estimación]**. |
| S13 | Disponibilidad de 99,9 % para juego y saldo, y de 99,5 % para registro y depósitos. Recuperación ≤ 15 min para juego y saldo | La regulación obliga al operador a fijar estos valores y Coljuegos puede exigir tiempos menores [F1]. Cada minuto caído empuja jugadores a sitios ilegales [F31]. |
| S14 | Los proveedores dan por fallida una operación sin respuesta en ~3 s. La meta interna es p99 ≤ 250 ms | No hay un valor público por proveedor **[Estimación]**. La meta deja un margen amplio frente al tiempo de espera. |
| S15 | Tolerancia de 0,05x entre el multiplicador mostrado y el liquidado en el *crash* | Hace verificable la equidad. El valor debe validarse con las reglas del proveedor. |
| S16 | Alerta antilavado al oficial de cumplimiento en ≤ 15 min | Permite actuar antes de que se pague un retiro **[Estimación]**. |
| S17 | Concesión de 5 años | Plazo usual de los contratos de concesión **[por verificar]**. |
| S18 | Consulta de listas restrictivas en el registro: p95 ≤ 5 s | La consulta es parte de la activación. Debe ser breve para no frenar el registro, que compite con sitios ilegales sin controles [F31] **[Estimación]**. |
| S19 | Las listas cambian varias veces al mes. La reconsulta completa de la base termina en **≤ 6 h** | Con ~3.000 retiros al día (S8), una ventana de 6 h deja expuestos a lo sumo ~750 retiros a una lista desactualizada. Además, regla absoluta: ningún retiro se paga a alguien que ya esté en una lista recibida. Ritmo mínimo: 350.000 / 6 h ≈ 16 consultas por segundo, sin afectar el juego **[Estimación]**. |
| S20 | Evaluación de riesgo de un depósito o retiro en p95 ≤ 1 s, actualizando el perfil del jugador | Debe caber holgadamente dentro de la confirmación del depósito (p95 ≤ 10 s) **[Estimación]**. |
| S21 | ~2 % de los retiros (~60 al día) requieren diligencia reforzada; se resuelven en ≤ 48 h | Deja 24 h de margen frente al máximo regulatorio de 72 h [F1]. El porcentaje es una estimación para dimensionar la revisión humana **[Estimación]**. |
| S22 | Ante una falla grave, el aviso a Coljuegos se da en **≤ 30 min** | El reglamento dice "de inmediato" (E3, E5), y eso no es verificable. Esta cifra lo vuelve medible. Aprobado en la fase 6. |
| S23 | Después de un abono, el jugador ve su saldo actualizado en **p95 ≤ 2 s** | El jugador decide su siguiente apuesta con el saldo que ve en pantalla; 2 s cabe dentro del ritmo de juego más rápido (un giro cada 5 s, S6) **[Estimación]**. Agregado en la fase 8 para respaldar ESC-04. |
| S24 | Tras restablecer el juego y el saldo, las operaciones vuelven a p99 ≤ 250 ms en **≤ 5 min** | Al volver coinciden las reconexiones y las jugadas incompletas por resolver; 5 min acota esa degradación dentro del pico **[Estimación]**. Agregado en la fase 8 para respaldar ESC-02. |
| S25 | Una petición de proveedor que repite proveedor, jugador e identificador con datos distintos genera una alerta técnica en **≤ 5 min** | Es señal de un error de integración o de manipulación, y debe atenderse antes de que se acumulen diferencias en la conciliación diaria **[Estimación]**. Agregado en la fase 8 para respaldar ESC-03. |

---

## 7. Verificación de la fase

| Criterio | Estado |
|----------|--------|
| Alguien externo puede puntuar los atributos sin inventar nada | ✔ La sección 1 tiene cifras para carga, tiempo, disponibilidad, regulación, seguridad, integraciones, cambio, presupuesto y usuarios |
| Puerta de revisión 1: cifras y no adjetivos | ✔ |
| No se nombran soluciones ni tecnologías | ✔ Solo hay necesidades, restricciones y actores externos. El cómo conectarse con los proveedores queda abierto |
| Cada número tiene origen ([F#] o [S#]) | ✔ |
| Rúbrica, criterio *Ficha de contexto*, nivel "Logrado" | Requisitos no funcionales cuantificados que anclarán la priorización y los escenarios |

---

## 8. Punto de control: qué debe validar el equipo

1. **Dimensionamiento (S2, S4, S6).** ¿Les parecen creíbles 120.000 jugadores activos al mes y 12.000 simultáneos en el pico? Si los cambian, se recalculan en cadena las operaciones de saldo, los depósitos y el GGR.
2. **Metas de disponibilidad (S13).** ¿99,9 % para juego y saldo, o más exigente? Esta cifra afecta directamente la tensión entre fiabilidad y costo.
3. **Tolerancia del *crash* (S15).** Es la cifra más "inventada" y la que más pesa en un posible escenario de desempeño.
4. **Plazo del laboratorio (S10).** Si alguien del equipo conoce el plazo real, conviene reemplazar la estimación.
5. **Métricas del SARLAFT (S18–S21).** Reconsulta de la base en ≤ 6 h, ~2 % de retiros con diligencia reforzada y resolución en ≤ 48 h.

> **Estado:** el equipo validó todas las métricas, incluidas las del SARLAFT (punto 5), el 25 de octubre de 2026. **Fase cerrada.**
