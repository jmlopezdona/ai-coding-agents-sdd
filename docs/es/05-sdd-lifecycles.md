# 5. Posibles enfoques del ciclo de vida

Si lees tres artículos sobre Spec-Driven Development, encontrarás tres versiones distintas del "ciclo de vida SDD". Eso no es un fallo de la disciplina; es señal de que la disciplina aún se está formando. Pero deja al lector con un problema concreto: *¿qué fases hay, en qué orden, y por qué dos fuentes serias dicen cosas distintas?*

Este capítulo desambigua. Hay **dos enfoques principales** del ciclo de vida SDD en la literatura, no son intercambiables, y la elección entre ellos dice algo sobre el tipo de proyecto en el que estás.

## Enfoque A

[El paper de arXiv sobre SDD](https://arxiv.org/html/2602.00180) propone un ciclo de cuatro fases:

1. **Specify** — definir qué tiene que hacer el software a través de descripciones de comportamiento, criterios de aceptación y requisitos. Sin prescribir cómo se implementa.
2. **Plan** — decidir cómo se construye: tecnologías, arquitectura, modelos de datos, interfaces.
3. **Implement** — producir código que realiza la spec según el plan, en incrementos pequeños y validados.
4. **Validate** — verificar que el código realmente cumple la spec mediante tests automáticos, escenarios BDD, aceptación de stakeholders.

Lo característico de este enfoque es que la **validación es una fase explícita y separada**. La verificación no es algo que pasa "al final si hay tiempo"; es uno de los cuatro vértices del ciclo, con el mismo peso que la implementación.

## Enfoque B

[El artículo más práctico sobre cómo usar SDD en el día a día](https://dev.to/pockit_tools/specification-driven-development-how-to-stop-vibe-coding-and-actually-ship-production-ready-5788) propone un ciclo distinto, también de cuatro fases, pero con nombres y fronteras diferentes:

1. **Requirements** — qué se construye, incluyendo user stories, casos límite, requisitos de seguridad y *no-goals* explícitos.
2. **Design** — traducir requisitos en decisiones técnicas: schema de base de datos, endpoints, arquitectura, error handling.
3. **Tasks** — descomponer el diseño en pasos discretos, ordenados por dependencias, lo bastante pequeños para una sesión enfocada del agente, cada uno con tests.
4. **Implementation** — el agente escribe código tarea a tarea, guiado por el contexto completo de las fases anteriores, con TDD verificando cada paso.

La diferencia importante con el enfoque A es que aquí **la validación está embebida en la fase de tareas** (cada tarea trae sus tests) y no aparece como fase autónoma. A cambio, hay una fase explícita — *Tasks* — que en el enfoque A no existe como tal y vive escondida dentro de "Plan".

## ¿Cuál es la "correcta"?

Ninguna. Son dos puntos de vista distintos sobre el mismo proceso, optimizados para situaciones distintas:

- **El enfoque A (Specify → Plan → Implement → Validate) es mejor cuando el sistema tiene invariantes fuertes que necesitan validación explícita y formal.** APIs públicas, sistemas regulados, código que cruza fronteras de equipos. Si la verificación es un evento social y técnico que merece su propia fase, este es tu enfoque.

- **El enfoque B (Requirements → Design → Tasks → Implementation) es mejor cuando el cuello de botella es la descomposición en trozos digeribles para el agente.** Features grandes que un humano podría hacer pero un agente sin ayuda no, porque el blast radius de los cambios excede su ventana de atención. Si tu problema es "el agente se pierde en mi repo de 200 archivos", este es tu enfoque.

En la práctica, los equipos maduros acaban combinando: usan los nombres del enfoque B en el flujo diario y separan una fase de validación (enfoque A) cuando la feature toca algo crítico. No es contradictorio. Es elegir la herramienta adecuada al riesgo del cambio.

## Las tres fases del ciclo de vida

Tanto el enfoque A como el B — y todos los frameworks del [capítulo 7](07-native-sdd-tools.md) — hacen las mismas tres cosas, aunque las nombren distinto y las distribuyan de formas diferentes. Estas tres fases son el modelo general del ciclo de vida SDD:

| Fase | Qué se hace | Enfoque A | Enfoque B |
|---|---|---|---|
| **1. Diseñar la spec** | Todo lo que ocurre antes de escribir código: especificar intención, planificar, descomponer en tareas | Specify + Plan | Requirements + Design + Tasks |
| **2. Implementar la spec** | Escribir código guiado por lo que la fase 1 produjo | Implement | Implementation |
| **3. Validar la spec** | Verificar que el código cumple lo que la spec pedía — y nada más | Validate | *(embebida en cada tarea)* |

Las herramientas nativas implementan estas mismas tres fases de formas distintas:

| Framework | Fase 1: Diseñar | Fase 2: Implementar | Fase 3: Validar |
|---|---|---|---|
| [**Kiro**](07-native-sdd-tools.md#kiro) | 3 documentos: Requirements → Design → Tasks | Agente ejecuta tasks | Hooks on-save por tarea |
| [**Spec-kit**](07-native-sdd-tools.md#spec-kit-github) | Constitution + feature specs con checkpoints | Agente implementa | Aspira a anclaje, pero es spec-first en la práctica |
| [**Tessl**](07-native-sdd-tools.md#tessl) | Spec formal única (spec-as-source) | Genera código desde spec | Regeneración como reconciliación |
| [**BMAD**](07-native-sdd-tools.md#bmad) | PM (requirements) + Architect (design) + QA (test plan) | Developer agent | QA agent verifica |

Lo que esta tabla revela es importante:

- **La Fase 1 es donde vive la complejidad.** Puede ser un solo acto (escribir una spec breve) o tres documentos con tres revisores distintos. La granularidad depende del contexto, no del método.
- **La Fase 2 es la más estable.** Todos hacen lo mismo: el agente escribe código guiado por lo que la Fase 1 produjo. No hay variación significativa aquí.
- **La Fase 3 es donde más divergen los frameworks** — y donde más fallan. El [capítulo 7](07-native-sdd-tools.md#lo-que-ninguna-de-estas-herramientas-resuelve) identifica la verificación post-código como la grieta que ninguna herramienta nativa resuelve completamente. Unos la hacen explícita (enfoque A), otros la embeben en cada tarea (enfoque B, Kiro), otros apenas la implementan (Spec-kit), y otros la delegan a un agente especializado (BMAD). Que la validación exista como fase con peso propio — no como algo opcional que se hace "si hay tiempo" — es lo que separa SDD de vibe coding con documentación.

## Fase 1 — Diseñar la spec

Esta fase cubre **todo lo que ocurre antes de escribir código**. Es la fase más variable en granularidad: en un proyecto simple puede ser un solo paso; en un proyecto enterprise con stakeholders y documentación previa puede desplegarse en varios sub-pasos con sus propios artefactos y revisiones.

!!! warning "Qué significa «spec» en este ciclo de vida"

    Cuando este capítulo dice *"la spec"*, no se refiere necesariamente a un único documento. Se refiere a **todo lo que el agente necesita para tener completitud y rigor**: qué implementar, qué no implementar, por qué, por qué no, y cómo validarlo.

    En una PoC, un prototipo o un proyecto pequeño greenfield, esa información cabe en un solo archivo — la spec del capítulo 3 es suficiente. Pero en un proyecto enterprise, brownfield o de cierta complejidad, la spec **es la suma** del documento SDD más la documentación funcional y técnica upstream que lo alimenta: requisitos validados por el cliente, decisiones arquitectónicas, contratos de API, reglas de negocio documentadas.

    Si esa documentación upstream no existe o no tiene el rigor necesario, el primer paso no es escribir la spec — es **conseguir ese upstream**. Saltar este paso y confiar en que el agente derivará los requisitos a partir de una conversación informal es exactamente cómo se cae en la falta de rigor que SDD pretende evitar. El capítulo 4 detalla [cómo consumir, producir y referenciar esos artefactos](04-spec-in-context.md#la-spec-y-sus-fuentes-externas-consumir-producir-modificar).

### Un documento o varios: dos estrategias

Los sub-pasos de esta fase producen artefactos — pero **¿viven en un solo archivo o en varios?** La respuesta depende de la escala del proyecto y de si existe documentación upstream previa.

**Documento integrado** — Los seis bloques del capítulo 3 más el plan y las tareas conviven en un solo archivo. Es la opción natural para PoCs, prototipos, proyectos greenfield y cambios pequeños o medianos. Su ventaja principal: el [bucle bidireccional del capítulo 6](06-living-specs.md) actualiza un solo sitio, y no hay riesgo de drift entre artefactos.

**Artefactos separados** — Cada sub-paso produce su propio documento: requisitos, diseño técnico, lista de tareas. Es el modelo que [Kiro](07-native-sdd-tools.md#kiro) implementa nativamente con su flujo **Requirements → Design → Tasks**, y el que encaja con equipos enterprise donde distintas personas revisan distintas capas (producto valida requisitos, arquitectura valida diseño, el equipo valida tareas). La spec del capítulo 3 actúa como **documento maestro** que referencia los demás sin duplicarlos — exactamente la regla de [consumir sin reimplementar](04-spec-in-context.md#tres-relaciones-tres-reglas) del capítulo 4.

El coste de los artefactos separados no es trivial: cada documento adicional es una superficie de drift. [Spec-kit](07-native-sdd-tools.md#spec-kit-github) sufrió exactamente esto — Fowler observó que los archivos generados eran *"más pesados de revisar que el propio código"*. Si eliges esta estrategia, necesitas disciplina o herramientas para mantener los artefactos sincronizados, y hoy [ninguna de las herramientas nativas lo resuelve completamente](07-native-sdd-tools.md#lo-que-ninguna-de-estas-herramientas-resuelve).

En ambos casos, la **"spec" conceptual es siempre una** — es la suma de todo lo que el agente necesita para implementar con rigor. Lo que cambia es cómo se distribuye físicamente. Y el factor decisivo suele ser la pregunta del capítulo 4: **¿existe documentación upstream validada?** Si sí, la spec la consume y referencia (artefactos separados tiene sentido). Si no, la spec tiene que contenerlo todo (documento integrado es más natural).

| Estrategia | Cuándo encaja | Framework de referencia | Riesgo principal |
|---|---|---|---|
| **Documento integrado** | PoC, greenfield, equipos pequeños, cambios medianos | Manual con plantilla del cap. 3 | Puede crecer demasiado en proyectos complejos |
| **Artefactos separados** | Enterprise, brownfield, stakeholders por capas, upstream validado | [Kiro](07-native-sdd-tools.md#kiro), [BMAD](07-native-sdd-tools.md#bmad) | Drift entre documentos; maintenance tax |

### Sub-paso 1.1 — Especificar la intención

Abres una sesión con el agente. **No le pides que escriba código**. Le pides que te ayude a redactar la spec usando la plantilla del capítulo 3, **alimentándola con la documentación upstream** que ya tengas — requisitos funcionales, criterios de aceptación, decisiones técnicas previas. Le das el objetivo de alto nivel y dejas que te haga preguntas. Si el agente no te hace preguntas, hazlas tú: "qué partes del sistema toca esto", "qué casos límite estoy olvidando", "qué no estamos construyendo y deberíamos decir explícitamente".

El output de este sub-paso es un archivo en `specs/` o donde tu proyecto los aloje. Cometido al repo, revisable, versionado.

!!! tip "Clarificación iterativa"

    El survey de [*Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) documenta que sistemas como **ClarifyGPT** y **TiCoder** introducen una fase de preguntas iterativas **antes** de fijar la spec: el agente, en lugar de tomar el primer prompt como verdad, saca a la superficie ambigüedades y huecos antes de que se materialicen como código equivocado.

    En la práctica, esto significa que especificar **no es lineal**. Es un mini-bucle: prompt → preguntas del agente → respuestas → spec borrador → más preguntas → spec final. Los equipos que tratan este sub-paso como un ping-pong en lugar de un dictado obtienen specs sustancialmente mejores.

    Esta clarificación conecta directamente con los **boundaries** del capítulo 3 — en particular con la categoría *"ask first"*: las preguntas que el agente debe hacer antes de actuar. La diferencia es que aquí las preguntas ocurren antes de que la spec exista, no después.

### Sub-paso 1.2 — Planificar la implementación

Con la spec en mano, le pides al agente un **plan de implementación**: qué archivos toca, en qué orden, qué dependencias hay entre tareas, qué tests pretende escribir. El plan no es código; es un documento intermedio. Lo lees, lo discutes, lo corriges si el agente ha entendido mal alguna restricción.

Este sub-paso es donde las herramientas tipo Traycer aportan más valor ([capítulo 8](08-architect-layers.md)), porque la calidad del plan determina la calidad de todo lo que viene después.

### Sub-paso 1.3 — Descomponer en tareas

Tomas el plan y lo cortas en **tareas pequeñas, secuenciales, cada una con un criterio de "hecho"**. Una tarea típica debería caber en una sola sesión enfocada del agente, sin ambigüedad. Si una tarea necesita más de una sesión, está mal cortada.

La regla heurística: si no puedes describir el estado "después de la tarea" en una frase, la tarea es demasiado grande.

!!! note "No todos los proyectos necesitan los tres sub-pasos"

    En un cambio pequeño o una PoC, los sub-pasos 1.2 y 1.3 pueden ser implícitos o inexistentes — la spec misma ya es el plan y las tareas son obvias. La Fase 1 completa puede durar cinco minutos. En un proyecto enterprise con múltiples stakeholders, cada sub-paso puede tener su propio ciclo de revisión y aprobación. **El peso de la Fase 1 debe ser proporcional al riesgo del cambio** — exactamente la regla del [capítulo 9](09-patterns-of-application.md).

## Fase 2 — Implementar la spec

El agente ejecuta las tareas de una en una. Para cada tarea: lee la spec, lee la tarea, escribe los tests que la tarea pide, escribe el código, ejecuta los tests, verifica. Si los tests no pasan, itera. Si pasan, marca la tarea como hecha y pasa a la siguiente.

Lo importante aquí es que **cada tarea es atómica respecto a la spec**: o se cumple completamente o se revierte. No hay tareas a medias que "luego se arreglan en otra".

Esta es la fase más estable del ciclo — todos los frameworks la implementan de forma similar. Lo que varía es el nivel de autonomía del agente (desde un agente general con la spec como contexto, hasta [BMAD](07-native-sdd-tools.md#bmad) con un agente Developer especializado) y si la validación ocurre dentro de cada tarea (enfoque B, [Kiro](07-native-sdd-tools.md#kiro)) o se difiere a la Fase 3.

## Fase 3 — Validar la spec

Cuando todas las tareas están hechas, alguien — el agente, tú, o un segundo agente revisor — toma la spec original y verifica que **todo lo que la spec pedía está hecho, y que no se ha hecho nada que la spec no pedía**. Esta fase es la que más fácilmente se salta y la más importante: es donde los criterios de aceptación dejan de ser texto y se convierten en una checklist tachada o no tachada.

Es también la fase donde **más divergen los frameworks** — y donde más fallan:

- **Validación explícita y separada** (enfoque A): la verificación es un evento propio con peso en el proceso. Es la opción más rigurosa, apropiada cuando hay invariantes fuertes, APIs públicas o código que cruza fronteras de equipos.
- **Validación embebida en cada tarea** (enfoque B, [Kiro](07-native-sdd-tools.md#kiro)): cada tarea trae sus tests y se auto-valida al completarse. Más ágil, pero sin una verificación final se pierde la visión global — que cada tarea pase sus tests no garantiza que el conjunto cumpla la spec.
- **Validación aspiracional** ([Spec-kit](07-native-sdd-tools.md#spec-kit-github)): aspira a spec-anchored pero en la práctica no tiene un mecanismo automático que compare spec contra código. Es spec-first con disfraz.
- **Validación por regeneración** ([Tessl](07-native-sdd-tools.md#tessl)): si regeneras código desde la spec y obtienes lo mismo, "valida". Pero el no-determinismo de los LLMs entra en conflicto directo con esta promesa.
- **Validación delegada a un rol** ([BMAD](07-native-sdd-tools.md#bmad)): un agente QA especializado verifica. La especialización ayuda con el foco, pero los fallos de coordinación entre agentes son sutiles.

!!! warning "La Fase 3 es lo que separa SDD de vibe coding con documentación"

    Si la Fase 3 no existe o se reduce a "confiar en que los tests pasan", no estás haciendo SDD — estás haciendo spec-first con buenas intenciones. La verificación de que el código cumple la spec **como un todo** (no solo tarea a tarea) es el mecanismo que cierra el ciclo. Sin ella, la spec envejece desde el momento en que se escribe.

    Esta es exactamente la grieta que el [capítulo 7 identifica como problema común](07-native-sdd-tools.md#lo-que-ninguna-de-estas-herramientas-resuelve) de las herramientas nativas, y donde las capas de arquitecto del [capítulo 8](08-architect-layers.md) se han posicionado.

## El ciclo de vida no es lineal

Una nota que casi siempre se omite en las descripciones canónicas: **el ciclo es raramente lineal en la práctica**. Lo que pasa de verdad es que en la Fase 2 descubres algo que invalida una asunción de la Fase 1, y vuelves atrás. La Fase 3 te enseña que un criterio no era verificable y tienes que reescribirlo. A mitad de una tarea, te das cuenta de que la spec tiene un hueco.

Esto es **normal y deseable**. La diferencia entre un ciclo sano y uno patológico no es que el primero no tenga retrocesos; es que el sano **actualiza la spec cuando descubre que estaba mal** en lugar de ignorarla y seguir codificando. Esa retroalimentación de la Fase 3 hacia la Fase 1 es exactamente la idea del próximo capítulo: las **especificaciones vivas**.

## Lo que viene a continuación

El capítulo 6 va sobre la diferencia entre specs estáticas (las que envejecen mal) y specs vivas (las que se mantienen útiles porque la implementación retroalimenta la spec). Es la pieza que convierte el ciclo de vida descrito en este capítulo en un proceso sostenible en lugar de un ritual de arranque.
