## El terreno que ya existe

FlowSync hoy solo resuelve la cuenta de usuario, SQLite con dos tablas, relación 1 usuario → N tokens


## El interrogatorio


Aquí van las 5, pensadas para acotar problema, alcance y usuarios antes de proponer nada:

1. ¿Quién y cuántos? Describe al equipo típico que usaría esto: ¿un solo equipo de N personas, o varios equipos dentro de una empresa? ¿Qué roles conviven (solo devs, o también PM/diseño/soporte), y hay alguien que "supervisa" o todos son iguales?
2. ¿Qué es "en qué está trabajando cada uno"? Cuando abres FlowSync por la mañana, ¿qué necesitas ver de un vistazo para no tener que preguntar en el chat? ¿El foco actual de cada persona (una sola cosa "ahora mismo"), su lista del día, o su progreso/bloqueos? ¿Con qué nivel de detalle es útil sin volverse una carga de reportar?
3. ¿Qué reunión estás matando exactamente? ¿El daily standup, los "¿en qué vas?" sueltos del día, o los reportes de estado a un jefe? De lo que hoy se dice en esas reuniones, ¿qué información concreta es la que de verdad hay que reemplazar, y qué parte no importa?
4. ¿Qué significa "tiempo real" para ti? ¿Necesitas ver cambios al instante mientras miras la pantalla (tipo presencia en vivo), o basta con que esté siempre actualizado cuando entres? ¿El equipo está en la misma franja horaria o repartido, de modo que gran parte se consume en asíncrono?
5. ¿Qué es el "rollo de Jira" que NO quieres? Nómbrame 2-3 cosas concretas que Jira te obliga a hacer y que aquí deberían estar prohibidas o ser invisibles (¿estimaciones, sprints, campos obligatorios, flujos de estados, permisos?). Y al revés: ¿cómo sabrías en un mes que FlowSync funcionó — qué comportamiento del equipo habría cambiado?

las respuestas fueron la ficha de hechos del curso, pegada entera y en el prompt 2 de prompts.md

# FlowSync — PRD de alcance (MVP)

> Documento de alcance, problema y usuarios. **No** cubre modelo de datos ni
> diseño de API a propósito. Los huecos que las respuestas de descubrimiento no
> cerraban están marcados como **[Supuesto]** y son decisiones tomadas por
> defecto, revisables.

## 1. El problema

En un equipo remoto pequeño, saber "en qué está trabajando cada uno" hoy cuesta
una interrupción o una reunión:

- La **ronda de "¿en qué estás?"** de la daily se come la mitad de los 15 minutos.
- Fuera de la daily, el "¿en qué vas?" suelto por Slack/chat interrumpe a alguien
  cada vez que alguien más necesita saber el estado del equipo.

Nadie puede ver el estado del equipo **sin interrumpir a una persona**.

**Episodio que lo ilustra:** dos personas tocaron el mismo módulo la misma semana
porque una empezó sin que la otra lo supiera. Dos días perdidos.

### Quién cobra el valor

Los **pares**, no un lead. No hay reporte hacia arriba; a un manager le daría
igual. El dolor lo sienten:

- las dos personas que descubren tarde que iban a lo mismo, y
- quien tiene que interrumpir a otro para preguntar cómo va algo.

### Qué decisión cambia (por qué el "tiempo real" vale su coste)

1. **No empezar algo que otra persona ya está tocando.**
2. **Elegir lo siguiente sabiendo qué está libre.**

Si el único beneficio fuera "sentirse informado", esto no justificaría el
esfuerzo. El valor está en esas dos decisiones.

## 2. Los usuarios

- Equipos remotos **pequeños, 3–10 personas**.
- **Roles planos:** en el MVP todos ven y editan lo mismo. Sin jerarquía de
  permisos.
- **Espacio único compartido**, sin entidad "equipo".

**Primer usuario (caso de estudio, no cliente real):** equipo de 6 personas de
producto SaaS, repartido en **3 husos horarios**, que hoy usa un gestor de tareas
pesado y una daily de 15 min por videollamada.

## 3. La solución, en una frase

El sitio **donde se hace el trabajo** (no donde se cuenta): una lista de tareas
compartida cuyo estado se ve de un vistazo y se actualiza en segundos, para que el
equipo sepa quién está en qué sin preguntar ni reunirse a sincronizar.

### Principios de diseño (lo que la hace sostenible)

- **Frescura de la tarea, no presencia de la persona.** El estado es de la TAREA.
  Nada de "quién está conectado", indicadores de actividad ni nada que huela a
  vigilancia — se rechaza a propósito.
- **Resumen que espera, no aviso que interrumpe.** El caso de uso es "llego por la
  mañana o vuelvo de una reunión y veo qué se ha movido". Sin notificaciones push.
- **Dos clics sobre una lista ya abierta.** Sin campos obligatorios, sin decidir
  sprint ni estimación. Actualizar tiene que costar casi nada.
- **Quien escribe el estado cobra en el momento:** esa misma lista es su cola de
  trabajo (la mira para decidir qué coge) y, de paso, deja de recibir
  interrupciones. Si el beneficio fuera solo para los demás, no lo escribiría.

## 4. Alcance del MVP (dentro)

- **Una tarea contiene:** título, responsable, estado y fecha de vencimiento.
  - La **fecha de vencimiento** existe para ver de un vistazo qué se ha pasado de
    plazo.
- **Crear tarea y cambiarle el estado en segundos**, sin flujos de configuración.
- **El estado lo teclea la persona que hace la tarea.** Manual, en segundos.
- **La lista se consume filtrando por estado**, para centrarse en lo pendiente.
- **Tiempo real = ver los cambios de estado sin refrescar ni preguntar.** El estado
  del equipo está siempre al día cuando entras.
- Sobre la base ya existente: **cuenta de usuario** (signup / login / perfil /
  logout) ya resuelta.


propuso 6 alcance y me quede con 5
cuenta de usuario se excluye,  la IA metió en el alcance algo que ya estaba construido. No llegué a tres exclusiones propias. Estaba revisando del tiempo real, y tuve dudas al respecto, cuando se acabó el tiempo

La decisión dudosa es el tiempo real.El cliente pedía mas en tiempo real, pero le bastaba que la lista esté al día al abrirla. Para que entre el usuario dejara la lista abierta todo el día mientras hace otras cosas, y ahora mismo no conocemos al usuario lo suficiente para saberlo.

### Qué es y qué no es "tiempo real" aquí

- **Sí:** los cambios de estado de las tareas se reflejan sin recargar.
- **No:** chat, videollamada, colaboración simultánea sobre el mismo documento,
  ni indicadores de presencia.

## 5. Fuera del MVP (renuncias explícitas)

Un equipo que necesite lo de abajo **no es nuestro usuario**:

- Sprints, estimaciones, épicas, backlog priorizado, informes/analítica.
- Notificaciones push, integración con Slack.
- Roles / permisos avanzados.
- Comentarios en tareas.
- **Derivar el estado de señales externas** (Git/PRs, CI, calendario): es otro
  producto, con integraciones y OAuth de terceros.
- **Convivir con otro gestor de tareas.** FlowSync **sustituye** al gestor, no lo
  lee. Convivir exigiría doble actualización, que es como muere esta categoría.
- **Bloqueos.** La parte de "bloqueos" de la daily sigue existiendo; este MVP no
  la resuelve (ver §6).
- Varios equipos separados, o gente en más de un equipo (ver **[Supuesto S1]**).

## 6. Qué desaparece y qué no (honesto)

- **Desaparece:** la ronda de "¿en qué estás?" de la daily y los "¿en qué vas?"
  sueltos por chat.
- **No desaparece:** la daily entera. La parte de **bloqueos** sigue, y este MVP
  no la ataca.

## 7. Criterios de éxito

- **Para el usuario:** deja de hacer la ronda de "¿en qué estás?" porque el estado
  del equipo se ve de un vistazo.
- **A una semana de uso real:** el equipo **cancela esa ronda y nadie pide que
  vuelva**. Si la siguen haciendo igual, no funcionó.

## 8. Riesgo principal a validar

**Que la información se quede vieja.** Si el estado no refleja la realidad, el
producto pierde el sentido. Es el riesgo #1, no un detalle.

- **Mitigación:** que actualizar cueste dos clics, no obligar a nadie con campos
  ni recordatorios.

## 9. Filosofía de construcción

Una **vertical fina y usable de punta a punta**, no el andamiaje amplio de un
producto. Preferimos una capability terminada a tres a medias.

## 10. Supuestos (huecos decididos por defecto)

Decisiones tomadas donde el descubrimiento no llegaba. Revisables.

- **[S1] Multi-equipo, fuera.** Hay un único espacio global compartido: cualquier
  usuario registrado ve y edita todas las tareas. Varios equipos aislados, o una
  persona en más de uno, queda fuera del MVP (se anota aquí, no se construye).
- **[S2] Estados de la tarea:** un conjunto mínimo y fijo, del tipo *pendiente →
  en curso → hecha*. Sin estados configurables ni "bloqueada" (los bloqueos van
  por la daily, §6). El conjunto exacto se cierra al diseñar la vertical.
- **[S3] Responsable único, y la tarea puede estar "libre".** Cada tarea tiene como
  mucho un responsable. Una tarea **sin responsable** representa lo que está libre
  para coger — necesario para la decisión "elegir lo siguiente" (§1).
- **[S4] Cualquiera edita cualquier tarea.** Coherente con roles planos: crear,
  reasignar, cambiar estado y borrar están abiertos a todo el equipo. Sin
  propiedad exclusiva ni auditoría de quién cambió qué.
- **[S5] Fecha de vencimiento opcional.** No es campo obligatorio; sirve para
  resaltar lo vencido cuando existe.
- **[S6] Sin histórico.** No se guarda registro de cambios de estado ni actividad
  pasada (coherente con "nada de vigilancia/reporting"). Solo importa el estado
  actual.
- **[S7] El mecanismo de "tiempo real"** (cómo se propaga el cambio sin refrescar)
  es una decisión de diseño técnico, no de alcance; se resuelve en implementación
  respetando "sin push que interrumpa".
