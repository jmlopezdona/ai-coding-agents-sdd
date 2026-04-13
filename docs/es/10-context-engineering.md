# 10. Code-embedded context: la otra cara de la moneda

El capítulo anterior cierra con seis críticas serias al SDD. Sería tramposo dejarte ahí, en la crítica sin complemento. Los mismos críticos — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development) en particular — proponen una idea distinta sobre dónde debe vivir la intención y los "por qués". La llaman "context engineering" (un término que merece matización, como veremos al final del capítulo), y este capítulo la presenta como lo que realmente es: **no una alternativa al SDD, sino su complemento natural** — la otra cara de la misma moneda.

> *Captura la intención y las restricciones de las discusiones, actualiza el contexto iterativamente a medida que evoluciona la comprensión, y preserva explícitamente las razones de las decisiones dentro del propio código.*
> — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development), *The Limits of Spec-Driven Development*

Esa frase resume el enfoque.

## En qué se diferencia del SDD

SDD pone la intención en **specs que son documentos paralelos al código**. Lo que Isoform propone es distribuir esa intención en **artefactos con distintos niveles de acoplamiento al propio código**: desde comentarios inline (máximo acoplamiento) hasta ADRs (acoplamiento bajo, similar al de una spec).

Esto no es una dicotomía binaria "dentro vs fuera del código". Es un **gradiente de acoplamiento**:

| Artefacto | Acoplamiento al código | Riesgo de desincronización |
|---|---|---|
| Comentario inline | Máximo — mismo archivo, misma línea | Muy bajo |
| Commit message | Alto — ligado al diff exacto vía `git blame` | Ninguno (inmutable) |
| AGENTS.md / CLAUDE.md | Medio — en el repo, orientativo | Bajo si se mantiene corto |
| ADR | Bajo — documento separado sobre decisiones | Medio (igual que una spec ligera) |
| Spec en el repo | Bajo — documento separado sobre implementación | Medio-alto |
| Spec en Confluence/Notion | Mínimo — fuera del repo | Alto |

Hay que ser honesto: un ADR en `docs/adr/0005-auth-strategy.md` es un documento separado que describe decisiones sobre código que vive en otro sitio — **puede desincronizarse exactamente igual que una spec**. Lo que cambia entre un ADR y una spec no es su adherencia al código, sino **qué captura** (decisiones puntuales vs implementación completa) y **su ambición** (un por qué concreto vs una fuente de verdad del sistema).

La propuesta de Isoform no es "todo adherido al código" — es **preferir los artefactos de la parte alta del gradiente** (comentarios, commits) y usar los de la parte baja (ADRs) solo para decisiones, no para describir el sistema entero. Las consecuencias prácticas:

1. **Menos superficie de mantenimiento.** No desaparece el *maintenance tax* del capítulo 9 por completo — los ADRs también pueden quedar obsoletos — pero se reduce porque los artefactos más acoplados (comentarios, commits) no pueden desincronizarse por construcción.
2. **Los por qués más críticos viajan con el código.** Un comentario de intención se borra cuando se borra la línea. Un commit message es inmutable. Los por qués que viven ahí no se pierden.
3. **No hay falsa ilusión de completitud** porque no hay un documento maestro que dé sensación de cobertura. La cobertura es exactamente la del código.

Una observación importante: **una spec que vive en el repositorio también es un artefacto del repo**, versionada con git, revisable en PRs. La frontera entre "spec" y "artefacto del proyecto" se desdibuja. La diferencia real no es dónde vive el archivo, sino el grado de acoplamiento con el código que describe y la ambición de lo que intenta capturar.

## Qué prácticas incluye

Code-embedded context no es "no hagas nada". Es un conjunto de prácticas concretas, todas orientadas a **dejar trazas legibles dentro del propio sistema**:

### 1. ADRs (Architecture Decision Records)

Cada decisión arquitectónica importante se captura como un ADR corto: contexto, decisión, alternativas consideradas, consecuencias. Vive en `docs/adr/` o equivalente, versionada como código. No describe el sistema entero — describe **decisiones**, una por documento, con fecha y dueño.

La diferencia con una spec: una spec dice "el sistema hace esto"; un ADR dice "elegimos hacerlo así por estas razones". ADRs son los por qués pelados, sin los qués. Y cuando un agente lee un ADR antes de tocar el módulo correspondiente, obtiene exactamente lo que la crítica de Isoform decía que las specs no daban.

### 2. Commit messages que explican

Un commit message rico — no "fix" o "update", sino tres frases que dicen *qué* cambia, *por qué* y *qué efecto secundario tiene* — es code-embedded context en su forma más barata. `git blame` se vuelve un sistema de recuperación de intención. Cualquier agente futuro que pregunte "¿por qué esta línea?" tiene la respuesta a un comando de distancia.

### 3. Comentarios sobre intención, no sobre mecánica

Los comentarios tradicionales explican *qué hace* el código. Code-embedded context pide comentarios que expliquen *por qué* el código hace lo que hace, especialmente cuando la decisión tiene una razón no obvia.

```python
# No usamos retry exponencial aquí porque el cliente
# de upstream tiene su propio backoff y los dos
# concatenados generan timeouts duplicados.
# (Decisión post-incidente del 2025-11-12.)
```

Ese comentario es code-embedded context puro: vive con el código, explica un por qué, y ningún agente futuro va a "limpiar" el retry por error porque la razón está justo ahí.

### 4. AGENTS.md (o CLAUDE.md) como mapa, no como spec

Un archivo `AGENTS.md` corto en la raíz del repo, que oriente al agente sobre **dónde están las cosas** y **qué convenciones se siguen**, es code-embedded context bien hecho. No es una spec del sistema; es un mapa de orientación para alguien que llega de fuera. Idealmente cabe en una pantalla.

La línea entre "AGENTS.md como mapa" y "AGENTS.md como spec gigante" es exactamente la línea entre code-embedded context y SDD malhecho. Cuando AGENTS.md crece más allá de orientación y empieza a describir el sistema, te estás deslizando hacia la disciplina de specs por la puerta de atrás, con todos sus costes.

## SDD y code-embedded context: comparativa

Para ver cómo encajan como complementos — no como enemigos — vale la pena tabular en qué coinciden y en qué difieren.

| Dimensión | SDD | Code-embedded context |
|---|---|---|
| Dónde vive la intención | Specs (documentos paralelos) | Distribuida en el gradiente de acoplamiento |
| Mantenimiento sincronizado | Sí, explícito | Reducido pero no eliminado (los ADRs también envejecen) |
| Captura los por qués | A veces (depende de disciplina) | Siempre, por diseño |
| Granularidad | Por feature/módulo | Por decisión |
| Visible en revisión de código | No directamente | Sí, en cada PR |
| Soporta spec-as-source | Sí (Tessl) | No |
| Coste de adopción | Alto | Bajo |
| Madurez de la práctica | En formación | Décadas (ADRs son de 2011) |

Los dos enfoques resuelven el mismo problema — el context collapse del capítulo 1 — con filosofías distintas. SDD apuesta por la **persistencia explícita en documentos paralelos**; code-embedded context apuesta por **distribuir la intención en artefactos con alto acoplamiento al código**.

## Cuándo es suficiente y cuándo necesitas specs

Code-embedded context — comentarios de intención, commits ricos, ADRs — es la **capa base que siempre debería estar**. La pregunta no es "¿cuál de los dos uso?" sino "¿necesito añadir specs formales encima?". La clave está en que cubren momentos distintos: code-embedded context captura los **por qués después** de las decisiones; las specs capturan la **intención antes** de implementar.

**Code-embedded context es suficiente cuando:**

- El cambio es lo bastante pequeño como para que el agente lo entienda con el código existente y un prompt bien escrito — refactors, bug fixes, features pequeñas.
- La intención se puede comunicar en el momento y no necesita sobrevivir semanas antes de implementarse.
- No hay múltiples personas que necesiten alinearse sobre qué se va a construir antes de que nadie escriba código.

**Necesitas añadir specs formales encima cuando:**

- La intención tiene que **sobrevivir en el tiempo** antes de implementarse — necesitas un artefacto que capture "qué vamos a hacer" y sea revisable por otros antes de que nadie escriba código.
- El alcance es **demasiado grande** para comunicarlo en un prompt — el agente necesita un documento de referencia que no cabe en un comentario ni en un commit.
- Hay **requisitos de auditoría o compliance** que exigen un artefacto separado y trazable.

## La síntesis honesta: usa los dos

Una de las cosas que más equipos descubren después de trabajar con ambos enfoques es que **funcionan mejor juntos que por separado**. El equipo maduro acaba haciendo una mezcla:

- **ADRs y commit messages ricos como capa base**, siempre, para todo. Esto es code-embedded context puro y es gratis (o muy barato).
- **Specs ligeras estilo capítulo 3** solo para features grandes con alcance amplio y vida larga, siguiendo el patrón 1 del capítulo 8.
- **Specs vivas con bucle bidireccional** solo para los módulos donde el coste de mantenimiento se amortiza por su criticidad.
- **Nada de eso** para refactors menores y bug fixes triviales.

Esto puede sonar a "haz lo que te dé la gana", y es exactamente lo opuesto. Es **modulación deliberada** del nivel de formalidad según el coste y el riesgo del cambio. Es la habilidad que distingue a un equipo que aplica SDD bien de uno que lo sufre universalmente.

## Una nota importante sobre la palabra "context engineering"

Hay que ser honesto con una colisión de términos. Cuando Isoform habla de "context engineering" en su artículo, se refiere a algo específico: **incrustar intención, decisiones y restricciones dentro del propio código y sus artefactos adyacentes** (ADRs, commits, comentarios de intención, AGENTS.md). Es una práctica de documentación ligada al código.

Pero el término **context engineering** tiene un significado mucho más amplio y anterior en la comunidad de AI. Popularizado por figuras como Andrej Karpathy y Simon Willison, se refiere a la **disciplina general de diseñar todo el contexto que llega al LLM en el momento de actuar**: system prompts, ejemplos few-shot, resultados de RAG, definiciones de herramientas, historial de conversación, y cualquier otro input que condicione la respuesta del modelo. Es la evolución natural de "prompt engineering" cuando el input dejó de ser un único prompt.

Las dos acepciones no son lo mismo:

| | Context engineering (general) | Lo que propone Isoform |
|---|---|---|
| **Alcance** | Todo input al LLM | Solo artefactos del repo |
| **Incluye** | System prompts, RAG, tools, few-shot, specs | ADRs, commits, comentarios, AGENTS.md |
| **Objetivo** | Que el agente actúe bien | Que la intención sobreviva en el código |
| **Quién lo hace** | Quien configura el agente | Quien escribe el código |

Lo que Isoform propone sería más preciso llamarlo **code-embedded context**: información relevante más allá de la propia codificación — los por qués, las restricciones, las decisiones — pero adherida al código en lugar de vivir en un documento externo como una spec. Es un subconjunto del context engineering general, no su sinónimo.

Dicho esto, tanto SDD como code-embedded context son **estrategias concretas dentro del context engineering general**. SDD elige specs externas como mecanismo principal para alimentar el contexto del agente; code-embedded context elige el propio código y sus artefactos adyacentes. La elección depende del contexto del equipo, y como veremos en la síntesis siguiente, no son excluyentes.

## Lo que viene a continuación

El **capítulo 11** es el más corto del curso y el más operativo: una lista de **anti-patrones** del SDD que verás en equipos reales, con su nombre, su síntoma y la corrección. Es la parte que querrás imprimir y poner en la pared del equipo.
