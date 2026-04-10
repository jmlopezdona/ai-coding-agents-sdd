# 9. Patrones de aplicación: features, refactors y bug fixes

Una de las trampas más fáciles de caer en SDD es **aplicar el mismo proceso a todo**. Es la trampa exacta que [Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) le señala a Kiro en el capítulo 7: tratar un bug fix de tres líneas como una feature multi-historia. El proceso no escala uniformemente porque los problemas no son uniformes. Este capítulo recorre los tres patrones de trabajo más comunes — feature nueva, refactor y bug fix — y describe **cómo modular el ciclo SDD** para cada uno.

La regla general que conviene tener delante: **el peso de la spec debe ser proporcional al coste del cambio si sale mal**. Una feature crítica con alcance amplio merece spec completa. Un bug fix trivial merece, como mucho, una nota.

## Patrón 1 — Feature nueva

Es el caso para el que SDD fue diseñado y donde más rinde. Hay intención que capturar, restricciones que escribir, criterios de aceptación que enumerar, casos límite que desenterrar antes de tocar código.

### El ciclo aplicado

1. **Especificar.** Plantilla completa del capítulo 3: objetivo, no-goals, restricciones, criterios, por qués, boundaries. Escribe esto *antes* de hablar con el agente sobre código.
2. **Clarificar.** Pasa la spec al agente y pídele que te haga preguntas. Esta fase raramente se omite gratis: las preguntas que el agente te hace son exactamente las ambigüedades que iban a romper el código después.
3. **Planificar.** El agente propone qué archivos toca, en qué orden, qué tests escribe. Tú revisas. Si el plan está mal, lo corriges *aquí*, no después.
4. **Implementar tarea por tarea.** Cada tarea es atómica respecto a la spec. Cada tarea trae sus tests.
5. **Verificar.** Comparas la spec original con el código final. Los criterios de aceptación se convierten en una checklist tachada.
6. **Update bidireccional.** Cualquier decisión que se tomó durante la implementación y no estaba en la spec original sube a la spec.

### Errores típicos de este patrón

- **Saltarse el paso 2** y tirar adelante con la primera spec que escribes. La elicitación parece coste; en realidad es ahorro.
- **Confundir el plan con el código**. El plan es discutible y barato; el código es caro de revertir. Discute en el plan, no en el diff.
- **Olvidar el paso 6** y dejar la spec congelada en su versión inicial. Esta es la diferencia entre spec-first y spec-anchored del capítulo 2.

### Cuándo este patrón **no** es para ti

- Features experimentales donde la intención es exactamente "ver qué sale". Si no tienes una intención clara antes de empezar, no hay nada que especificar todavía.
- Spikes de research. La spec viene *después* del spike, como conclusión, no antes.

## Patrón 2 — Refactor

El refactor es el patrón donde más equipos confunden cómo aplicar SDD. La tentación es tratarlo como una feature nueva — "vamos a especificar cómo queremos que quede el módulo" — y eso casi siempre lleva a una spec inflada que nadie respeta. El refactor merece un proceso distinto.

### La pregunta correcta para empezar

> *¿Qué invariantes externos deben seguir siendo verdad después del refactor?*

Esta es la pregunta tesis del SDD aplicado a refactors. No "cómo queda el código por dentro" — eso es decisión del agente — sino "qué garantías observables externamente no pueden romperse". Esos invariantes son tu spec. Todo lo demás es libertad de implementación.

### El ciclo aplicado

1. **Capturar invariantes.** Una lista corta de cosas que deben seguir siendo verdad: la API pública no cambia, los tests existentes siguen pasando, el contrato de errores se mantiene, la performance no empeora más de un X%.
2. **Caracterizar el estado actual.** Si los tests actuales no cubren bien los invariantes, escribe los tests que faltan *antes* de tocar nada. Un refactor sin red de tests es vibe coding disfrazado.
3. **Plan del refactor.** Pasos pequeños, cada uno revertible, ningún paso rompe los invariantes mientras se ejecuta.
4. **Implementar paso a paso, con tests verdes en cada paso.**
5. **Verificar contra los invariantes.** Los mismos tests, las mismas APIs, la misma observabilidad externa.

### Lo que no escribes

- No escribes "diseño nuevo" porque el diseño nuevo es el output, no el input.
- No escribes criterios de aceptación de comportamiento porque el comportamiento *no debe cambiar*. Los invariantes son tus criterios.
- No escribes no-goals como en una feature; en un refactor el principal no-goal está implícito ("no introducir cambios funcionales") y solo lo escribes si hay excepciones.

### Errores típicos

- **Hacer el refactor y "de paso" arreglar dos bugs y añadir una pequeña mejora.** No. Un refactor que cambia comportamiento ya no es un refactor — es una feature, y se especifica como una feature.
- **Saltar el paso 2.** Si los tests actuales no cubren los invariantes, refactorizar es ruleta rusa. La caracterización es la fase más subestimada del refactor.

## Patrón 3 — Bug fix

El bug fix es el patrón donde el SDD pleno es contraproducente. Un bug fix de tres líneas con plantilla completa, clarificación iterativa y plan formal es exactamente la burocracia que el capítulo 10 critica. No lo hagas.

Pero "no lo hagas" no significa "haz vibe coding". Significa que el bug fix tiene su propia versión muy ligera del ciclo, y vale la pena nombrarla.

### El ciclo aplicado (versión mínima)

1. **Reproducir el bug** con un test que falla. Esto es tu spec mínima viable: el test que pasa es el criterio de aceptación, todo en uno.
2. **Diagnosticar la causa raíz.** Si el agente la encuentra, perfecto. Si no, hazlo tú.
3. **Aplicar el fix** con el test rojo a la vista.
4. **Comprobar que el test pasa y los demás también.**
5. **Una nota corta en la spec relevante** (si la hay) explicando el bug y el fix. Una línea, no un párrafo.

### Cuándo el bug fix se convierte en patrón 1 o 2

A veces, mientras diagnosticas un bug, descubres que el bug no es local — es síntoma de un problema arquitectónico. En ese momento **subes de nivel**: paras el bug fix y abres una spec de feature o de refactor. La señal de que esto está pasando es que el fix te pide tocar más de dos archivos no relacionados, o que ya hay tres bugs distintos que apuntan al mismo lugar.

Esta transición de "bug → refactor" es uno de los movimientos más valiosos del SDD bien aplicado y casi nunca se enseña explícitamente. La heurística: **si el fix es mecánico, es bug fix; si el fix te obliga a entender cinco cosas que no entendías ayer, ha dejado de ser un bug fix**.

## Una matriz para elegir el patrón correcto

| Pregunta | Sí → patrón |
|---|---|
| ¿Hay intención nueva que capturar? | Feature |
| ¿El comportamiento observable cambia? | Feature |
| ¿El comportamiento debe quedarse igual y solo cambia la estructura? | Refactor |
| ¿Hay un test rojo concreto que arreglar? | Bug fix |
| ¿El fix toca cinco archivos no relacionados? | Refactor (o feature) |
| ¿Sabes en una frase qué tiene que cambiar? | Bug fix |

Si dudas entre dos patrones, elige el más ligero y promociónalo si descubres que se queda corto. La promoción tardía duele menos que la sobre-especificación temprana.

## El meta-patrón: leer el tamaño antes de elegir el proceso

Si te quedas con una sola idea de este capítulo, que sea esta: **antes de aplicar SDD, lee el tamaño del problema**. No el tamaño aparente — el tamaño real, que casi siempre se descubre haciendo dos preguntas:

1. *¿Cuántos sitios distintos del sistema necesito tocar para que esto funcione?*
2. *¿Cuánto duele si esto sale mal y nadie lo nota durante una semana?*

Las respuestas a esas dos preguntas, multiplicadas, te dicen cuánto SDD necesitas. Mucho × mucho = patrón 1 con todo. Poco × poco = patrón 3 minimal. Las combinaciones intermedias caen en variantes del patrón 2.

Las herramientas del capítulo 7 (Kiro especialmente) tienden a aplicar el patrón 1 a todo, y por eso a veces se sienten pesadas. El criterio de modulación que estás aprendiendo aquí no lo ofrece ninguna herramienta hoy: lo aporta el ingeniero, y es exactamente el tipo de criterio que distingue a un equipo que usa SDD de uno que lo sufre.

## Lo que viene a continuación

Llegamos al capítulo más incómodo del curso. El **capítulo 10** es la crítica honesta a SDD: dónde falla, qué cuesta, por qué algunas de las críticas son justas y qué hacer al respecto. Si vienes leyendo el curso convencido de que esta disciplina es la respuesta, ese capítulo está diseñado para descolocarte un poco. Es deliberado y es necesario.
