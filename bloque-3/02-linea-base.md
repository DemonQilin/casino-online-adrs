# Fase 2 — Línea base

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Sección de la plantilla:** 2 (Línea base: la solución actual)
**Criterio de la rúbrica:** Línea base y cuello de botella. Esta fase aporta la mitad del criterio: una línea base clara y sin tácticas. El componente que se satura primero, su límite con fuente y lo que ve el usuario llegan en la fase 4.

**Insumos:**
- `01-punto-de-partida.md`: operación crítica (saldo) y retiro como otra dimensión; tipos de cifra.
- `bloque-2/07-adrs.md`: las opciones de cada ADR, de donde sale la versión más simple.
- `bloque-2/02-ficha-contexto.md` §4, §5 y §6, y `bloque-2/01-investigacion-dominio.md` §3: restricciones del reglamento (E1, E7, E11, C6, C11) y supuestos.
- `docs/class-material/PedalVa Tácticas.html`: catálogo de tácticas, para el chequeo de la sección 6.

**Qué fija esta fase:**
1. Cómo funciona la operación de saldo en la versión más simple, en un párrafo y componente por componente.
2. Cómo funciona el retiro en esa misma versión.
3. Qué hay en la línea base porque el reglamento lo exige y qué queda libre para las tácticas de la fase 5.

No entran límites de componentes ni fuentes nuevas: esos nacen en la fase 4.

---

## 1. Criterio para construir la línea base

Casino Andino no existe, así que la línea base es **la versión más simple que funcionaría, sin ninguna de las tácticas que se van a proponer** (material del Bloque 3, sección 2).

Se arma con dos reglas:

1. **De cada ADR se toma la opción más simple del Bloque 2.** Así la línea base es una de las opciones que el equipo ya comparó, no una invención nueva.
2. **Se agrega el piso que exige el reglamento**, en su forma más débil que todavía cumple la norma. Lo que el reglamento obliga no es una táctica elegida (sección 4).

| ADR | Opción que entra en la línea base | Medida del escenario que incumple |
|-----|-----------------------------------|-----------------------------------|
| ADR-02 | **B** · Confirmar tras la grabación local y copiar después | **0 operaciones confirmadas perdidas**: si el servidor falla antes de copiar, se pierden las últimas operaciones ya confirmadas |
| ADR-03 | **A** · Buscar el identificador en el historial de operaciones | **0 abonos duplicados**: dos copias simultáneas pueden pasar ambas la búsqueda. Además, el tiempo de búsqueda crece con el historial |
| ADR-04 | **A** · Clave fija y lista de direcciones de red permitidas por proveedor | **100 % de la batería rechazada**: la copia de una petición legítima con el monto modificado pasa la verificación |
| ADR-01 | **A** · Retener los retiros hasta que termine la reconsulta | **Evaluación del retiro en p95 ≤ 1 s**: durante la reconsulta, un retiro espera hasta 6 h [S19 · supuesto] |

**Por qué así:** las cuatro opciones incumplen su medida. Eso es lo que se busca: si la línea base ya cumpliera los escenarios, las tácticas de la fase 5 no tendrían contra qué compararse.

---

## 2. Cómo funciona hoy, en un párrafo

*Texto para la plantilla.*

> El operador tiene un sitio principal en un centro de datos Tier III con un servidor de aplicación y un servidor de base de datos, y un sistema alterno en un sitio separado con la misma composición. Cuando un proveedor de juego pide descontar, abonar o revertir saldo, la petición llega cifrada por internet, se acepta solo si viene de una dirección de red registrada para ese proveedor y trae su clave fija, y la aplicación la interpreta con el código propio del protocolo de ese proveedor. Luego busca el identificador de la transacción en el historial de operaciones del jugador: si ya está, reconstruye la respuesta; si no, en una sola transacción de la base de datos comprueba que el saldo alcanza, cambia el saldo y anota la operación en el historial, y responde al proveedor en cuanto la escritura queda grabada en el disco del sitio principal. Después, un proceso copia las operaciones al sistema alterno, segundos más tarde, y otro extrae periódicamente las transacciones para el sistema de inspección de Coljuegos. Los retiros se evalúan contra el perfil de riesgo y la marca de listas restrictivas guardada en cada cuenta; cuando llega una lista nueva, una reconsulta recorre todas las cuentas consultando al proveedor de listas, y mientras dura, todos los retiros quedan retenidos.

Composición del sitio: 1 servidor de aplicación y 1 servidor de base de datos [S30 · supuesto]. Centro Tier III y sistema alterno [F1 · referencia].

---

## 3. Componentes que intervienen, en orden

### 3.1 Operación de saldo (operación crítica)

| # | Componente | Qué hace en la operación | Cifras que pasan por aquí |
|---|-----------|--------------------------|---------------------------|
| 1 | **Proveedor de juego** (externo) | Al resolverse una jugada, envía la petición de descuento, abono o reversión con su propio protocolo y su identificador de transacción. Si no recibe respuesta en ~3 s, la da por fallida, la reenvía o la revierte | 5 proveedores [dato]; ~3 s de espera [S14 · supuesto][F24 · referencia] |
| 2 | **Red pública** hasta el sitio principal | Lleva la petición por un canal cifrado, autenticado en ambos sentidos | Clave de ≥ 2048 bits (E7) [F1 · referencia] |
| 3 | **Entrada del sitio principal** | Descarta toda petición que no venga de una dirección de red registrada para algún proveedor | — |
| 4 | **Servidor de aplicación: interfaz de saldo** | (a) Compara la clave fija de la petición con la del proveedor. (b) Interpreta el protocolo de ese proveedor con un código propio dentro de la misma aplicación. (c) Busca el identificador en el historial de operaciones del jugador, con proveedor y jugador (la unicidad depende del proveedor [F26 · referencia]). Si lo encuentra, reconstruye la respuesta original y salta al paso 6 | 1 servidor [S30 · supuesto]; ~3.000 operaciones/s sostenidas, ~4.000 en ráfaga [S6 · supuesto] |
| 5 | **Servidor de base de datos: base única** | En **una transacción**: comprueba que el saldo alcanza, cambia el saldo y agrega la operación al historial. Confirma cuando la escritura queda grabada en el disco local. La misma base guarda cuentas, perfiles de riesgo, depósitos y retiros | 1 servidor [S30 · supuesto]; ~24 M de operaciones al día [S4][S6 · supuesto]; historial ≥ 6 meses (C11) [F1 · referencia] |
| 6 | **Servidor de aplicación: respuesta** | Devuelve al proveedor el resultado y el saldo nuevo | Meta p99 ≤ 250 ms medida en el operador [S14 · supuesto] |
| 7 | **Copia al sistema alterno** (después de responder) | Un proceso lee las operaciones ya confirmadas y las graba en el alterno, segundos después | Sitio separado, misma certificación (E1, E5) [F1 · referencia] |
| 8 | **Extracción para el sistema de inspección** (después de responder) | Un proceso lee del historial las transacciones nuevas y las envía en el formato de Coljuegos, con la periodicidad del modelo de datos | Formato y periodicidad de Coljuegos [F1 · referencia] |

Los pasos 1 a 6 están en el camino de la respuesta y consumen los 250 ms. Los pasos 7 y 8 no la retrasan, pero leen la misma base de datos.

### 3.2 Retiro (otra dimensión)

| # | Componente | Qué hace | Cifras |
|---|-----------|----------|--------|
| 1 | **Jugador** | Solicita el retiro desde el sitio web | ~3.000 retiros al día [S8 · supuesto] |
| 2 | **Servidor de aplicación** (el mismo de la operación de saldo) | Recibe la solicitud y llama a la evaluación de riesgo | — |
| 3 | **Evaluación de riesgo** (dentro de la misma aplicación) | Lee de la base única el perfil de riesgo y la marca de listas de la cuenta. **Si hay una reconsulta en curso, retiene el retiro** hasta que termine. Si la evaluación no está disponible, el retiro se detiene (fallo cerrado, restricción 9) | Meta p95 ≤ 1 s [S20 · supuesto] |
| 4 | **Proveedor de listas** (externo) | Entrega la actualización de listas restrictivas varias veces al mes y responde consultas por documento | Varias veces al mes [S19 · supuesto] |
| 5 | **Reconsulta masiva** (proceso en el mismo servidor de aplicación) | Al llegar la lista, recorre las cuentas: lee cada una de la base única, consulta su documento al proveedor de listas y guarda la marca. Corre sin ritmo máximo, tan rápido como respondan la base y el proveedor | 350.000 cuentas [S3 · supuesto]; ≤ 6 h, ≥ 16 consultas/s [S19 · supuesto] |
| 6 | **Orden de transferencia** al medio de pago | Si el retiro pasa la evaluación, se emite la orden | Meta ≤ 24 h [S8 · supuesto]; máximo legal 72 h [F1 · referencia] |

---

## 4. Frontera con la regulación (tarea 2.3)

Algunas piezas de la línea base coinciden con tácticas del catálogo. Están porque el reglamento las exige, y entran en su forma más débil. La fase 5 no puede presentarlas como mejora; solo puede proponer **reforzarlas**, y el efecto se mide contra la forma débil.

| Pieza de la línea base | Por qué está | Táctica del catálogo con la que coincide | Qué queda abierto para la fase 5 |
|------------------------|-------------|------------------------------------------|----------------------------------|
| Sistema alterno con copia posterior | E1: sistema alterno certificado, con los cambios replicados de inmediato [F1] | **Redundancia pasiva** ("una réplica recibe el estado periódicamente") | **Cuándo** se copia (antes o después de confirmar) y **cuántas** copias: es la decisión de ADR-02 |
| Transacción de saldo e historial | C6: ningún saldo negativo; C11: reconstruir cada jugada [F1] | **Transacciones** | Qué más entra en el paso indivisible, por ejemplo el registro de identificadores de ADR-03 |
| Historial de operaciones | C11: registro de toda operación ≥ 6 meses [F1] | **Auditoría** | Nada: se usa tal cual |
| Clave fija por proveedor | E11: autenticar todos los accesos, también de otros sistemas [F1] | **Identificar actores**, **Autenticar actores** | Autenticar el contenido de cada petición: ADR-04 |
| Filtro de direcciones de red | Parte de la opción A de ADR-04, la más simple que cumple E11 | **Limitar el acceso** | Nada nuevo |
| Canal cifrado y autenticado en ambos sentidos | E7: cifrado ≥ 2048 bits y autenticación mutua [F1] | **Cifrar datos** | Nada |
| Fallo cerrado en los retiros | Restricción 9 de la ficha (decisión de la fase 3 del Bloque 2) | — | Nada: es una regla de negocio |

**Riesgo de interpretación (E1).** El reglamento dice "de inmediato" y la línea base copia segundos después. El equipo lo lee como la forma mínima que cumple la norma; otra lectura exigiría copia antes de confirmar, y entonces la opción A de ADR-02 sería el piso y no una táctica. Se deja anotado para la sustentación: si el profesor pregunta, la respuesta es que la diferencia entre las dos lecturas es justamente la ventana de pérdida que mide ESC-02.

**Por qué no un repuesto en frío.** Sería más simple que la copia posterior, pero un alterno que se enciende cuando hace falta no recibe los cambios de inmediato: no cumple E1.

---

## 5. Piezas que se pueden saturar

Candidatos para la fase 4. Aquí solo se nombran; sus límites y fuentes llegan en esa fase.

| Candidato | Por qué puede saturarse |
|-----------|-------------------------|
| Base de datos única: escrituras | Recibe las ~3.000 escrituras por segundo del saldo [S6], los depósitos y los retiros, y cada una espera la grabación en disco |
| Base de datos única: búsqueda del identificador | Cada operación busca en un historial que crece ~24 M de operaciones al día [S4][S6] y debe conservarse ≥ 6 meses [F1] |
| Base de datos única: lecturas de fondo | La copia al alterno, la extracción para Coljuegos y la reconsulta leen la misma base que el juego, sin ritmo máximo |
| Servidor de aplicación | Uno solo [S30] atiende las peticiones de los 5 proveedores, el sitio de los jugadores y la reconsulta |
| Red pública y canal cifrado | Cada petición cruza internet y el cifrado tiene un costo de cómputo por conexión |
| Proveedor de listas (externo) | La reconsulta le pide ≥ 16 consultas por segundo [S19]; su límite no lo controla el operador |
| Copia al alterno | Si se atrasa frente a las ~3.000 operaciones por segundo, crece la ventana de operaciones confirmadas que se perderían |

---

## 6. Chequeo: ninguna táctica de la fase 5 en la línea base (tarea 2.4)

Se recorrió el catálogo del curso. Las tácticas se dividen en tres grupos.

**Presentes, solo en la forma que exige el reglamento** (sección 4): Redundancia pasiva, Transacciones, Auditoría, Identificar actores, Autenticar actores, Limitar el acceso, Cifrar datos.

**Ausentes de la línea base y libres para la fase 5**, con el ADR donde podrían entrar:

| Familia del catálogo | Tácticas | ADR candidato |
|----------------------|----------|---------------|
| Disponibilidad | Redundancia activa, Votación, Latido (heartbeat), Ping / eco, Monitor, Marca de tiempo, Resincronización de estado, Reconfiguración, Reintento, Interruptor de circuito, Degradación | ADR-02, ADR-01, ADR-04 |
| Rendimiento | Múltiples copias de cómputo, Múltiples copias de datos, Introducir concurrencia, Aumentar la eficiencia, Reducir el sobrecosto, Procesamiento asíncrono, Limitar la tasa de eventos, Priorizar eventos, Planificar recursos, Acotar el tamaño de las colas | ADR-01, ADR-02, ADR-03 |
| Seguridad | Verificar integridad del mensaje, Detectar retraso del mensaje, Detectar intrusiones, Detectar denegación de servicio, Validar entradas, Revocar acceso, Informar a los actores | ADR-04, ADR-03 |
| Modificabilidad | Usar un intermediario, Encapsular, Restringir dependencias, Componentes enchufables | ADR-03, ADR-04 |

**Puntos a vigilar:**
- **Usar un intermediario.** En la línea base cada protocolo se interpreta con código propio *dentro de la misma aplicación*. No hay una pieza separada por proveedor; esa pieza es la opción C de ADR-03 y la B de ADR-04.
- **Validar entradas.** La línea base solo rechaza lo que no puede interpretar. Rechazar una petición repetida con datos distintos (ESC-03) no está.
- **Múltiples copias de cómputo.** Por eso la línea base tiene un solo servidor de aplicación [S30]. Ver la variante V2.

---

## 7. Cifra nueva

| ID | Supuesto | Razón |
|----|----------|-------|
| S30 | La línea base tiene **1 servidor de aplicación y 1 servidor de base de datos** en el sitio principal, y la misma composición en el sistema alterno | Es la versión más simple que funcionaría. Cada servidor adicional en el sitio principal sería una táctica del catálogo (Múltiples copias de cómputo o Redundancia activa) y debe entrar en la fase 5 con su efecto y su costo. Agregado el 13 de noviembre de 2026. |

---

## 8. Variantes

| Variante | Qué cambia | A favor | En contra |
|----------|-----------|---------|-----------|
| **V1 (recomendada)** | Lo descrito en las secciones 2 y 3: opciones B, A, A, A y un servidor de cada tipo | Ninguna táctica elegible queda dentro; cada ADR tiene una medida incumplida contra la cual comparar | Un solo servidor de aplicación se ve poco realista para 12.000 jugadores. Es justamente lo que la fase 4 debe mostrar con números |
| **V2** | Dos servidores de aplicación detrás de un distribuidor de entrada | Más parecido a una plataforma real; los casos del material lo usan como línea base | Mete *Múltiples copias de cómputo* en la línea base: la fase 5 ya no podría usarla como táctica, y el cuello de la fase 4 se movería con certeza a la base de datos |
| **V3** | En ADR-01, la opción B (consultar al proveedor de listas en cada retiro) en lugar de la A | Los retiros no esperan las 6 h de la reconsulta | No es la más simple: añade una dependencia externa al camino de cada retiro. Además "Depende" de la latencia del proveedor, así que la línea base podría cumplir el p95 ≤ 1 s y ADR-01 perdería contra qué compararse |

---

## 9. Verificación de la fase

| Comprobación | Estado |
|--------------|--------|
| La operación de saldo se puede seguir de punta a punta | ✔ §3.1, 8 pasos; los pasos 1 a 6 forman el camino de la respuesta |
| Se ven las piezas que se pueden saturar | ✔ §5, 7 candidatos para la fase 4 |
| Cada opción de la línea base coincide con su texto en `bloque-2/07-adrs.md` | ✔ ADR-02 B, ADR-03 A, ADR-04 A, ADR-01 A |
| La línea base no cumple ningún escenario crítico | ✔ §1, una medida incumplida por ADR |
| Ninguna táctica de la fase 5 está en la línea base, salvo el piso regulatorio declarado | ✔ §4 y §6 |
| Cada cifra tiene tipo y origen | ✔ S3, S4, S6, S8, S14, S19, S20 y S30 (supuestos); F1, F24 y F26 (referencias); 5 proveedores (dato) |
| Ningún mecanismo nombra tecnologías ni marcas | ✔ "base de datos única", "entrada del sitio", "canal cifrado" |
| No hay límites ni fuentes nuevas | ✔ Solo S30, que describe la composición; los límites son de la fase 4 |

---

## 10. Punto de control (cerrado el 13 de noviembre de 2026)

**Decisiones del equipo:** se aprueba la variante **V1** tal como está propuesta.

1. **Composición de la línea base: V1**, un servidor de aplicación y uno de base de datos en el sitio principal. V2 queda registrada como variante.
2. **Retiro: opción A de ADR-01.** V3 queda registrada como variante.
3. **Lectura de E1:** la copia posterior es el piso regulatorio (§4). La opción A de ADR-02 sigue siendo una opción de la fase 5.
4. **Supuesto S30 aprobado**, con su razón.

La fase 3 (perfil de carga) puede arrancar.
