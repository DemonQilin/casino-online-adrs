# Fase 5 — Selección de escenarios

> [← Anterior](./04-priorizacion.md) · [Índice](./README.md) · [Siguiente →](./06-escenarios.md)

**Proyecto:** Casino Andino, casino online regulado en Colombia
**Insumos:**
- [`04-priorizacion.md`](./04-priorizacion.md): priorización adoptada.
- [`03-mapa-tensiones.md`](./03-mapa-tensiones.md): tensiones T1–T6 y L1.
- [`02-ficha-contexto.md`](./02-ficha-contexto.md): cifras y restricciones.

**Priorización adoptada:**

| Seguridad | Fiabilidad | Desempeño | Compatibilidad | Adec. funcional | Usabilidad | Mantenibilidad | Portabilidad |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **4** | **4** | **3** | **3** | 1 | 1 | 1 | 1 |

**Atributos altos** (los "más altos de la priorización"): Seguridad, Fiabilidad, Desempeño y Compatibilidad.

**Prueba de criticidad** (del widget de ADR del curso). Un escenario es crítico solo si responde "sí" a las dos preguntas:
1. ¿Su atributo dominante está entre los más altos de la priorización?
2. ¿Cumplirlo obliga a pagar un costo en otro atributo que también puntuamos alto?

> En esta fase cada candidato se escribe en una línea (fuente, estímulo y entorno). Las seis partes completas se redactan en la fase 6.

---

## 1. Candidatos

| ID | Tensión | Fuente | Estímulo | Entorno |
|----|---------|--------|----------|---------|
| K1 | T1 · Seguridad × Desempeño | Jugador incluido en una lista restrictiva recibida hace 30 minutos, cuya cuenta aún no ha sido reconsultada | Solicita un retiro de COP 3.000.000 a su billetera digital | Viernes de quincena a las 21:00, 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo y reconsulta masiva de 350.000 cuentas en curso |
| K2 | T1 · Seguridad × Desempeño | Proveedor de consulta de listas restrictivas | Publica una actualización de las listas | Pico de diseño: 12.000 jugadores simultáneos, ~3.000 operaciones de saldo por segundo |
| K3 | T2 · Fiabilidad × Desempeño | Falla de hardware en el servidor que registra las operaciones de saldo | Deja de responder abruptamente, con operaciones en curso | Pico de diseño: ~3.000 operaciones de saldo por segundo y una ronda de *crash* en curso con 2.400 jugadores |
| K4 | T2 · Desempeño × Fiabilidad | Los 3 proveedores de tragamonedas y ruleta | Envían peticiones de descuento y abono del saldo | Pico de diseño: ~3.000 por segundo, con ráfagas de 4.000 |
| K5 | T3 · Compatibilidad × Fiabilidad | Proveedor del juego *crash* | Envía 3 veces la misma petición de abono, con el mismo identificador de transacción, porque no recibió respuesta a tiempo | Pico de diseño con red congestionada entre el proveedor y el operador; ronda en curso con 2.400 jugadores |
| K6 | L1 · Seguridad × Fiabilidad | Proveedor de consulta de listas restrictivas | Deja de responder durante 2 horas | Viernes de quincena, ~300 retiros solicitados en ese lapso |
| K7 | T6 · Desempeño × Adecuación funcional | 720 jugadores del *crash* | Se retiran en el mismo segundo, cuando el multiplicador llega a 2,00x | Pico de diseño, ronda con 2.400 jugadores |
| K8 | T4 · Mantenibilidad × Seguridad | Gobierno nacional | Expide un decreto que cambia la base de cálculo de un impuesto sobre el GGR bimestral | Sistema en producción; equipo de 14 personas; validación del laboratorio en 2–4 semanas |
| K9 | T5 · Seguridad × Usabilidad | Jugador legítimo que olvidó su contraseña | Falla 5 intentos seguidos de inicio de sesión | Sábado a las 22:00, desde el celular, con un saldo de COP 80.000 |

---

## 2. Prueba de criticidad

| ID | Atributo dominante | ¿Dominante alto? | Atributo que paga | ¿Paga un alto? | Resultado |
|----|-------------------|:----------------:|-------------------|:--------------:|-----------|
| K1 | Seguridad (4) | ✔ | Desempeño (3): evaluar contra la lista recién recibida, en el camino del retiro, mientras corre la reconsulta masiva en pleno pico | ✔ | **Crítico** |
| K2 | Seguridad (4) | ✔ | Desempeño (3) | ✔ | Crítico, pero **misma tensión que K1** |
| K3 | Fiabilidad (4) | ✔ | Desempeño (3): asegurar que ninguna operación confirmada se pierda alarga cada una de las ~3.000 por segundo | ✔ | **Crítico** |
| K4 | Desempeño (3) | ✔ | Fiabilidad (4) | ✔ | Crítico, pero **misma tensión que K3** |
| K5 | Compatibilidad (3) | ✔ | Fiabilidad (4): aceptar el protocolo del proveedor (reintentos, reversiones) arriesga la integridad del saldo | ✔ | **Crítico** |
| K6 | Seguridad (4) | ✔ | Fiabilidad (4) | ✔ | Pasa la prueba, pero **la decisión ya está tomada** (fallo cerrado, restricción 9) |
| K7 | Desempeño (3) | ✔ | Adecuación funcional (1): la corrección del premio, que además calcula el proveedor | ✘ | **No crítico.** Es importante, pero cumplirlo no sacrifica un atributo alto |
| K8 | Mantenibilidad (1) | ✘ | Seguridad (4) | ✔ | No crítico. El dominante no está entre los altos (riesgo asumido en la fase 4) |
| K9 | Seguridad (4) | ✔ | Usabilidad (1) | ✘ | No crítico. Además, el bloqueo a los 5 intentos es piso regulatorio fijo |

---

## 3. Selección propuesta

**4 escenarios: 3 críticos y 1 no crítico.** Eso da **3 ADRs**.

| Escenario | Candidato | Tensión | Atributo dominante | ¿Crítico? | ¿Lleva ADR? |
|-----------|-----------|---------|--------------------|:---------:|:-----------:|
| **ESC-01** | K1 · Retiro de un jugador en lista durante la reconsulta masiva | T1 | Seguridad | ✔ | **ADR-01** |
| **ESC-02** | K3 · Falla del servidor de saldo en pleno pico | T2 | Fiabilidad | ✔ | **ADR-02** |
| **ESC-03** | K5 · Abono repetido del proveedor del *crash* | T3 | Compatibilidad | ✔ | **ADR-03** |
| **ESC-04** | K7 · Ráfaga de retiros del *crash* | T6 | Desempeño | ✘ | No |

### Justificación de la selección (en una línea)

> Elegimos ESC-01, ESC-02 y ESC-03 como críticos porque son las únicas tensiones fuertes entre dos atributos altos (T1, T2 y T3), con un escenario por tensión. ESC-04 se escribe sin ADR porque su costo cae en un atributo que priorizamos bajo.

### Por qué quedan fuera los demás

- **K2 y K4:** repiten la tensión de K1 y K3. El material pide un escenario por tensión. K1 ya incorpora la actualización de listas de K2 en su entorno.
- **K6:** la decisión ya está tomada (fallo cerrado). Un ADR cuya decisión está decidida de antemano sería un anuncio, no un registro de alternativas. **Entra como restricción en el Context de ADR-01.**
- **K8 y K9:** su dominante o su costo está en un atributo priorizado bajo. Es exactamente lo que aceptamos en la fase 4.

### Cobertura que logra la selección

- **Los cuatro atributos altos son dominantes** en algún escenario: Seguridad (ESC-01), Fiabilidad (ESC-02), Compatibilidad (ESC-03) y Desempeño (ESC-04).
- **Los retos del tema quedan cubiertos:**
  - antilavado y regulación (ESC-01);
  - continuidad del dinero (ESC-02);
  - interoperabilidad con proveedores (ESC-03);
  - tiempo real (ESC-04).
- **ESC-04 demuestra el criterio de selección:** muestra un escenario importante que, a propósito, no lleva ADR. La rúbrica premia eso, y pone en nivel Inicial escribir ADRs para todos.

---

## 4. Variante: 5 escenarios

Si el equipo quiere mostrar un segundo ejemplo de escenario no crítico, se puede agregar **K9** (bloqueo a los 5 intentos) como **ESC-05**. La razón para no hacerlo es que la mayor parte de K9 la fija la regulación y aporta poco análisis nuevo. En cualquier caso, la cantidad de ADRs sigue siendo 3.

---

## 5. Verificación de la fase

| Criterio | Estado |
|----------|--------|
| Al menos 6 candidatos derivados de tensiones | ✔ 9 candidatos: el plan pedía de 6 a 8; se agregó uno más para descartar con criterio los no críticos |
| Prueba de criticidad aplicada con las dos preguntas | ✔ Sección 2 |
| Entre 3 y 5 escenarios elegidos, críticos marcados | ✔ 4 escenarios, 3 críticos |
| Selección justificada en una línea | ✔ Sección 3 |
| Rúbrica, criterio *Selección de escenarios críticos*, nivel "Logrado" | Cada ADR responde a un escenario con dominante alto que cuesta otro alto; la selección está justificada |

---

## 6. Punto de control (cerrado el 25 de octubre de 2026)

1. **Aprobado:** 4 escenarios, 3 de ellos críticos (3 ADRs). Se descarta la variante de 5.
2. **Aprobado:** el fallo cerrado entra como restricción en el Context de ADR-01 y no tiene ADR propio.

---

> [← Anterior](./04-priorizacion.md) · [Índice](./README.md) · [Siguiente →](./06-escenarios.md)
