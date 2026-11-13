# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es este repositorio

Proyecto de aula de **Arquitectura de Software (ITM, Equipo 8)**. Desarrolla de forma **teórica** la arquitectura de *Casino Andino*, un casino online regulado en Colombia. Avanza bloque a bloque, según el material que publica el profesor.

No hay código de producto, ni build, ni lint, ni pruebas. Los entregables son documentos markdown y una presentación HTML por bloque.

## Mapa del curso

| Bloque | Tema | Estado | Efecto sobre los ADR |
|--------|------|--------|----------------------|
| 2 | Atributos de calidad, escenarios y ADR | Entregado (`bloque-2/`) | Context y opciones consideradas. Estado: Propuesto |
| 3 | Tácticas | Pendiente | Se completan Decision y Consequences. Estado: Aceptado |
| 4 | Estilos arquitectónicos | Pendiente | Un estilo puede condicionar o reemplazar decisiones; los ADR se enlazan |

## Fuente de autoridad

El material del profesor en `docs/class-material/` define qué se entrega, con qué formato y con qué rúbrica. Léelo antes de empezar un bloque. Si contradice algo ya escrito en el proyecto, prevalece el material.

Esos archivos son widgets HTML interactivos: casi todo el contenido está en constantes JavaScript dentro de `<script>`, no en el texto visible. Lee los datos del script, no solo el HTML renderizado.

## Estructura

| Ruta | Contenido |
|------|-----------|
| `README.md` | Índice del proyecto |
| `docs/class-material/` | Material del profesor, con el bloque en el nombre del archivo |
| `bloque-N/` | Entregables de un bloque: `NN-nombre.md` por fase y la presentación HTML |

Cada bloque nuevo empieza con `bloque-N/00-plan-de-trabajo.md`, derivado del material de clase: qué se entrega, fases, criterio de terminado por fase y puntos de control. `bloque-2/00-plan-de-trabajo.md` es el modelo.

## Flujo de trabajo

1. **Markdown primero.** El contenido se redacta y se valida en los markdown por fase. La presentación HTML se integra al final a partir de ellos y nunca es la fuente.
2. **Una fase por archivo.** Cada fase declara al inicio sus insumos (las fases anteriores) y cierra con una sección de verificación y un punto de control.
3. **Punto de control.** Al cerrar una fase, detente y espera la validación del equipo. No empieces una fase sobre otra sin validar.
4. **La rúbrica es el criterio de terminado.** Cada fase indica qué criterio alimenta y debe alcanzar el nivel "Logrado".

## Rol de Claude

Propone, justifica y redacta el entregable completo de cada fase.

- Deja el razonamiento escrito: el equipo sustenta el trabajo ante el profesor y debe poder defender cada decisión.
- Marca como **decisión del equipo** lo que sea discutible (puntajes, selección de escenarios, elección entre opciones) y preséntalo con variantes en el punto de control.
- No elijas por el equipo en silencio ni cambies texto ya aprobado sin señalarlo.

## Reglas del contenido

Estas reglas cruzan varios archivos. Romper una rompe la cadena completa.

| Regla | Detalle |
|-------|---------|
| Toda cifra tiene origen | Cada número se marca **[Fuente]** (`F#`, definidas en `01`) o **[Supuesto]** (`S#`, definidos en `02` con su razón). Una cifra nueva en un escenario o ADR exige agregar su supuesto a la ficha |
| Mecanismos, no marcas | Fichas, escenarios y opciones de ADR describen lo que el sistema hace, sin nombres de tecnologías ni productos. Los nombres de tácticas entran solo cuando el Bloque 3 las introduce |
| El escenario es el Context | El Context de cada ADR es su escenario copiado tal cual. Si cambia el escenario, cambia el Context y la presentación |
| Los ADR no se reescriben | Un ADR cambia de estado (Propuesto, Aceptado, reemplazado por otro) y se enlaza; no se borra ni se edita para cambiar la historia |
| Un ADR por escenario crítico | Es crítico si su atributo dominante es alto y cumplirlo le cuesta a otro atributo alto. Un escenario por tensión, no por atributo |
| Presupuesto de 18 puntos | Ocho atributos ISO 25010, de 1 a 5, suma exacta 18, máximo tres atributos en 4 o más. Reparto adoptado: Seguridad 4, Fiabilidad 4, Compatibilidad 3, Desempeño 3 y el resto en 1 |
| Identificadores estables | `ESC-NN`, `ADR-NN`, `T#` (tensiones fuertes), `L#` (leves), `S#`, `F#`, `H#` (hallazgos). No se renumeran: un elemento que cambia de categoría recibe un identificador nuevo y los demás conservan el suyo |
| Vocabulario ISO 25010 | Se usan las ocho características del widget del curso. La tabla de equivalencias con los términos del taller GPSMetro está en `bloque-2/00-plan-de-trabajo.md` §5 |
| Alcance | Solo juegos de casino de proveedores externos. Fuera: apuestas deportivas, juegos propios, póker entre jugadores y bingo |

## Cuando algo cambia

Un cambio se propaga en el orden de la cadena: investigación → ficha → tensiones → priorización → selección → escenarios → ADR → presentación HTML.

1. Actualiza cada archivo afectado, no solo el que originó el cambio.
2. Si la fase ya tenía el punto de control cerrado, registra el cambio con fecha y la lista de archivos tocados, como en `bloque-2/08-revision.md` §7.
3. Repite las verificaciones de la sección siguiente.

## Verificación

No hay herramientas automáticas en el repositorio. Antes de dar una fase por terminada, comprueba y reporta:

- [ ] Las cifras derivadas se recalcularon y cuadran con la ficha.
- [ ] Cada cifra de un escenario o ADR se rastrea hasta una `F#` o un `S#`.
- [ ] El Context de cada ADR coincide con su escenario.
- [ ] La priorización suma 18 y respeta los límites.
- [ ] Ningún escenario nombra tácticas, tecnologías o marcas.
- [ ] La presentación HTML dice lo mismo que los markdown.

## La presentación HTML

`bloque-2/Casino Andino.html` es un único archivo autocontenido; su única dependencia externa son las fuentes de Google. El contenido vive en constantes JavaScript (`ATTRS`, `TENS`, `SC`, `ADR`, `SUPS`, `SRCS`, entre otras) y el marcado solo lo pinta. Para cambiar contenido, edita esas constantes.

## Git

No se trabaja con ramas: todo va directo a `main`, en commits organizados.

- Un commit por unidad de trabajo: una fase, una corrección o un cambio de estructura. No se mezclan en el mismo commit.
- Un cambio que se propaga por la cadena va completo en un solo commit, con todos los archivos que toca.
- Mensajes en formato Conventional Commits, en inglés, que digan el resultado y no la lista de archivos.

## Idioma

Los entregables se escriben en español neutro y profesional. Los nombres de las secciones del ADR quedan en inglés, como en el material del curso: Context, Decision, Consequences.
