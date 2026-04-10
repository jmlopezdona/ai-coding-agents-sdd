# 4. La spec en contexto: detalle, componentes y calibración

En la [sección anterior](03-anatomy-of-a-spec.md) vimos qué poner dentro de una spec — los seis bloques, la plantilla, la maldición de las instrucciones y las trampas de la generación con agente. Ahora toca la pregunta que sigue: **cuánto detalle**, **cómo incorporar información de documentos upstream**, y **cómo se relaciona la spec con los componentes técnicos** del sistema que toca.

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

## Reutilizar criterios de documentos upstream (user stories, contratos)

Una pregunta inevitable: si la user story upstream **ya contiene criterios de aceptación**, ¿qué hago con ellos? La respuesta corta es **reescribir con traza** — ni copiar literal (importa la imprecisión del lenguaje de producto), ni referenciar a secas (`"ver JIRA-1234"` rompe la auto-contención del capítulo 1 y abre drift invisible). Reescribes los criterios en la spec al nivel de precisión que el validador del capítulo 6 necesita, y citas la user story en la sección de "por qués" como fuente del *qué motivó la decisión*, no como contenedor del *qué hay que hacer*.

La regla unificada: **la spec contiene su propia versión, precisa y verificable, de los criterios. El documento upstream se cita como fuente del por qué.** El anti-patrón a evitar — *fusionar user story y spec en un único archivo híbrido porque "dicen lo mismo"* — lo desarrollamos como [anti-patrón #13](12-anti-patterns.md#13-fusionar-user-story-y-spec-en-un-solo-archivo) en el capítulo 12.

**Cuando la user story vive en el mismo repo que la spec**, las objeciones más graves se desactivan: la auto-contención se conserva (todo está en git), el drift deja de ser invisible (queda un commit), y aparece una posibilidad nueva — un sensor automático (hook o agente recurrente) que detecte cuando la user story cambia sin que la spec se actualice. Pero la regla de "reescribir con traza" sigue siendo correcta: siguen siendo dos artefactos con propósitos distintos (la user story responde al *qué quiere el usuario*; la spec, al *qué garantías cumple el sistema*), y cada uno cambia por razones diferentes.

Un riesgo concreto de este escenario: la tentación de editar ambos documentos con frecuencia sin que cada cambio esté plenamente justificado. Cada edición cruzada es una oportunidad de desalineamiento — y cuantas más haya, más difícil es saber cuál de los dos refleja la verdad actual. La traza (la referencia explícita que conecta un criterio de la spec con su origen en la user story) cumple un doble propósito: sirve como **punto de comprobación en el momento del cambio** (el humano o el agente que edita la spec puede verificar que el origen sigue siendo válido) y como **material de auditoría posterior** para un agente de doc-gathering que busque discrepancias entre ambos documentos de forma recurrente.

**Una excepción legítima**: cuando el documento upstream es realmente autoritativo y vive bajo su propia disciplina de validación (un contrato OpenAPI/Protobuf con su propio CI, una política de seguridad corporativa, un RFC), la spec **resume las implicaciones** sin copiar el contenido. La distinción crítica: ese documento upstream tiene su *propia* validación operando sobre él. Una user story de Jira casi nunca tiene esa propiedad.

## La spec y los componentes técnicos: consumir, producir, modificar

La anatomía no vive en el vacío: toda spec aterriza sobre código que ya existe. Y en la práctica, casi todas las specs reales **tocan varios componentes a la vez** — alguno se construye nuevo, otro se modifica, y varios se consumen sin tocarse. Cada una de esas relaciones tiene reglas distintas, y mezclarlas en un mismo lenguaje es una de las formas más rápidas de inflar la spec sin ganar precisión.

Esta sección extiende la anterior sobre documentos upstream — donde tratamos el caso de las user stories — a la pregunta más general: ¿qué dice una spec sobre los componentes técnicos del sistema con los que se relaciona?

### Tres relaciones, tres reglas

Una spec puede tener tres relaciones distintas con cada componente que toca:

**1. Consumido (existe; la spec lo usa).** El componente ya está construido, tiene su propio contrato (API, módulo, librería), y la spec se apoya en él sin modificarlo. La regla es **referencia el contrato, no lo reimplementes**. La spec documenta *cómo* se usa el componente — qué endpoints llama, qué errores espera manejar — pero no documenta lo que el componente hace por sí mismo. Eso vive en el contrato del componente, que es su propia fuente de verdad.

Si el componente es un ADR, una API contract, una política de seguridad o un estándar externo, la regla es la misma con un matiz: **cita el invariante específico que esta spec depende, no el documento entero**. *"Esta spec respeta ADR-007 (single Postgres instance) y ADR-012 (no llamadas síncronas entre servicios)"* es útil. *"Relacionado: ADR-007"* es ruido.

**2. Producido (no existe; la spec lo crea).** Es el caso típico de una feature nueva: *"esta spec crea un nuevo endpoint / un nuevo servicio / un nuevo módulo"*. Aquí la trampa es obvia y peligrosa: como el componente no existe todavía, sientes que tienes que "definirlo" en la spec, y casi siempre acabas describiéndolo con clases, métodos, firmas y payloads. Eso es exactamente el [anti-patrón #12](12-anti-patterns.md#12-spec-generada-que-es-pseudocodigo-disfrazado) (pseudocódigo disfrazado de spec).

La forma correcta: la spec describe el **contrato observable** que el componente nuevo tiene que ofrecer, no su estructura interna. *"Tras esta spec, debe existir una capacidad para aceptar uploads de avatares autenticados (JPEG/PNG, ≤10 MB) y devolver una URL recuperable"* es contrato observable. *"Crear `AvatarUploadService` con método `upload(file, user_id) -> AvatarMetadata`"* es pseudocódigo.

Hay un matiz conceptualmente importante: una spec que produce un componente nuevo es **la fuente de verdad transitoria que hace handoff**. Mientras el componente no existe, la spec es lo único que lo describe. En cuanto el componente existe, **su propia documentación** (su README, su contrato OpenAPI, sus tests, su propio código) se vuelve la fuente operacional, y la spec original pasa a ser el *"por qué se construyó así"* — intención histórica, no contrato vivo. Si confundes los dos roles, acabas con dos artefactos compitiendo por ser la fuente de verdad del mismo componente, y vuelves a la fusión silenciosa del [anti-patrón #13](12-anti-patterns.md#13-fusionar-user-story-y-spec-en-un-solo-archivo).

**3. Modificado (existe; la spec añade, cambia o quita capacidades).** Aquí la regla es **describe el delta observable, no el estado completo del componente**. La spec no tiene que volver a documentar cómo funciona `User`; tiene que decir *"añade el campo `avatar_url` (opcional, URL del avatar actual o ausente). Se actualiza al subir un avatar y se borra al borrarlo. Ningún otro comportamiento del modelo cambia."*

La parte de **"ningún otro comportamiento cambia"** es lo que salva a la spec de inflarse. Sin ella, el equipo tiende a reescribir el componente entero "para que la spec sea completa". El delta es la spec; el resto del componente vive en su propia documentación.

> **En resumen de las tres relaciones**: *consumido* = referencia el contrato y describe solo el uso; *producido* = define el contrato observable y prepara el handoff a la doc del componente; *modificado* = describe el delta y declara el resto invariante.

### Capturar lo fundamental en tres dimensiones

Para cada componente que la spec produce o modifica, la pregunta inmediata es *"¿cuánto detalle pongo?"*. La respuesta vaga lleva al pseudocódigo. La respuesta útil es decomponer en **tres dimensiones explícitas** y, para cada una, capturar solo lo load-bearing — y referenciar a docs más profundas cuando existan.

Las tres dimensiones, aplicadas al **delta** del componente:

- **Funcional** — qué cambia en el comportamiento observable. *"El modelo `User` ahora puede tener un avatar; el campo es opcional; se actualiza al subir y se elimina al borrar."*
- **No funcional** — qué restricciones cruzadas tiene que respetar el cambio. Performance, seguridad, compatibilidad, observabilidad, datos. *"La migración no debe requerir downtime; el campo no debe romper la serialización existente del modelo."*
- **Técnico** — qué decisiones de contrato/integración el cambio cierra explícitamente. *"El campo es indexado por `user_id`; la URL del avatar pasa por el CDN existente, no por el bucket directamente."*

Para cada dimensión, una **referencia opcional al documento más profundo** si existe: *"Más detalle de la migración en `docs/db/migrations/0042-add-avatar.md`. Política de CDN en `infra/cdn/policy.md`."*

Tres aclaraciones importantes sobre este patrón:

1. **Si una dimensión no aporta, omítela.** No fuerces tres líneas por componente por simetría. La ausencia es informativa: dice que esa dimensión no cambia.
2. **Lo que va en la spec es lo load-bearing**: lo que el validador del cap. 6 puede comparar contra el código y lo que el equipo quiere que sea contractual. El detalle más fino vive en la documentación del componente, que tiene su propio mecanismo de validación (tests, type checks, contratos versionados).
3. **El término "no funcional" es clásico pero contestado** ([Fowler](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) argumenta que todo es funcional, solo cambia la dimensión). Para los efectos de este capítulo lo usamos como categoría operativa — tres preguntas distintas que hacerle a cada componente — no como ontología. Si a tu equipo le encaja mejor llamarlas *comportamiento*, *cualidades*, *contratos*, da exactamente igual. Lo que importa es que sean **tres preguntas distintas**, no una sola.

> **En resumen de las tres dimensiones**: *funcional* describe el comportamiento observable; *no funcional* las restricciones cruzadas; *técnico* las decisiones de contrato e integración. Solo lo load-bearing entra en la spec; el resto se referencia.

### Dónde va esta información en los seis bloques

La taxonomía consumir/producir/modificar y las tres dimensiones no crean una sección nueva en la spec — ayudan a escribir mejor las secciones que ya existen:

- **Componentes consumidos** → van en **restricciones técnicas**: "la auth pasa por `requireAuth`; `avatar-storage` se usa vía `POST /upload`, manejando 413 y 415 explícitamente".
- **Componentes producidos o modificados** → sus garantías observables van en **criterios de aceptación**: "acepta uploads JPEG/PNG ≤10 MB; devuelve URL; falla con 413/415 con mensaje legible". Sus límites de diseño van en **restricciones técnicas**: "no se crea tabla nueva; el avatar es un campo en `User`".
- **Las tres dimensiones** (funcional, no funcional, técnico) son las tres preguntas que te haces al escribir cada criterio o restricción, no tres subsecciones dentro de la spec.
- **Los por qués** de cada decisión van, como siempre, en **por qués**.

Cuando un componente se consume, la spec es más corta para él — solo describe *cómo lo usa esta feature*, no lo que el componente hace (eso vive en su propio contrato). Cuando un componente se modifica, la spec declara explícitamente qué **no** cambia — eso es lo que salva a la spec de inflarse. Esos dos hábitos son los que distinguen una spec útil de una spec inflada.

### Heurística de tamaño: cuándo la spec necesita partirse

Una regla práctica que funciona: si al escribir restricciones y criterios descubres que la spec produce o modifica más de **3-4 componentes**, probablemente tienes uno de tres problemas, y conviene diagnosticar antes de seguir:

1. Una feature demasiado grande que debería partirse en varias specs (modulación del cap. 9).
2. Un cambio arquitectónico disfrazado de feature, que merece un ADR separado del que esta spec luego cuelgue.
3. Un refactor camuflado de feature (también modulación del cap. 9): lo que estás cambiando es la *forma* del sistema, no añadiendo capacidad de negocio.

Las tres tienen tratamiento distinto en el resto del curso, y la spec sola no es el artefacto correcto para ninguna.

### Tabla resumen consolidada

Reuniendo esta sección con la subsección anterior sobre documentos upstream, esta es la regla unificada por tipo de relación:

| Relación | Regla principal | Sección de la spec | Trampa típica |
|---|---|---|---|
| **Componente consumido** | Referencia el contrato; documenta el uso | Restricciones técnicas | Re-documentar el componente |
| **Componente producido** | Define contrato observable, no estructura | Criterios + Restricciones | Pseudocódigo (#12); no marcar el handoff a la doc del componente |
| **Componente modificado** | Describe el delta, declara el resto invariante | Criterios + Restricciones | Reescribir el componente entero |
| **ADR / arquitectura** | Cita invariantes específicos | Restricciones técnicas | Copiar invariantes (drift garantizado) |
| **API contract / estándar externo** | Referencia + resumen de implicaciones | Restricciones técnicas | Re-documentar el contrato |
| **Feature brief / doc de producto** | Extrae los por qués cardinales | Por qués | Importar el lenguaje vago de producto |
| **User story** | Reescribir criterios con traza | Criterios + Por qués | Copia literal o referencia opaca |

Y un anti-patrón nuevo que aparece cuando las referencias se acumulan sin disciplina: **sopa de referencias** — una spec donde el contenido propio queda enterrado bajo una lista larga de citas a ADRs, contratos, briefs y componentes, todos sin priorización. El agente y el humano no saben cuáles son load-bearing y cuáles son tangenciales, y acaban ignorándolos a todos. Es el inverso de la *maldición de las instrucciones*, — una spec ahogada por sus propias citas — y lo desarrollamos como [anti-patrón #14](12-anti-patterns.md#14-sopa-de-referencias) en el capítulo 12. La regla preventiva: **referencia solo lo que el agente o el humano necesita para esta tarea, y anota el porqué de cada referencia**.

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
