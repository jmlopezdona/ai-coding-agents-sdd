# 11. Anti-patrones del SDD

Una lista corta de los errores más caros que se ven en equipos que adoptan Spec-Driven Development. No están aquí para asustarte sino para que cuando los reconozcas en tu propio proceso — y los vas a reconocer — sepas pararlos antes de que se conviertan en hábito.

Cada anti-patrón sigue el mismo formato: **nombre**, síntoma observable, por qué pasa, y la corrección.

---

## 1. Spec-as-Theatre

**Síntoma.** El equipo escribe specs completas con plantilla, las commitea al repo, y nunca las vuelve a leer. Ni el agente, ni los humanos. Las specs existen para "demostrar que hacemos SDD" en revisiones de proceso, no para ser usadas.

**Por qué pasa.** Adopción top-down sin convicción. Alguien decidió "vamos a hacer SDD" y la respuesta del equipo fue cumplir con el ritual mínimo. Las specs son tributo a la autoridad, no herramientas.

**Corrección.** O las specs se usan o se eliminan. No hay punto medio. La señal de que la spec se usa es que cuando hay drift entre código y spec **alguien lo discute en un PR**. Si nunca aparece esa discusión, las specs son teatro y vale más reconocerlo.

---

## 2. Big Spec Up Front

**Síntoma.** Antes de escribir una sola línea de código se invierten dos semanas en una spec exhaustiva de 800 líneas. Para cuando el equipo empieza a implementar, la mitad de las asunciones ya han cambiado.

**Por qué pasa.** Confusión entre SDD y waterfall. La gente que viene de procesos pesados aplica los reflejos pesados al SDD y produce specs gigantes que recuerdan a los design docs de antaño.

**Corrección.** Specs ligeras y vivas. La plantilla del capítulo 3 cabe en una pantalla. Si tu spec necesita más de eso, casi siempre estás describiendo dos features que deberían separarse, o estás especificando cosas que el agente ya sabe inferir del contexto del código.

---

## 3. Spec sin por qués

**Síntoma.** La spec describe en detalle el *qué* — schemas, criterios, contratos — pero no dice por qué se eligieron las decisiones. Seis meses después, nadie sabe si una decisión de la spec sigue siendo válida o es legacy de una restricción que ya no existe.

**Por qué pasa.** Los por qués son la sección más fácil de recortar cuando el equipo va con prisa, y a corto plazo no se nota. A medio plazo, es lo que mata el valor de la spec.

**Corrección.** Los por qués son sección obligatoria, no opcional. En revisiones de PR, una spec sin por qués se trata como una spec incompleta. Y si una decisión no tiene un por qué claro, eso mismo debería bloquear el merge — porque significa que no se ha pensado.

---

## 4. Spec teatral con anclaje fingido

**Síntoma.** El equipo se declara en spec-anchored (capítulo 2) pero no tiene ningún mecanismo automático que detecte drift entre spec y código. Las dos cosas viven en paralelo, ambas se modifican, y nadie sabe en qué momento dejaron de coincidir.

**Por qué pasa.** Ambición sin infraestructura. Aspirar a spec-anchored es fácil; implementar el anclaje (tests, validadores, agentes recurrentes) requiere inversión real que el equipo no hizo.

**Corrección.** Honestidad sobre el nivel real. Si no tienes anclaje automático, estás en spec-first, no en spec-anchored. Llámalo por su nombre. Y si quieres subir de nivel, primero invierte en el mecanismo, después declara el cambio.

---

## 5. Patrón único para cualquier tarea

**Síntoma.** Cada cambio — feature, refactor, bug fix de tres líneas — pasa por el mismo proceso completo: spec con plantilla, plan formal, descomposición en tareas, validación. El equipo se queja de que SDD "es muy lento" y tiene razón.

**Por qué pasa.** Falta de modulación. La adopción se hizo sin enseñar la diferencia entre los patrones del capítulo 8.

**Corrección.** Proceso proporcional al riesgo. Bug fixes triviales no merecen spec; features grandes sí. La regla del capítulo 8 — *el peso de la spec proporcional al coste del cambio si sale mal* — no es opcional, es lo que hace al proceso sostenible.

---

## 6. La spec que devora al código en revisión

**Síntoma.** Los PRs incluyen tantos cambios de markdown (specs, plans, tasks, checklists) que la revisión del propio código queda enterrada. Los reviewers leen markdown durante 20 minutos y miran el diff de código durante 3.

**Por qué pasa.** Fowler lo identifica como uno de los problemas concretos de Spec-kit: el proceso genera más artefactos para revisar que código real. Si no se acota, el "revisar" se vuelve "revisar markdown".

**Corrección.** Medir la ratio markdown/código de los PRs durante un sprint. Si el markdown gana sistemáticamente, simplifica las plantillas o automatiza la generación de los artefactos repetitivos. La revisión humana debe concentrarse donde el juicio aporta valor — el código y las decisiones nuevas — no donde un linter podría pasar.

---

## 7. La spec eterna

**Síntoma.** Una spec se escribe al inicio de la feature y nunca más se actualiza. El código evoluciona, la spec se queda. Seis meses después, leer la spec engaña activamente: dice cosas que el código ya no hace.

**Por qué pasa.** Ausencia de bucle bidireccional (capítulo 5). La definición de "hecho" no incluye actualizar la spec, así que el incentivo es no hacerlo.

**Corrección.** "Hecho" = "código mergeado **y** spec actualizada". Sin lo segundo, la tarea no está hecha. Esto suena duro y lo es, pero es la única forma en que el bucle bidireccional sobrevive a un equipo bajo presión.

---

## 8. Confiar en la spec más que en el código

**Síntoma.** Cuando hay un desacuerdo entre lo que dice la spec y lo que hace el código, el equipo asume que la spec tiene razón y "el código está mal". Este reflejo es exactamente lo opuesto al correcto.

**Por qué pasa.** Una spec densa y bien escrita inspira confianza, y la confianza se convierte en autoridad. Pero la autoridad es injustificada: la spec puede estar mal, desactualizada, o haber sido escrita con asunciones que ya no aplican.

**Corrección.** El código es la fuente de verdad operacional. Cuando hay desacuerdo, **investiga** en lugar de asumir. A veces el código está mal y la spec acierta. A veces — más frecuentemente de lo que el ego del equipo aceptará — es la spec la que está mal.

---

## 9. Promocionar prematuramente a spec-as-source

**Síntoma.** El equipo, entusiasmado con Tessl o con la idea de la generación, declara que "ahora editamos la spec y regeneramos el código". A las dos semanas hay cambios manuales en archivos marcados como `// GENERATED — DO NOT EDIT`, y la promesa se ha desinflado.

**Por qué pasa.** Spec-as-source sigue siendo, hoy, viable solo en dominios estrechos (capítulo 2). Aplicarlo a software de propósito general choca con el no-determinismo de los LLMs y con casos límite que el spec no anticipaba.

**Corrección.** Spec-as-source es experimentación, no producción, salvo en los dominios donde lleva décadas funcionando. Si quieres explorarlo, hazlo en una superficie aislada y reversible. No lo apuestes en el path crítico.

---

## 10. SDD como excusa para no hablar

**Síntoma.** El equipo deja de discutir las decisiones porque "está en la spec". La spec se usa como sustituto de la conversación humana sobre trade-offs. Las preguntas mueren con un "léete la spec".

**Por qué pasa.** Escribir es más cómodo que discutir, especialmente para equipos distribuidos. Una spec elaborada empieza a funcionar como un decreto en lugar de como una propuesta.

**Corrección.** La spec es un punto de partida para la conversación, no su fin. Si una decisión importante de la spec no ha sido cuestionada en seis meses, no es señal de calidad — es señal de que la spec ha matado la conversación que debería estar viva.

---

## 11. SDD aplicado a problemas que no son de SDD

**Síntoma.** El equipo adopta SDD esperando que resuelva problemas que en realidad son de otra cosa: producto mal definido, arquitectura mal pensada, equipo sin experiencia, deploy infrastructure rota. La disciplina no mueve la aguja porque no era la palanca correcta.

**Por qué pasa.** Reificación de la metodología. "Si SDD es bueno, mi problema debe poder resolverse con SDD." No siempre.

**Corrección.** Diagnóstico antes de tratamiento. Pregunta qué está fallando exactamente, antes de decidir qué metodología aplicar. Si el problema es de producto o de infraestructura, ninguna spec lo va a arreglar.

---

## 12. Spec generada que es pseudocódigo disfrazado

**Síntoma.** El equipo usa un agente para generar specs a partir de user stories, features o componentes arquitectónicos. Las specs resultantes están llenas de clases, métodos, firmas de funciones, payloads de API y reglas de negocio inline. Parecen completísimas. Cuando llega la implementación, el equipo descubre que (a) está implementando dos veces, porque las decisiones ya estaban tomadas en la "spec", y (b) la revisión humana de esas specs es tan cara como revisar el código que aún no existe — pero sin la red de seguridad de tests ni ejecución real.

**Por qué pasa.** Cuando le pides a un LLM "genera una spec", el modelo optimiza por *cobertura aparente* y baja al nivel de detalle donde está cómodo: el código. Es donde su corpus de entrenamiento le da más confianza. Subir al nivel correcto de una spec — intención, restricciones, por qués — requiere abstracción, que es justo lo que el modelo hace peor. El resultado es predecible: pseudocódigo en markdown, disfrazado de documento de diseño.

A esto se suma el sesgo del equipo: una spec generada que tiene 600 líneas, clases, métodos y diagramas *parece* rigurosa. El revisor humano, cansado, asume que algo tan detallado tiene que estar bien pensado. Y la "spec" pasa el filtro sin que nadie verifique de verdad si captura intención o solo describe una implementación posible entre muchas. El capítulo 3 desarrolla este anti-patrón en su sección *"¿Y si la spec la genera un agente?"*: aquí solo damos las correcciones operativas.

**Corrección.** Tres reglas concretas:

1. **Si tu spec menciona firmas de función, nombres de clase o estructuras de payload, bórralas.** La spec habla de qué garantías tiene que cumplir el sistema, no de cómo se construye. Si después de borrarlas la spec queda vacía, no necesitabas una spec — necesitabas código bien escrito (cap. 10).
2. **Usa el agente como draft generator solo del *qué*, no del *cómo*.** El agente puede arrancar bien el objetivo, los criterios de aceptación y una lista candidata de no-goals. El humano tiene que poner sí o sí los por qués, los no-goals reales y las restricciones tácitas. Lo que el agente no sabe, lo inventa o lo omite — y los por qués inventados son peores que los por qués ausentes.
3. **Mide la ratio detalle-implementación / detalle-intención de tus specs.** Si más del 30% de la spec describe estructuras de código, estás escribiendo pseudocódigo, no una spec. Es señal de que el agente generador (o tú mismo) ha bajado un nivel de abstracción que rompe el propósito del documento.

El test sencillo: una buena spec sobrevive a **dos implementaciones distintas** del mismo equipo y sigue siendo válida para las dos. Una spec generada como pseudocódigo no — describe *una* forma específica de implementar, y si el código se desvía mínimamente ya está en drift. Si la spec no pasa ese test, no tienes una spec; tienes un boceto de implementación al que llamas spec.

---

## 13. Fusionar user story y spec en un solo archivo

**Síntoma.** El equipo, intentando "no duplicar trabajo", junta la user story y la spec en un único archivo híbrido. El resultado intenta ser las dos cosas a la vez y no es ninguna: es demasiado técnico para que el PM lo refine como user story, y demasiado vago en su lenguaje de producto para que el agente lo use como contrato técnico verificable. Cuando llega un cambio de producto, alguien edita el archivo y un detalle técnico queda inconsistente. Cuando llega un descubrimiento de implementación, alguien edita el archivo y una decisión de producto cambia sin que el PM se entere.

**Por qué pasa.** Es muy tentador, especialmente cuando la documentación de producto vive en el mismo repo Git que las specs. La proximidad física hace que el equipo pregunte "¿para qué tener dos archivos si dicen lo mismo?" — y la respuesta intuitiva es fusionarlos. Pero la pregunta está mal planteada: no dicen lo mismo. La user story responde a *qué quiere el usuario y por qué le importa al negocio*. La spec responde a *qué garantías observables tiene que cumplir el sistema*. Tienen un solapamiento del 60-70%, pero no son la misma cosa, y el solapamiento no justifica fundirlas — cada artefacto sirve a una audiencia distinta y cambia por motivos distintos.

A esto se suma una causa más sutil: el equipo confunde *trazabilidad* con *unificación*. Quiere que el lazo entre intención de producto y contrato técnico sea explícito, y asume que la única forma de lograrlo es ponerlos en el mismo documento. Pero la trazabilidad se consigue con referencias, traces y sensores automáticos (capítulo 5), no fundiendo dos artefactos con propósitos distintos.

**Corrección.** Tres reglas:

1. **Mantén la separación física**, incluso cuando los dos archivos vivan en el mismo repo. La user story en `docs/product/`, la spec en `specs/`, cada una con su propia historia de cambios y su propia audiencia. La proximidad facilita la trazabilidad y la validación cruzada — no la fusión.
2. **El test del autor**. Si el archivo lo edita tanto el PM (por razones de producto) como un ingeniero (por razones de implementación) sin coordinación, es señal de que estás manteniendo dos artefactos colapsados en uno. Sepáralos.
3. **Habilita un sensor de divergencia** entre ambos archivos en lugar de fusionarlos. Un git hook o un agente recurrente del capítulo 5 puede vigilar cuando la user story cambie sin que la spec se actualice (o al revés), y abrir un issue para reconciliarlas conscientemente. Eso preserva las dos audiencias y hace explícita la relación entre los dos artefactos.

El capítulo 3 desarrolla este tema en su sección *"Reutilizar criterios de documentos upstream"* — aquí solo señalamos por qué la fusión silenciosa es un anti-patrón con nombre propio. Es una de las patologías que más amplifica meter la documentación de producto en el repo sin pensar: lo que era una buena idea organizativa se convierte en un arquetipo nuevo de spec malhecha.

---

## 14. Sopa de referencias

**Síntoma.** La spec contiene una lista larga de referencias — ADRs, contratos, feature briefs, componentes, user stories, documentos de arquitectura — todas citadas sin priorización ni anotación. El bloque "Referencias" es más largo que el contenido propio de la spec. Cuando el agente lee la spec, no sabe cuáles de esas referencias son load-bearing para esta tarea y cuáles son contexto tangencial. Cuando el humano la revisa en seis meses, le pasa lo mismo. El resultado es que **nadie las sigue**, y la spec efectiva acaba siendo solo el contenido visible — el resto es ruido decorativo.

**Por qué pasa.** Hay dos causas que se refuerzan. Primero, el equipo confunde *exhaustividad de citas* con *rigor*: cita todo lo "relacionado" para parecer riguroso, sin distinguir lo que esta spec realmente depende de lo que solo está en el mismo barrio del repo. Segundo, hay un sesgo defensivo: añadir referencias se siente "barato" (no rompe nada, no obliga a pensar) frente a quitarlas (que requiere defender por qué algo no se necesita).

A esto se suma el efecto compuesto del capítulo 3: cuando el equipo aprende a referenciar documentos upstream en lugar de copiarlos (que es la regla correcta), la pendiente fácil es referenciarlo *todo*. La regla "referencia, no copies" se interpreta como "referencia siempre", y se pierde la otra mitad: "referencia solo lo que esta tarea necesita". Es el inverso de la *maldición de las instrucciones* del capítulo 3 — y el efecto paralizante es el mismo.

**Corrección.** Tres reglas:

1. **Referencia solo lo que el agente o el humano necesita para esta tarea**, no lo que está tangencialmente relacionado. Si una referencia desaparecería sin que la implementación cambie, no debe estar en la spec.
2. **Anota el porqué de cada referencia**. *"Citado porque este endpoint debe respetar el invariante de la sección 3 de ADR-007"* es útil. *"Relacionado: ADR-007"* es ruido. La anotación es la prueba de que has pensado por qué la referencia está ahí.
3. **Mide la ratio referencias / contenido propio**. Si el bloque de citas es más largo que las secciones de criterios + por qués + restricciones técnicas, hay sopa. Recorta hasta que el contenido propio domine.

El test sencillo: pídele a alguien externo al equipo que lea la spec y subraye las referencias *imprescindibles* para entender qué hay que construir. Si subraya menos del 30% de las que aparecen, las otras son sopa. Bórralas o mueve al final como "contexto opcional", separado del cuerpo de la spec.

El capítulo 3 desarrolla este tema en su sección *"La spec y los componentes técnicos: consumir, producir, modificar"* — aquí solo damos el síntoma con nombre y la corrección operativa. Es un anti-patrón especialmente común en equipos que vienen de adoptar la disciplina de "no copies, referencia": han aprendido la mitad correcta de la regla pero no la otra mitad.

---

## Cómo usar esta lista

Una vez al trimestre, durante una retrospectiva, lee esta lista en voz alta y pregunta al equipo: *¿reconocemos alguno de estos en nosotros?* La gente, contra lo que el ego promete, casi siempre reconoce uno o dos. Y reconocerlos es la mitad de pararlos.

Si reconoces tres o más, **el problema no es uno de los anti-patrones; es la adopción entera de SDD**. Vuelve al capítulo 9 y al 10 y replantea si SDD es la herramienta correcta para tu situación.

## Lo que viene a continuación

El **capítulo 12** cierra el curso conectando SDD con el siguiente paso: el harness. Cuáles son los puntos exactos donde la disciplina se acopla con la ingeniería del harness, y por qué los dos juntos producen mucho más que los dos por separado.
