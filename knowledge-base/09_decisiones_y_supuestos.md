# Decisiones y Supuestos

## Decisiones documentadas
### DD-01 — Uso de n8n en lugar de código nativo o Zapier
**Decisión**: Utilizar n8n para orquestar la lógica del negocio.
**Contexto**: El presupuesto de la PyME es acotado, al igual que los conocimientos de programación avanzada.
**Justificación**: n8n provee una capa low-code, es de código abierto (evitando altos costos por uso de Zapier/Make) y ofrece nodos nativos de LangChain.

### DD-02 — Google Sheets como base de datos
**Decisión**: Utilizar Google Sheets para la persistencia.
**Contexto**: Necesidad de un gestor de datos amigable para personas sin conocimientos técnicos.
**Justificación**: Provee una interfaz familiar al dueño del taller y elimina la curva de aprendizaje respecto a consolas SQL.
**Trade-offs aceptados**: Ausencia de transacciones ACID, posibles condiciones de carrera en alta concurrencia y límites de escalabilidad.

### DD-03 — Mistral mistral-small-latest a temperatura 0.3
**Decisión**: Uso del LLM de Mistral Cloud configurado con temp 0.3.
**Contexto**: Encontrar un balance entre extracción precisa de datos de entidades (DNI) y naturalidad en respuestas.
**Justificación**: Temperaturas muy bajas (0.1) resultaron rígidas; temperaturas altas (0.7) alucinaban o fallaban al extraer datos.

## Supuestos inferidos
### SU-01 — Concurrencia de usuarios
**Supuesto**: El taller maneja volúmenes bajos de turnos simultáneos.
**Origen**: Entrevista con el taller mecánico que indicó 12 a 15 vehículos semanales.
**Riesgo si es falso**: Solapamiento de turnos o problemas de concurrencia al actualizar Google Sheets sin mecanismos de bloqueo de celdas.
**Cómo validar**: Monitoreo de uso del bot en producción.
