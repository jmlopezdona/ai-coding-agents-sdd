# Spec-Driven Development: producir software con intención

Hay un momento, casi siempre en torno a la quinta o sexta sesión seria con un agente de codificación, en el que el truco deja de funcionar. Hasta ahí todo iba razonablemente: el agente entendía lo que pedías, generaba algo que se compilaba, lo iterabas dos o tres veces y salías con código que parecía sano. Y de repente algo se rompe. La feature funciona pero rompe otras tres que el agente nunca miró. El refactor es estructuralmente correcto pero ignora una convención no escrita del proyecto. La sesión siguiente empieza desde cero como si la anterior no hubiera existido. El agente "entiende" perfectamente y aún así produce algo subtilmente equivocado.

La reacción habitual es echarle la culpa al modelo, esperar al siguiente, o convencerse de que hay que escribir mejores prompts. Las tres son evasivas. El problema no está en el modelo y no se arregla con prompts mejores. El problema está en que **estás intentando que un agente ejecute una intención que nunca llegaste a especificar con claridad**, y lo estás haciendo en un medio que no perdona la ambigüedad. A esa práctica la comunidad la ha bautizado *vibe coding* — programar por intuición, prompt a prompt, sin estructura — y a la disciplina que intenta corregirla, *Spec-Driven Development*.

## Qué es exactamente SDD (y qué no)

SDD no es escribir mucha documentación. No es waterfall disfrazado. No es ceremonia de gestión de proyectos. SDD es una idea muy concreta:

> **La especificación de la intención — qué se construye, por qué, con qué restricciones y bajo qué criterios de aceptación — es el artefacto principal del trabajo. El código es la realización de esa especificación.**

Cambian dos cosas. Primero, **el orden**: especificas antes de pedirle al agente que codifique, no después. Segundo, **la persistencia**: la especificación queda en el repositorio, versionada, y se actualiza cuando el código evoluciona. Es la diferencia entre "le pongo un ticket al agente y veo qué sale" y "le doy un contrato con el que comparar lo que produce".

Lo que SDD no es: una promesa de que el agente acertará a la primera, una garantía de que el código generado será correcto, un sustituto de revisar lo que sale, ni un proceso que escala uniformemente a cualquier tarea — un bug fix de tres líneas y una migración de base de datos no piden la misma cantidad de spec, y forzar lo contrario es una de las cosas más rápidas de matar un equipo (capítulo 9).

## Por qué hace falta una disciplina

El argumento más limpio para SDD viene de aceptar dos hechos sobre cómo funcionan los agentes hoy.

El primero es el **context collapse**: cada sesión empieza con cero conocimiento del proyecto. Lo que el agente sabía ayer no lo sabe hoy. Las convenciones de nomenclatura, los patrones de manejo de errores, las decisiones arquitectónicas que el equipo discutió en Slack hace dos meses, todo eso es invisible al agente a menos que esté explícitamente escrito en algún sitio que el agente pueda leer. En un proyecto de 200 archivos, el conocimiento implícito es enorme, y el agente lo adivina a partir de su entrenamiento general — que casi nunca coincide con el tuyo.

El segundo es el **context drift**, que es el primo operacional del context collapse pero ocurre *dentro* de una sesión: el agente arregla un bug en un archivo y rompe otros tres que no miró. No es que se confunda; es que su ventana de atención no abarca el blast radius real del cambio. Cuanto más grande el repo, peor.

Una spec resuelve los dos problemas a la vez. Resuelve el collapse porque es persistente: la próxima sesión empieza con la spec leída. Y resuelve el drift porque convierte el cambio en un *contrato comparable*: si la spec dice "estos invariantes no se tocan", la verificación post-código tiene un punto de referencia.

Por eso "mejor prompt" no es solución. Un prompt mejor es una corrección efímera. Una spec es una corrección que persiste y que cualquier agente futuro puede usar.

## El cambio de rol: del programador al arquitecto de intenciones

Si aceptas SDD, tu trabajo cambia menos de lo que parece y más de lo que esperas. Sigues siendo ingeniero, pero el medio es otro:

- Antes pasabas la mayor parte del tiempo escribiendo código que ejecutaba una intención que tenías clara en la cabeza.
- Ahora pasas la mayor parte del tiempo **haciendo esa intención explícita** — restricciones, no-goals, criterios de aceptación, los "por qués" — para que un agente la pueda ejecutar.

Es un trabajo menos artesanal y más arquitectónico. Lo difícil deja de ser teclear rápido y empieza a ser pensar con precisión: *qué* quiero exactamente, *por qué* lo quiero así, *qué* no quiero, *cómo* sé que está bien. Cuando un equipo dice "el agente no entiende lo que le pido", casi siempre la traducción honesta es "yo no sabía exactamente qué le estaba pidiendo". La spec es la herramienta que fuerza esa precisión antes de que el agente tenga la oportunidad de adivinar.

## El espectro de la especificación, en una sola idea

Una de las contribuciones útiles del paper *Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants* (arXiv) es que no presenta el SDD como una cosa única, sino como un espectro de tres niveles de compromiso:

- **Spec-First**: escribes una spec antes de codificar; sirve para arrancar; luego el código diverge libremente.
- **Spec-Anchored**: spec y código evolucionan a la par; tests automáticos garantizan la alineación; es donde viven los equipos serios.
- **Spec-as-Source**: la spec *es* el código fuente; el código se regenera; los humanos solo editan la spec. Hoy solo es viable en dominios maduros (Simulink, embedded automotriz).

La mayoría de los equipos confunden lo que están haciendo: dicen "estamos haciendo SDD" cuando en realidad están haciendo *spec-first* aspiracional, que se degrada a *vibe coding* en cuanto el código tiene seis meses. El capítulo 2 desarrolla este espectro porque entender en qué nivel estás operando es la decisión más importante del curso.

## La crítica que también es parte del curso

Una guía honesta sobre SDD tiene que aceptar que la disciplina también puede salir mal. El capítulo 9 desarrolla las críticas más fuertes — el *maintenance tax* de Isoform, el paralelo con el viejo Model-Driven Development de Martin Fowler, la pérdida de los "por qués", la falsa ilusión de control — y el capítulo 10 presenta la alternativa que propone Isoform: *context engineering*, donde la intención y los "por qués" se preservan **dentro del código** en vez de en specs externas.

No están ahí para hacer ruido. Están porque mal aplicada esta disciplina es indistinguible de la burocracia, y si el curso solo te enseña los patrones positivos te va a dejar sin defensa contra el momento — que llegará — en el que tu propio proceso se convierta en el cuello de botella.

## Cómo encaja en la trilogía

Esta guía es la pieza intermedia entre dos cursos hermanos:

- **[Fundamentals](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/)** te enseñó qué es un agente, cómo se le habla, qué herramientas tiene. Es el "qué".
- **Spec-Driven Development** (este curso) te enseña a estructurar el trabajo para que el agente pueda ejecutarlo bien. Es el "cómo trabajar con él".
- **[Harness Engineering](https://jmlopezdona.github.io/ai-coding-agents-harness/)** te enseña a ingeniar el sistema alrededor del agente — sandboxes, sensores, hooks, observabilidad, el repo entero como sustrato. Es el "qué se construye alrededor".

SDD existe porque entre saber usar un agente y saber montarle un harness completo hay una disciplina intermedia que vale la pena aprender por separado. Sin SDD, el harness se convierte en infraestructura sin propósito. Sin harness, las specs se convierten en burocracia sin tracción. Los tres cursos forman una progresión, y cada uno se apoya en el anterior.

## Por dónde seguir

Si quieres entender el dolor concreto que SDD intenta resolver, salta al **capítulo 1**. Si ya lo tienes claro y quieres el modelo conceptual, empieza por el **capítulo 2** (el espectro). Si vienes a por herramientas, los capítulos **6 y 7** son tu ruta corta. Y si vienes a buscar criterio para no tragarte el hype, lee primero el **capítulo 9** y vuelve al principio con otros ojos.

Lo que no hace falta es leerla en orden de tapa a tapa. Lo que sí hace falta es no quedarse solo con los capítulos cómodos.

---

*Este texto sintetiza ideas del paper de arXiv sobre SDD, los artículos de Addy Osmani y Augment Code sobre specs vivas y boundaries, los análisis críticos de Isoform y Martin Fowler (Thoughtworks) sobre los límites de SDD, y la discusión de la comunidad alrededor de herramientas como Kiro, Spec-kit, Tessl, BMAD y Traycer. URLs en la [sección de fuentes del índice](index.md#fuentes).*
