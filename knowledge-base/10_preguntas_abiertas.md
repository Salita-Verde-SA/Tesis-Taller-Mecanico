# Preguntas Abiertas

## Inconsistencias detectadas
### IN-01 — Gestión de Horarios en Turnos
**Documento A dice**: El registro incluye "Fecha y hora pactada".
**Problema**: No se detalla cómo el sistema previene que un usuario reserve un horario que ya está ocupado.
**Impacto**: Posibles turnos dobles en el mismo slot de tiempo.
**Resolución propuesta**: Implementar una lectura previa de los turnos en el Agente Público antes de confirmar un nuevo turno, o incorporar una integración oficial con Google Calendar.

## Preguntas abiertas (priorizadas)
| Prioridad | Pregunta | Bloquea | Decisor |
|-----------|----------|---------|---------|
| Alta | ¿Cómo se revocan los accesos de administradores? Actualmente requiere cambiar el workflow de n8n manualmente. | Escalabilidad | Administrador del sistema |
| Media | ¿Se requerirá manejo de cancelaciones de turnos por parte de los clientes? | MVP V1.1 | Dueño del Taller |
| Baja | ¿Cuál es la política para manejo de datos personales bajo la Ley 25.326? | Producción | Dueño del Taller |
