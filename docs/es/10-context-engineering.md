# 10. Context engineering como alternativa

El capítulo anterior cierra con seis críticas serias al SDD. Sería tramposo dejarte ahí, en la crítica sin alternativa. Por suerte, los mismos críticos — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development) en particular — proponen una idea distinta sobre dónde debe vivir la intención y los "por qués". La llaman **context engineering**, y este capítulo la presenta sin endulzarla pero también sin dramatizar la diferencia con SDD.

> *Captura la intención y las restricciones de las discusiones, actualiza el contexto iterativamente a medida que evoluciona la comprensión, y preserva explícitamente las razones de las decisiones dentro del propio código.*
> — [Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development), *The Limits of Spec-Driven Development*

Esa frase resume el desplazamiento.

## La diferencia esencial con SDD

SDD pone la intención **fuera del código**, en specs que el agente lee. Context engineering pone la intención **dentro del código y de los artefactos directamente legibles** del proyecto: comentarios estructurados, ADRs (Architecture Decision Records), commit messages ricos, notas en el README y, sí, instrucciones puntuales en archivos tipo `AGENTS.md` que viven en el repo pero **sin la pretensión de ser una fuente de verdad paralela**.

La diferencia parece pequeña pero tiene tres consecuencias profundas:

1. **No hay segundo sistema que mantener.** Las decisiones viven donde vive el código, así que no puedes desincronizarlas porque son la misma cosa. El *maintenance tax* del capítulo 9 desaparece por construcción.
2. **Los por qués no se pierden** porque están adheridos al código que los implementa. Quitas la unidad lógica y se va con ella.
3. **No hay falsa ilusión de completitud** porque no hay un documento maestro que dé sensación de cobertura. La cobertura es exactamente la del código.

## Qué cosas hace context engineering

Context engineering no es "no hagas nada". Es un conjunto de prácticas concretas, todas orientadas a **dejar trazas legibles dentro del propio sistema**:

### 1. ADRs (Architecture Decision Records)

Cada decisión arquitectónica importante se captura como un ADR corto: contexto, decisión, alternativas consideradas, consecuencias. Vive en `docs/adr/` o equivalente, versionada como código. No describe el sistema entero — describe **decisiones**, una por documento, con fecha y dueño.

La diferencia con una spec: una spec dice "el sistema hace esto"; un ADR dice "elegimos hacerlo así por estas razones". ADRs son los por qués pelados, sin los qués. Y cuando un agente lee un ADR antes de tocar el módulo correspondiente, obtiene exactamente lo que la crítica de Isoform decía que las specs no daban.

### 2. Commit messages que explican

Un commit message rico — no "fix" o "update", sino tres frases que dicen *qué* cambia, *por qué* y *qué efecto secundario tiene* — es context engineering en su forma más barata. `git blame` se vuelve un sistema de recuperación de intención. Cualquier agente futuro que pregunte "¿por qué esta línea?" tiene la respuesta a un comando de distancia.

### 3. Comentarios sobre intención, no sobre mecánica

Los comentarios tradicionales explican *qué hace* el código. Context engineering pide comentarios que expliquen *por qué* el código hace lo que hace, especialmente cuando la decisión tiene una razón no obvia.

```python
# No usamos retry exponencial aquí porque el cliente
# de upstream tiene su propio backoff y los dos
# concatenados generan timeouts duplicados.
# (Decisión post-incidente del 2025-11-12.)
```

Ese comentario es context engineering puro: vive con el código, explica un por qué, y ningún agente futuro va a "limpiar" el retry por error porque la razón está justo ahí.

### 4. AGENTS.md (o CLAUDE.md) como mapa, no como spec

Un archivo `AGENTS.md` corto en la raíz del repo, que oriente al agente sobre **dónde están las cosas** y **qué convenciones se siguen**, es context engineering bien hecho. No es una spec del sistema; es un mapa de orientación para alguien que llega de fuera. Idealmente cabe en una pantalla.

La línea entre "AGENTS.md como mapa" y "AGENTS.md como spec gigante" es exactamente la línea entre context engineering y SDD malhecho. Cuando AGENTS.md crece más allá de orientación y empieza a describir el sistema, te estás deslizando hacia la disciplina de specs por la puerta de atrás, con todos sus costes.

## En qué se parece a SDD y en qué no

Para evitar que parezca que son enemigos irreconciliables — no lo son — vale la pena tabular en qué coinciden y en qué difieren.

| Dimensión | SDD | Context engineering |
|---|---|---|
| Dónde vive la intención | Specs externas | Dentro del código y del repo |
| Mantenimiento sincronizado | Sí, explícito | No requerido por construcción |
| Captura los por qués | A veces (depende de disciplina) | Siempre, por diseño |
| Granularidad | Por feature/módulo | Por decisión |
| Visible en revisión de código | No directamente | Sí, en cada PR |
| Soporta spec-as-source | Sí (Tessl) | No |
| Coste de adopción | Alto | Bajo |
| Madurez de la práctica | En formación | Décadas (ADRs son de 2011) |

Los dos enfoques resuelven el mismo problema — el context collapse del capítulo 1 — con filosofías distintas. SDD apuesta por la **persistencia explícita**; context engineering apuesta por la **adherencia al sustrato**.

## Cuándo elegir context engineering en lugar de SDD

Hay situaciones donde context engineering es objetivamente la mejor opción:

- **Equipos pequeños** donde el coste de mantener specs externas es desproporcionado.
- **Proyectos en fase de descubrimiento** donde la intención cambia más rápido de lo que se puede escribir una spec.
- **Bases de código maduras con buenas convenciones** donde la mayor parte del contexto ya vive en el código y solo falta capturar los por qués.
- **Equipos que ya practican ADRs disciplinados** y querrían no añadir un segundo sistema documental encima.

Y hay situaciones donde context engineering se queda corto:

- **Sistemas regulados** donde la spec separada es un requisito de auditoría, no una elección estilística.
- **Coordinación entre múltiples equipos** donde hace falta un artefacto compartido que no esté enterrado en commits.
- **Features grandes con alcance amplio** donde el plan formal previo aporta más que ADRs sueltos.

## La síntesis honesta: usa los dos

Una de las cosas que más equipos descubren después de leer las dos posiciones es que **no son alternativas excluyentes**. El equipo maduro acaba haciendo una mezcla:

- **ADRs y commit messages ricos como capa base**, siempre, para todo. Esto es context engineering puro y es gratis (o muy barato).
- **Specs ligeras estilo capítulo 3** solo para features grandes con alcance amplio y vida larga, siguiendo el patrón 1 del capítulo 8.
- **Specs vivas con bucle bidireccional** solo para los módulos donde el coste de mantenimiento se amortiza por su criticidad.
- **Nada de eso** para refactors menores y bug fixes triviales.

Esto puede sonar a "haz lo que te dé la gana", y es exactamente lo opuesto. Es **modulación deliberada** del nivel de formalidad según el coste y el riesgo del cambio. Es la habilidad que distingue a un equipo que aplica SDD bien de uno que lo sufre universalmente.

## Una nota sobre la palabra "context engineering"

El término está ganando tracción precisamente porque captura algo que el SDD no: que la batalla real no es escribir specs, es **gestionar el contexto** que llega al agente en el momento de actuar. Y el contexto puede venir de muchos sitios: specs, ADRs, commits, comentarios, AGENTS.md, búsquedas en el código, RAG sobre el repo, y herramientas de retrieval. Llamar a todo eso "context engineering" es honesto sobre lo que realmente está pasando.

SDD es una **estrategia concreta** dentro de context engineering: la estrategia que elige specs externas como mecanismo principal. Y como toda estrategia, tiene contextos donde brilla y contextos donde estorba. Visto así, las dos posiciones del capítulo 9 y este capítulo no son enemigos: son dos puntos del mismo espectro de soluciones para el mismo problema, y la elección depende del contexto del equipo.

## Lo que viene a continuación

El **capítulo 11** es el más corto del curso y el más operativo: una lista de **anti-patrones** del SDD que verás en equipos reales, con su nombre, su síntoma y la corrección. Es la parte que querrás imprimir y poner en la pared del equipo.
