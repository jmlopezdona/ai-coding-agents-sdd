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

## Cómo usar esta lista

Una vez al trimestre, durante una retrospectiva, lee esta lista en voz alta y pregunta al equipo: *¿reconocemos alguno de estos en nosotros?* La gente, contra lo que el ego promete, casi siempre reconoce uno o dos. Y reconocerlos es la mitad de pararlos.

Si reconoces tres o más, **el problema no es uno de los anti-patrones; es la adopción entera de SDD**. Vuelve al capítulo 9 y al 10 y replantea si SDD es la herramienta correcta para tu situación.

## Lo que viene a continuación

El **capítulo 12** cierra el curso conectando SDD con el siguiente paso: el harness. Cuáles son los puntos exactos donde la disciplina se acopla con la ingeniería del harness, y por qué los dos juntos producen mucho más que los dos por separado.
