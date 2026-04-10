# 5. Los ciclos SDD: dos variantes y cuándo usar cada una

Si lees tres artículos sobre Spec-Driven Development, encontrarás tres versiones distintas del "ciclo SDD". Eso no es un fallo de la disciplina; es señal de que la disciplina aún se está formando. Pero deja al lector con un problema concreto: *¿qué fases hay, en qué orden, y por qué dos fuentes serias dicen cosas distintas?*

Este capítulo desambigua. Hay **dos variantes principales** del ciclo SDD en la literatura, no son intercambiables, y la elección entre ellas dice algo sobre el tipo de proyecto en el que estás.

## Variante A — La del paper de arXiv

El paper de arXiv sobre SDD propone un ciclo de cuatro fases:

1. **Specify** — definir qué tiene que hacer el software a través de descripciones de comportamiento, criterios de aceptación y requisitos. Sin prescribir cómo se implementa.
2. **Plan** — decidir cómo se construye: tecnologías, arquitectura, modelos de datos, interfaces.
3. **Implement** — producir código que realiza la spec según el plan, en incrementos pequeños y validados.
4. **Validate** — verificar que el código realmente cumple la spec mediante tests automáticos, escenarios BDD, aceptación de stakeholders.

Lo característico de esta variante es que la **validación es una fase explícita y separada**. La verificación no es algo que pasa "al final si hay tiempo"; es uno de los cuatro vértices del ciclo, con el mismo peso que la implementación.

## Variante B — La del artículo de DEV.to (Pockit)

El artículo más práctico sobre cómo usar SDD en el día a día propone un ciclo distinto, también de cuatro fases, pero con nombres y fronteras diferentes:

1. **Requirements** — qué se construye, incluyendo user stories, casos límite, requisitos de seguridad y *no-goals* explícitos.
2. **Design** — traducir requisitos en decisiones técnicas: schema de base de datos, endpoints, arquitectura, error handling.
3. **Tasks** — descomponer el diseño en pasos discretos, ordenados por dependencias, lo bastante pequeños para una sesión enfocada del agente, cada uno con tests.
4. **Implementation** — el agente escribe código tarea a tarea, guiado por el contexto completo de las fases anteriores, con TDD verificando cada paso.

La diferencia importante con la variante A es que aquí **la validación está embebida en la fase de tareas** (cada tarea trae sus tests) y no aparece como fase autónoma. A cambio, hay una fase explícita — *Tasks* — que en la variante A no existe como tal y vive escondida dentro de "Plan".

## ¿Cuál es la "correcta"?

Ninguna. Son dos puntos de vista distintos sobre el mismo proceso, optimizados para situaciones distintas:

- **La variante A (Specify → Plan → Implement → Validate) es mejor cuando el sistema tiene invariantes fuertes que necesitan validación explícita y formal.** APIs públicas, sistemas regulados, código que cruza fronteras de equipos. Si la verificación es un evento social y técnico que merece su propia fase, esta es tu variante.

- **La variante B (Requirements → Design → Tasks → Implementation) es mejor cuando el cuello de botella es la descomposición en trozos digeribles para el agente.** Features grandes que un humano podría hacer pero un agente sin ayuda no, porque el blast radius de los cambios excede su ventana de atención. Si tu problema es "el agente se pierde en mi repo de 200 archivos", esta es tu variante.

En la práctica, los equipos maduros acaban combinando: usan los nombres de la variante B en el flujo diario y separan una fase de validación (variante A) cuando la feature toca algo crítico. No es contradictorio. Es elegir la herramienta adecuada al riesgo del cambio.

## El ciclo, paso a paso, con un agente real

Te lo cuento como flujo concreto, neutral respecto a la variante, que es cómo se ve cuando lo haces de verdad.

### Paso 1 — Especificar la intención

Abres una sesión con el agente. **No le pides que escriba código**. Le pides que te ayude a redactar la spec usando la plantilla del capítulo 3. Le das el objetivo de alto nivel y dejas que te haga preguntas. Si el agente no te hace preguntas, hazlas tú: "qué partes del sistema toca esto", "qué casos límite estoy olvidando", "qué no estamos construyendo y deberíamos decir explícitamente".

El output de este paso es un archivo en `specs/` o donde tu proyecto los aloje. Cometido al repo, revisable, versionado.

### Paso 2 — Planificar la implementación

Con la spec en mano, le pides al agente un **plan de implementación**: qué archivos toca, en qué orden, qué dependencias hay entre tareas, qué tests pretende escribir. El plan no es código; es un documento intermedio. Lo lees, lo discutes, lo corriges si el agente ha entendido mal alguna restricción.

Este paso es donde las herramientas tipo Traycer aportan más valor (capítulo 8), porque la calidad del plan determina la calidad de todo lo que viene después.

### Paso 3 — Descomponer en tareas

Tomas el plan y lo cortas en **tareas pequeñas, secuenciales, cada una con un criterio de "hecho"**. Una tarea típica debería caber en una sola sesión enfocada del agente, sin ambigüedad. Si una tarea necesita más de una sesión, está mal cortada.

La regla heurística: si no puedes describir el estado "después de la tarea" en una frase, la tarea es demasiado grande.

### Paso 4 — Implementar tarea a tarea

El agente ejecuta las tareas de una en una. Para cada tarea: lee la spec, lee la tarea, escribe los tests que la tarea pide, escribe el código, ejecuta los tests, verifica. Si los tests no pasan, itera. Si pasan, marca la tarea como hecha y pasa a la siguiente.

Lo importante aquí es que **cada tarea es atomic respecto a la spec**: o se cumple completamente o se revierte. No hay tareas a medias que "luego se arreglan en otra".

### Paso 5 — Validar contra la spec original

Cuando todas las tareas están hechas, el agente (o tú, o un segundo agente revisor) toma la spec original y verifica que todo lo que la spec pedía está hecho, y que no se ha hecho nada que la spec no pedía. Esta fase es la que más fácilmente se salta y la más importante: es donde los criterios de aceptación dejan de ser texto y se convierten en una checklist tachada o no tachada.

## Clarificación iterativa: lo que el survey de arXiv añade

Hay un detalle que el survey de *Code Generation with LLM-based Agents* documenta y que merece estar en este ciclo aunque no aparezca explícitamente en ninguna de las dos variantes anteriores. Sistemas como **ClarifyGPT** y **TiCoder** introducen una fase **previa** a la spec: el agente, en lugar de tomar el primer prompt como verdad, hace preguntas iterativas para sacar a la superficie ambigüedades y huecos antes de que se materialicen como código equivocado.

En la práctica, esto significa que tu fase 1 del ciclo (especificar) **no es lineal**. Es un mini-bucle dentro del bucle: prompt → preguntas del agente → respuestas → spec borrador → más preguntas → spec final. Los equipos que tratan la fase 1 como un ping-pong en lugar de un dictado obtienen specs sustancialmente mejores, y esto es independiente de qué herramienta uses.

## El ciclo no es lineal

Una nota final que casi siempre se omite en las descripciones canónicas: **el ciclo es raramente lineal en la práctica**. Lo que pasa de verdad es que en el paso 4 descubres algo que invalida una asunción del paso 1, y vuelves atrás. El paso 5 te enseña que el criterio que escribiste no era verificable y tienes que reescribirlo. A mitad de una tarea, te das cuenta de que la spec tiene un hueco.

Esto es **normal y deseable**. La diferencia entre un ciclo sano y uno patológico no es que el primero no tenga retrocesos; es que el sano **actualiza la spec cuando descubre que estaba mal** en lugar de ignorarla y seguir codificando. Esa es exactamente la idea del próximo capítulo: las **especificaciones vivas**.

## Lo que viene a continuación

El capítulo 6 va sobre la diferencia entre specs estáticas (las que envejecen mal) y specs vivas (las que se mantienen útiles porque la implementación retroalimenta la spec). Es la pieza que convierte el ciclo de este capítulo en un proceso sostenible en lugar de un ritual de arranque.
