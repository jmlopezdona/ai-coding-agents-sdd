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

## Lo que distingue una spec buena de una mediocre

Si tienes que recordar tres cosas de este capítulo, que sean estas:

1. **Los no-goals te ahorran más drift que cualquier otra sección**. Escríbelos siempre.
2. **Los por qués son lo que envejece bien**. Sin ellos, la spec dura semanas. Con ellos, dura años.
3. **El "ask first" es la categoría que falta en casi todas las specs**. Es donde vive la colaboración real con el agente.

Las specs malas son las que enumeran trivialidades técnicas y no dicen nada de la intención. Las specs mediocres son las que dicen mucho del *qué* y nada del *por qué*. Las specs buenas son cortas, dicen *qué*, *por qué* y *qué no*, y se pueden leer en cinco minutos sin perder nada importante.

## Lo que viene a continuación

Hasta aquí hemos visto qué hay *dentro* de una spec. En el **capítulo 4** vamos a ver el ciclo en el que una spec vive: las cuatro fases del proceso SDD (con sus dos variantes en la literatura), cómo encadenarlas con un agente, y dónde encaja la verificación que evita que el código se aleje de la spec.
