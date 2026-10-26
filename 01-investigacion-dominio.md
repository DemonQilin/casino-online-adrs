# Fase 1 — Investigación del dominio

> [← Anterior](./00-plan-de-trabajo.md) · [Índice](./README.md) · [Siguiente →](./02-ficha-contexto.md)

**Proyecto:** casino online de juegos de azar regulado en Colombia (sin apuestas deportivas)
**Fecha de corte de la investigación:** 25 de octubre de 2026
**Propósito:** reunir hechos verificables para que la ficha de contexto (fase 2) no invente nada.

**Convenciones de este documento:**
- **[F#]** remite a la lista de fuentes al final.
- **[Estimación]** marca un dato sin fuente directa. Debe convertirse en supuesto declarado en la fase 2.
- Este documento describe **obligaciones, actores y flujos**. No propone soluciones ni arquitectura.

---

## Resumen: lo que más pesa para la arquitectura

1. **El regulador está "dentro" del sistema.** El operador debe mantener un *sistema de inspección* en territorio colombiano, en un centro de datos de nivel Tier III como mínimo. Ese sistema captura las transacciones en archivos XML con el formato que define Coljuegos y las carga en una bodega de datos que Coljuegos consulta directamente [F1].
2. **Cada cambio relevante se recertifica.** Toda modificación que afecte un elemento crítico debe ser validada por un laboratorio autorizado **antes** de salir a producción. Además, hay una recertificación integral cada 12 meses [F1]. Esto choca de frente con la mantenibilidad.
3. **La continuidad es obligatoria y exigente.** El operador debe tener un sistema de juego alterno con la misma certificación que el principal y replicarle de inmediato cualquier cambio. Ante una falla grave, se suspenden partidas, apuestas y pagos [F1].
4. **El dinero del jugador es intocable.** Ningún saldo puede quedar negativo, no se puede jugar sin saldo, todo movimiento queda registrado para auditoría y las partidas deben poder reconstruirse [F1].
5. **Los juegos los ponen terceros.** Los proveedores de juego descuentan y abonan el saldo del jugador contra el sistema del operador en cada jugada, y deshacen la operación cuando algo falla [F24][F26]. La integración externa queda en el camino crítico de cada apuesta.
6. **La regulación cambia con frecuencia.** En 2026 Coljuegos está revisando los requerimientos técnicos [F14], amplió los juegos de premio inmediato [F13] y el Gobierno creó por decreto un impuesto del 16 % sobre los ingresos brutos del juego (GGR) [F15]. Esto exige que el sistema pueda adaptarse a cambios de reglas.

---

## 1. Marco regulatorio

### 1.1 Normas base

| Norma | Qué establece | Fuente |
|-------|---------------|--------|
| Ley 643 de 2001, art. 38 (modificado por la Ley 1753 de 2015, art. 93) | Los juegos por internet son "novedosos". Solo pueden operarlos personas jurídicas con contrato de concesión con Coljuegos. | [F1][F2] |
| Acuerdo 4 de 2016, hoy compilado en el **Acuerdo 8 de 2020** (Título 1) | Reglamento de juegos operados por internet: tipos de juego, retorno al jugador y obligaciones del operador. | [F2][F8] |
| Requerimientos Técnicos de Juegos Operados por Internet (Res. 20161200013324 de 2016; última modificación consultada: **Res. 20224000026034 de 2022**) | Requisitos técnicos detallados: plataforma, inspección, seguridad, continuidad y jugador. | [F1] |
| Res. 20161200013844 de 2016 | Requisitos para laboratorios certificadores autorizados por Coljuegos. | [F11] |
| Res. 20195100044514 de 2019 | Sistema de prevención de lavado de activos y financiación del terrorismo (SIPLAFT) para operadores concesionados. | [F9] |
| Res. 20244000022654 de 2024 | Lineamientos de juego responsable. Derogó la Res. 20214000036784 de 2021. | [F12] |
| Ley 1581 de 2012, Decreto 1377 de 2013, Ley 1266 de 2008, Ley 1480 de 2011 | Protección de datos personales, habeas data y estatuto del consumidor. Los requerimientos técnicos las citan expresamente. | [F1] |
| Acuerdo 5 de 2020 | Incorpora el **casino en vivo** como tipo de juego autorizado. | [F5] |
| Acuerdo 1 de 2026 | Amplía los juegos de premio inmediato a mecánicas de multiplicador (tipo *crash*). | [F13][F14] |
| Decreto 0240 de 2026 | Impuesto nacional al consumo del 16 % sobre el GGR bimestral de juegos operados por internet. | [F15][F16] |

> **Aviso de vigencia:** Coljuegos abrió en 2026 una revisión formal de los requerimientos técnicos, con implementación progresiva [F14]. Este documento usa la versión de 2022, que es la última publicada que pudimos consultar completa.

### 1.2 Obligaciones técnicas que condicionan la arquitectura

La última columna anticipa qué característica de ISO 25010 toca cada obligación. Sirve como insumo para la fase 3.

#### A. Infraestructura e inspección

| # | Obligación | Fuente | Atributo que toca |
|---|-----------|--------|-------------------|
| A1 | El sistema técnico de juego debe estar en un centro de datos de nivel **Tier III como mínimo**. El propio documento define Tier III como 99,982 % de disponibilidad, es decir, no más de 1,6 horas de falla al año y sin interrupciones por mantenimiento planificado. | [F1] | Fiabilidad |
| A2 | El **sistema de inspección** (capturador, almacenamiento de XML y bodega de datos) debe estar en un centro Tier III **en territorio colombiano**. Toda su operación es responsabilidad del operador. | [F1] | Compatibilidad, Portabilidad, Fiabilidad |
| A3 | El capturador traduce las transacciones a XML según el modelo de datos de Coljuegos (XSD), y la bodega usa un modelo dimensional definido por Coljuegos. | [F1] | Compatibilidad |
| A4 | El sistema de inspección conserva el histórico durante toda la concesión: **mínimo 2 años en línea** y el resto en respaldo, que se entrega formalmente a Coljuegos. | [F1] | Fiabilidad, Desempeño (capacidad) |
| A5 | Coljuegos debe poder acceder (por VPN o credenciales) a la trazabilidad de cambios de los datos. | [F1] | Seguridad, Compatibilidad |
| A6 | Debe existir certificación del origen de los datos: el emisor de los informes debe estar autorizado. | [F1] | Seguridad |
| A7 | Los relojes de los componentes críticos se sincronizan con una fuente de tiempo confiable en Colombia (referencia: Instituto Nacional de Metrología), con controles contra la alteración de marcas de tiempo. | [F1] | Seguridad (integridad) |

#### B. Certificación y gestión de cambios

| # | Obligación | Fuente | Atributo que toca |
|---|-----------|--------|-------------------|
| B1 | Certificación **integral** inicial por un laboratorio autorizado: cuentas, juegos, pagos, inspección, seguridad, carga, caídas y recuperación. | [F1] | Todos |
| B2 | Recertificación integral **cada 12 meses**, que se presenta dentro de los 20 días hábiles siguientes a cada año de operación. | [F1] | Mantenibilidad |
| B3 | Todo cambio que afecte **elementos críticos** debe validarse con el laboratorio **antes** de producción. Son elementos críticos el saldo, el registro, la verificación de identidad, la bodega de datos, la seguridad de las comunicaciones y la integridad del juego. | [F1] | Mantenibilidad × Seguridad |
| B4 | Agregar un juego certificado sobre la **misma** plataforma ya certificada solo requiere aportar el certificado del juego. Si el juego viene sobre **otra** plataforma o versión, hace falta recertificar la integración. | [F1] | Compatibilidad × Mantenibilidad |
| B5 | En emergencias de seguridad se permite aplicar cambios críticos de inmediato, con la documentación completa en un plazo máximo de 24 horas. | [F1] | Seguridad × Mantenibilidad |
| B6 | Si el operador tiene ISO 27001 vigente, el laboratorio solo ejecuta las pruebas técnicas de seguridad. | [F1] | Seguridad |
| B7 | Ambientes de desarrollo, pruebas y producción separados, con un procedimiento documentado de gestión de cambios. | [F1] | Mantenibilidad |

#### C. Jugador, cuenta y dinero

| # | Obligación | Fuente | Atributo que toca |
|---|-----------|--------|-------------------|
| C1 | Registro con datos obligatorios (documento CC, CE, PPT o PEP; nombres, nacimiento, residencia según DIVIPOLA, correo y teléfono). Validación y actualización de datos **como mínimo una vez al año**. | [F1] | Adecuación funcional |
| C2 | Impedir el juego a menores de edad, a interdictos judiciales, al personal del operador y a los autoexcluidos. | [F1] | Seguridad, Adecuación funcional |
| C3 | **Bloqueo de la cuenta tras 5 intentos fallidos** de autenticación. Se registran todos los intentos y se muestra la fecha del último acceso. Se recuerda cambiar la contraseña cada 12 meses. | [F1] | Seguridad × Usabilidad |
| C4 | Detectar suplantación analizando **cambios bruscos de comportamiento**, en particular en montos de depósito o retiro. | [F1] | Seguridad |
| C5 | Autoexclusión: bloquea apuestas y depósitos, pero **siempre permite retirar y consultar**. Debe impedir, en lo posible, que el autoexcluido cree otra cuenta. | [F1] | Adecuación funcional |
| C6 | **Ningún saldo negativo** y ninguna jugada sin saldo disponible. Se prohíben las transferencias entre cuentas de distintos jugadores. | [F1] | Fiabilidad (integridad), Seguridad |
| C7 | Los retiros exigen identificación plena del jugador. La orden de transferencia al banco se emite en **72 horas como máximo**. Hay conciliación mensual de pagos y cobros. | [F1] | Adecuación funcional |
| C8 | Si se guardan datos de tarjetas, aplica **PCI DSS**. | [F1] | Seguridad |
| C9 | Límites de depósito definidos por el jugador, con historial de cada modificación. | [F1] | Adecuación funcional |
| C10 | Historial descargable de los **últimos 30 días**. Las correcciones se muestran por separado, de forma que el jugador pueda conciliar con sus propios registros. | [F1] | Usabilidad |
| C11 | Registros de toda operación con efecto en el juego, la cuenta o los pagos: **mínimo 6 meses** disponibles y luego en almacenamiento externo. Los datos deben permitir **reconstruir cada jugada**. Los códigos de autorización de pago se conservan durante toda la concesión. | [F1] | Seguridad (responsabilidad), Mantenibilidad (analizabilidad) |
| C12 | Desconexión por inactividad a los **20 minutos como máximo**. | [F1] | Seguridad × Usabilidad |
| C13 | Tiempo de sesión **visible permanentemente**, aviso de juego responsable fijo y herramientas para limitar apuestas y duración. | [F1] | Usabilidad |

#### D. Juego

| # | Obligación | Fuente | Atributo que toca |
|---|-----------|--------|-------------------|
| D1 | Generador de números aleatorios (GNA) certificado, impredecible y sin sesgos. Debe **monitorearse para detectar fallas** y existir un criterio para deshabilitar el juego si falla. | [F1] | Fiabilidad, Seguridad |
| D2 | Toda la lógica que determina resultados se ejecuta en el **servidor**, independiente del dispositivo del jugador. | [F1] | Seguridad, Portabilidad |
| D3 | Retorno al jugador (RTP) **mínimo del 83 %**, con registro de los cambios de parámetros. Coljuegos revisa el RTP real como mínimo cada semestre. | [F3][F2] | Adecuación funcional |
| D4 | El monto del *jackpot* debe verse actualizado en **todos** los dispositivos que participan. | [F1] | Desempeño, Fiabilidad |
| D5 | Debe informarse al jugador si el tiempo de respuesta afecta su probabilidad de ganar, y avisarle de inmediato si se pierde la comunicación. | [F1] | Desempeño × Usabilidad |
| D6 | **Juego incompleto:** al reconectarse, el jugador ve el resultado y el estado de su apuesta. Se registran el inicio, la duración y la causa de cada interrupción. | [F1] | Fiabilidad |
| D7 | Repetición de jugada: debe poder reconstruir las **últimas 10 partidas** de la sesión. | [F1] | Adecuación funcional |
| D8 | Prohibidos los diseños de "casi gano" que inducen a seguir apostando. | [F1] | Adecuación funcional |

#### E. Continuidad, disponibilidad y seguridad

| # | Obligación | Fuente | Atributo que toca |
|---|-----------|--------|-------------------|
| E1 | **Sistema técnico de juego alterno** con la misma certificación que el principal. Todo cambio en el principal se replica **de inmediato** en el alterno. | [F1] | Fiabilidad × Mantenibilidad |
| E2 | El plan de continuidad cubre como mínimo: acceso a cuentas y saldos, retiros, continuación de juegos incompletos, sistema de inspección y restablecimiento completo. Cada escenario tiene tiempos de recuperación respaldados por un análisis de impacto del negocio (BIA). | [F1] | Fiabilidad |
| E3 | Ante una falla grave se **suspenden** partidas, apuestas y pagos, y se cancelan las apuestas cuando las reglas lo permiten. Se informa a Coljuegos de inmediato, y **Coljuegos puede exigir tiempos de recuperación menores**. | [F1] | Fiabilidad |
| E4 | Plan de disponibilidad por servicio (registro y cuentas, depósitos y retiros, juego) con **indisponibilidad máxima mensual y tiempo máximo de recuperación definidos por el operador**. | [F1] | Fiabilidad |
| E5 | Copias de información en un sitio **físicamente separado**. Toda pérdida de datos se reporta de inmediato a Coljuegos. | [F1] | Fiabilidad |
| E6 | Sistema de gestión de seguridad guiado por ISO 27001, 27002 y 27005. Acceso a través de un equipo de gestión unificada de amenazas (firewall e IDS/IPS). Sin usuarios técnicos genéricos, redes segmentadas y registros de auditoría protegidos contra alteración. | [F1] | Seguridad |
| E7 | Datos personales y transaccionales cifrados con **clave de al menos 2048 bits**. Autenticación en ambos sentidos: el sistema identifica al jugador y el jugador identifica al sistema. | [F1] | Seguridad × Desempeño |
| E8 | Pruebas de penetración de caja blanca y caja negra a cargo del laboratorio certificador. | [F1] | Seguridad |
| E9 | Todo acceso desde Colombia se redirige al dominio ".co" del operador, y se identifica el tráfico por IP cuando sea técnicamente viable. | [F1] | Compatibilidad, Seguridad |
| E10 | Las diferencias de funcionalidad en dispositivos móviles se documentan y se informan al jugador, que debe aceptarlas. | [F1] | Portabilidad × Usabilidad |

### 1.3 Prevención de lavado de activos (SIPLAFT)

- Los operadores concesionados deben implementar el SIPLAFT con los criterios mínimos de la Res. 20195100044514 de 2019 [F9].
- Son "clientes" sujetos de control quienes **compran créditos, retiran fondos o ganan premios** [F9]. Los reportes van a la UIAF y a Coljuegos [F9].
- La norma de 2016 (hoy derogada) definía dos estados de cuenta que siguen siendo útiles como referencia [F10]:
  - **suspendida por precaución**, ante fraude, colusión o uso por terceros;
  - **cancelada por tiempo**, tras 11 meses en estado suspendido.
- Coljuegos anunció la evolución de SIPLAFT hacia un modelo **SARLAFT basado en riesgos** [F30]. No encontramos todavía la norma que lo exija a los operadores **[por verificar]**.
- Los requerimientos técnicos exigen además "medidas contra el fraude y lavado de activos" como parte de la certificación [F1].
- La norma vigente permite que un operador que ya tenga un SARLAFT lo use, siempre que cumpla como mínimo los criterios del SIPLAFT [F9].
- Coljuegos y la Superintendencia Financiera discuten que los bancos reconozcan el SARLAFT de los operadores, porque hoy estos tienen barreras de acceso al sistema financiero [F33].

**Decisión del equipo:** el operador implementa **SARLAFT** (ver sección 5). Elementos que esto añade frente a un SIPLAFT básico:

| Elemento | En qué consiste | Atributo que toca |
|----------|-----------------|-------------------|
| Segmentación de factores de riesgo | Se clasifica el riesgo por **cliente** (jugador), **producto** (tipo de juego), **canal** (medio de pago) y **jurisdicción** (municipio de residencia) | Adecuación funcional |
| Perfil de riesgo por jugador | Se calcula en el registro y se **actualiza con cada depósito y retiro** | Seguridad × Desempeño |
| Consulta de listas restrictivas | En el registro y **reconsulta de toda la base** cada vez que las listas se actualizan | Seguridad × Desempeño (capacidad), Compatibilidad |
| Debida diligencia reforzada | Para jugadores de riesgo alto y **personas expuestas políticamente**: por ejemplo, se pide el origen de los fondos antes de un retiro | Seguridad × Usabilidad |
| Monitoreo continuo y señales de alerta | Revisión del comportamiento transaccional y alertas al oficial de cumplimiento | Seguridad |

> **Ojo con la sigla "PEP".** En SARLAFT significa *Persona Expuesta Políticamente*. En los requerimientos técnicos de Coljuegos significa *Permiso Especial de Permanencia* (un documento de identidad). En la entrega conviene escribirlo completo la primera vez.

### 1.4 Juego responsable

- La norma vigente es la Res. 20244000022654 de 2024 [F12].
- Elementos confirmados:
  - autoexclusión;
  - límites de depósito y de duración;
  - mensajes de juego responsable;
  - enlace al micrositio de Coljuegos;
  - informes periódicos del programa [F1][F12].
- No revisamos el texto completo de la resolución de 2024, así que puede haber obligaciones adicionales **[por verificar si se necesita detalle]**.

### 1.5 Protección de datos

- Se requiere autorización **previa, expresa e informada** del titular.
- El operador debe hacer **evaluaciones de impacto permanentes y documentadas** y aplicar **privacidad por diseño y por defecto** [F1].

### 1.6 Contexto fiscal

Afecta sobre todo al presupuesto, aunque también tiene impacto en el reporte.

- **Derechos de explotación:** mínimo el 15 % del GGR para juegos con RTP de al menos 83 % [F3].
- **Impuesto al consumo del 16 % sobre el GGR bimestral** (Decreto 0240 de 2026) [F15].
  - El gremio advierte una inconsistencia entre el hecho generador (el depósito) y la base gravable (el GGR) [F16].
  - Decretos anteriores con impuestos similares fueron suspendidos por la Corte Constitucional [F16].
- **Implicación:** el sistema debe calcular el GGR por periodos y adaptarse a reglas fiscales que cambian por decreto.

---

## 2. Modelo operativo

### 2.1 Actores

| Actor | Rol | Interacción con el sistema |
|-------|-----|---------------------------|
| Jugador | Persona mayor de edad, registrada y verificada | Registro, depósitos, jugadas, retiros, límites y autoexclusión |
| Operador (concesionario) | Persona jurídica con contrato de concesión | Dueño de la plataforma, la cuenta de juego y el cumplimiento regulatorio |
| Proveedor de juegos | Estudio que desarrolla los juegos con GNA o transmite el casino en vivo | Ejecuta la partida y descuenta o abona el saldo en cada jugada [F24][F25] |
| Agregador de juegos | Intermediario que unifica muchos proveedores en una sola integración | Lanzamiento de juegos, catálogo y operaciones de saldo estandarizadas [F25] |
| Medios de pago | PSE, billeteras digitales (Nequi, Daviplata), tarjetas y efectivo (Efecty y similares) | Depósitos y retiros. **PSE concentraba cerca del 60 % de las transacciones** [F22] |
| Coljuegos | Regulador y concedente | Consulta el sistema de inspección, recibe reportes y certificaciones, y puede exigir cambios |
| UIAF | Unidad de inteligencia financiera | Recibe los reportes de operaciones sospechosas y de ganadores [F9] |
| Laboratorio certificador | Uno de los laboratorios autorizados por Coljuegos (nueve a 2026) [F14] | Certifica la plataforma, los juegos, la seguridad y cada cambio crítico |
| Servicio de verificación de identidad | **[Estimación]** Proveedor externo para validar documento y edad | Validación en el registro y en la reactivación de cuentas |
| Proveedor de consulta de listas restrictivas | **[Estimación]** Servicio externo que mantiene listas de sanciones y de personas expuestas políticamente | Consulta en el registro y reconsulta masiva cuando las listas cambian (decisión SARLAFT) |

### 2.2 Tipos de juego posibles en el alcance

| Tipo | Quién determina el resultado | Exigencia temporal | Fuente |
|------|------------------------------|--------------------|--------|
| Tragamonedas | GNA del proveedor, en servidor | El jugador decide su propio ritmo; el resultado se muestra al instante | [F1][F4] |
| Juegos de mesa con GNA (ruleta, blackjack) | GNA en servidor | El jugador decide su propio ritmo | [F4] |
| Premio inmediato / *crash* | GNA con multiplicador progresivo; el jugador decide cuándo retirarse | **El instante del retiro define el premio**, así que la latencia importa | [F13] |
| Casino en vivo | Crupier real en un estudio | Ventana de apuesta cerrada por el servidor antes del resultado. El video tarda típicamente de 1 a 3 s, y los proveedores líderes apuntan a menos de 500 ms en juegos de cartas | [F5][F28][F29] |

**Dato de demanda:** según una encuesta de Playtech de 2025, ruleta, bingo y tragamonedas son los juegos online preferidos por los colombianos [F22].

### 2.3 Flujos principales

Los flujos se describen como pasos observables, no como diseño.

- **F1 · Registro y activación.** El jugador completa el formulario obligatorio y acepta términos, condiciones y política de datos (queda registrada la fecha y hora). Se verifican edad e identidad y se revisan las listas de prohibidos y autoexcluidos. Solo entonces la cuenta queda activa.
- **F2 · Autenticación.** El jugador inicia sesión y ve la fecha de su último acceso. A los 5 fallos la cuenta se bloquea. A los 20 minutos de inactividad se cierra la sesión. La reactivación exige una verificación reforzada.
- **F3 · Depósito.** El jugador elige un medio de pago a su nombre. El pago se confirma y se acreditan los créditos. Se aplican sus límites de depósito. Se registra el código de autorización. Los controles antilavado evalúan la operación.
- **F4 · Jugada con proveedor externo (modelo habitual de la industria).**
  1. El operador lanza el juego con un identificador de sesión.
  2. En cada apuesta, el proveedor pide al operador **descontar** el monto.
  3. Al resolverse la jugada, pide **abonar** el premio (que puede ser cero).
  4. Si algo falla, pide **revertir** la operación.

  Cada operación lleva un identificador. Su alcance de unicidad lo define cada proveedor: en algunos protocolos es único solo por jugador, y dos jugadores pueden compartir el mismo identificador [F26]. Si una operación llega repetida, debe responderse igual que la primera vez [F26]. Los identificadores se conservan para conciliación (un agregador exige al menos 4 meses) [F24]. Si el operador no responde a un descuento, el proveedor lo considera fallido y genera una reversión [F24].
- **F5 · Casino en vivo.** Se abre la ventana de apuestas y el servidor la cierra antes del resultado. El crupier juega, el resultado se registra y se liquidan las apuestas. Si el jugador pierde la conexión, la partida sigue las reglas de juego incompleto.
- **F6 · Retiro.** Se identifica plenamente al jugador, se pasan los controles antilavado y se emite la orden de transferencia en 72 horas como máximo. En el mercado, los retiros tardan típicamente entre 24 y 72 horas según el medio [F27].
- **F7 · Juego responsable.** El jugador fija límites de depósito o de sesión, o se autoexcluye. Si se autoexcluye, se bloquean depósitos y apuestas, pero se mantienen los retiros y las consultas.
- **F8 · Reporte al regulador.** Las transacciones se capturan, se convierten a XML según el modelo de datos, se almacenan y se cargan en la bodega que consulta Coljuegos. Aparte se reporta a la UIAF.
- **F9 · Falla y recuperación.** Se detecta la falla y se suspende lo afectado. Se informa al jugador y a Coljuegos. Se activa el plan de continuidad. Al volver, se resuelven las jugadas incompletas y se registran las evidencias.

---

## 3. Cifras de referencia

| Dato | Valor | Fecha | Fuente | Nota de uso |
|------|-------|-------|--------|-------------|
| Operadores online con contrato vigente | 15 plataformas | 2026 | [F17] | Mercado maduro y concentrado |
| Concentración del mercado | BetPlay 51,43 %, Wplay 14,5 %, Codere 11,7 %, Yajuego 5,27 %, Rushbet 4,7 % | Publicado en nov. 2024 | [F20] | Un operador nuevo sería mediano o pequeño |
| Cuentas "activas" en juegos online | 2,85 M (2020), 3,75 M (cierre de 2020), ~7 M (dic. 2022), **más de 12,4 M** (2025) | 2020–2025 | [F7][F6][F23][F18] | **Ojo:** el reglamento llama "activa" a toda cuenta no suspendida ni cancelada [F1]. Una persona puede tener cuentas en varios operadores, así que **no equivale a jugadores activos** |
| Usuarios activos proyectados | 2,08 M en 2028 | Proyección de 2025 | [F21] | Más cercano a jugadores reales |
| Tamaño del mercado online | ~USD 1.290 M en 2024; crecimiento anual compuesto proyectado del 9,53 % hasta 2029 | 2025 | [F21] | — |
| Peso de las apuestas deportivas | 66 % de los ingresos online | 2025 | [F21] | Por diferencia, casino y otros ≈ 34 % **[Estimación derivada]** |
| Recaudo de Coljuegos por juego online | COP 473.023 millones (ene.–ago. 2026), casi la mitad del recaudo total de Coljuegos. El +39,38 % anual corresponde al recaudo **total** de la entidad, no solo al online; las apuestas deportivas en línea crecieron 84,24 % por el Mundial | 2026 | [F17] | El pico de 2026 es deportivo, no de casino |
| Derechos de explotación online | COP 317.977 millones | 2023 | [F19] | — |
| Gasto típico del jugador | ~30 % de los encuestados gasta entre COP 20.000 y 50.000 al mes | 2025 | [F22] | Muchos montos pequeños: alto volumen y bajo valor por transacción |
| Medio de pago dominante | PSE, con ~60 % de las transacciones | 2023 | [F22] | — |
| Depósitos | Inmediatos en la mayoría de medios; rangos típicos de COP 1.000 a 5.000.000 según el medio | — | [F27] | — |
| Competencia ilegal | 118 marcas activas en el mercado colombiano, incluidas no autorizadas | Feb. 2026 | [F31] | Si la experiencia legal es peor, el jugador migra a sitios ilegales |
| Latencia del casino en vivo | Típica de 1–3 s; los líderes buscan menos de 500 ms en cartas | 2026 | [F28][F29] | — |
| Ventana de apuesta en vivo | 10–20 s según la mesa | 2026 | [F32] | Fuente secundaria; usar como orden de magnitud |
| Tier III | 99,982 % (≤ 1,6 h de falla al año) | — | [F1] | Piso regulatorio de la infraestructura |

**Cifras sin fuente pública, que deben decidirse como supuestos en la fase 2:**
- Jugadores concurrentes en hora pico **[Estimación]**.
- Patrón de picos de casino (se intuyen noches, fines de semana y quincenas) **[Estimación]**.
- Jugadas por segundo **[Estimación]**.
- Tamaño del catálogo de juegos y número de proveedores **[Estimación]**.
- Presupuesto y tamaño del equipo **[Estimación]**.

---

## 4. Tensiones que ya se asoman

Esto es un adelanto para la fase 3, sin soluciones.

| Hallazgo | Tensión que sugiere |
|----------|---------------------|
| Recertificación previa de todo cambio crítico (B3) | **Mantenibilidad** frente a Seguridad y Adecuación funcional. Cada cambio es lento y costoso. |
| Sistema alterno certificado y replicado de inmediato (E1) | **Fiabilidad** frente a Mantenibilidad y Desempeño. |
| El proveedor descuenta y abona el saldo en cada jugada (F4) | **Compatibilidad × Fiabilidad × Desempeño**. La integración externa está en el camino crítico de cada apuesta. |
| Bloqueo a los 5 intentos y cierre a los 20 minutos (C3, C12) | **Seguridad** frente a **Usabilidad**. Es el mismo ejemplo del widget de tensiones del curso. |
| Detección de suplantación por comportamiento (C4) y SARLAFT: perfil de riesgo en cada depósito y retiro y reconsulta masiva de listas (sección 1.3) | **Seguridad** frente a **Desempeño**, porque se evalúa en el camino de cada depósito o retiro. Y **Seguridad** frente a **Usabilidad**, por la diligencia reforzada, que retiene retiros. |
| Inspección en Colombia, con formato fijado por Coljuegos (A2, A3) | **Compatibilidad** obligatoria y **Portabilidad** restringida. |
| Ventana de apuestas en vivo y juegos *crash* (sección 2.2) | **Desempeño** frente a equidad del juego (Adecuación funcional y Fiabilidad). |
| Reglas fiscales y técnicas que cambian por decreto (secciones 1.1 y 1.6) | **Mantenibilidad** (capacidad de modificación). |

---

## 5. Punto de control: decisiones del equipo (cerrado el 25 de octubre de 2026)

| # | Decisión | Resultado |
|---|----------|-----------|
| 1 | Alcance de juegos | **Tragamonedas, ruleta con GNA, blackjack en vivo y un juego tipo *crash*** (estilo "el avión") |
| 2 | Perfil del operador | **Nuevo concesionario mediano** |
| 3 | Origen de los juegos | **Todos los juegos los suministran proveedores externos certificados.** El operador no desarrolla juegos propios. *Cómo* se conecta con ellos (integración directa o mediante un intermediario) queda abierto como posible decisión de arquitectura. |
| 4 | Sistema antilavado | **SARLAFT** (administración del riesgo de lavado de activos y financiación del terrorismo, basada en riesgos) en lugar de un SIPLAFT básico, cumpliendo de todos modos los mínimos del SIPLAFT que exige la Res. 20195100044514 de 2019 [F9]. Razones: es la dirección que anunció el regulador [F30] y facilita el acceso del operador al sistema bancario [F33]. |

---

## 6. Vacíos y puntos por verificar

- Los **requerimientos técnicos están en revisión en 2026**. Si se publica una versión nueva antes de la entrega, hay que contrastarla.
- No está confirmado que **SARLAFT** ya se exija a los operadores. El equipo decidió adoptarlo de todos modos, así que esto ya no bloquea.
- No leímos completa la **resolución de juego responsable de 2024**.
- El **Decreto 0240 de 2026** tiene un estado jurídico incierto, por el precedente de suspensión de la Corte.
- No hay datos públicos de concurrencia ni de picos del casino online en Colombia. Todo eso será supuesto declarado.

---

## Fuentes

- **[F1]** Coljuegos, Res. 20224000026034 de 2022 y Requerimientos Técnicos de Juegos Operados por Internet (Diario Oficial 52.193). https://sidn.ramajudicial.gov.co/SIDN/NORMATIVA/TEXTOS_COMPLETOS/8_RESOLUCIONES/RESOLUCIONES%202022/AMRJSA%20Resoluciones%2020224000026034%20de%202022%20(Requerimientos%20T%C3%A9cnicos%20de%20Juegos%20Operados%20por%20Internet).pdf
- **[F2]** Coljuegos, Acuerdo 4 de 2016. https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0004_2016.htm
- **[F3]** Coljuegos, Acuerdo 2 de 2019 (retorno al jugador) y documento de Coljuegos sobre la modificación del retorno (revisión semestral). https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0002_2019.htm · https://www.coljuegos.gov.co/descargar.php?id=201359
- **[F4]** Coljuegos, Acuerdo 4 de 2019 (tipos de juego). https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0004_2019.htm
- **[F5]** Coljuegos, Acuerdo 5 de 2020 (casino en vivo). https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0005_2020.htm
- **[F6]** Coljuegos, Acuerdo 2 de 2021. https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0002_2021.htm
- **[F7]** Coljuegos, Acuerdo 2 de 2020. https://normativa.colpensiones.gov.co/colpens/docs/acuerdo_coljuegos_0002_2020.htm
- **[F8]** Coljuegos, Acuerdo 8 de 2020 (compilatorio). https://normograma.supersalud.gov.co/compilacion/docs/acuerdo_coljuegos_0008_2020.htm
- **[F9]** Coljuegos, Res. 20195100044514 de 2019 (SIPLAFT). https://normograma.supersalud.gov.co/compilacion/docs/resolucion_coljuegos_44514_2019.htm
- **[F10]** Coljuegos, Res. 20161200032334 de 2016 (derogada; definiciones de estados de cuenta). https://normograma.supersalud.gov.co/compilacion/docs/resolucion_coljuegos_32334_2016.htm
- **[F11]** Coljuegos, Res. 20161200013844 de 2016 (laboratorios). https://vlex.com.co/vid/resolucion-numero-20161200013844-2016-643663249
- **[F12]** Coljuegos, Res. 20214000036784 de 2021, con nota de derogación por la Res. 20244000022654 de 2024. https://normograma.supersalud.gov.co/compilacion/docs/resolucion_coljuegos_36784_2021.htm
- **[F13]** Focus Gaming News, "Coljuegos refuerza el marco regulatorio para los juegos de premio inmediato online" (2026). https://focusgn.com/latinoamerica/coljuegos-refuerza-el-marco-regulatorio-para-los-juegos-de-premio-inmediato-online
- **[F14]** Vixio, requerimientos técnicos de Colombia para juegos de casino (2026). https://www.vixio.com/articles/what-tool-can-help-a-casino-game-provider-understand-technical-approval-requirements-in-colombia
- **[F15]** Yogonet, "El Gobierno de Petro introduce un nuevo impuesto del 16 % al juego online" (mar. 2026). https://www.yogonet.com/latinoamerica/noticias/2026/03/16/108079-colombia-el-gobierno-de-petro-introduce-un-nuevo-impuesto-del-16-al-juego-online
- **[F16]** Yogonet, entrevista a Evert Montero, Fecoljuegos (mar. 2026), y Focus Gaming News sobre los decretos suspendidos. https://www.yogonet.com/latinoamerica/noticias/2026/03/30/108265-evert-montero-nuestra-posicion-es-clara-el-sector-esta-dispuesto-a-tributar-pero-no-bajo-esquemas-improvisados · https://focusgn.com/latinoamerica/el-gobierno-de-colombia-crea-por-decreto-un-nuevo-impuesto-nacional-al-consumo-del-16-para-el-juego-online
- **[F17]** SiGMA, "Las apuestas del Mundial elevaron el recaudo del juego en Colombia" (2026). https://sigma.world/es/news/recaudo-coljuegos-2026-apuestas-mundial/
- **[F18]** SiGMA, "Diferencias en los datos del juego en línea en Colombia" (2025). https://sigma.world/es/news/diferencias-datos-juego-en-linea-colombia/
- **[F19]** Yogonet, listado de operadores legales y recaudo de 2023 (feb. 2024). https://www.yogonet.com/latinoamerica/noticias/2024/02/16/98043-coljuegos-publico-una-actualizacion-del-listado-de-operadores-legales-de-apuestas-online-en-colombia
- **[F20]** La República, "Colombia cuenta con 15 operadores autorizados en el mercado de apuestas en línea" (nov. 2024). https://www.larepublica.co/empresas/mercado-de-apuestas-online-en-colombia-4002300
- **[F21]** Vanguardia, "Colombia afianza su posición en el mercado de casinos latinoamericano" (oct. 2025). https://www.vanguardia.com/mundo/2025/10/21/colombia-afianza-su-posicion-en-el-mercado-de-casinos-latinoamericano/
- **[F22]** La República, índice de notas de Fecoljuegos: encuesta de Playtech de 2025 y medios de pago de 2023. https://www.larepublica.co/fecoljuegos
- **[F23]** Las2orillas, "Los 3 poderosos dueños de los juegos en línea en Colombia". https://www.las2orillas.co/?p=757561
- **[F24]** Hub88, documentación de su Wallet API. https://docs.hub88.io/developer-docs/operator-api-reference/wallet-api
- **[F25]** DEV Community, "Casino API Integration Explained". https://dev.to/betnex_a4734ded26ea85377f/casino-api-integration-explained-architecture-wallet-callbacks-and-scaling-an-igaming-platform-414j
- **[F26]** Blue Ocean Gaming, preguntas frecuentes de su integración de billetera. https://blueoceangaming.atlassian.net/wiki/x/gIASS
- **[F27]** El Tiempo, guías de depósito y retiro en operadores colombianos. https://www.eltiempo.com/apuestas-deportivas/betsson-deposito-852029 · https://www.eltiempo.com/apuestas-deportivas/betsson-retiro-853268
- **[F28]** Stats Perform, latencia en *streaming* de apuestas y casino en vivo. https://www.statsperform.com/?p=33709
- **[F29]** Programming Insider, el casino en vivo como estudio de televisión (2026). https://programminginsider.com/what-a-live-dealer-table-has-in-common-with-a-television-studio-and-where-the-two-part-ways/
- **[F30]** Focus Gaming News, "Coljuegos destruyó cerca de 10.000 elementos de juego ilegal" (evolución SIPLAFT → SARLAFT, 2025). https://focusgn.com/latinoamerica/coljuegos-destruyo-cerca-de-10-000-elementos-de-juego-ilegal
- **[F31]** Blask, "Colombia: deposit tax returns, opening doors for offshore" (2026). https://blask.com/newsroom/colombia-deposit-tax-returns-opening-doors-for-offshore/
- **[F32]** MercoPress, "How Live Dealer Casino Games Actually Work" (sept. 2026). https://en.mercopress.com/2026/09/01/how-live-dealer-casino-games-actually-work-at-olimp
- **[F33]** Focus Gaming News, "Coljuegos y la Superintendencia Financiera de Colombia buscan facilitar el acceso de operadores de juego al sistema bancario". https://focusgn.com/latinoamerica/coljuegos-y-la-superintendencia-financiera-de-colombia-buscan-facilitar-el-acceso-de-operadores-de-juego-al-sistema-bancario

---

> [← Anterior](./00-plan-de-trabajo.md) · [Índice](./README.md) · [Siguiente →](./02-ficha-contexto.md)
