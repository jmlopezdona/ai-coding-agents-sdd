# 1. Del vibe coding al SDD: context collapse y context drift

Antes de hablar de specs hay que entender exactamente qué falla cuando no las hay. La conversación pública sobre las limitaciones de los agentes de codificación tiende a quedarse en lugares cómodos — "alucinan", "no entienden el contexto", "el modelo no es lo bastante bueno" — y casi nunca diagnostica los dos fallos concretos que el SDD existe para resolver. Vamos a ponerles nombre.

## Vibe coding, la práctica por defecto

Llamamos *vibe coding* a la forma en que la mayoría de la gente usa hoy los agentes de codificación: abres una sesión, escribes en lenguaje natural lo que quieres ("añade un endpoint para subir fotos"), miras lo que sale, lo iteras un par de veces a base de prompts adicionales, y o bien lo aceptas o lo descartas. Es ágil, es divertido, y para prototipos funciona razonablemente bien.

Y rompe sistemáticamente en cuanto el proyecto deja de ser un prototipo. La razón no es que el agente sea malo. Es que estás operando en un régimen donde dos fallos específicos se vuelven inevitables.

## Fallo nº1: context collapse

El **context collapse** es un fallo *entre* sesiones. Cada conversación nueva con el agente empieza desde cero. El conocimiento que el agente "tenía" ayer no está hoy. Las convenciones de nomenclatura del proyecto, los patrones de manejo de errores que el equipo decidió hace dos sprints, la regla no escrita de que el módulo `legacy/` no se toca porque está en proceso de deprecación — todo eso es invisible al agente nuevo.

En un proyecto de cinco archivos no importa. En uno de doscientos, importa muchísimo. El agente, cuando no encuentra información explícita, hace lo que hace cualquier modelo entrenado en internet: extrapola desde su training general. Y la extrapolación casi nunca coincide con tu proyecto, porque tu proyecto tiene historia, decisiones, y compromisos que no están en el training.

El síntoma típico: el agente produce código sintácticamente correcto, idiomático en general, pero **estilísticamente extranjero al resto del repo**. Nombres que no encajan. Patrones de error handling distintos a los que ya hay. Una capa nueva donde tu equipo nunca habría puesto una capa. Cosas que un humano que conoce el proyecto detectaría en cinco segundos y que el agente, sin contexto, no puede ni sospechar.

## Fallo nº2: context drift

El **context drift** es distinto y vale la pena no confundirlo con el anterior. Ocurre *dentro* de una misma sesión, cuando el agente arregla algo en un archivo y rompe otros tres que no miró. No es que se haya "olvidado" del contexto — es que su ventana de atención no cubre el blast radius real del cambio.

Tú le pides "renombra esta función". El agente la renombra en el archivo donde la encontró. Pero la función se importa desde otros nueve sitios, y el agente no los miró porque su búsqueda se cerró cuando encontró la primera coincidencia. Resultado: nueve roturas que aparecerán cuando ejecutes los tests — si tienes tests — o cuando llegue a producción, si no los tienes.

El context drift es operacional: ocurre incluso si al agente le has dado todo el contexto que cabía en la ventana. Es un fallo de cobertura, no de memoria.

## Por qué los dos fallos juntos son letales

Por separado cada fallo es manejable. Juntos producen un patrón muy concreto que es la firma del vibe coding en producción:

1. Le pides una feature al agente.
2. El agente la implementa razonablemente, sin ver convenciones (collapse) ni dependencias laterales (drift).
3. Tú apruebas el cambio porque "se ve bien".
4. Una semana después, otro cambio rompe esto. Otro agente lo arregla. Otro contexto perdido.
5. Repites veinte veces.
6. El repo es ahora un mosaico de estilos, con una pila de bugs sutiles que nadie sabe cómo se introdujeron.

Eso es vibe coding a escala. No hay un momento donde algo se rompa de forma espectacular. Hay una erosión continua que solo se nota cuando intentas hacer un cambio grande y descubres que el repo se ha vuelto frágil de una forma difícil de explicar.

## Otros síntomas que conviene aprender a nombrar

Más allá de los dos fallos centrales, el vibe coding produce un puñado de síntomas que la literatura ha aprendido a etiquetar:

- **Regresión silenciosa de patrones**: cada feature nueva es ligeramente distinta a las anteriores porque el agente no vio las anteriores.
- **Tokens desperdiciados**: prompts interminables de ida y vuelta para arreglar errores que una spec habría prevenido. Es el coste *económico* del vibe coding y casi nadie lo mide.
- **Pérdida de trazabilidad**: dentro de un año, nadie sabe por qué existe esa lógica. La respuesta vive en un chat que ya no existe.
- **Happy path bias**: el agente, sin spec que enumere casos límite, optimiza para el camino feliz. Los timeouts, las race conditions, los inputs malformados no aparecen en el código generado a menos que se los pidas explícitamente.

Cada uno de estos síntomas es individualmente arreglable. Lo que el SDD propone es atacarlos todos a la vez con un único movimiento estructural: **mover la fuente de verdad del prompt al artefacto persistente**.

## El punto de inflexión

El momento en que vale la pena dejar el vibe coding es muy concreto y muchos equipos lo cruzan sin darse cuenta: cuando empiezas a invertir más tiempo *arreglando* lo que el agente produce que el que habrías invertido escribiéndolo a mano. En ese punto, el agente ha dejado de ser un multiplicador y se ha vuelto un generador de retrabajo.

La medida no es perfecta — un día malo no cuenta — pero como heurística sostenida en el tiempo, es la señal más fiable de que necesitas estructura. Y la estructura no es "más prompts" ni "modelo más caro". Es una spec.

## Qué hace una spec, exactamente

Una spec, en el sentido más mínimo posible, es **un artefacto persistente que captura intención y restricciones, y que el agente lee antes de actuar**. Eso es todo. No tiene que ser larga. No tiene que ser exhaustiva. Tiene que existir, estar en el repo, y ser leída.

Lo que esto resuelve, fila por fila:

| Fallo | Cómo lo resuelve la spec |
|---|---|
| Context collapse | La spec persiste entre sesiones; el contexto deja de ser efímero. |
| Context drift | La spec define invariantes que la verificación post-código puede comparar. |
| Regresión de patrones | La spec referencia los patrones del proyecto; el agente los lee, no los inventa. |
| Tokens desperdiciados | Menos iteraciones porque la primera tiene mejor contexto. |
| Pérdida de trazabilidad | La spec sobrevive al chat. Dentro de un año sigue ahí. |
| Happy path bias | Los casos límite están enumerados explícitamente, no asumidos. |

No es magia. Es simplemente trasladar lo que estaba implícito en tu cabeza a un sitio donde el agente puede leerlo. El precio que pagas es escribir la spec. El precio que evitas es todo lo de arriba.

## Lo que viene a continuación

Si aceptas que el vibe coding tiene un techo y que mover la fuente de verdad al artefacto resuelve la mayoría de los síntomas, la siguiente pregunta es *qué tipo de spec* y *con cuánto compromiso*. Esa es la pregunta del **capítulo 2**, donde introducimos el espectro de la especificación: tres niveles de adopción de SDD, con ventajas y costes muy distintos. La mayoría de equipos creen que están en uno y en realidad están en otro, y entender eso es la decisión más importante que vas a tomar en el curso.
