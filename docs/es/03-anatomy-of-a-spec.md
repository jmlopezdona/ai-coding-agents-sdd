# 3. Anatomía de una buena especificación

Hay muchas formas de escribir una spec mal y bastantes pocas de escribirla bien. Este capítulo intenta destilar lo que distingue una spec útil para un agente de IA de una que es ceremonia disfrazada. La frase corta: una buena spec captura **intención, restricciones y los "por qués"** con la suficiente precisión para que el agente no tenga que adivinar, y con la suficiente brevedad para que un humano la siga leyendo dentro de seis meses.

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

Decisiones de diseño técnico *específicas de esta feature* que el agente necesita conocer para no salirse del camino. No son convenciones generales del proyecto (esas viven en la configuración global del agente — CLAUDE.md, rules, skills —, no en cada spec). Son las decisiones arquitecturales que acotan el espacio de soluciones para *este* cambio concreto.

> *Las imágenes se sirven desde el bucket S3 ya existente, no se introduce uno nuevo. La autenticación pasa por el middleware `requireAuth`, no se inventa otro. No se crea una tabla nueva; el avatar se almacena como campo en el modelo `User` existente.*

Las restricciones son lo que hace que la feature encaje en el resto del sistema en lugar de aterrizar como un cuerpo extraño. Son el antídoto del context collapse del capítulo 1.

### 4. Criterios de aceptación verificables

Cómo sabes que está hecho. La palabra clave es **verificable**: una persona externa, leyendo solo el criterio, debería poder decidir si el código lo cumple o no.

> *Un usuario autenticado puede subir una imagen JPEG o PNG de hasta 10 MB. El upload falla con un mensaje claro si el archivo es de otro tipo o supera el tamaño. Solo el dueño del archivo puede borrarlo. Un test de integración cubre los tres casos.*

"Funciona bien" no es un criterio de aceptación. "Devuelve 413 con mensaje legible si el archivo supera los 10 MB" sí lo es.

#### Reutilizar criterios de documentos upstream (user stories, contratos)

Una pregunta inevitable: si la user story upstream **ya contiene criterios de aceptación**, ¿qué hago con ellos? La respuesta corta es **reescribir con traza** — ni copiar literal (importa la imprecisión del lenguaje de producto), ni referenciar a secas (`"ver JIRA-1234"` rompe la auto-contención del capítulo 1 y abre drift invisible). Reescribes los criterios en la spec al nivel de precisión que el validador del capítulo 6 necesita, y citas la user story en la sección de "por qués" como fuente del *qué motivó la decisión*, no como contenedor del *qué hay que hacer*.

La regla unificada: **la spec contiene su propia versión, precisa y verificable, de los criterios. El documento upstream se cita como fuente del por qué.** El anti-patrón a evitar — *fusionar user story y spec en un único archivo híbrido porque "dicen lo mismo"* — lo desarrollamos como anti-patrón #13 en el capítulo 12.

**Cuando la user story vive en el mismo repo que la spec**, las objeciones más graves se desactivan: la auto-contención se conserva (todo está en git), el drift deja de ser invisible (queda un commit), y aparece una posibilidad nueva — un sensor automático (hook o agente recurrente) que detecte cuando la user story cambia sin que la spec se actualice. Pero la regla de "reescribir con traza" sigue siendo correcta: siguen siendo dos artefactos con propósitos distintos (la user story responde al *qué quiere el usuario*; la spec, al *qué garantías cumple el sistema*), y cada uno cambia por razones diferentes.

Un riesgo concreto de este escenario: la tentación de editar ambos documentos con frecuencia sin que cada cambio esté plenamente justificado. Cada edición cruzada es una oportunidad de desalineamiento — y cuantas más haya, más difícil es saber cuál de los dos refleja la verdad actual. La traza (la referencia explícita que conecta un criterio de la spec con su origen en la user story) cumple un doble propósito: sirve como **punto de comprobación en el momento del cambio** (el humano o el agente que edita la spec puede verificar que el origen sigue siendo válido) y como **material de auditoría posterior** para un agente de doc-gathering que busque discrepancias entre ambos documentos de forma recurrente.

**Una excepción legítima**: cuando el documento upstream es realmente autoritativo y vive bajo su propia disciplina de validación (un contrato OpenAPI/Protobuf con su propio CI, una política de seguridad corporativa, un RFC), la spec **resume las implicaciones** sin copiar el contenido. La distinción crítica: ese documento upstream tiene su *propia* validación operando sobre él. Una user story de Jira casi nunca tiene esa propiedad.

### 5. Los "por qués"

Aquí es donde las specs ordinarias mueren y las buenas se separan del resto. Casi todas las specs explican el *qué*. Casi ninguna explica el *por qué*. Y los por qués son justo lo que el agente necesita para tomar decisiones inteligentes en los huecos que la spec no cubre.

> *No comprimimos la imagen automáticamente porque el equipo de diseño quiere preservar la calidad para los avatares de cuentas verificadas. (Decisión de septiembre 2025, dueña: @maria.)*
>
> *Solo el dueño puede borrar porque añadir moderación cambiaría el modelo de permisos y queremos mantener la spec mínima en esta iteración.*

Los por qués son el ingrediente que [la crítica de Isoform](https://isoform.ai/blog/the-limits-of-spec-driven-development) identifica como lo que el SDD malhecho pierde, y es la razón principal por la que las specs se vuelven inútiles a meses vista. Una spec sin por qués envejece a la velocidad de las decisiones que la rodean. Una spec con por qués sobrevive a esas decisiones porque las explica.

### 6. Boundaries — el sistema de tres niveles

Esta es la contribución más útil de [Addy Osmani](https://addyosmani.com/blog/good-spec/) al pensamiento sobre specs para agentes. En lugar de una lista plana de "haz" y "no hagas", divide las restricciones en **tres tiers**:

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
| **Restricciones técnicas** | Decisiones de diseño que acotan la solución para esta feature | Mezclar con convenciones globales del proyecto |
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
- Decisiones de diseño específicas de esta feature
- Componentes existentes que se deben usar (y cuáles no tocar)
- (Las convenciones globales del proyecto viven en CLAUDE.md / rules, no aquí)

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

## Superficies afectadas
### [Componente] — [producido | modificado | consumido]
- **Funcional:** ...
- **No funcional:** ...
- **Técnico:** ...
- **Más detalle:** [referencia a doc más profunda si existe]
```

*Recomendado siempre que la spec toque más de un componente. Ver [sección 4](04-spec-in-context.md#la-spec-y-los-componentes-técnicos-consumir-producir-modificar) para las reglas de cada tipo de relación.*

Esta plantilla cabe en menos de una pantalla y cubre el 80% del valor que una spec puede aportar. El otro 20% son detalles específicos del dominio — diagramas de estado, contratos de API, ejemplos concretos de input/output — que se añaden cuando el dominio los pide, no por defecto.

## La maldición de las instrucciones

**Más información en una sola spec no es mejor — es peor.** Una spec global de 500 líneas es un mal plan; cinco specs modulares de 100 líneas, donde la tarea de hoy referencia solo dos, es un buen plan. Esta es la regla práctica más importante de todo el capítulo, y va contra la intuición de casi todo el mundo que empieza con SDD.

La razón tiene nombre. [Addy Osmani](https://addyosmani.com/blog/good-spec/), citando estudios académicos, la llama **the curse of instructions**: cuanto más metes en un prompt, peor cumple el modelo *cada una* de las cosas. No es lineal — es un colapso cualitativo. Una spec con cinco restricciones se cumple razonablemente bien. Una spec con cincuenta se cumple razonablemente mal en todas, y muchas veces el agente actúa como si no existiera ninguna.

La estrategia ganadora, entonces, es dividir specs grandes en módulos pequeños y pasarle al agente solo el módulo relevante para la tarea actual.

Esta es también una de las razones por las que las herramientas tipo [Spec-kit y Kiro](07-native-sdd-tools.md) generan **muchos archivos pequeños** en vez de uno grande, y por las que esa proliferación, mal gestionada, se convierte en su propio problema (lo veremos en el capítulo 10).

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

**Implementas dos veces.** Has pagado el coste de pensar la implementación a nivel de clases y métodos en la spec, y vas a pagar otra vez el mismo coste cuando codifiques. Peor: cualquier cosa que descubras al implementar (y al implementar siempre se descubren cosas) te obliga a volver atrás y editar la spec, o a aceptar drift inmediato. Estás en la peor parte de la curva del capítulo 10: maintenance tax desde el día uno, sin haber escrito todavía una línea de código.

**La revisión humana se vuelve tan cara como revisar código.** Esto es exactamente lo que [Fowler reporta de Spec-kit](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) en el capítulo 7 — que los archivos generados eran *más pesados de revisar que el propio código*. Y es peor que revisar código directamente, porque al menos el código se ejecuta y los tests lo verifican; el pseudocódigo en markdown no tiene ninguna red de seguridad. Estás revisando algo del nivel de detalle del código pero **sin las garantías del código**. El reviewer humano, cansado, lo lee por encima — y la "spec" pasa el filtro como si hubiera sido revisada.

La combinación de las dos es venenosa: doblas el trabajo y *además* lo revisas peor. Es casi siempre peor que no haber escrito ninguna spec.

### Por qué no es solo un problema de plantilla

Es tentador pensar que esto se arregla con mejor prompting o una plantilla más estricta. *"Le digo al agente que no incluya firmas, que no mencione clases, que exprese todo como invariantes externamente observables."* Y sí, eso reduce el problema. Pero **no lo elimina**, por dos razones que conviene tener claras.

Primero, el sesgo del modelo a bajar al nivel del código es muy fuerte porque es donde "siente que produce valor". Cuando le restringes todo lo que sabe hacer bien y le pides solo lo que hace mal — abstraer intención, capturar por qués que no están en el input — la spec resultante es corta, pobre, y el equipo intuye que "le falta algo". El siguiente paso es casi siempre pedirle más detalle, y vuelves al pseudocódigo. Es un equilibrio inestable.

Segundo, hay cosas que el agente **no puede** generar bien por mucha plantilla que le des. Los **por qués** casi nunca están en el input — viven en la cabeza del PM, en una conversación de Slack, en un trade-off histórico que se decidió hace dos años. Lo que el agente no sabe, lo inventa o lo omite. Una spec generada con "por qués" inventados es peor que una spec sin "por qués", porque miente con apariencia de autoridad.

### Uso legítimo: el agente como draft generator del *qué*, no del *cómo*

Esto no significa que la generación con agente no sirva. Sirve, pero en un papel más estrecho del que la mayoría de la gente imagina:

- **El agente puede arrancar bien**: el bloque de **objetivo**, los **criterios de aceptación verificables** (con cuidado), una primera lista de **no-goals candidatos** que el humano luego refina, y una propuesta de **boundaries** basada en patrones que ya ve en el repo.
- **El humano tiene que poner sí o sí**: los **por qués**, los **no-goals reales** (especialmente los políticos o de scope que no están en ningún input), las **restricciones técnicas tácitas** del equipo, y la decisión de *cuánta* spec merece la tarea (lo que en el capítulo 9 llamaremos *modulación*: ajustar el peso de la spec a la complejidad y el riesgo reales de la feature). Y, sobre todo, tiene que **borrar** todo lo que el agente metió de pseudocódigo: clases, métodos, firmas, payloads. Si después de borrarlos la spec queda vacía, no necesitabas una spec; necesitabas escribir bien el código (capítulo 11, *context engineering*).

La regla práctica más útil: **si tu spec generada menciona una firma de función o un nombre de clase, bórrala**. La spec habla de qué garantías cumple el sistema, no de cómo se construye.

Y la advertencia honesta: usar el agente como generador de drafts **no reduce el coste total** de hacer SDD bien. Lo que reduce es el coste de empezar el primer borrador, que para muchos equipos es la barrera psicológica que más pesa. El coste real — pensar la intención con precisión, capturar los por qués, mantener viva la spec — sigue estando ahí, sin atajo, y el agente no lo paga por ti. Si tu adopción de SDD depende de que la generación automática elimine ese coste, lo que vas a tener no es SDD — vas a tener el anti-patrón #12 del capítulo 12.

## Lo que viene a continuación

Hasta aquí hemos visto qué poner dentro de una spec y las dos trampas más caras al escribirla: specs demasiado grandes y specs generadas por agentes sin supervisión. Pero la anatomía de una spec no termina en sus bloques — **el nivel de detalle apropiado depende del contexto**: del nivel del espectro en el que operas, de los componentes técnicos que tocas, y del tipo de validación que vas a aplicar.

En la **[sección 4](04-spec-in-context.md)** abordamos exactamente eso: cómo calibrar la spec para que el detalle no sea ni desperdicio ni ceguera.
