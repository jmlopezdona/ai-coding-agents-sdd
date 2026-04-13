# 12. De la disciplina a la fábrica de software

Si has llegado hasta aquí, ya tienes el SDD como disciplina: sabes especificar intención, mantener el bucle bidireccional, modular el proceso según el tipo de tarea, y tienes criterio para no confundirlo con burocracia. Lo que falta es entender **hacia dónde lleva todo esto** cuando dejas de depender de la disciplina individual y empiezas a construir un sistema que funcione solo.

## La progresión de autonomía

El camino natural no es "SDD y luego otra cosa". Es una progresión continua donde el sistema asume cada vez más responsabilidad y el humano se desplaza del centro a la periferia:

### Nivel 1 — Human in the loop (SDD puro)

Es donde estás ahora. El humano escribe specs, revisa el output del agente, mantiene el bucle bidireccional manualmente, y detecta drift con los ojos. Funciona, pero tiene un techo: **depende de que cada miembro del equipo aplique disciplina perfecta, y la disciplina perfecta no escala**. Si Alice se va de vacaciones una semana, las specs se desactualizan. Si nadie revisa el bucle, se rompe.

### Nivel 2 — Human on the loop (SDD + automatización)

El humano ya no ejecuta cada paso — **supervisa un sistema que ejecuta por él**. Hooks automáticos disparan validaciones cuando el código cambia. Sensores detectan drift entre spec y código y abren issues sin intervención humana. Agentes recurrentes mantienen las specs vivas. El humano define intención, revisa excepciones, y toma decisiones que el sistema no puede tomar solo.

La diferencia operativa es profunda: en el nivel 1, olvidarte de actualizar la spec es un fallo de disciplina. En el nivel 2, es un bug de infraestructura — y los bugs de infraestructura se arreglan una vez.

### Nivel 3 — Human over the loop (software factory)

El humano define intención a alto nivel y el sistema se encarga del resto: descompone la intención en specs, implementa, verifica, detecta problemas, y se autocorrige. El humano interviene en excepciones y decisiones estratégicas, no en el flujo normal. Cada iteración del sistema lo hace más autónomo y más fiable.

No es ciencia ficción — es la dirección en la que herramientas como Traycer ([capítulo 7](07-native-sdd-tools.md#traycer)) ya apuntan con su ciclo de elicitación → planificación → verificación. La diferencia es que en el nivel 3, ese ciclo no necesita un humano supervisando cada sesión.

## Los mecanismos que habilitan cada salto

La progresión no es abstracta. Cada nivel se habilita con mecanismos concretos que conectan directamente con lo que has aprendido en este curso:

### Specs como contexto persistente

Todo lo que el agente debe saber tiene que estar materializado en el repo, no en Slack ni en cabezas. Las specs del SDD son una especialización de ese principio: contexto persistente con una forma específica — *intención + restricciones + por qués + criterios verificables*.

En el nivel 1, las specs las escribe y mantiene un humano. En el nivel 2, el sistema se asegura de que las specs lleguen al agente en la forma correcta, en el momento correcto, con los hooks correctos. En el nivel 3, el sistema genera y actualiza specs como parte de su propio ciclo.

### Sensores que validan specs

La fase de verificación del capítulo 5 es un sensor manual: alguien compara lo que el agente hizo con lo que la spec pedía. En el nivel 2, eso se convierte en infraestructura: tests que comparan código contra spec, agentes recurrentes que detectan drift, linters que validan que las restricciones se siguen respetando.

Esta es la forma técnica de cerrar el bucle bidireccional del capítulo 6. Sin automatización, el bucle depende de la disciplina humana ("acuérdate de actualizar la spec"). Con automatización, el bucle es automático ("el sensor detecta que el código y la spec divergieron y abre un issue para reconciliarlos").

### Hooks que eliminan el "acuérdate de..."

Los hooks que se disparan al guardar archivos — como los de Kiro en el capítulo 7 — son la prefiguración de un principio más general: **eventos del repo que disparan acciones automáticas** (regenerar docs, actualizar índices, validar invariantes, refrescar el bucle de specs vivas).

La regla operativa: **cualquier cosa que en SDD puro requiera "acuérdate de hacer X cuando pase Y" debe convertirse en un hook automático**. Si algo se queda en "acuérdate", se va a olvidar. Cada hook que añades es un paso del nivel 1 al nivel 2.

### Herramientas SDD como prototipos de fábrica

Herramientas como Traycer son, técnicamente, **prototipos del nivel 2**: interceptan entradas, planifican, verifican salidas. Es la dinámica de guías y sensores aplicada al ciclo concreto de una sesión con un agente.

Si la fábrica completa te parece demasiado para empezar, una herramienta SDD sobre tu agente es la versión miniatura de la idea: prueba qué se siente tener guías y sensores envolviendo al agente, antes de comprometerte a construir tu propia infraestructura.

### Anti-patrones que solo la automatización resuelve

Varios de los [anti-patrones del capítulo 11](11-anti-patterns.md) son específicamente difíciles de evitar con disciplina humana sola:

- **Spec-as-Theatre** muere cuando hay un sensor automático que mide adhesión: si la spec no se respeta, el sistema lo dice.
- **La spec eterna** muere cuando hay un agente recurrente que detecta drift y abre issues automáticos.
- **Confiar en la spec más que en el código** se vuelve raro cuando hay validadores automáticos que ante desacuerdo te obligan a investigar antes de aceptar el merge.
- **El review burden** se reduce cuando los hooks generan los markdowns repetitivos automáticamente y la atención humana queda libre para lo que aporta juicio.

En otras palabras: **una parte significativa de lo que mata al SDD malhecho es exactamente lo que la automatización está diseñada para resolver**.

## Señales de que es momento de subir de nivel

No todos los equipos que adoptan SDD necesitan saltar al nivel 2 inmediatamente. Hay señales específicas:

- **La sostenibilidad depende de individuos.** Si una persona se va una semana y las specs se desactualizan, necesitas mecanismo, no más disciplina.
- **El bucle bidireccional funciona cuando alguien lo cuida y se rompe cuando nadie lo mira.** Eso es señal de que necesita pasar de proceso humano a infraestructura.
- **Los [anti-patrones del capítulo 11](11-anti-patterns.md) reaparecen periódicamente** aunque el equipo los conoce. La conciencia individual no escala; la infraestructura sí.
- **Las herramientas del capítulo 7 se quedan cortas** porque tu sistema necesita varias capas (sensores + hooks + sandboxes + observabilidad) que ninguna herramienta individual cubre.

Si reconoces dos o más, es momento de empezar a construir infraestructura. La [guía del harness](https://jmlopezdona.github.io/ai-coding-agents-harness/) desarrolla los mecanismos concretos para dar ese paso.

## Una despedida sin promesas

Una guía honesta no termina con "y ahora cambiará tu vida". Termina con dos verdades sencillas:

Primero, **SDD bien hecho mejora el output del agente y la sostenibilidad del proyecto**. Esto está documentado en la práctica de los equipos que llevan tiempo aplicándolo y se nota en métricas concretas: menos retrabajo, menos drift, mejor onboarding. No es magia, pero es real.

Segundo, **SDD malhecho es indistinguible de la burocracia y a veces es peor**. Esto también está documentado y es por lo que el capítulo 9 existe. Aplicarlo sin criterio convierte la disciplina en su propio cuello de botella, y a las pocas semanas el equipo vuelve al vibe coding con un nivel adicional de cinismo.

La diferencia entre las dos no está en la herramienta. Está en el juicio del equipo para modular el nivel de formalidad según el problema, mantener vivos los por qués, y entender que la disciplina manual es solo el primer paso — el objetivo es construir un sistema donde la calidad sea consecuencia de la infraestructura, no del heroísmo individual.

Si te llevas eso, te llevas lo mejor que esta guía puede ofrecer.

---

*Este capítulo cierra el curso. Si quieres dar el siguiente paso hacia la automatización, lee la [guía del harness](https://jmlopezdona.github.io/ai-coding-agents-harness/). Y si te faltan los fundamentos sobre qué es un agente y cómo se pilota, la [guía de fundamentos](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/) es el punto de partida natural de la trilogía. Todos los capítulos de aquí son auto-contenidos: salta al que necesites cuando aparezca el problema que resuelve.*
