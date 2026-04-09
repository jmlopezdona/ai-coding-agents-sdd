# 2. El espectro de la especificación

Una de las cosas más útiles que aporta el paper *Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants* (arXiv) es que se niega a presentar el SDD como una cosa única. Lo presenta como un **espectro de tres niveles de compromiso**, y entender en cuál de los tres niveles operas — y en cuál crees que operas — es probablemente la decisión más consecuente que vas a tomar en este curso.

> *La spec declara la intención. El código la realiza.*
> — Spec-Driven Development paper, arXiv

Esta es la frase tesis del paper, y vale la pena tenerla delante mientras leemos los tres niveles.

## Nivel 1 — Spec-First

El nivel más ligero. Escribes una especificación **antes** de empezar a codificar, se la pasas al agente, el agente genera el código, y a partir de ahí la spec **se abandona**. El código toma vida propia. Cualquier cambio futuro se hace directamente sobre el código y la spec se queda fosilizada en su versión inicial.

Es el modo en que la mayoría de equipos creen que están haciendo SDD. "Sí, escribimos un design doc antes de empezar la feature." Vale, pero ese doc es solo *spec-first*. No es nada más. Sirve para una cosa concreta — arrancar una feature con una intención clara — y para nada más.

**Cuándo funciona:**

- Features nuevas con un alcance definido y un horizonte corto.
- Prototipos donde el código es desechable y la spec no necesita seguir viva.
- Equipos pequeños donde el conocimiento del cambio cabe en la cabeza de quien lo hace.

**Cuándo se rompe:**

- En cuanto el código entra en mantenimiento.
- En cuanto otra persona (o agente) tiene que tocarlo seis meses después y la spec ya no describe la realidad.
- En cuanto la cantidad de drift entre spec y código es tal que la spec engaña en lugar de informar.

El error más común con spec-first es **creer que es spec-anchored**. Un equipo escribe specs al principio de cada feature y cree que tiene SDD. No lo tiene. Tiene un ritual de arranque.

## Nivel 2 — Spec-Anchored

El nivel intermedio, y donde viven la mayoría de los equipos serios que adoptan SDD bien. Spec y código **evolucionan a la par**, como compañeros iguales, y existe algún mecanismo automático que detecta cuando se desincronizan. Tests que verifican comportamientos descritos en la spec, BDD frameworks tipo Cucumber, validadores que comparan firmas declaradas en la spec con firmas reales en el código, o agentes recurrentes que detectan drift entre uno y otro.

Lo que distingue spec-anchored de spec-first no es la calidad de la spec inicial. Es que **hay un coste tangible asociado a dejar que la spec se desactualice**. Si el test falla, alguien tiene que decidir: ¿actualizo el código para que cumpla la spec, o actualizo la spec para reflejar lo que el código ahora hace? Esa decisión, hecha cien veces, es lo que mantiene la spec viva.

El paper dice algo importante sobre este nivel:

> *BDD frameworks like Cucumber exemplify this approach, making it the practical choice for most production systems.*

Es una pista honesta. Spec-anchored no es una invención nueva del SDD. Es BDD bien hecho, llevado al nivel de la intención arquitectónica en lugar del comportamiento de cara al usuario. Si tu equipo ya practica BDD con disciplina, está más cerca de SDD del que cree.

**Cuándo funciona:**

- Sistemas en producción con vida larga.
- Equipos donde el código pasa por varias manos (incluidas manos artificiales).
- Dominios donde los invariantes son importantes y no negociables.

**Cuándo se rompe:**

- Cuando el coste de mantener spec y código sincronizados se vuelve mayor que el beneficio. Esto es real y el capítulo 9 lo desarrolla con honestidad.
- Cuando los tests automáticos no cubren lo suficiente como para cumplir su rol de "anclaje".

## Nivel 3 — Spec-as-Source

El nivel más radical y, hoy por hoy, el menos viable en la mayoría de contextos. La spec **es** el código fuente. Los humanos solo editan la spec. El código se genera a partir de ella, siempre, y nunca se edita directamente. Si algo está mal en el código, se arregla la spec y se regenera.

Tessl es el ejemplo más visible de esta filosofía hoy: genera código con un comentario `// GENERATED FROM SPEC - DO NOT EDIT` al principio de cada archivo. La promesa es que vuelves a tener una sola fuente de verdad, como en compilación clásica.

El paper es claro sobre dónde funciona esto realmente:

> *Currently viable mainly in mature domains like automotive embedded systems using tools such as Simulink.*

**Simulink** es el ejemplo paradigmático y conviene entenderlo porque ilustra qué condiciones tiene que cumplir un dominio para que spec-as-source funcione de verdad. Es una herramienta de MathWorks (los mismos de MATLAB) en la que diseñas sistemas dibujando **diagramas de bloques visuales** — sensores, controladores, filtros, conexiones — y la herramienta **genera automáticamente código C/C++ embebido** a partir de ese diagrama. Los ingenieros editan el diagrama, no el código generado. Si hay un bug en el `.c`, se arregla el bloque y se regenera. Las industrias de automoción, aeroespacial y control industrial llevan décadas — desde los años 90 — produciendo firmware crítico de esta forma. No es teoría, es producción real a gran escala.

Lo que hace que Simulink funcione, y que es exactamente lo que falta en software de propósito general, son tres condiciones:

1. **Semántica formal estrecha**: cada bloque tiene un significado matemático preciso (una integral, un PID, un filtro Kalman). No hay ambigüedad sobre qué hace.
2. **Dominio acotado**: sistemas de control, dinámica física, procesamiento de señales. Problemas con leyes bien establecidas.
3. **Generación determinística**: el mismo diagrama produce el mismo código. Siempre.

Para software de propósito general — APIs web, frontends, scripts internos, todo lo que la mayoría de equipos hacen el 95% del tiempo — ninguna de las tres condiciones se cumple. Las specs están en lenguaje natural (ambigüedad), el dominio cambia con cada feature (no acotado), y los LLMs son no determinísticos por diseño. Por eso spec-as-source aplicado a este tipo de software sigue siendo más promesa que realidad. La razón fundamental es la misma que mató al Model-Driven Development en los años 2000, y el capítulo 9 vuelve sobre esto.

**Cuándo funciona:**

- Dominios maduros con semántica formal (embedded, control, hardware/software co-design).
- Capas finas de glue code donde el mapeo spec→código es trivial.

**Cuándo se rompe:**

- En la mayoría del software real. La traducción spec→código no es determinística cuando la spec está en lenguaje natural y el generador es un LLM, y ese no-determinismo es exactamente lo que rompe la promesa central del nivel.

## Spec-Anchored vs Spec-as-Source: la diferencia que más confunde

A primera vista parece que la diferencia entre los dos niveles es solo técnica — *"en los dos la spec es la fuente, ¿no?"*. A nivel filosófico sí. A nivel operacional, son modelos muy distintos, y entender la diferencia te ahorra elegir mal.

La distinción clave es **quién puede editar el código**:

- **En spec-anchored, el código se edita directamente** (por un humano o por un agente). Spec y código son dos artefactos paralelos, los dos se mantienen, los dos pueden modificarse directamente. Lo que el "anclaje" añade es un mecanismo automático (tests, validadores, agentes recurrentes) que **detecta cuándo se desincronizan**. Cuando hay drift, alguien decide: actualizo el código para que cumpla la spec, o actualizo la spec para reflejar lo que el código hace ahora. Las dos direcciones son válidas. Un dev (o un agente) puede arreglar un bug puntual editando directamente un archivo, y el sensor abrirá un issue para reconciliar la spec después.

- **En spec-as-source, el código nunca se edita directamente.** Está marcado como generado. Si hay un bug, no abres el `.c` y lo arreglas — tienes que cambiar la spec y regenerar. El código es **solo lectura por contrato**, como un binario compilado.

Esto tiene tres consecuencias prácticas que no son matices:

1. **No hay escape hatch en spec-as-source.** Si la generación produce algo que no encaja con lo que necesitas, no puedes "tocar dos líneas para arreglarlo". Tienes que volver a la spec, refinarla, regenerar todo. En spec-anchored sí puedes tocar las dos líneas y luego actualizar la spec con la decisión.

2. **Cambia quién puede contribuir.** En spec-anchored, cualquiera que sepa el lenguaje de programación puede contribuir. En spec-as-source, tienes que aprender el formato de la spec y entender cómo el generador la interpreta. Es un skill nuevo y obligatorio.

3. **El coste de los cambios pequeños sube.** Un fix trivial en spec-anchored es trivial. En spec-as-source, hasta el fix más pequeño tiene que pasar por "edita la spec, regenera, valida que el resto del código no cambió de forma indeseada". Es exactamente el problema que mató al MDD en los 2000.

Resumido en una tabla:

| | Spec-First | Spec-Anchored | Spec-as-Source |
|---|---|---|---|
| ¿Quién edita el código? | Humanos / agentes directamente | Humanos / agentes directamente | Nadie — se genera |
| ¿Dónde haces un cambio pequeño? | Directamente en el código (la spec está abandonada) | Spec o código, después sincronizas | Solo spec, regeneras |
| ¿Hay escape hatch para casos raros? | Siempre (no hay restricción) | Sí (edita el código) | No (refactoriza la spec) |
| Fuente única de verdad | Ninguna — el código deriva de facto | Dos artefactos sincronizados | Una sola, el resto deriva |
| Mecanismo de sincronización | Ninguno | Tests / validadores / agentes | Generación |
| Skill necesario | Programar (+ escribir una spec inicial) | Programar + escribir specs + diseñar el anclaje (tests, validadores) | Escribir specs formales (la programación queda absorbida) |

Y ahí está exactamente por qué Simulink funciona en su dominio — semántica formal estrecha = la generación es predecible y rara vez necesitas el escape hatch — y por qué Tessl tiene difícil replicarlo en software de propósito general: la generación con LLMs no es determinística, así que el escape hatch se necesita constantemente, y prohibirlo te deja atascado.

## Cómo saber en qué nivel estás de verdad

Una pregunta diagnóstica simple: **¿qué pasa cuando alguien cambia el código directamente sin tocar la spec?**

- Si la respuesta es "nada, la spec se queda como estaba" → estás en spec-first, aunque te llames SDD.
- Si la respuesta es "el CI falla, o un linter me avisa, o un test BDD se rompe" → estás en spec-anchored.
- Si la respuesta es "no se puede, el código está marcado como generado" → estás en spec-as-source.

Si no puedes responder con seguridad, la respuesta operativa es la primera. Estás en spec-first. No es un drama, pero no te cuentes la historia de que estás en otro nivel.

## La trampa del "queremos estar en spec-anchored"

La trampa más cara que vemos en equipos que adoptan SDD es esta: aspiran a spec-anchored, escriben specs muy detalladas como si lo estuvieran, pero **no invierten en el mecanismo de anclaje**. No hay tests que comparen spec y código. No hay validador. No hay agente recurrente. Solo hay una carpeta `specs/` con markdowns que poco a poco se desactualizan.

Eso es spec-first vestido de spec-anchored, y es peor que spec-first puro porque genera la falsa sensación de tener una fuente de verdad cuando lo que tienes es ficción documentada. El capítulo 9 vuelve sobre esto bajo el nombre de "ilusión de control".

La regla operativa: **no declares que haces spec-anchored hasta que tengas el mecanismo automático de anclaje funcionando**. Sin ese mecanismo, lo que tienes es spec-first con más papeleo.

## Qué nivel deberías elegir

No hay una respuesta universal, pero hay una recomendación honesta:

- **Empieza por spec-first, sin disimular**. Escribe specs antes de las features importantes. No te obligues a mantenerlas.
- **Promociona a spec-anchored solo lo que tenga vida larga**. Las partes del sistema que vayan a sobrevivir años, donde el coste del anclaje se amortiza. No lo apliques al código que sabes que será reemplazado en seis meses.
- **No vayas a spec-as-source** salvo que estés en uno de los dominios maduros donde funciona, o estés deliberadamente experimentando con Tessl en una superficie aislada.

Esta progresión imita lo que la industria ha hecho con los tests: empezamos sin tests, luego con tests donde duele, luego con TDD donde el código es crítico, y nunca con TDD universal salvo en proyectos donde el rigor lo justifica. SDD sigue la misma curva, y entender eso te ahorra dos años de overcommit a un nivel que no necesitas.

## Lo que viene a continuación

Una vez que sabes en qué nivel estás operando, la pregunta siguiente es muy concreta: *¿cómo se escribe una spec que de verdad sirva?* Ese es el contenido del **capítulo 3**. Vamos a hablar de qué partes tiene una buena spec, cuáles son las que de verdad mueven la aguja, y de los "por qués" — el ingrediente que la mayoría de specs olvidan y que es exactamente lo que las hace útiles a meses vista.
