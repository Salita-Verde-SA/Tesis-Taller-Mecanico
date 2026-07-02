# Tesis para la tecnicatura universitaria en programación de la UTN

Las instrucciones del agente están en AGENTS.md.

## TODO

### General

- [x] Actualizar el informe para agregar los subworkflows, cancelación de turno y herramientas. También borrar el párrafo de una vulnerabilidad en 6.5 que ya no aplica (e investigar por qué).

### Agentes

#### Admin

- [x] Modificar el prompt para que muestre los clientes y el stock de forma correcta, actualmente los formatea como tablas de markdown.
- [x] Mostrar de forma más corta las consultas de clientes, turnos, stock, que pueden llegar a ser muy largas; y paginar de a 4 resultados usando botones de telegram para pasar de pestaña. Consultar la API de telegram para el uso de botones y mensajes dinámicos.
- [ ] Que se maneje todo por botones en lugar de decir textualmente, por ejemplo, "si para confirmar, no para cancelar". Que el mensaje de bienvenida tenga botones inline para mandar mensajes que digan "ver clientes", "ver stock", etc.
- [x] Corregir problema al actualizar stock, está mal la herramienta, devuelve el siguiente error: "columns.matchingColumns is required for the Append or Update Row operation
Set columns.matchingColumns to a non-empty string\[\] of header names that uniquely identify the row to update (e.g. \["id"\] or \["email"\]). If there is no key column to match on, use the append operation instead."

#### Cliente

- [ ] Cuando hace una recomendación de fecha al estar ocupada la que se solicita, que también compruebe la disponibilidad antes de recomendar, actualmente no lo hace y recomienda fechas y horarios que pueden estar ocupados.
- [ ] Elaborar mejor el mensaje de turno confirmado, está usando la dirección del cliente como dirección del taller al intentar decir "te esperamos en...".
- [ ] No debe pasar el código al consultar el stock.
- [ ] Antes de intentar agendar un turno, a la hora de preguntar, comprobar si tiene tres turnos, no puede tener más de tres turnos.
- [ ] No formatear como markdown, ni "#", ni "---", ni otras cosas que puedan haber.
- [ ] No debe responder a temas no relacionados al taller, ni aceptar bromas
