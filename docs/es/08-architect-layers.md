# 8. Capas de arquitecto sobre agentes: Traycer y el patrón de envoltura

Las herramientas del capítulo anterior comparten una característica: **definen su propio workflow completo** — su entorno, su estructura de documentos, su flujo de fases, sus convenciones. Adoptarlas implica trabajar dentro de su forma de hacer las cosas. Pero hay otra categoría que está creciendo en paralelo y que vale la pena entender por separado: las herramientas que **se insertan en el workflow que ya tengas**, situándose encima del agente que ya usas sin pedirte que lo cambies. Las llamamos *capas de arquitecto*, y el ejemplo más representativo de esta categoría hoy es **[Traycer](https://traycer.ai/)**.

## La pregunta que esta categoría responde

Cuando un equipo ya usa Cursor, Claude Code, Codex o similar y los conoce bien, la fricción de adoptar las herramientas del capítulo anterior tiene dos caras. En algunos casos implica **cambiar de herramienta** — Kiro, por ejemplo, es un IDE distinto donde pierdes los atajos, configuraciones y hábitos de trabajo que tu equipo ha acumulado. En otros implica **adaptar tu modelo de trabajo** al flujo que la herramienta impone — su estructura de documentos, sus fases, sus convenciones — aunque sigas usando tu agente habitual. En ambos casos, el coste de adopción es real aunque la nueva herramienta sea objetivamente mejor en algunas dimensiones.

Las capas de arquitecto resuelven esa fricción al no pedirla. Tu agente sigue siendo Cursor o Claude Code o el que uses. La capa se inserta en el flujo de trabajo en dos puntos concretos: **antes** del agente (planificando) y **después** del agente (verificando). En medio, el agente trabaja como siempre.

## Qué hace Traycer concretamente

Según se describe en su material y en la discusión de la comunidad, Traycer aporta tres cosas que un agente desnudo no:

### 1. Elicitación

Antes de dejarte hablar con tu agente principal, Traycer te hace una ronda de **preguntas** para sacar a la superficie requisitos que probablemente habrías olvidado mencionar. Es la fase de "clarificación iterativa" del capítulo 5 incorporada como mecanismo automático en lugar de como recordatorio mental.

La idea es directa: la mayoría de prompts que la gente le da a su agente son pobremente especificados, y la mayoría de fallos del agente vienen de esa pobreza. Si una herramienta automatiza el "espera, antes de empezar dime qué debería pasar si...", la calidad de todo lo que viene después sube.

### 2. Planificación

Una vez clarificado el requisito, Traycer genera un **plan de implementación detallado** — qué archivos toca, qué dependencias hay entre tareas, qué orden seguir — *antes* de que el agente principal escriba código. El plan es revisable. Si está mal, lo corriges. Si está bien, lo bajas al agente principal como contexto rico, no como un prompt suelto.

Esto resuelve directamente el problema del *context drift* del capítulo 1: el plan obliga a explicitar el blast radius del cambio antes de empezar, en lugar de descubrirlo a posteriori cuando el agente ya rompió cosas que no miró.

### 3. Verificación automática

Cuando el agente termina, Traycer **compara lo que el agente hizo con la spec/plan original** y señala las divergencias. Es la fase de validación del capítulo 5, automatizada y obligatoria, en lugar de "si me acuerdo, lo reviso".

Esta verificación post-código es exactamente la grieta que el capítulo 7 identifica como no resuelta por las herramientas SDD nativas. Traycer la cierra.

## El patrón general, más allá de Traycer

Aunque Traycer sea el ejemplo más representativo hoy, el patrón vale la pena entenderlo abstraído de la herramienta concreta:

> **Una capa de arquitecto envuelve a un agente de codificación con tres cosas: clarificación de requisitos antes del prompt, planificación explícita antes del código, y verificación automática después del código.**

El patrón es independiente de la herramienta y va a aparecer en otras formas. Habrá más herramientas que ocupen este espacio. La pregunta arquitectónica para tu equipo no es "¿uso Traycer?" — es "¿le falta a mi flujo alguno de estos tres ganchos?". Si la respuesta es sí, entonces vale la pena buscar una capa que los aporte, sea esta o sean otras.

## Una nota sobre el sesgo de la fuente

La discusión más completa sobre Traycer en la comunidad viene de un post de r/vibecoding que es **claramente promocional**. Su autor lo declara como "mi mejor opción" y la estructura del post es la de marketing, no la de análisis neutral. Esto no significa que las afirmaciones sean falsas — la categoría existe y la lógica del patrón es sólida — pero sí que conviene matizar el entusiasmo del material original.

La forma honesta de evaluar Traycer (o cualquier herramienta del patrón) es: monta una semana de trabajo real con ella, mide cuántos retrabajos te ahorras y cuántos prompts adicionales necesitas para arreglar lo que la herramienta no anticipó. Si la diferencia es notable, sigue. Si no, vuelve atrás sin coste.

## Capas vs. herramientas nativas: cómo elegir

| Pregunta | Si tu respuesta es… | Mira… |
|---|---|---|
| ¿Tu equipo ya tiene años de uso de Cursor/Claude Code? | sí | una capa (cap. 8) |
| ¿Tu workflow sufre más por falta de plan que por falta de proceso? | sí | una capa (cap. 8) |
| ¿Quieres adoptar SDD desde cero, sin agente preferido? | sí | herramienta nativa (cap. 7) |
| ¿Quieres una constitution global y disciplina de checkpoints? | sí | Spec-kit (cap. 7) |
| ¿Tu problema real es organizar features grandes en pasos? | sí | Kiro (cap. 7) |
| ¿Te interesa empujar la frontera de spec-as-source? | sí | Tessl (cap. 7) |
| ¿Tu cuello de botella es la coordinación entre roles distintos? | sí | BMAD (cap. 7) |

Las dos categorías no son excluyentes. Un equipo maduro puede perfectamente usar Spec-kit para la disciplina global de constitution y boundaries, y Traycer (o equivalente) sobre su agente diario para los planes y verificaciones tácticas. Son capas distintas del mismo edificio.

## Lo que las capas de arquitecto **no** resuelven

Hay un límite real que conviene nombrar: las capas de arquitecto **no resuelven** el problema de mantener specs vivas a meses vista (capítulo 6). Resuelven el ciclo táctico de una feature concreta — clarificar, planificar, ejecutar, verificar — pero no la disciplina sostenida de mantener una spec actualizada cuando el código evoluciona y el equipo cambia.

Para esa parte, las capas son insuficientes por diseño. Su unidad de trabajo es la sesión, no el ciclo de vida del módulo. La disciplina de mantenimiento sigue siendo trabajo del equipo, y ese trabajo tiene un coste real (capítulo 10).

## El gancho con el harness

Hay una frase que vale la pena dejar grabada porque conecta este capítulo con el siguiente curso de la trilogía. El patrón "capa de arquitecto sobre agente" es, técnicamente, **una pieza de harness aplicada a SDD**. Lo que Traycer hace — interceptar entradas, planificar, verificar salidas — es exactamente el tipo de envoltorio que el curso de harness desarrolla para muchísimas otras dimensiones (tests, sandboxes, sensores, hooks).

Visto así, Traycer no es solo una herramienta SDD: es un ejemplo concreto de cómo un harness bien hecho mejora el SDD. Y los hooks de Kiro mencionados en el capítulo 7 son otro ejemplo del mismo principio: harness + SDD se refuerzan mutuamente. El capítulo 13 vuelve sobre esto.

## Lo que viene a continuación

Hasta aquí hemos visto el ciclo de vida (cap. 5), las specs vivas (cap. 6), y dos categorías de herramientas (cap. 7 y 8). En el **capítulo 9** bajamos a la práctica: cómo se aplica todo esto a tres tipos de trabajo distintos — features nuevas, refactors, y bug fixes — porque el proceso óptimo no es el mismo en los tres casos.
