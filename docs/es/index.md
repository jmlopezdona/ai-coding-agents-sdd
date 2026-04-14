# Spec-Driven Development

Una guía para equipos técnicos que ya saben usar agentes de codificación (Claude Code, Codex, Cursor, Copilot…) y han chocado con el muro: los resultados son inconsistentes, el agente se desvía a media sesión, los refactors rompen archivos que el agente nunca vio, y "escribe un mejor prompt" dejó de funcionar hace tiempo.

Es la **segunda guía de una trilogía**: viene después de la [guía de fundamentos](https://jmlopezdona.github.io/ai-coding-agents-fundamentals/es/) (qué es un agente, cómo se usa) y antes de la [guía de harness engineering](https://jmlopezdona.github.io/ai-coding-agents-harness/es/) (cómo se implementa el sistema alrededor del agente). El SDD es la disciplina intermedia: **cómo estructurar el trabajo para que el agente pueda ejecutarlo bien**.

## Premisas

- Sabes pilotar un agente y has visto sus límites en proyectos reales.
- No buscas un producto mágico, buscas una disciplina.
- Estás dispuesto a pagar el coste de escribir specs si eso evita el coste mayor de retrabajo, drift y deuda invisible.
- Quieres criterio, no dogma: te interesa saber cuándo SDD ayuda y cuándo es burocracia.

## Tesis central

> El cuello de botella ya no es escribir código. Es **especificar la intención** con la suficiente claridad como para que un agente la pueda ejecutar — y mantener esa especificación viva mientras el código evoluciona.

## Cómo leer esta guía

Los capítulos están ordenados de **problema → fundamentos → ciclo → práctica → puente**, pero cada uno se sostiene por sí solo. Si ya tienes claro el "por qué", salta al capítulo que te resuelva un problema concreto. Si vienes de cero, léelos en orden.

Hay dos capítulos que **no son opcionales** aunque parezcan negativos: el 9 (la crítica honesta) y el 10 (code-embedded context como complemento). Sin ellos, el curso es marketing. Con ellos, da criterio.

## Índice

### Capítulo 0 — Puerta de entrada

- [Spec-Driven Development: producir software con intención](00-spec-driven-development.md)

### Parte 1 — El problema

- [1. Del vibe coding al SDD: context collapse y context drift](01-from-vibe-coding-to-sdd.md)

### Parte 2 — Fundamentos

- [2. El espectro de la especificación](02-specification-spectrum.md)
- [3. Anatomía de una buena especificación](03-anatomy-of-a-spec.md)
- [4. La spec en contexto](04-spec-in-context.md)

### Parte 3 — Ciclo de vida

- [5. Posibles enfoques del ciclo de vida](05-sdd-lifecycles.md)
- [6. Especificaciones vivas: el bucle bidireccional](06-living-specs.md)

### Parte 4 — Puesta en práctica

- [7. Herramientas SDD: Kiro, Spec-kit, Tessl, BMAD y Traycer](07-native-sdd-tools.md)
- [8. Patrones de aplicación: features, refactor, bugfix](08-patterns-of-application.md)
- [9. La crítica honesta: maintenance tax, MDD y la ilusión de control](09-honest-critique.md)
- [10. Code-embedded context: la otra cara de la moneda](10-context-engineering.md)
- [11. Anti-patrones del SDD](11-anti-patterns.md)

### Parte 5 — Qué sigue

- [12. De la disciplina a la fábrica de software](12-from-sdd-to-harness.md)

## Qué *no* encontrarás en esta versión

- Comparativas exhaustivas de herramientas. El ecosistema cambia cada trimestre; el capítulo 7 es un mapa, no una guía de compra.
- Plantillas de spec listas para producción. Hay esqueletos en los capítulos 3 y 9, pero la spec útil se escribe contra tu dominio, no contra una plantilla genérica.
- Promesas de productividad. Si alguien te promete 10×, probablemente no ha leído el capítulo 9.

## Fuentes

Esta guía sintetiza ideas de:

- [*Spec-Driven Development: From Code to Contract in the Age of AI Coding Assistants*](https://arxiv.org/html/2602.00180) — arXiv
- [*A Survey on Code Generation with LLM-based Agents*](https://arxiv.org/html/2508.00083v1) — arXiv 2508.00083
- Addy Osmani — [*How to write a good spec for AI agents*](https://addyosmani.com/blog/good-spec/)
- Augment Code — [*How to Write Living Specs for AI Agent Development*](https://www.augmentcode.com/guides/living-specs-for-ai-agent-development)
- Pockit Tools — [*Specification-Driven Development: How to Stop Vibe Coding*](https://dev.to/pockit_tools/specification-driven-development-how-to-stop-vibe-coding-and-actually-ship-production-ready-5788)
- r/vibecoding — [*Everything one should know about Spec-Driven Development*](https://www.reddit.com/r/vibecoding/comments/1qs80k4/everything_one_should_know_about_specdriven/)
- Isoform — [*The Limits of Spec-Driven Development*](https://isoform.ai/blog/the-limits-of-spec-driven-development)
- Martin Fowler / Thoughtworks — [*Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
