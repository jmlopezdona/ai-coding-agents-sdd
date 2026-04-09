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

Lo que el agente *no puede* tocar, y lo que *tiene que* respetar. Son los invariantes del sistema.

> *Las imágenes se sirven desde el bucket S3 ya existente, no se introduce uno nuevo. La autenticación pasa por el middleware `requireAuth`, no se inventa otro. El error handling sigue el patrón `Result<T, AppError>` que se usa en `src/api/`.*

Las restricciones son lo que hace que la feature encaje en el resto del sistema en lugar de aterrizar como un cuerpo extraño. Son el antídoto del context collapse del capítulo 1.

### 4. Criterios de aceptación verificables

Cómo sabes que está hecho. La palabra clave es **verificable**: una persona externa, leyendo solo el criterio, debería poder decidir si el código lo cumple o no.

> *Un usuario autenticado puede subir una imagen JPEG o PNG de hasta 10 MB. El upload falla con un mensaje claro si el archivo es de otro tipo o supera el tamaño. Solo el dueño del archivo puede borrarlo. Un test de integración cubre los tres casos.*

"Funciona bien" no es un criterio de aceptación. "Devuelve 413 con mensaje legible si el archivo supera los 10 MB" sí lo es.

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
```

Esta plantilla cabe en menos de una pantalla y cubre el 80% del valor que una spec puede aportar. El otro 20% son detalles específicos del dominio — diagramas de estado, contratos de API, ejemplos concretos de input/output — que se añaden cuando el dominio los pide, no por defecto.

## La maldición de las instrucciones

Hay una trampa concreta a evitar. Addy Osmani, citando estudios académicos, identifica lo que llama **the curse of instructions**: cuanto más metes en un prompt, peor cumple el modelo *cada una* de las cosas. No es lineal — es un colapso cualitativo. Una spec con cinco restricciones se cumple razonablemente bien. Una spec con cincuenta se cumple razonablemente mal en todas, y muchas veces el agente actúa como si no existiera ninguna.

La consecuencia operativa es contraintuitiva: **más información en una sola spec no es mejor**. La estrategia ganadora es dividir specs grandes en módulos pequeños y pasarle al agente solo el módulo relevante para la tarea actual. Una spec global de 500 líneas es un mal plan; cinco specs modulares de 100 líneas cada una, donde la tarea de hoy referencia solo dos, es un buen plan.

Esta es también una de las razones por las que las herramientas tipo Spec-kit y Kiro generan **muchos archivos pequeños** en vez de uno grande, y por las que esa proliferación, mal gestionada, se convierte en su propio problema (lo veremos en el capítulo 9).

## ¿Y si la spec la genera un agente?

Es una idea natural: si el agente sabe escribir código, también debería saber escribir specs. Le pasas una user story, una descripción de la feature o un componente arquitectónico, y te devuelve la plantilla anterior rellena. El coste de arranque baja, la barrera de entrada desaparece, y aparentemente has resuelto el problema más caro de SDD — que es escribir specs.

Es una idea que vale la pena intentar y que **falla casi siempre por defecto**, no por mal prompting sino por una razón estructural que conviene entender antes de gastar semanas descubriéndola.

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
- **El humano tiene que poner sí o sí**: los **por qués**, los **no-goals reales** (especialmente los políticos o de scope que no están en ningún input), las **restricciones técnicas tácitas** del equipo, y la decisión de *cuánta* spec merece la tarea (la modulación del capítulo 8). Y, sobre todo, tiene que **borrar** todo lo que el agente metió de pseudocódigo: clases, métodos, firmas, payloads. Si después de borrarlos la spec queda vacía, no necesitabas una spec; necesitabas escribir bien el código (capítulo 10, *context engineering*).

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

## Lo que distingue una spec buena de una mediocre

Si tienes que recordar tres cosas de este capítulo, que sean estas:

1. **Los no-goals te ahorran más drift que cualquier otra sección**. Escríbelos siempre.
2. **Los por qués son lo que envejece bien**. Sin ellos, la spec dura semanas. Con ellos, dura años.
3. **El "ask first" es la categoría que falta en casi todas las specs**. Es donde vive la colaboración real con el agente.

Las specs malas son las que enumeran trivialidades técnicas y no dicen nada de la intención. Las specs mediocres son las que dicen mucho del *qué* y nada del *por qué*. Las specs buenas son cortas, dicen *qué*, *por qué* y *qué no*, y se pueden leer en cinco minutos sin perder nada importante.

## Lo que viene a continuación

Hasta aquí hemos visto qué hay *dentro* de una spec. En el **capítulo 4** vamos a ver el ciclo en el que una spec vive: las cuatro fases del proceso SDD (con sus dos variantes en la literatura), cómo encadenarlas con un agente, y dónde encaja la verificación que evita que el código se aleje de la spec.
