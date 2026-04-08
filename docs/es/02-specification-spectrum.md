# 2. El espectro de la especificación

Una de las cosas más útiles que aporta el paper *Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants* (arXiv) es que se niega a presentar el SDD como una cosa única. Lo presenta como un **espectro de tres niveles de compromiso**, y entender en cuál de los tres operas — y en cuál crees que operas — es probablemente la decisión más consecuente que vas a tomar en este curso.

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

Traducido: en dominios donde la generación de código a partir de modelos lleva décadas funcionando porque los modelos tienen una semántica muy estrecha y muy bien entendida. Para software de propósito general — APIs web, frontends, scripts internos, todo lo que la mayoría de equipos hacen el 95% del tiempo — el spec-as-source sigue siendo más promesa que realidad. La razón fundamental es la misma que mató al Model-Driven Development en los años 2000, y el capítulo 9 vuelve sobre esto.

**Cuándo funciona:**
- Dominios maduros con semántica formal (embedded, control, hardware/software co-design).
- Capas finas de glue code donde el mapeo spec→código es trivial.

**Cuándo se rompe:**
- En la mayoría del software real. La traducción spec→código no es determinística cuando la spec está en lenguaje natural y el generador es un LLM, y ese no-determinismo es exactamente lo que rompe la promesa central del nivel.

## Cómo saber en qué nivel estás de verdad

Una pregunta diagnóstica simple: **¿qué pasa cuando alguien cambia el código directamente sin tocar la spec?**

- Si la respuesta es "nada, la spec se queda como estaba" → estás en spec-first, aunque te llames SDD.
- Si la respuesta es "el CI falla, o un linter me avisa, o un test BDD se rompe" → estás en spec-anchored.
- Si la respuesta es "no se puede, el código está marcado como generado" → estás en spec-as-source.

Si no puedes responder con seguridad, la respuesta operativa es la primera. Estás en spec-first. No es un drama, pero no te cuentes la historia de que estás en otro nivel.

## La trampa del "queremos estar en spec-anchored"

La trampa más cara que vemos en equipos que adoptan SDD es esta: aspiran a spec-anchored, escriben specs muy detalladas como si lo estuvieran, pero **no invierten en el mecanismo de anclaje**. No hay tests que comparen spec y código. No hay validador. No hay agente recurrente. Solo hay una carpeta `specs/` con markdowns que poco a poco se desactualizan.

Eso es spec-first vestido de spec-anchored, y es peor que spec-first puro porque genera la falsa sensación de tener una fuente de verdad cuando lo que tienes es ficción documentada. El capítulo 9 vuelve sobre esto bajo el nombre de "ilusión de control".

La regla operativa: **no declares que haces spec-anchored hasta que tengas el mecanismo automático de anclaje funcionando**. Sin ese mecanismo, lo que tienes es spec-first con más papeles.

## Qué nivel deberías elegir

No hay una respuesta universal, pero hay una recomendación honesta:

- **Empieza por spec-first, sin disimular**. Escribe specs antes de las features importantes. No te obligues a mantenerlas.
- **Promociona a spec-anchored solo lo que tenga vida larga**. Las partes del sistema que vayan a sobrevivir años, donde el coste del anclaje se amortiza. No lo apliques al código que sabes que será reemplazado en seis meses.
- **No vayas a spec-as-source** salvo que estés en uno de los dominios maduros donde funciona, o estés deliberadamente experimentando con Tessl en una superficie aislada.

Esta progresión imita lo que la industria ha hecho con los tests: empezamos sin tests, luego con tests donde duele, luego con TDD donde el código es crítico, y nunca con TDD universal salvo en proyectos donde el rigor lo justifica. SDD sigue la misma curva, y entender eso te ahorra dos años de overcommit a un nivel que no necesitas.

## Lo que viene a continuación

Una vez que sabes en qué nivel estás operando, la pregunta siguiente es muy concreta: *¿cómo se escribe una spec que de verdad sirva?* Ese es el contenido del **capítulo 3**. Vamos a hablar de qué partes tiene una buena spec, cuáles son las que de verdad mueven la aguja, y de los "por qués" — el ingrediente que la mayoría de specs olvidan y que es exactamente lo que las hace útiles a meses vista.
