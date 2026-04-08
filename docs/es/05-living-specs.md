# 5. Especificaciones vivas: el bucle bidireccional

El problema que mata a la mayoría de iniciativas de Spec-Driven Development no es escribir las specs. Es lo que pasa **después**. Las specs envejecen, el código las deja atrás, y dentro de tres meses tienes una carpeta de markdowns que describe un sistema que ya no existe. Ese es exactamente el patrón que la gente de Augment Code llama el **spec gap**, y todo este capítulo es sobre cómo evitarlo.

## El problema del flujo unidireccional

Una spec estática funciona así:

1. El humano escribe la spec.
2. El agente la lee y produce código.
3. El código evoluciona, se modifica, se refactoriza.
4. La spec se queda donde estaba.

La información fluye en una sola dirección: spec → código. Y eso significa que cualquier decisión que se tome durante la implementación — y siempre se toman muchas: trade-offs, descubrimientos, casos límite que aparecen al teclear — **no vuelve a la spec**. La spec describe la intención inicial; el código describe la realidad actual; entre ambos hay una grieta que crece con cada commit.

Augment lo dice con una precisión incómoda:

> *Gaps in the specification widen with direct code changes and keep resurfacing because AI generation is non-deterministic.*

Es decir: **el problema no solo es que la spec envejezca, es que la generación con LLMs es no determinística**, así que cada vez que regeneras código a partir de una spec desactualizada, introduces inconsistencias nuevas. El gap no solo crece — cada ciclo de regeneración lo amplifica.

## Qué es una spec viva

Una spec viva (*living spec*) es una spec que se actualiza cuando el código cambia. Esa es la definición corta. La definición operativa es algo más exigente:

> Una spec viva tiene un **bucle de retroalimentación bidireccional**: las decisiones tomadas durante la implementación se vuelven a escribir en la spec, de forma que la spec siempre describe el estado actual del sistema, no solo su estado deseado inicial.

La diferencia con una spec estática no es la primera versión del documento. Las dos pueden empezar idénticas. La diferencia es lo que pasa el día 30, el día 90 y el día 365.

## Las cuatro fases del bucle vivo

El framework de Augment descompone el bucle bidireccional en cuatro fases:

### Fase 1 — Especificar la intención inicial

Idéntica a la fase 1 del ciclo del capítulo 4. Una spec mínima viable: objetivo, restricciones, criterios, no-goals, por qués. Nada nuevo aquí.

### Fase 2 — Implementar contra la spec

El agente ejecuta tareas guiado por la spec. Toma decisiones tácticas que la spec no anticipó: qué librería usa para una utilidad, cómo nombra una variable, qué patrón aplica para un caso límite. La spec no las dictó, así que el agente las inventa.

### Fase 3 — Bidirectional update (la fase clave)

Esta es la fase que define una spec viva y la que casi todos los equipos olvidan. Cuando una tarea termina, **alguien — agente o humano — escribe en la spec lo que se decidió de verdad durante la implementación**. No "lo que la spec decía", sino "lo que el código acabó haciendo y por qué". Las decisiones tácticas suben de nivel y se quedan registradas.

Augment dice exactamente esto:

> *Agents or developers update the spec to reflect what was actually built.*

La consecuencia práctica: la spec deja de ser un plan y empieza a ser una **descripción**. Sigue capturando intención, pero también captura las decisiones que se tomaron al realizar esa intención. La grieta entre intención y realidad se cierra.

### Fase 4 — Production feedback

Esta es la fase que añade Augment respecto a casi cualquier otra descripción del proceso SDD, y es brillante. Una vez que el código está en producción, **las métricas, incidentes y aprendizajes operacionales también se reescriben en la spec**. Si una asunción de la spec resulta ser falsa en producción ("creíamos que las imágenes serían de menos de 5 MB de media, en realidad son de 12"), eso vuelve a la spec como nota de hallazgo. Si un incidente revela un caso límite que la spec no contemplaba, ese caso límite se añade.

La spec, en este modelo, es un **documento que aprende**. Empieza describiendo intención, atraviesa decisiones de implementación, y acaba conteniendo las cicatrices de lo que la realidad le ha enseñado al sistema.

## Por qué esta fase 4 importa más de lo que parece

Una de las críticas más fuertes a SDD (capítulo 9, Isoform) es que las specs **pierden el contexto real** porque solo capturan "lo que íbamos a hacer", no "lo que aprendimos al hacerlo". La fase 4 de las specs vivas es la respuesta directa a esa crítica. Si tu spec recoge production feedback, el contexto real **no se pierde** — se acumula.

Es la diferencia entre un documento de diseño y un manual de operación. Y para un agente que llega seis meses después a hacer un cambio, leer la spec con production feedback es radicalmente más útil que leer la spec original.

## Mecanismos para que el bucle no muera

La razón por la que las specs estáticas se imponen no es ideológica — es entrópica. Mantener una spec viva requiere disciplina, y la disciplina sin mecanismo se evapora. Aquí van los mecanismos que de verdad funcionan:

- **Hacer del update parte de la definición de "hecho"**. Una tarea no está terminada hasta que la spec refleja lo que se hizo. Esto entra en el checklist de PR, no en el "sería bueno que..." informal.
- **Usar agentes recurrentes que detecten drift**. Un agente que cada noche compara specs y código y abre issues cuando hay divergencias. Es trabajo del harness, no del SDD propiamente, y por eso este punto se desarrolla en el curso siguiente.
- **Tratar las decisiones de implementación como mini-PRs a la spec**. Cada commit no trivial pregunta: ¿esto cambia algo de lo que la spec asume? Si sí, el commit incluye una entrada en la spec.
- **Hacer que la spec sea el primer lugar al que mira el agente**. Si tu workflow lleva al agente al código antes de la spec, el agente nunca aprenderá a actualizar la spec. Si la spec es siempre el primer documento que se carga, el bucle se cierra.

Ninguno de estos mecanismos es gratis, y ahí está el coste real del SDD. El capítulo 9 desarrolla este coste con honestidad — el famoso *maintenance tax* — y por qué no es despreciable.

## Lo que distingue a una spec viva sana de una muerta

Test rápido. Mira la última modificación significativa de tu spec. Compárala con la última modificación del código que la spec describe. Si la diferencia es de horas o días, tu spec está viva. Si es de semanas o meses, tu spec está muerta y nadie te lo ha dicho todavía.

Otro test: cuando un dev nuevo entra al equipo, ¿le mandas a leer las specs o le mandas a leer el código? Si le mandas al código y le adviertes "las specs están desactualizadas, no te fíes", la respuesta es clara — has aceptado oficialmente que las specs ya no son útiles. Y a partir de ese momento son peor que inútiles, porque generan ruido sin señal.

## La granularidad correcta de los updates

Una pregunta práctica que aparece pronto: ¿hay que actualizar la spec con cada cambio? La respuesta honesta es no. Hay dos tipos de cambios al código:

- **Cambios que la spec ya cubría** (refactor interno, optimización, fix tipográfico). No tocan la spec.
- **Cambios que la spec no cubría o que cambian una asunción de la spec** (decisión nueva, caso límite descubierto, trade-off elegido). Sí tocan la spec.

La distinción no siempre es nítida, y es ahí donde el juicio del ingeniero importa. Pero la regla práctica es: **si dentro de seis meses alguien leyendo la spec se sorprendería al ver lo que el código hace, la spec necesita ese update ahora**. Si no, no.

## Lo que viene a continuación

Hasta aquí hemos hablado del proceso. En el **capítulo 6** vamos a aterrizar en herramientas concretas: las cuatro herramientas SDD nativas que están definiendo el estado del arte hoy — **Kiro, Spec-kit, Tessl y BMAD** — con qué hacen bien, qué hacen mal y para qué tipo de equipo encaja cada una. Y en el capítulo 7, una categoría distinta: las herramientas que se sitúan **encima** de tu agente actual, como Traycer.
