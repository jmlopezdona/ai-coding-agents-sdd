# 13. Del SDD al harness: cómo encajan las piezas

Si has llegado hasta aquí, ya tienes el SDD como disciplina: sabes especificar intención, mantener el bucle bidireccional, modular el proceso según el tipo de tarea, y tienes criterio para no confundirlo con burocracia. Lo que falta es entender **dónde se conecta** esto con la siguiente capa de la trilogía: el **harness**, el sistema de infraestructura interna que rodea al agente para que la disciplina escale a un equipo entero y a meses vista.

Este capítulo es el puente. No repite el contenido del curso de harness — eso es otro repositorio entero — pero sí señala con precisión los cinco puntos exactos donde SDD se acopla con harness, y por qué la combinación produce algo mejor que los dos por separado.

## La tesis del puente

> SDD te dice **qué** especificar y cómo. Harness te dice **dónde** vive esa especificación, **quién** la lee, **cuándo** se valida, y **cómo** se mantiene viva sin disciplina humana sostenida.

SDD sin harness es ceremonia: depende de que cada miembro del equipo aplique disciplina perfecta, y la disciplina perfecta no escala. Harness sin SDD es infraestructura sin propósito: tienes sandboxes y sensores y bucles, pero el agente no sabe qué construir porque la intención nunca quedó capturada. Los dos juntos forman un sistema; los dos por separado son piezas sueltas.

## Punto de acople 1 — Las specs como contexto persistente

El primer pilar del harness, según el curso correspondiente, es **el contexto como sistema de registro**: la idea de que todo lo que el agente debe saber tiene que estar materializado en el repo, no en Slack ni en cabezas. Las specs del SDD son **una especialización de ese principio**.

Una spec bien escrita es contexto persistente con una forma específica: *intención + restricciones + por qués + criterios verificables*. El harness aporta la disciplina general ("el contexto vive en el repo"); el SDD aporta la forma concreta de uno de los artefactos que viven en él.

La consecuencia operativa: cuando un equipo monta un harness, las specs no son una carpeta más — son una de las piezas centrales del contexto. Y cuando ese mismo equipo aplica SDD, el harness se asegura de que las specs lleguen al agente de la forma correcta, en el momento correcto, con los hooks correctos.

## Punto de acople 2 — Los sensores que validan specs

El segundo pilar del harness es la dualidad **guías y sensores**: artefactos que orientan al agente *antes* de actuar (guías) y verificadores que detectan cuándo el agente se ha desviado *después* (sensores). La fase 5 del ciclo SDD del capítulo 5 — verificación — es exactamente un tipo de sensor.

Sin harness, esa fase 5 es manual o, en el mejor caso, un script ad-hoc que alguien ejecuta. Con harness, se convierte en infraestructura: tests que comparan código contra spec, agentes recurrentes que detectan drift entre uno y otro, linters que validan que las restricciones de la spec se siguen respetando.

Esta es la forma técnica de cerrar el bucle bidireccional del capítulo 6. Sin harness, el bucle depende de la disciplina humana ("acuérdate de actualizar la spec"). Con harness, el bucle es automático ("el sensor detecta que el código y la spec divergieron y abre un issue para reconciliarlos").

## Punto de acople 3 — Hooks que mantienen specs vivas

El detalle de Kiro del capítulo 7 — los **hooks que se disparan al guardar archivos** — no es un detalle. Es la prefiguración de uno de los mecanismos centrales del harness: **eventos del repo que disparan acciones del agente** (regenerar docs, actualizar índices, validar invariantes, refrescar el bucle de specs vivas).

Cuando una herramienta tipo Kiro tiene esos hooks, está implementando una micro-pieza de harness dentro de su propio producto. Cuando montas tu harness completo, tienes hooks para cualquier evento, y los ejes de mantenimiento de specs que requieren disciplina humana en SDD puro pasan a ser disparados por máquina en SDD + harness.

La regla operativa: **cualquier cosa que en SDD puro requiera "acuérdate de hacer X cuando pase Y", en SDD + harness debe convertirse en un hook automático**. Si algo se queda en "acuérdate", se va a olvidar.

## Punto de acople 4 — Las capas de arquitecto como prototipo de harness

Las herramientas del capítulo 8 — Traycer y similares — son, técnicamente, **harness aplicado a SDD**. Lo que hacen — interceptar entradas, planificar, verificar salidas — es exactamente la dinámica de guías y sensores del harness, aplicada al ciclo concreto de una sesión con un agente.

Si el harness completo te parece demasiado para empezar, una capa de arquitecto sobre tu agente es la versión miniatura de la idea: prueba qué se siente tener guías y sensores envolviendo al agente, antes de comprometerte a montar harness propio. Y cuando estés listo para el harness completo, descubrirás que la lógica de Traycer es un caso particular del patrón general — porque lo es.

## Punto de acople 5 — Los anti-patrones que solo el harness resuelve

Varios de los anti-patrones del capítulo 12 son específicamente difíciles de evitar sin harness:

- **Spec-as-Theatre** muere cuando hay un sensor automático que mide adhesión: si la spec no se respeta, el harness lo dice.
- **La spec eterna** muere cuando hay un agente recurrente que detecta drift y abre issues automáticos.
- **Confiar en la spec más que en el código** se vuelve raro cuando hay validadores automáticos que cuando hay desacuerdo te obligan a investigar antes de aceptar el merge.
- **El review burden** se reduce cuando los hooks generan los markdowns repetitivos automáticamente y la atención humana queda libre para lo que aporta juicio.

En otras palabras: **una parte significativa de lo que mata al SDD malhecho es exactamente lo que el harness está diseñado para resolver**. Los dos cursos no son secuenciales por capricho — son secuenciales porque los problemas que descubres haciendo bien la disciplina son los problemas que el harness aborda.

## Cuándo dar el siguiente paso

No todos los equipos que adoptan SDD necesitan saltar al harness inmediatamente. Hay señales específicas de que ya estás listo para la transición:

- **El equipo aplica SDD con disciplina pero la sostenibilidad depende de individuos.** Si Alice se va una semana de vacaciones y las specs se desactualizan, necesitas mecanismo, no más disciplina.
- **El bucle bidireccional del capítulo 6 funciona cuando alguien lo cuida y se rompe cuando nadie lo mira.** Eso es señal de que necesita pasar de proceso humano a infraestructura.
- **Los anti-patrones del capítulo 12 reaparecen periódicamente** aunque el equipo los conoce. La conciencia individual no escala; la infraestructura sí.
- **Las herramientas del capítulo 7 se quedan cortas porque son únicas para una dimensión** y tu sistema necesita varias capas (sensors + hooks + sandboxes + observabilidad). Eso es un harness completo, no una herramienta más.

Si reconoces dos o más, el siguiente curso de la trilogía es tu siguiente paso natural.

## Lo que el harness te va a enseñar (en avance corto)

Para que el puente sea concreto, aquí va una idea muy condensada de qué desarrolla el curso de harness y cómo conecta con lo que has aprendido aquí:

- **Guías y sensores como dos pilares.** El cap. 3 del harness desarrolla esta dualidad. Las specs son guías; los validadores y agentes recurrentes son sensores.
- **El bucle como primitiva.** El cap. 4 del harness presenta la idea de "iteración como primitiva". El ciclo del cap. 5 de este curso es un caso particular.
- **Entornos aislados y reproducibles.** El cap. 5 del harness habla de sandboxes desechables. Aquí mejoras la fiabilidad de la fase de implementación del SDD.
- **Contexto como sistema de registro.** El cap. 6 del harness es donde tus specs encajan como uno de los artefactos centrales.
- **Arquitectura para agentes.** El cap. 7 del harness te enseña a diseñar el repo de forma que el agente pueda navegarlo. Las specs solo funcionan cuando el agente las puede encontrar.

La progresión es natural y vale la pena hacerla en orden: primero esta guía para tener la disciplina, después harness para tener la infraestructura.

## Una despedida sin promesas

Una guía honesta no termina con "y ahora cambiará tu vida". Termina con dos verdades sencillas:

Primero, **SDD bien hecho mejora el output del agente y la sostenibilidad del proyecto**. Esto está documentado en la práctica de los equipos que llevan tiempo aplicándolo y se nota en métricas concretas: menos retrabajo, menos drift, mejor onboarding. No es magia, pero es real.

Segundo, **SDD malhecho es indistinguible de la burocracia y a veces es peor**. Esto también está documentado y es por lo que el capítulo 10 existe. Aplicarlo sin criterio convierte la disciplina en su propio cuello de botella, y a las pocas semanas el equipo vuelve al vibe coding con un nivel adicional de cinismo.

La diferencia entre las dos no está en la herramienta. Está en el juicio del equipo para modular el nivel de formalidad según el problema, mantener vivos los por qués, leer las críticas honestas como parte del curso, y entender que SDD es **una** estrategia dentro de context engineering, no la única.

Si te llevas eso, te llevas lo mejor que esta guía puede ofrecer.

---

*Este capítulo cierra el curso. Si quieres dar el siguiente paso, lee la [guía del harness](https://jmlopezdona.github.io/ai-coding-agents-harness/). Y si te faltan los fundamentos sobre qué es un agente y cómo se pilota, la [guía de fundamentos](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) es el punto de partida natural de la trilogía. Todos los capítulos de aquí son auto-contenidos: salta al que necesites cuando aparezca el problema que resuelve.*
