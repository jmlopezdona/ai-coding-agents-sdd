# 7. Herramientas SDD nativas: Kiro, Spec-kit, Tessl y BMAD

Hay un puñado de herramientas que se han propuesto explícitamente como infraestructura para hacer Spec-Driven Development. Las llamamos **nativas** porque la spec es el centro de su workflow, no un complemento. En este capítulo recorremos las cuatro más visibles hoy — Kiro, Spec-kit, Tessl y BMAD — situándolas en el espectro del capítulo 2 y siendo honestos sobre dónde encaja cada una y dónde se rompe.

> **Nota importante:** este capítulo envejecerá rápido. El ecosistema SDD está en plena ebullición y lo que aquí se describe es el estado del arte a principios de 2026. Trátalo como una foto del momento, no como una recomendación definitiva.

## La advertencia de Fowler

Antes de entrar herramienta por herramienta, conviene tener presente la conclusión a la que llega [Martin Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) tras evaluar las tres principales:

> *Ninguna de ellas es adecuada para la mayoría de los problemas reales de programación.*

No es marketing inverso. Es la observación honesta de que las herramientas actuales sufren todas el mismo problema — *workflow mismatch*: aplican un proceso de talla única a problemas de tamaños radicalmente distintos. Una bug fix de tres líneas tratada como una feature multi-historia es burocracia. Una migración crítica tratada como un commit cualquiera es negligencia. Las herramientas de hoy no distinguen bien entre los dos extremos, y eso es su limitación común.

Con eso delante, vamos una por una.

## Kiro

Kiro es el IDE agéntico de AWS. Su workflow es probablemente el más simple del cuarteto: tres documentos markdown — **Requirements → Design → Tasks** — que se generan en orden y guían al agente a través del ciclo. Encaja casi exactamente con el "enfoque B" del ciclo de vida del capítulo 5.

**Lo que hace bien:**

- El flujo es legible a primera vista. Un humano que llega de fuera entiende el proceso en cinco minutos.
- Tiene **hooks que se disparan al guardar archivos** para ejecutar tareas en segundo plano (actualizar docs, regenerar tests, validar contra la spec). Esto es relevante porque conecta directamente con uno de los temas centrales del curso de harness.
- Es pragmático. No intenta forzarte a Spec-as-Source. Vive cómodamente en spec-anchored.

**Lo que Fowler señala como problemático:**

- **Excesivo para bug fixes pequeños.** Trata cada cambio como una feature multi-story, lo que para arreglar un typo es absurdo. Si tu día a día son muchos cambios pequeños, Kiro te va a frustrar.
- La estructura rígida no se adapta al tamaño del problema.

**Para quién encaja:**

- Equipos con features medianas-grandes donde el ritual de tres documentos amortiza su coste.
- Equipos que quieren un onramp suave a SDD sin reescribir su tooling.

:material-book-open-variant: [Guía de inicio de Kiro](https://kiro.dev/docs/getting-started)

## Spec-kit (GitHub)

Spec-kit es el toolkit oficial de GitHub para Spec-Driven Development. Es un CLI y un conjunto de plantillas que introducen *checkpoints* en cada etapa del proceso. Su rasgo distintivo es la **constitution**: un documento de reglas fundamentales del proyecto que vive por encima de las specs individuales y que el agente lee antes de cualquier tarea.

**Lo que hace bien:**

- La idea de la constitution es buena: separa lo que es invariante del proyecto (capítulo 3, "boundaries") de lo que es específico de una feature.
- Usa checklists explícitas que obligan al agente a no saltarse pasos.
- Es open source y se integra naturalmente con flujos de GitHub (PRs, issues, Actions).

**Lo que Fowler señala como problemático:**

- **Aspira a spec-anchored pero en la práctica es spec-first.** Genera muchos archivos interconectados, pero el mecanismo automático de mantener spec y código sincronizados no está realmente implementado. Es spec-first con disfraz.
- **Genera markdown verboso y repetitivo.** Fowler cuenta que, en su evaluación, los archivos generados eran *más pesados de revisar que el propio código*. Esto es exactamente lo que los críticos predicen del SDD malhecho: el tax de revisión sube en lugar de bajar.

**Para quién encaja:**

- Equipos profundamente integrados en GitHub que quieren la ergonomía y no temen el coste de revisar markdown.
- Equipos que quieren probar la idea de la "constitution" como capa global de boundaries.

:material-book-open-variant: [Repositorio de Spec-kit](https://github.com/github/spec-kit)

## Tessl

Tessl es la propuesta más radical del cuarteto. Apunta directamente al nivel **Spec-as-Source** del espectro: la spec es la única fuente que humanos editan, el código se genera, los archivos generados llevan una marca `// GENERATED FROM SPEC - DO NOT EDIT`. Si el código está mal, arreglas la spec y regeneras.

**Lo que hace bien:**

- Es la única herramienta del cuarteto que persigue de verdad spec-as-source. Si esa es la dirección estratégica que crees correcta, no tienes muchas alternativas.
- Fuerza una claridad en la spec que las otras no exigen, porque la spec **es** el sistema, no una descripción de él.

**Lo que Fowler señala como problemático:**

- **Está en beta privada**, con todas las limitaciones que eso implica (acceso, soporte, riesgo de cambios disruptivos, riesgo de desaparecer).
- **El no-determinismo de los LLMs entra en conflicto directo con la promesa de "regenera y obtienes lo mismo"**. Si dos generaciones del mismo spec producen código distinto, la promesa de "una sola fuente de verdad" se desinfla. Esto es casi exactamente el problema que mató al Model-Driven Development en los 2000 y que el capítulo 10 desarrolla.

**Para quién encaja:**

- Experimentación deliberada en superficies aisladas y pequeñas. No es una herramienta para apostar tu repo principal en 2026.
- Equipos con curiosidad por hacia dónde podría ir SDD a largo plazo.

:material-book-open-variant: [Quickstart de Tessl](https://docs.tessl.io/introduction-to-tessl/quickstart-skills-docs-rules)

## BMAD

BMAD es el caso más distinto del cuarteto, y por eso lo dejo para el final. En lugar de centrarse en la spec como artefacto, BMAD despliega un **equipo de agentes especializados con roles** — Product Manager, Arquitecto, QA, Developer — que gestionan el ciclo ágil completo manteniendo contexto consistente entre ellos. Es Spec-Driven Development implementado como sistema multi-agente.

**Lo que hace bien:**

- La especialización por roles ayuda con la *curse of instructions* del capítulo 3. Cada agente solo ve las instrucciones relevantes para su rol, en lugar de un megaprompt que mezcla todo.
- Encaja con lo que el survey académico de arXiv ([*A Survey on Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1)) describe como la familia de **multi-agent role-playing systems** (junto a ChatDev y MetaGPT en el lado más académico).
- Para equipos que ya piensan en términos de roles ágiles, la mental model translation es trivial.

**Lo que hay que mirar con cuidado:**

- La complejidad operativa es alta. Coordinar varios agentes con roles es un harness en miniatura, y los fallos de coordinación entre agentes son sutiles y difíciles de depurar.
- La promesa multi-agente está sobrerrepresentada en marketing y subrepresentada en evidencia empírica fuera de demos.

**Para quién encaja:**

- Equipos con apetito experimental y procesos ágiles maduros que pueden absorber la complejidad.
- Casos donde la separación de preocupaciones por rol aporta más que la simplicidad de un solo agente bien dirigido.

:material-book-open-variant: [Repositorio de BMAD](https://github.com/bmad-code-org/BMAD-METHOD)

## Cómo elegir (o no elegir)

Una recomendación honesta, sabiendo que envejecerá:

1. **Si nunca has hecho SDD**, no elijas herramienta todavía. Empieza con specs-first puro escritas a mano usando la plantilla del capítulo 3 y un agente de propósito general. Aprende qué partes del proceso te duelen *antes* de buscar herramienta que las resuelva.

2. **Si has hecho specs a mano y sabes qué te falta**, evalúa Kiro o Spec-kit dependiendo de si te molesta más la rigidez (Kiro) o la verbosidad (Spec-kit).

3. **Si te interesa empujar la frontera**, prueba Tessl en una superficie aislada o monta una proof-of-concept con BMAD. No los pongas en el path crítico.

4. **Si tu problema real es el *blast radius* de los cambios y no la estructura del proceso**, ninguna herramienta de este capítulo te va a resolver el problema. Lo que necesitas vive en el capítulo siguiente: una **capa de arquitecto** sobre tu agente actual.

## Lo que ninguna de estas herramientas resuelve

Hay un patrón que vale la pena nombrar y que ninguna de las cuatro herramientas resuelve por sí sola: **el problema de la verificación post-código**. Las cuatro te ayudan a estructurar la spec y el plan; ninguna verifica con suficiente rigor que el código generado **cumple** la spec original. Esa verificación, cuando se hace, es típicamente otra ronda de prompt manual.

Esa es exactamente la grieta donde Traycer (capítulo 8) y otras herramientas del patrón "capa de arquitecto" se han posicionado.

## Lo que viene a continuación

El capítulo 8 cubre la **otra categoría** de herramientas SDD: las que no reemplazan tu agente sino que lo envuelven. Es una distinción importante porque responde a una pregunta distinta. Las herramientas de este capítulo responden a "¿qué herramienta uso?". Las del próximo responden a "¿cómo mejoro la herramienta que ya estoy usando?".
