# 4. La spec en contexto: detalle, componentes y calibración

En la [sección anterior](03-anatomy-of-a-spec.md) vimos qué poner dentro de una spec — los seis bloques, la plantilla, la maldición de las instrucciones y las trampas de la generación con agente. Ahora toca la pregunta que sigue: **cuánto detalle**, y **cómo se relaciona la spec con sus fuentes externas** — documentos de producto, decisiones arquitectónicas, componentes de código.

## El nivel de detalle depende del nivel del espectro

Hasta aquí hemos hablado de la anatomía de una spec como si fuera una sola cosa. No lo es. **El nivel de detalle apropiado depende de en qué nivel del espectro del capítulo 2 estés operando**, y escribir una spec con el nivel de detalle equivocado para tu nivel del espectro es una de las formas más comunes — y más caras — de sufrir SDD malhecho.

### Spec-first → detalle ligero, intencional, no exhaustivo

En spec-first la spec se lee una vez, al arrancar la feature, y a partir de ahí el código deriva libremente. Su única función es **alinear al equipo y al agente al principio**. Nadie la va a leer de vuelta, así que cada línea extra que escribas es trabajo que nadie va a recuperar.

El detalle apropiado: objetivo, no-goals, criterios de aceptación, los por qués críticos, y poco más. Si tu spec-first ocupa más de una pantalla, casi siempre es porque estás escribiendo spec-anchored *aspiracional* ([anti-patrón #4](12-anti-patterns.md#4-spec-teatral-con-anclaje-fingido) del capítulo 12) o porque caíste en pseudocódigo ([anti-patrón #12](12-anti-patterns.md#12-spec-generada-que-es-pseudocodigo-disfrazado)). La spec-first óptima es la mínima viable para arrancar con intención clara.

### Spec-anchored → detalle medio, acotado por lo que el anclaje puede verificar

Aquí cambia la lógica. La spec sí se va a leer otra vez — por los validadores, por los tests, por los agentes recurrentes que detectan drift. El detalle ya no es opcional: tiene que ser **suficiente para que el mecanismo de anclaje pueda comparar**.

Pero hay un tope superior contraintuitivo: **el detalle no debe ir más allá de lo que tu anclaje sabe verificar**. Si tu validador comprueba contratos de API y tu spec describe reglas de UI, la parte de UI no está anclada — es spec-first disfrazada de spec-anchored. Y como ese trozo no se verifica, drifta libre.

La regla operativa: el detalle de una spec-anchored se mide contra el **alcance del anclaje**, no contra una idea abstracta de "completitud". Si lo que escribes no se puede verificar automáticamente, escribirlo no te aporta más anclaje — solo te aporta más maintenance tax.

### Spec-as-source → detalle exhaustivo, pero de un tipo distinto

Aquí es donde la conversación se vuelve interesante. Spec-as-source sí necesita el máximo detalle, porque el código se genera a partir de la spec. Pero el detalle es de **una naturaleza diferente** del de los otros dos niveles.

Es detalle **formal o semi-formal**: signaturas de tipos, contratos, invariantes, gramáticas, reglas de transformación. Es detalle **generator-friendly**: pensado para que un generador (LLM o no) pueda producir código determinista a partir de él. Y es legítimo que aparezcan firmas, esquemas y estructuras concretas — porque en este nivel **la spec es el código fuente, solo que en otra notación**.

Aquí está la conexión incómoda con la [sección anterior](03-anatomy-of-a-spec.md#y-si-la-spec-la-genera-un-agente): la "spec" generada por un agente que es pseudocódigo disfrazado se *parece* superficialmente a una spec-as-source. Tiene clases, métodos, payloads, reglas. Pero hay una diferencia crítica: **una spec-as-source legítima viene acompañada de un generador determinista que produce el código**. Sin ese generador, lo que tienes es la peor combinación posible: el detalle exhaustivo de spec-as-source con el no-determinismo de spec-first. Es el equivalente de escribir especificaciones formales exhaustivas sin tener un generador que las ejecute — exactamente lo que [Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) criticó del Model-Driven Development original: formalismo sin ejecución que lo respalde.

### La regla unificadora

Si tienes que destilar todo lo anterior a una sola frase:

> **El nivel de detalle apropiado de una spec es el que tu mecanismo de validación — humano, determinista o generativo — sabe consumir. Más detalle que eso es desperdicio; menos detalle es ceguera.**

En spec-first el "validador" es el equipo en una sola lectura inicial, así que el detalle apropiado es lo que cabe en esa lectura. En spec-anchored el validador es el mecanismo de anclaje, así que el detalle apropiado es lo que el anclaje sabe comparar. En spec-as-source el validador es el generador, así que el detalle apropiado es lo que el generador necesita para producir código sin ambigüedad.

Casi todas las patologías que veremos en el capítulo 12 vienen de **desalinear estas tres cosas**: escribir spec-anchored sin anclaje ([#4](12-anti-patterns.md#4-spec-teatral-con-anclaje-fingido)), escribir spec-as-source sin generador ([#9](12-anti-patterns.md#9-promocionar-prematuramente-a-spec-as-source) + [#12](12-anti-patterns.md#12-spec-generada-que-es-pseudocodigo-disfrazado)), o escribir spec-first con el detalle de spec-as-source ([#2](12-anti-patterns.md#2-big-spec-up-front)). La anatomía no es absoluta — es relativa a qué tipo de validación se va a hacer sobre lo que escribes.

## La spec y sus fuentes externas: consumir, producir, modificar

Toda spec se relaciona con artefactos que ya existen — documentos de producto, decisiones arquitectónicas, componentes de código. La taxonomía es siempre la misma: **¿lo consumes, lo produces o lo modificas?** Entender qué relación tienes con cada artefacto externo determina qué va en la spec, cuánto detalle pones, y dónde cae dentro de los seis bloques.

### Tres relaciones, tres reglas

**1. Consumido (existe; la spec lo usa sin modificarlo).** Aplica tanto a componentes de código (un servicio existente, una librería) como a documentos (una user story, un ADR, un contrato de API). La regla es **referencia el contrato o el artefacto, no lo reimplementes**. La spec documenta *cómo* lo usa esta feature, no lo que el artefacto hace por sí mismo — eso vive en su propia fuente de verdad.

Un matiz importante: **cita el invariante específico del que esta spec depende, no el documento entero**. *"Esta spec respeta ADR-007 (single Postgres instance) y ADR-012 (no llamadas síncronas entre servicios)"* es útil. *"Relacionado: ADR-007"* es ruido.

**2. Producido (no existe; la spec lo crea).** Es el caso típico de una feature nueva: *"esta spec crea un nuevo endpoint / un nuevo servicio / un nuevo módulo"*. Aquí la trampa es obvia: como no existe todavía, sientes que tienes que "definirlo" en la spec, y casi siempre acabas describiéndolo con clases, métodos, firmas y payloads. Eso es exactamente el [anti-patrón #12](12-anti-patterns.md#12-spec-generada-que-es-pseudocodigo-disfrazado) (pseudocódigo disfrazado de spec).

La forma correcta: la spec describe el **contrato observable** que el artefacto nuevo tiene que ofrecer, no su estructura interna. *"Debe existir una capacidad para aceptar uploads de avatares autenticados (JPEG/PNG, ≤10 MB) y devolver una URL recuperable"* es contrato observable. *"Crear `AvatarUploadService` con método `upload(file, user_id) -> AvatarMetadata`"* es pseudocódigo.

**Cuando ya existe un documento de diseño previo** (un tech design, un documento de interfaces, un contrato de API diseñado antes de la implementación), el componente a nivel de código se *produce*, pero a nivel de diseño se *consume*. En ese caso, la spec referencia el documento de diseño y describe solo las implicaciones para esta feature — exactamente como con cualquier artefacto consumido. La regla de "definir el contrato observable" aplica solo cuando no hay ningún diseño previo que ya lo haga.

Un matiz conceptualmente importante: una spec que produce un artefacto nuevo es **la fuente de verdad transitoria que hace handoff**. En cuanto el artefacto existe, **su propia documentación** se vuelve la fuente operacional, y la spec pasa a ser el *"por qué se construyó así"* — intención histórica, no contrato vivo. Confundir los dos roles lleva a la fusión silenciosa del [anti-patrón #13](12-anti-patterns.md#13-fusionar-user-story-y-spec-en-un-solo-archivo).

Un ejemplo concreto de este handoff: las APIs. Cuando una API está por implementar, el documento de diseño (un borrador de contrato, un tech design) es la fuente transitoria y la spec lo referencia. Pero una vez implementada, si el stack tecnológico genera documentación a partir del código (OpenAPI/Swagger generado desde anotaciones, por ejemplo), esa documentación generada pasa a ser la **fuente viva** — es la que refleja lo que el código realmente hace. A partir de ese momento, la spec y cualquier referencia futura deben apuntar al artefacto generado, no al documento de diseño original, que puede haber quedado obsoleto sin que nadie lo detecte.

**3. Modificado (existe; la spec añade, cambia o quita capacidades).** La regla es **describe el delta observable, no el estado completo**. La spec no tiene que volver a documentar cómo funciona `User`; tiene que decir *"añade el campo `avatar_url` (opcional). Se actualiza al subir un avatar y se borra al borrarlo. Ningún otro comportamiento del modelo cambia."*

La parte de **"ningún otro comportamiento cambia"** es lo que salva a la spec de inflarse. El delta es la spec; el resto vive en su propia documentación.

Ese delta explícito es también una traza para un doc-gathering agent: puede comparar lo que la spec declaró contra el estado real del componente y detectar dos tipos de deriva — que el código cambió más de lo que la spec declaró, o que la documentación del componente no se actualizó para reflejar el delta.

> **En resumen**: *consumido* = referencia y describe solo el uso; *producido* = define el contrato observable y prepara el handoff; *modificado* = describe el delta y declara el resto invariante.

### Caso especial: documentos de producto (user stories, briefs)

Los documentos de producto se *consumen* — la spec se alimenta de ellos pero no los modifica. La pregunta es **cuánto gap hay entre el upstream y lo que la spec necesita**, y eso depende del rigor con el que se hayan elaborado los requisitos funcionales.

En un extremo, una user story dice *"el usuario puede subir una foto fácilmente"* — la spec tiene que derivar criterios de ingeniería completos: tipos de archivo, tamaño máximo, códigos de error, edge cases. En el otro extremo, unos criterios de aceptación bien elaborados ya cubren las reglas de negocio, los casos no felices y los límites funcionales — la spec solo necesita referenciarlos y añadir las restricciones técnicas que el upstream no cubre (qué middleware usar, qué componentes no tocar, qué patrones respetar).

En el contexto de trabajo ágil, la user story y los criterios de aceptación son el **inicio de la conversación** con los desarrolladores para refinar el detalle funcional. SDD no cambia eso — lo que cambia es que el gap, sea del tamaño que sea, tiene que quedar cubierto **explícitamente** en algún sitio (la spec), no implícitamente en la cabeza del desarrollador. Cuanto más rigor haya en los requisitos funcionales upstream, menos trabajo de derivación necesitará la spec.

Cómo incorporar el upstream depende de si el agente puede acceder al documento original:

**Sin acceso** (Jira sin MCP, Notion sin API, un documento suelto): **derivar con traza**. Escribes tus propios criterios precisos en la spec y citas la user story en la sección de "por qués" como fuente del *qué motivó la decisión*. Referenciar a secas (`"ver JIRA-1234"`) rompe la auto-contención del capítulo 1 — el agente no puede leer ese enlace, y la user story puede cambiar sin que nadie lo detecte.

**Con acceso** (mismo repo Git o MCP a Jira/Linear/Notion): no necesitas copiar — pero sí necesitas **derivar tus propios criterios al nivel de precisión de la spec** y referenciar el origen. En este escenario el drift deja de ser invisible (queda un commit o un timestamp), y aparece la posibilidad de un **sensor automático** que detecte cuando la user story cambia sin que la spec se actualice.

Un riesgo concreto en ambos casos: la tentación de editar ambos documentos con frecuencia sin que cada cambio esté plenamente justificado. Cada edición cruzada es una oportunidad de desalineamiento. La traza cumple un doble propósito: **punto de comprobación en el momento del cambio** y **material de auditoría** para un agente de doc-gathering que busque discrepancias de forma recurrente.

La regla: **la spec siempre contiene sus propios criterios, derivados al nivel de precisión que la validación necesita. El documento upstream se cita como fuente del por qué, no como contenedor del qué.** El anti-patrón a evitar — *fusionar user story y spec en un híbrido* — lo desarrollamos como [anti-patrón #13](12-anti-patterns.md#13-fusionar-user-story-y-spec-en-un-solo-archivo) en el capítulo 12.

### Caso especial: documentos con validación automática contra el código

Cuando el artefacto consumido se valida automáticamente contra el código de forma continua (un contrato OpenAPI con CI, un esquema Protobuf con checks de compatibilidad), basta con referenciar y resumir las implicaciones para tu feature. Una user story de Jira no tiene validación automática contra el código — su consistencia depende de disciplina humana puntual, lo que la hace vulnerable a drift silencioso.

### Cuánto detalle poner: tres dimensiones

Para cada artefacto que la spec produce o modifica, la pregunta es *"¿cuánto detalle?"*. La respuesta útil es hacerse **tres preguntas explícitas** y capturar solo lo load-bearing:

- **Funcional** — qué cambia en el comportamiento observable. *"El modelo `User` ahora puede tener un avatar; el campo es opcional; se actualiza al subir y se elimina al borrar."*
- **No funcional** — qué restricciones cruzadas tiene que respetar el cambio. *"La migración no debe requerir downtime; el campo no debe romper la serialización existente."*
- **Técnico** — qué decisiones de contrato/integración el cambio cierra explícitamente. *"El campo es indexado por `user_id`; la URL pasa por el CDN existente."*

Para cada dimensión, una **referencia opcional al documento más profundo** si existe. Si una dimensión no aporta, omítela — la ausencia es informativa.

Dos aclaraciones:

1. **Lo que va en la spec es lo load-bearing**: lo que el validador del cap. 6 puede comparar contra el código. El detalle más fino vive en la documentación del artefacto.
2. **El término "no funcional" es clásico pero contestado** ([Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) argumenta que todo es funcional, solo cambia la dimensión). Lo usamos como categoría operativa — tres preguntas distintas — no como ontología.

### Dónde va esta información en los seis bloques

La taxonomía consumir/producir/modificar y las tres dimensiones no crean una sección nueva en la spec — ayudan a escribir mejor las secciones que ya existen:

- **Artefactos consumidos** (componentes, ADRs, contratos, user stories) → van en **restricciones técnicas** (los límites) y en **por qués** (la motivación derivada del upstream).
- **Artefactos producidos o modificados** → sus garantías observables van en **criterios de aceptación**; sus límites de diseño van en **restricciones técnicas**.
- **Las tres dimensiones** son las preguntas que te haces al escribir cada criterio o restricción, no subsecciones dentro de la spec.

### Heurística de tamaño: cuándo la spec necesita partirse

Si al escribir restricciones y criterios descubres que la spec produce o modifica más de **3-4 artefactos**, probablemente tienes uno de tres problemas:

1. Una feature demasiado grande que debería partirse en varias specs (modulación del cap. 9).
2. Un cambio arquitectónico disfrazado de feature, que merece un ADR separado.
3. Un refactor camuflado de feature (también modulación del cap. 9): lo que estás cambiando es la *forma* del sistema, no añadiendo capacidad de negocio.

### Tabla resumen por tipo de relación

| Relación | Regla principal | Sección de la spec | Trampa típica |
|---|---|---|---|
| **Componente consumido** | Referencia el contrato; documenta el uso | Restricciones técnicas | Re-documentar el componente |
| **Componente producido** | Define contrato observable, no estructura | Criterios + Restricciones | Pseudocódigo (#12); no marcar el handoff |
| **Componente modificado** | Describe el delta, declara el resto invariante | Criterios + Restricciones | Reescribir el componente entero |
| **ADR / arquitectura** | Cita invariantes específicos | Restricciones técnicas | Copiar invariantes (drift garantizado) |
| **API contract / estándar** | Referencia + resumen de implicaciones | Restricciones técnicas | Re-documentar el contrato |
| **Doc de producto / brief** | Extrae los por qués cardinales | Por qués | Importar lenguaje vago de producto |
| **User story** | Derivar criterios con traza | Criterios + Por qués | Copia literal o referencia opaca |

Y un anti-patrón que aparece cuando las referencias se acumulan sin disciplina: **sopa de referencias** — una spec ahogada por sus propias citas, donde el agente y el humano no saben cuáles son load-bearing y acaban ignorándolas todas. Lo desarrollamos como [anti-patrón #14](12-anti-patterns.md#14-sopa-de-referencias) en el capítulo 12. La regla preventiva: **referencia solo lo que el agente o el humano necesita para esta tarea, y anota el porqué de cada referencia**.

## En resumen: lo que distingue una spec buena de una mediocre

Si tienes que recordar tres cosas de estas dos secciones, que sean estas:

1. **Los no-goals te ahorran más drift que cualquier otra sección**. Escríbelos siempre.
2. **Los por qués son lo que envejece bien**. Sin ellos, la spec dura semanas. Con ellos, dura años.
3. **El "ask first" es la categoría que falta en casi todas las specs**. Es donde vive la colaboración real con el agente.

### Mediocre vs buena, lado a lado

La diferencia se ve mejor en contraste. Misma feature, dos specs:

**Mediocre** (parece completa, no lo es):

> *Spec: Avatar upload. El sistema permitirá a los usuarios subir un avatar. Se creará un endpoint `POST /api/avatar` que recibirá el archivo y lo guardará. Se validará que el archivo sea una imagen. Se devolverá la URL del avatar guardado. Se añadirá un campo `avatar_url` al modelo `User`. Se implementarán tests.*

Habla del *qué* en términos de implementación, no menciona ningún *por qué*, no hay no-goals, no hay boundaries, los criterios no son verificables ("se validará que el archivo sea una imagen" — ¿qué tipos? ¿qué tamaño? ¿qué error?), y menciona detalles de código (`POST /api/avatar`, `avatar_url`) que son decisiones de implementación, no contrato.

**Buena** (corta, intencional, envejece):

> **Spec: Avatar upload**
>
> **Objetivo:** permitir a un usuario autenticado subir una foto de perfil visible para sus seguidores y eliminable solo por él.
>
> **No-goals:** no se soporta GIF animado; no se comprime automáticamente; la moderación de contenido vive en otra spec.
>
> **Restricciones:** se sirve desde el bucket S3 existente; auth pasa por `requireAuth`; no se crea tabla nueva, el avatar es un campo en `User`.
>
> **Criterios:** un usuario autenticado puede subir JPEG o PNG ≤10 MB; falla con 413 si excede tamaño y 415 si tipo inválido, ambos con mensaje legible; solo el dueño puede borrar; un test de integración cubre los tres casos.
>
> **Por qués:** no comprimimos porque diseño quiere preservar calidad para cuentas verificadas (sept-2025, @maria); solo el dueño borra porque añadir moderación cambiaría el modelo de permisos y queremos esta iteración mínima.
>
> **Boundaries:** ✅ ejecutar tests antes de declarar terminado · ⚠️ preguntar antes de tocar el esquema de `User` · 🚫 nunca deshabilitar tests existentes ni commitear secretos.

La buena cabe en la misma extensión que la mediocre, pero captura intención, restricciones, criterios verificables y los por qués que la harán sobrevivir a las decisiones que la rodean.

**La regla final**: las specs malas enumeran trivialidades técnicas y no dicen nada de la intención. Las specs mediocres dicen mucho del *qué* y nada del *por qué*. Las specs buenas son cortas, dicen *qué*, *por qué* y *qué no*, y se pueden leer en cinco minutos sin perder nada importante.

## Lo que viene a continuación

Hasta aquí hemos visto qué hay *dentro* de una spec y cómo calibrar su detalle según el contexto. La regla unificadora: **escribe solo el detalle que tu mecanismo de validación sabe consumir**. Si la anatomía responde al *qué*, lo siguiente es el *cuándo* y el *cómo se valida*: el momento en que una spec deja de ser documento y empieza a comportarse como contrato.

En el **capítulo 5** vamos a ver el ciclo en el que una spec vive: las cuatro fases del proceso SDD (con sus dos variantes en la literatura), cómo encadenarlas con un agente, y dónde encaja la verificación que evita que el código se aleje de la spec.
