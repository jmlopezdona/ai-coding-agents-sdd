# 3. Anatomía de una buena especificación

Hay muchas formas de escribir una spec mal y bastantes pocas de escribirla bien. Este capítulo intenta destilar lo que distingue una spec útil para un agente de IA de una que es ceremonia disfrazada. La frase corta: una buena spec captura **intención, restricciones y los "por qués"** con la suficiente precisión para que el agente no tenga que adivinar, y con la suficiente brevedad para que un humano la siga leyendo dentro de seis meses.

**Mapa del capítulo.** Empezamos por los **seis bloques** que toda spec mínima debería tener y una **plantilla** que cabe en una pantalla. Después atacamos las tres trampas más caras: la **maldición de las instrucciones** (specs grandes rinden peor que módulos pequeños), la **generación con agente** (que casi siempre produce pseudocódigo disfrazado), y el **nivel de detalle equivocado** para tu nivel del espectro. Luego abrimos una sección sobre cómo una spec se relaciona con los **componentes técnicos** del sistema (consumir, producir, modificar), cerramos con un **ejemplo end-to-end** de spec real, y rematamos con las tres cosas que separan una spec buena de una mediocre.

## Lo que tiene que estar dentro

Una spec mínima útil tiene seis bloques. No todos los seis tienen que ser largos, pero los seis tienen que estar.

### 1. Objetivo

Una o dos frases describiendo qué se construye y *para quién*. No qué API expone, no qué endpoints tiene — eso viene después. Solo: el resultado observable que justifica el trabajo.

> *Permitir que un usuario suba una foto de perfil que será visible para sus seguidores y eliminable solo por él.*

Si el objetivo no se puede escribir en dos frases, casi siempre es porque está mezclando dos features distintas. Sepáralas y escribe dos specs.

### 2. No-goals

Esta es la sección que casi nadie escribe y la que más previene drift. **Qué no se construye**, explícitamente.

> *No se va a soportar GIF animado en esta versión. No se va a comprimir la imagen automáticamente. No se va a integrar con el sistema de moderación; eso vive en otra spec.*

Los no-goals son una vacuna contra el scope creep del agente. Sin ellos, el agente — siguiendo su training — añadirá compresión, validación de contenido, soporte multi-formato y miniaturas, porque todas esas cosas son "lo que normalmente se hace". Y todas esas cosas son trabajo que tú no pediste.

### 3. Restricciones técnicas

Lo que el agente *no puede* tocar, y lo que *tiene que* respetar. Son los invariantes del sistema.

> *Las imágenes se sirven desde el bucket S3 ya existente, no se introduce uno nuevo. La autenticación pasa por el middleware `requireAuth`, no se inventa otro. El error handling sigue el patrón `Result<T, AppError>` que se usa en `src/api/`.*

Las restricciones son lo que hace que la feature encaje en el resto del sistema en lugar de aterrizar como un cuerpo extraño. Son el antídoto del context collapse del capítulo 1.

### 4. Criterios de aceptación verificables

Cómo sabes que está hecho. La palabra clave es **verificable**: una persona externa, leyendo solo el criterio, debería poder decidir si el código lo cumple o no.

> *Un usuario autenticado puede subir una imagen JPEG o PNG de hasta 10 MB. El upload falla con un mensaje claro si el archivo es de otro tipo o supera el tamaño. Solo el dueño del archivo puede borrarlo. Un test de integración cubre los tres casos.*

"Funciona bien" no es un criterio de aceptación. "Devuelve 413 con mensaje legible si el archivo supera los 10 MB" sí lo es.

#### Reutilizar criterios de documentos upstream (user stories, contratos)

Una pregunta inevitable: si la user story upstream **ya contiene criterios de aceptación**, ¿qué hago con ellos? La respuesta corta es **reescribir con traza** — ni copiar literal (importa la imprecisión del lenguaje de producto), ni referenciar a secas (`"ver JIRA-1234"` rompe la auto-contención del capítulo 1 y abre drift invisible). Reescribes los criterios en la spec al nivel de precisión que el validador del capítulo 5 necesita, y citas la user story en la sección de "por qués" como fuente del *qué motivó la decisión*, no como contenedor del *qué hay que hacer*.

La regla unificada: **la spec contiene su propia versión, precisa y verificable, de los criterios. El documento upstream se cita como fuente del por qué.** El anti-patrón a evitar — *fusionar user story y spec en un único archivo híbrido porque "dicen lo mismo"* — lo desarrollamos como anti-patrón #13 en el capítulo 11.

**Una excepción legítima**: cuando el documento upstream es realmente autoritativo y vive bajo su propia disciplina de validación (un contrato OpenAPI/Protobuf con su propio CI, una política de seguridad corporativa, un RFC), la spec **resume las implicaciones** sin copiar el contenido. La distinción crítica: ese documento upstream tiene su *propia* validación operando sobre él. Una user story de Jira casi nunca tiene esa propiedad.

### 5. Los "por qués"

Aquí es donde las specs ordinarias mueren y las buenas se separan del resto. Casi todas las specs explican el *qué*. Casi ninguna explica el *por qué*. Y los por qués son justo lo que el agente necesita para tomar decisiones inteligentes en los huecos que la spec no cubre.

> *No comprimimos la imagen automáticamente porque el equipo de diseño quiere preservar la calidad para los avatares de cuentas verificadas. (Decisión de septiembre 2025, dueña: @maria.)*
>
> *Solo el dueño puede borrar porque añadir moderación cambiaría el modelo de permisos y queremos mantener la spec mínima en esta iteración.*

Los por qués son el ingrediente que la crítica de Isoform identifica como lo que el SDD malhecho pierde, y es la razón principal por la que las specs se vuelven inútiles a meses vista. Una spec sin por qués envejece a la velocidad de las decisiones que la rodean. Una spec con por qués sobrevive a esas decisiones porque las explica.

### 6. Boundaries — el sistema de tres niveles

Esta es la contribución más útil de Addy Osmani al pensamiento sobre specs para agentes. En lugar de una lista plana de "haz" y "no hagas", divide las restricciones en **tres tiers**:

- **✅ Always do** — cosas que el agente debe hacer sin pedir permiso.
  *Ejemplo: ejecutar la suite de tests antes de declarar terminada cualquier tarea.*
- **⚠️ Ask first** — cosas que el agente debe **pedir confirmación** antes de hacer.
  *Ejemplo: modificar el esquema de la base de datos, añadir una dependencia nueva, tocar archivos de configuración de despliegue.*
- **🚫 Never do** — cosas que el agente nunca debe hacer.
  *Ejemplo: commitear secretos o claves, modificar `vendor/`, deshabilitar tests para que pasen, hacer force push.*

La distinción crucial es la del medio. Sin el "ask first", el agente solo tiene dos modos: actuar o paralizarse. El "ask first" introduce un tercer modo — "consulta antes de proceder" — que es donde de verdad cabe la conversación humano-agente productiva. Es la pieza que más equipos olvidan, y es la que convierte una spec restrictiva en una spec colaborativa.

#### Los seis bloques de un vistazo

| Bloque | Propósito | Trampa típica |
|---|---|---|
| **Objetivo** | Resultado observable y para quién, en 1-2 frases | Mezclar dos features en una |
| **No-goals** | Vacuna contra el scope creep del agente | No escribirlos |
| **Restricciones técnicas** | Invariantes que la feature debe respetar | Reinventar lo que ya existe |
| **Criterios de aceptación** | Cómo sabes que está hecho, verificables | "Funciona bien" como criterio |
| **Por qués** | Razones con dueño y fecha; lo que envejece bien | Documentar el *qué* y omitir el *por qué* |
| **Boundaries** | Always / Ask first / Never para el agente | Olvidar el "Ask first" |

## Una plantilla mínima

```markdown
# Spec: [nombre de la feature]

## Objetivo
Una o dos frases. Resultado observable. Para quién.

## No-goals
- Cosa que NO se hace
- Otra cosa que NO se hace

## Restricciones técnicas
- Invariantes que se respetan
- Patrones del proyecto que se siguen
- Lo que no se puede tocar

## Criterios de aceptación
- [ ] Criterio verificable 1
- [ ] Criterio verificable 2

## Por qués
- Por qué esta decisión y no la otra (con dueño y fecha)
- Trade-offs aceptados

## Boundaries
- ✅ Always: ...
- ⚠️ Ask first: ...
- 🚫 Never: ...

## Superficies afectadas (recomendado siempre que la spec toque >1 componente; ver sección dedicada más abajo)
### [Componente] — [producido | modificado | consumido]
- **Funcional:** ...
- **No funcional:** ...
- **Técnico:** ...
- **Más detalle:** [referencia a doc más profunda si existe]
```

Esta plantilla cabe en menos de una pantalla y cubre el 80% del valor que una spec puede aportar. El otro 20% son detalles específicos del dominio — diagramas de estado, contratos de API, ejemplos concretos de input/output — que se añaden cuando el dominio los pide, no por defecto.

## La maldición de las instrucciones

**Más información en una sola spec no es mejor — es peor.** Una spec global de 500 líneas es un mal plan; cinco specs modulares de 100 líneas, donde la tarea de hoy referencia solo dos, es un buen plan. Esta es la regla práctica más importante de todo el capítulo, y va contra la intuición de casi todo el mundo que empieza con SDD.

La razón tiene nombre. Addy Osmani, citando estudios académicos, la llama **the curse of instructions**: cuanto más metes en un prompt, peor cumple el modelo *cada una* de las cosas. No es lineal — es un colapso cualitativo. Una spec con cinco restricciones se cumple razonablemente bien. Una spec con cincuenta se cumple razonablemente mal en todas, y muchas veces el agente actúa como si no existiera ninguna.

La estrategia ganadora, entonces, es dividir specs grandes en módulos pequeños y pasarle al agente solo el módulo relevante para la tarea actual.

Esta es también una de las razones por las que las herramientas tipo Spec-kit y Kiro generan **muchos archivos pequeños** en vez de uno grande, y por las que esa proliferación, mal gestionada, se convierte en su propio problema (lo veremos en el capítulo 9).

## ¿Y si la spec la genera un agente?

**La tesis de esta sección, por adelantado:** el agente sirve como *draft generator del qué* — objetivo, criterios candidatos, no-goals tentativos, boundaries derivados del repo —, pero **no** del *cómo*. Y los **por qués** no los puede generar bien por mucho prompting que le pongas, porque no están en el input. Si te quedas con una sola regla: *si tu spec generada menciona una firma de función o un nombre de clase, bórrala*.

El resto de la sección explica por qué la versión optimista — *"el agente escribe la spec entera"* — falla casi siempre por defecto, no por mal prompting sino por una razón estructural que conviene entender antes de gastar semanas descubriéndola.

Es una idea natural: si el agente sabe escribir código, también debería saber escribir specs. Le pasas una user story, una descripción de la feature o un componente arquitectónico, y te devuelve la plantilla anterior rellena. El coste de arranque baja, la barrera de entrada desaparece, y aparentemente has resuelto el problema más caro de SDD — que es escribir specs. Pero hay un problema estructural debajo.

### El modo por defecto: pseudocódigo disfrazado de spec

Cuando le pides a un LLM "genera una spec para esta feature", el modelo optimiza por la métrica que mejor sabe medir: *cobertura aparente*. Y la forma más fácil de parecer exhaustivo es bajar al nivel de detalle del código, porque ahí el modelo está cómodo — su corpus de entrenamiento es código. Subir al nivel de *intención*, *restricciones* y *por qués* requiere abstracción, que es justo lo que el modelo hace peor.

El resultado es predecible y casi universal: la "spec" generada es en realidad un **pseudocódigo comentado** disfrazado de documento de diseño. Tiene clases, métodos, firmas de funciones, payloads de API, reglas de negocio inline, diagramas de secuencia. Parece completísima. Y es exactamente lo *contrario* de lo que una buena spec debería ser, según todo lo que hemos visto en este capítulo.

Una buena spec opera en un nivel de abstracción **distinto** al código. Si menciona clases y métodos, ya no es una spec: es un boceto de implementación. La pregunta de la spec no es "¿qué clase llama a qué método?", es "¿qué garantías observables tiene que cumplir esto, bajo qué restricciones y por qué?". Cuando el agente baja al primer nivel, la spec pierde su razón de ser, porque deja de ser **comparable contra el código desde un punto de vista distinto**. Si la spec dice lo mismo que el código pero en markdown, sólo es ruido.

### Las dos consecuencias graves

Cuando te encuentras con specs así — y, si dejas al agente en su modo por defecto, te las vas a encontrar — pagas dos costes que se refuerzan entre sí.

**Implementas dos veces.** Has pagado el coste de pensar la implementación a nivel de clases y métodos en la spec, y vas a pagar otra vez el mismo coste cuando codifiques. Peor: cualquier cosa que descubras al implementar (y al implementar siempre se descubren cosas) te obliga a volver atrás y editar la spec, o a aceptar drift inmediato. Estás en la peor parte de la curva del capítulo 9: maintenance tax desde el día uno, sin haber escrito todavía una línea de código.

**La revisión humana se vuelve tan cara como revisar código.** Esto es exactamente lo que Fowler reporta de Spec-kit en el capítulo 6 — que los archivos generados eran *más pesados de revisar que el propio código*. Y es peor que revisar código directamente, porque al menos el código se ejecuta y los tests lo verifican; el pseudocódigo en markdown no tiene ninguna red de seguridad. Estás revisando algo del nivel de detalle del código pero **sin las garantías del código**. El reviewer humano, cansado, lo lee por encima — y la "spec" pasa el filtro como si hubiera sido revisada.

La combinación de las dos es venenosa: doblas el trabajo y *además* lo revisas peor. Es casi siempre peor que no haber escrito ninguna spec.

### Por qué no es solo un problema de plantilla

Es tentador pensar que esto se arregla con mejor prompting o una plantilla más estricta. *"Le digo al agente que no incluya firmas, que no mencione clases, que exprese todo como invariantes externamente observables."* Y sí, eso reduce el problema. Pero **no lo elimina**, por dos razones que conviene tener claras.

Primero, el sesgo del modelo a bajar al nivel del código es muy fuerte porque es donde "siente que produce valor". Cuando le restringes todo lo que sabe hacer bien y le pides solo lo que hace mal — abstraer intención, capturar por qués que no están en el input — la spec resultante es corta, pobre, y el equipo intuye que "le falta algo". El siguiente paso es casi siempre pedirle más detalle, y vuelves al pseudocódigo. Es un equilibrio inestable.

Segundo, hay cosas que el agente **no puede** generar bien por mucha plantilla que le des. Los **por qués** casi nunca están en el input — viven en la cabeza del PM, en una conversación de Slack, en un trade-off histórico que se decidió hace dos años. Lo que el agente no sabe, lo inventa o lo omite. Una spec generada con "por qués" inventados es peor que una spec sin "por qués", porque miente con apariencia de autoridad.

### Uso legítimo: el agente como draft generator del *qué*, no del *cómo*

Esto no significa que la generación con agente no sirva. Sirve, pero en un papel más estrecho del que la mayoría de la gente imagina:

- **El agente puede arrancar bien**: el bloque de **objetivo**, los **criterios de aceptación verificables** (con cuidado), una primera lista de **no-goals candidatos** que el humano luego refina, y una propuesta de **boundaries** basada en patrones que ya ve en el repo.
- **El humano tiene que poner sí o sí**: los **por qués**, los **no-goals reales** (especialmente los políticos o de scope que no están en ningún input), las **restricciones técnicas tácitas** del equipo, y la decisión de *cuánta* spec merece la tarea (lo que en el capítulo 8 llamaremos *modulación*: ajustar el peso de la spec a la complejidad y el riesgo reales de la feature). Y, sobre todo, tiene que **borrar** todo lo que el agente metió de pseudocódigo: clases, métodos, firmas, payloads. Si después de borrarlos la spec queda vacía, no necesitabas una spec; necesitabas escribir bien el código (capítulo 10, *context engineering*).

La regla práctica más útil: **si tu spec generada menciona una firma de función o un nombre de clase, bórrala**. La spec habla de qué garantías cumple el sistema, no de cómo se construye.

Y la advertencia honesta: usar el agente como generador de drafts **no reduce el coste total** de hacer SDD bien. Lo que reduce es el coste de empezar el primer borrador, que para muchos equipos es la barrera psicológica que más pesa. El coste real — pensar la intención con precisión, capturar los por qués, mantener viva la spec — sigue estando ahí, sin atajo, y el agente no lo paga por ti. Si tu adopción de SDD depende de que la generación automática elimine ese coste, lo que vas a tener no es SDD — vas a tener el anti-patrón #12 del capítulo 11.

## El nivel de detalle depende del nivel del espectro

Hasta aquí hemos hablado de la anatomía de una spec como si fuera una sola cosa. No lo es. **El nivel de detalle apropiado depende de en qué nivel del espectro del capítulo 2 estés operando**, y escribir una spec con el nivel de detalle equivocado para tu nivel del espectro es una de las formas más comunes — y más caras — de sufrir SDD malhecho.

### Spec-first → detalle ligero, intencional, no exhaustivo

En spec-first la spec se lee una vez, al arrancar la feature, y a partir de ahí el código deriva libremente. Su única función es **alinear al equipo y al agente al principio**. Nada la va a leer de vuelta, así que cada línea extra que escribas es trabajo que nadie va a recuperar.

El detalle apropiado: objetivo, no-goals, criterios de aceptación, los por qués críticos, y poco más. Si tu spec-first ocupa más de una pantalla, casi siempre es porque estás escribiendo spec-anchored *aspiracional* (anti-patrón #4 del capítulo 11) o porque caíste en pseudocódigo (anti-patrón #12). La spec-first óptima es la mínima viable para arrancar con intención clara.

### Spec-anchored → detalle medio, acotado por lo que el anclaje puede verificar

Aquí cambia la lógica. La spec sí se va a leer otra vez — por los validadores, por los tests, por los agentes recurrentes que detectan drift. El detalle ya no es opcional: tiene que ser **suficiente para que el mecanismo de anclaje pueda comparar**.

Pero hay un tope superior contraintuitivo: **el detalle no debe ir más allá de lo que tu anclaje sabe verificar**. Si tu validador comprueba contratos de API y tu spec describe reglas de UI, la parte de UI no está anclada — es spec-first disfrazada de spec-anchored. Y como ese trozo no se verifica, drifta libre.

La regla operativa: el detalle de una spec-anchored se mide contra el **alcance del anclaje**, no contra una idea abstracta de "completitud". Si lo que escribes no se puede verificar automáticamente, escribirlo no te aporta más anclaje — solo te aporta más maintenance tax.

### Spec-as-source → detalle exhaustivo, pero de un tipo distinto

Aquí es donde la conversación se vuelve interesante. Spec-as-source sí necesita el máximo detalle, porque el código se genera a partir de la spec. Pero el detalle es de **una naturaleza diferente** del de los otros dos niveles.

Es detalle **formal o semi-formal**: signaturas de tipos, contratos, invariantes, gramáticas, reglas de transformación. Es detalle **generator-friendly**: pensado para que un generador (LLM o no) pueda producir código determinista a partir de él. Y es legítimo que aparezcan firmas, esquemas y estructuras concretas — porque en este nivel **la spec es el código fuente, solo que en otra notación**.

Aquí está la conexión incómoda con la sección anterior: la "spec" generada por un agente que es pseudocódigo disfrazado se *parece* superficialmente a una spec-as-source. Tiene clases, métodos, payloads, reglas. Pero hay una diferencia crítica: **una spec-as-source legítima viene acompañada de un generador determinista que produce el código**. Sin ese generador, lo que tienes es la peor combinación posible: el detalle exhaustivo de spec-as-source con el no-determinismo de spec-first. Es el equivalente, en lo conceptual, de escribir XML+OCL en los 2000 sin tener un compilador MDA detrás. Fowler sabría exactamente cómo se llama.

### La regla unificadora

Si tienes que destilar todo lo anterior a una sola frase:

> **El nivel de detalle apropiado de una spec es el que tu mecanismo de validación — humano, automático o generativo — sabe consumir. Más detalle que eso es desperdicio; menos detalle es ceguera.**

En spec-first el "validador" es el equipo en una sola lectura inicial, así que el detalle apropiado es lo que cabe en esa lectura. En spec-anchored el validador es el mecanismo de anclaje, así que el detalle apropiado es lo que el anclaje sabe comparar. En spec-as-source el validador es el generador, así que el detalle apropiado es lo que el generador necesita para producir código sin ambigüedad.

Casi todas las patologías que veremos en el capítulo 11 vienen de **desalinear estas tres cosas**: escribir spec-anchored sin anclaje (#4), escribir spec-as-source sin generador (#9 + #12), o escribir spec-first con el detalle de spec-as-source (#2 + #12). La anatomía no es absoluta — es relativa a qué tipo de validación se va a hacer sobre lo que escribes.

## La spec y los componentes técnicos: consumir, producir, modificar

La anatomía no vive en el vacío: toda spec aterriza sobre código que ya existe. Y en la práctica, casi todas las specs reales **tocan varios componentes a la vez** — alguno se construye nuevo, otro se modifica, y varios se consumen sin tocarse. Cada una de esas relaciones tiene reglas distintas, y mezclarlas en un mismo lenguaje es una de las formas más rápidas de inflar la spec sin ganar precisión.

Esta sección extiende la subsección anterior sobre *"Reutilizar criterios de documentos upstream"* — donde tratamos el caso de las user stories — a la pregunta más general: ¿qué dice una spec sobre los componentes técnicos del sistema con los que se relaciona?

### Tres relaciones, tres reglas

Una spec puede tener tres relaciones distintas con cada componente que toca:

**1. Consumido (existe; la spec lo usa).** El componente ya está construido, tiene su propio contrato (API, módulo, librería), y la spec se apoya en él sin modificarlo. La regla es **referencia el contrato, no lo reimplementes**. La spec documenta *cómo* se usa el componente — qué endpoints llama, qué errores espera manejar — pero no documenta lo que el componente hace por sí mismo. Eso vive en el contrato del componente, que es su propia fuente de verdad.

Si el componente es un ADR, una API contract, una política de seguridad o un estándar externo, la regla es la misma con un matiz: **cita el invariante específico que esta spec depende, no el documento entero**. *"Esta spec respeta ADR-007 (single Postgres instance) y ADR-012 (no llamadas síncronas entre servicios)"* es útil. *"Relacionado: ADR-007"* es ruido.

**2. Producido (no existe; la spec lo crea).** Es el caso típico de una feature nueva: *"esta spec crea un nuevo endpoint / un nuevo servicio / un nuevo módulo"*. Aquí la trampa es obvia y peligrosa: como el componente no existe todavía, sientes que tienes que "definirlo" en la spec, y casi siempre acabas describiéndolo con clases, métodos, firmas y payloads. Eso es exactamente el anti-patrón #12 (pseudocódigo disfrazado de spec).

La forma correcta: la spec describe el **contrato observable** que el componente nuevo tiene que ofrecer, no su estructura interna. *"Tras esta spec, debe existir una capacidad para aceptar uploads de avatares autenticados (JPEG/PNG, ≤10 MB) y devolver una URL recuperable"* es contrato observable. *"Crear `AvatarUploadService` con método `upload(file, user_id) -> AvatarMetadata`"* es pseudocódigo.

Hay un matiz conceptualmente importante: una spec que produce un componente nuevo es **la fuente de verdad transitoria que hace handoff**. Mientras el componente no existe, la spec es lo único que lo describe. En cuanto el componente existe, **su propia documentación** (su README, su contrato OpenAPI, sus tests, su propio código) se vuelve la fuente operacional, y la spec original pasa a ser el *"por qué se construyó así"* — intención histórica, no contrato vivo. Si confundes los dos roles, acabas con dos artefactos compitiendo por ser la fuente de verdad del mismo componente, y vuelves a la fusión silenciosa del anti-patrón #13.

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
2. **Lo que va en la spec es lo load-bearing**: lo que el validador del cap. 5 puede comparar contra el código y lo que el equipo quiere que sea contractual. El detalle más fino vive en la documentación del componente, que tiene su propio mecanismo de validación (tests, type checks, contratos versionados).
3. **El término "no funcional" es clásico pero contestado** (Fowler argumenta que todo es funcional, solo cambia la dimensión). Para los efectos de este capítulo lo usamos como categoría operativa — tres preguntas distintas que hacerle a cada componente — no como ontología. Si a tu equipo le encaja mejor llamarlas *comportamiento*, *cualidades*, *contratos*, da exactamente igual. Lo que importa es que sean **tres preguntas distintas**, no una sola.

> **En resumen de las tres dimensiones**: *funcional* describe el comportamiento observable; *no funcional* las restricciones cruzadas; *técnico* las decisiones de contrato e integración. Solo lo load-bearing entra en la spec; el resto se referencia.

### El bloque "Superficies afectadas"

Cuando una spec toca varios componentes, conviene un bloque explícito al principio que **enumere las superficies afectadas con su tipo de relación**. No es work breakdown (no enumera tareas, no menciona archivos, no asigna orden — eso vive en la fase de plan del cap. 4). Es **mapeo de superficie**, y hace tres cosas que ninguna otra parte de la spec hace bien:

1. Le da al agente y al validador del cap. 5 una **lista enumerable** del blast radius del cambio, que en el cap. 1 era invisible.
2. Le da al humano una **señal del tamaño de la spec**: si la lista tiene 12 entradas, casi seguro la spec son realmente *dos features* mal cosidas o un refactor camuflado de feature.
3. **No es work breakdown**, lo cual la mantiene del lado del "qué" en vez del "cómo".

Un ejemplo del bloque enriquecido con las tres dimensiones:

```markdown
## Superficies afectadas

### Modelo `User` — modificado

- **Funcional:** añade `avatar_url` (opcional, URL del avatar actual o ausente).
  Se actualiza al subir avatar; se borra al borrarlo. Resto del modelo invariante.
- **No funcional:** la migración no debe requerir downtime; serialización existente
  no puede romperse (test `test_user_serialization_back_compat`).
- **Técnico:** campo indexado por `user_id`. No cambia el esquema de permisos.
- **Más detalle:** `docs/db/migrations/0042-add-avatar.md` (cuando se cree).

### Endpoint `POST /api/avatar` — producido

- **Funcional:** acepta uploads autenticados (JPEG/PNG, ≤10 MB), devuelve URL.
  Errores: 401 sin auth, 413 si excede tamaño, 415 si tipo inválido.
- **No funcional:** rate limit existente del API gateway aplica sin cambios.
- **Técnico:** consume `services/avatar-storage` (contrato actual, sin cambios)
  y `auth-middleware` (contrato actual, sin cambios).
- **Más detalle:** este bloque es la fuente transitoria; tras la implementación,
  la doc operacional vive en el README del nuevo servicio.

### `services/avatar-storage` — consumido

- **Funcional:** se llama a `POST /upload`; se manejan 413 y 415 explícitamente,
  los 5xx se propagan al caller.
- Resto: ver `services/avatar-storage/contract.openapi.yaml@v2`.
```

Nota cómo el componente *consumido* tiene un bloque mucho más corto — solo describe *cómo lo usa esta feature*, no lo que el componente hace, porque eso vive en su propio contrato. Y nota cómo el componente *modificado* declara explícitamente qué **no** cambia. Esos dos hábitos son los que distinguen una spec útil de una spec inflada.

### Heurística de tamaño

Una regla práctica que funciona: si una spec produce o modifica más de **3-4 superficies**, probablemente tienes uno de tres problemas, y conviene diagnosticar antes de seguir:

- **(a)** Una feature demasiado grande que debería partirse en varias specs (modulación del cap. 8).
- **(b)** Un cambio arquitectónico disfrazado de feature, que merece un ADR separado del que esta spec luego cuelgue.
- **(c)** Un refactor camuflado de feature (también modulación del cap. 8): lo que estás cambiando es la *forma* del sistema, no añadiendo capacidad de negocio.

Las tres tienen tratamiento distinto en el resto del curso, y la spec sola no es el artefacto correcto para ninguna.

### Tabla resumen consolidada

Reuniendo esta sección con la subsección anterior sobre documentos upstream, esta es la regla unificada por tipo de relación:

| Relación | Regla principal | Sección de la spec | Trampa típica |
|---|---|---|---|
| **Componente consumido** | Referencia el contrato; documenta el uso | Restricciones técnicas | Re-documentar el componente |
| **Componente producido** | Define contrato observable, no estructura | Superficies afectadas + Criterios | Pseudocódigo (#12); no marcar el handoff a la doc del componente |
| **Componente modificado** | Describe el delta, declara el resto invariante | Superficies afectadas + Criterios | Reescribir el componente entero |
| **ADR / arquitectura** | Cita invariantes específicos | Restricciones técnicas | Copiar invariantes (drift garantizado) |
| **API contract / estándar externo** | Referencia + resumen de implicaciones | Restricciones técnicas | Re-documentar el contrato |
| **Feature brief / doc de producto** | Extrae los por qués cardinales | Por qués | Importar el lenguaje vago de producto |
| **User story** | Reescribir criterios con traza | Criterios + Por qués | Copia literal o referencia opaca |

Y un anti-patrón nuevo que aparece cuando las referencias se acumulan sin disciplina: **sopa de referencias** — una spec donde el contenido propio queda enterrado bajo una lista larga de citas a ADRs, contratos, briefs y componentes, todos sin priorización. El agente y el humano no saben cuáles son load-bearing y cuáles son tangenciales, y acaban ignorándolos a todos. Es el inverso de la *maldición de las instrucciones*, — una spec ahogada por sus propias citas — y lo desarrollamos como anti-patrón #14 en el capítulo 11. La regla preventiva: **referencia solo lo que el agente o el humano necesita para esta tarea, y anota el porqué de cada referencia**.

## En resumen: lo que distingue una spec buena de una mediocre

Si tienes que recordar tres cosas de este capítulo, que sean estas:

1. **Los no-goals te ahorran más drift que cualquier otra sección**. Escríbelos siempre.
2. **Los por qués son lo que envejece bien**. Sin ellos, la spec dura semanas. Con ellos, dura años.
3. **El "ask first" es la categoría que falta en casi todas las specs**. Es donde vive la colaboración real con el agente.

### Mediocre vs buena, lado a lado

La diferencia se ve mejor en contraste. Misma feature, dos specs:

**Mediocre** (parece completa, no lo es):

> *Spec: Avatar upload. El sistema permitirá a los usuarios subir un avatar. Se creará un endpoint `POST /api/avatar` que recibirá el archivo y lo guardará. Se validará que el archivo sea una imagen. Se devolverá la URL del avatar guardado. Se añadirá un campo `avatar_url` al modelo `User`. Se implementarán tests.*

Habla del *qué* en términos de implementación, no menciona ningún *por qué*, no hay no-goals, no hay boundaries, los criterios no son verificables ("se validará que el archivo sea una imagen" — ¿qué tipos? ¿qué tamaño? ¿qué error?), y menciona detalles de código (`POST /api/avatar`, `avatar_url`) que son decisiones de implementación, no contrato.

**Buena** (corta, intencional, envejece):

> *Spec: Avatar upload. **Objetivo:** permitir a un usuario autenticado subir una foto de perfil visible para sus seguidores y eliminable solo por él. **No-goals:** no se soporta GIF animado; no se comprime automáticamente; la moderación de contenido vive en otra spec. **Restricciones:** se sirve desde el bucket S3 existente; auth pasa por `requireAuth`; errores siguen el patrón `Result<T, AppError>`. **Criterios:** un usuario autenticado puede subir JPEG o PNG ≤10 MB; falla con 413 si excede tamaño y 415 si tipo inválido, ambos con mensaje legible; solo el dueño puede borrar; un test de integración cubre los tres casos. **Por qués:** no comprimimos porque diseño quiere preservar calidad para cuentas verificadas (sept-2025, @maria); solo el dueño borra porque añadir moderación cambiaría el modelo de permisos y queremos esta iteración mínima. **Boundaries:** ✅ ejecutar tests antes de declarar terminado; ⚠️ preguntar antes de tocar el esquema de `User`; 🚫 nunca deshabilitar tests existentes ni commitear secretos.*

La buena cabe en la misma extensión que la mediocre, pero captura intención, restricciones, criterios verificables y los por qués que la harán sobrevivir a las decisiones que la rodean.

**La regla final**: las specs malas enumeran trivialidades técnicas y no dicen nada de la intención. Las specs mediocres dicen mucho del *qué* y nada del *por qué*. Las specs buenas son cortas, dicen *qué*, *por qué* y *qué no*, y se pueden leer en cinco minutos sin perder nada importante.

## Lo que viene a continuación

Hasta aquí hemos visto qué hay *dentro* de una spec, y la regla unificadora que recorre todo el capítulo: **el detalle apropiado es el que tu mecanismo de validación sabe consumir** — ni más, ni menos. Si la anatomía responde al *qué*, lo siguiente es el *cuándo* y el *cómo se valida*: el momento en que una spec deja de ser documento y empieza a comportarse como contrato.

En el **capítulo 4** vamos a ver el ciclo en el que una spec vive: las cuatro fases del proceso SDD (con sus dos variantes en la literatura), cómo encadenarlas con un agente, y dónde encaja la verificación que evita que el código se aleje de la spec.
