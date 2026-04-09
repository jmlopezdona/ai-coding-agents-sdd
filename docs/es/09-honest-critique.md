# 9. La crítica honesta: maintenance tax, MDD y la ilusión de control

Hasta aquí el curso ha sido constructivo: por qué hace falta SDD, cómo se hace, con qué herramientas. Este capítulo no. Este capítulo es el contrapeso, y existe porque cualquier guía que solo te enseñe los patrones positivos te está dejando sin defensa contra el momento — que llegará — en que tu propio proceso se vuelva el cuello de botella.

Las críticas que vamos a recoger no son objeciones marginales. Vienen de dos fuentes que conviene leer enteras: **Isoform** (*The Limits of Spec-Driven Development*) y **Martin Fowler / Thoughtworks** (*Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*). Las dos están escritas por gente que ha probado SDD en serio y ha decidido publicar lo que vio.

## Crítica 1 — El maintenance tax

La crítica más demoledora de Isoform es que las specs detalladas crean **deuda de documentación disfrazada de disciplina de ingeniería**.

> *Comprehensive specifications create documentation debt disguised as engineering discipline.*

La lógica es directa. Cuando el sistema tiene una spec viva (capítulo 5), cada cambio en el código pide un cambio en la spec. Cuando los requisitos cambian — y siempre cambian — pides cambios coordinados en ambos sitios. El coste sube. Y a diferencia del coste de mantenimiento del código, que es visible y se discute en sprint reviews, el coste de mantenimiento de las specs es invisible: no aparece en ningún burndown, nadie lo cuenta, y se acumula hasta que un día el equipo se da cuenta de que está dedicando una proporción incómoda de su tiempo a mantener documentación de un sistema en vez de a evolucionarlo.

Isoform lo dice así: el SDD no reduce el overhead, lo *traslada*. Lo que antes era código sin documentar ahora es código documentado, sí, pero el coste no desaparece — solo cambia de lugar. Y a veces ese nuevo lugar es peor, porque la spec se ha convertido en un segundo sistema que mantener, con su propia entropía, sus propios bugs (specs internamente contradictorias) y su propia carga cognitiva.

**Cómo defenderte:** mide el coste. Honestamente, durante un sprint, anota cuántas horas el equipo dedicó a actualizar specs y cuántas a evolucionar código. Si el ratio sube por encima del 20% y la spec sigue desactualizada, **algo está mal en tu proceso**. O las specs son demasiado detalladas, o el bucle del capítulo 5 no está calibrado, o estás aplicando spec completa a sitios donde no lo necesitabas (capítulo 8). Cualquiera de las tres, paras y rediseñas.

## Crítica 2 — La pérdida de los "por qués"

Esta es la crítica que más vale la pena tomarse en serio porque ataca el corazón del SDD.

> *Specs describe what systems should do, but cannot capture why decisions were made. The missing context is where the real problems show up.*

Isoform argumenta que las specs típicas se centran demasiado en los **cómos** — schemas, firmas, contratos, criterios — y se quedan cortas en los **por qués**: las suposiciones, las restricciones del mundo real que llevaron a una decisión, los trade-offs que se eligieron y, sobre todo, los que se rechazaron y por qué.

Y los por qués son justo lo que un agente (o un humano nuevo) necesita seis meses después para tomar decisiones inteligentes en los huecos que la spec no cubre. Una spec sin por qués envejece a la velocidad de las decisiones que la rodean, porque pierde la capacidad de explicarse a sí misma.

El capítulo 3 ya intenta blindar contra esto haciendo de los "por qués" una sección obligatoria de la plantilla. Pero el blindaje es solo de proceso; en la práctica, los por qués son la sección que más fácilmente se omite cuando el equipo va con prisa, y la que peor se actualiza con el bucle bidireccional del capítulo 5. Si tienes que recortar tiempo de mantenimiento de la spec, casi todo el mundo recorta primero los por qués — y eso es exactamente lo peor que puedes recortar.

**Cómo defenderte:** trata los por qués como tier 0 de la spec. Si una spec no tiene por qués, es spec incompleta. Si el bucle bidireccional del capítulo 5 no actualiza por qués, no es bidireccional de verdad — es half-loop.

## Crítica 3 — La falsa ilusión de completitud

> *Detailed specifications create misleading confidence.*

Esta es sutil y muy real. Una spec detallada parece exhaustiva. Cuando lees una spec de 800 líneas con diagramas, criterios y plantillas, tu cerebro asume que el sistema está bien pensado. La spec funciona como **prueba social interna**: "alguien se sentó a pensar esto en serio, así que tiene que estar bien".

La trampa es que detalle ≠ corrección. Una spec puede ser detallada y estar equivocada. Una spec puede estar internamente consistente y no encajar con la realidad. Una spec puede cubrir los 47 casos que se te ocurrieron y no cubrir el 48 que solo se descubre en producción.

Y peor: una spec detallada **inhibe la iteración creativa**. Cuando hay 800 líneas que todo el mundo aceptó, cambiar el rumbo se vuelve un movimiento social, no técnico. La spec se convierte en un freno cultural. Isoform cita esto como "reinforcing waterfall-like thinking", y la coincidencia con waterfall no es accidental: cuanto más completa la spec antes de empezar, más se parece el proceso a waterfall, con todos los problemas que waterfall tenía.

**Cómo defenderte:** sospecha de tu propia confianza en una spec densa. Si nadie ha cuestionado una decisión de la spec en dos sprints, no es señal de calidad — es señal de que la spec se ha convertido en obstáculo para cuestionarla. Pregúntate periódicamente: *si reescribiera esta spec hoy, desde cero, ¿saldría igual?*

## Crítica 4 — El paralelo con MDD

Esta es la crítica de Fowler, y es la que más conviene entender por contexto histórico. En los años 2000, hubo una ola de **Model-Driven Development** (MDD) que prometía exactamente lo que SDD promete hoy: tú modelas, las herramientas generan, una sola fuente de verdad, mantenibilidad mágica. Los modelos eran XML con OCL, pero la promesa era la misma.

MDD fracasó. Lo hizo por dos razones que vale la pena recordar:

1. **El nivel de abstracción era incómodo.** Los modelos eran demasiado abstractos para detalles de bajo nivel y demasiado concretos para arquitectura. Acababas escribiendo modelos casi tan verbosos como el código que generaban, sin la flexibilidad del código.
2. **La generación era inflexible.** Cuando el código generado no encajaba en una situación específica, modificarlo "rompía" el modelo. Tenías que elegir entre vivir con código subóptimo o abandonar la generación.

Fowler ve un eco directo en SDD:

> *SDD sits at an awkward abstraction level and just creates too much overhead and constraints. While LLMs reduce some MDD constraints, they introduce non-determinism — potentially combining MDD's inflexibility with LLM unpredictability.*

Esa última frase merece quedarse en la pared. **SDD potencialmente combina la inflexibilidad de MDD con la impredictibilidad de los LLMs**. Lo peor de los dos mundos: la rigidez del modelo más la sorpresa del modelo generativo. Si MDD fracasó por uno solo de esos problemas, SDD podría fracasar por la suma.

**Cómo defenderte:** leer el aviso histórico. Cuando estés construyendo specs muy detalladas, pregúntate si lo que estás escribiendo es más mantenible que el código que generaría, y cuánta certeza tienes de que el agente generará código consistente entre runs. Si las dos respuestas no son claras, estás en el sitio donde MDD murió.

## Crítica 5 — El review burden

Una crítica menor pero práctica de Fowler: spec-kit, en su evaluación, generó archivos markdown verbosos y repetitivos que **eran más pesados de revisar que el propio código** habría sido. Es una observación específica de una herramienta concreta, pero apunta a un patrón general: si tu proceso SDD obliga a revisar más artefactos que el código que reemplaza, has invertido el signo del beneficio.

**Cómo defenderte:** medir el tiempo de revisión real. Una semana de tracking honesto te dice si la spec ahorra revisión o la añade. Si añade, simplifícala.

## Crítica 6 — El control que no es control

Fowler también observa que, incluso con specs elaboradas y workflows estructurados, **los agentes siguen ignorando instrucciones, sobreinterpretando otras, creando duplicados y produciendo cosas inesperadas**. La spec da una sensación de control que no se corresponde con el comportamiento real del agente. La elaboración del proceso ha aumentado, la previsibilidad del output no.

Es la observación más cruda del capítulo: **una spec más larga no produce un agente más obediente**. Produce un humano más confiado y un agente igualmente impredecible. Y la diferencia entre esos dos hechos es exactamente donde los problemas se cuelan.

**Cómo defenderte:** medir adhesión, no completitud. La pregunta no es "¿qué tan detallada es la spec?", es "¿qué porcentaje de las restricciones de la spec se respeta sin re-prompteo?". Si la spec crece y la adhesión no mejora, has añadido peso sin tracción.

## Cuándo SDD claramente *no* es para ti

Después de leer estas seis críticas, hay señales concretas de que SDD no es la respuesta para tu situación:

- Tu código tiene un horizonte de vida menor a 6 meses.
- Estás en fase de descubrimiento donde la intención cambia semana a semana.
- Tu equipo es pequeño (1-3 personas) y la coordinación no es el cuello de botella.
- Tu cuello de botella real es la deploy infrastructure, no la calidad del código.
- Cada vez que escribes una spec, el código relevante ha cambiado antes de terminarla.

Si reconoces tu situación en cualquiera de las cinco, **el coste del SDD probablemente excede su beneficio para ti**. No es un fallo de la disciplina. Es que la herramienta no encaja en tu problema, y forzarla es malgastar el tiempo del equipo.

## Lo que viene a continuación

Las críticas dejan al lector con una pregunta legítima: *si SDD malhecho es esto, ¿qué hago entonces?* El capítulo 10 desarrolla la alternativa que propone Isoform: **context engineering**. No es "vibe coding pero peor"; es una propuesta deliberadamente distinta sobre dónde debe vivir la intención.
