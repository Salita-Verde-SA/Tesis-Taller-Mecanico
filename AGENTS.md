# 🤖 Arquitectura de Agentes AI (Taller Mecánico) + Reglas del asistente

## Rol del asistente

Sos el asistente de desarrollo del Taller Mecánico. Tu responsabilidad es mantener el flujo de n8n y la documentación sincronizados.

## Source of Truth — Flujo de n8n

Los archivos JSON en este repositorio son la fuente de verdad definitiva del flujo de n8n:
- `Taller Mecanico.json` — Workflow principal
- `SubWorkflow_Consultar_Disponibilidad.json` — Subworkflow de verificación de disponibilidad

## Documentación de referencia (tesis)

La carpeta `docs/` contiene el informe de tesis. El archivo `docs/Tesis_Taller_Mecanico_V2.md` es la especificación funcional del sistema en formato Markdown legible para el agente. TODO el trabajo en el flujo de n8n debe alinearse con lo que describe ese documento.

### Reglas de sincronización

1. **No modificar docs/ salvo sincronización explícita**: La carpeta `docs/` contiene el informe de tesis final. Solo se actualiza cuando se sincroniza con el workflow.
2. **Alineación con la tesis**: Cualquier cambio en el flujo de n8n debe ser coherente con lo especificado en `docs/Tesis_Taller_Mecanico_V2.md`.
3. **Importación (repo → n8n)**: Si el flujo en n8n está desactualizado respecto a los JSON del repo, importalos a n8n.
4. **Exportación (n8n → repo)**: Si hacés cambios directamente en n8n, exportá el workflow actualizado a su JSON correspondiente y commitealo.
5. **Verificación**: Antes de considerar una tarea completa, verificá que los JSON del repo reflejen el estado real del flujo en n8n.
6. **Credenciales vacías en JSON**: Todos los bloques `credentials` en los JSON del repo deben contener solo el `"name"` de la credencial, sin `"id"`. Ejemplo: `"credentials": {"telegramApi": {"name": "Telegram account"}}`. n8n matchea automáticamente por nombre al importar. Nunca incluir IDs.

---

## 🔀 Enrutamiento & Orquestación (Telegram)

El flujo se dispara mediante un **Telegram Trigger**.
Inmediatamente después, un nodo **Validar Admin** (Code) extrae el `from.id` del mensaje y lo compara contra una lista de User IDs de administradores autorizados, seteando la variable booleana `admin`.

Luego un nodo **Switch** evalúa en orden de prioridad:
1. **admin == true** → se deriva al **Agente Administrador** (`Agente Admin`).
2. **admin == false AND texto == "/start"** → se envía un **Mensaje de Bienvenida** estático.
3. **Default** → se deriva al **Agente de Clientes** (`Agente Clientes`).

Las respuestas generadas por los agentes retornan a Telegram mediante los nodos finales **Responder Cliente1** y **Responder Admin1**.

---

## 🦸‍♂️ Agente: Agente Clientes (Asistente de Clientes)

**Rol**: Asistente virtual del Taller Mecánico. Atiende a los clientes por Telegram de forma amable, profesional y eficiente. Gestiona consultas de stock público, registro de clientes y agendamiento de turnos.
**Modelo**: Mistral Cloud Chat Model (`mistral-large-latest`)
**Memoria**: `Simple Memory3` (Window Buffer Memory), sessionKey `chat.id}-v7`, contextWindowLength=30.

### 📝 System Prompt (Reglas Principales)
> - **Horarios**: Lunes a Sábados, de 09:00 a 18:00 hs. Domingos cerrado.
> - **Prioridad disponibilidad**: Ante cualquier mención de horario, consultar `consultar_disponibilidad` ANTES de pedir datos personales.
> - **IDs Incrementales**: ID_Cliente e ID_Turno se calculan como máximo existente + 1.
> - **Confirmación**: Nunca agendar sin confirmación explícita del cliente.
> - **Sin markdown**: Respuestas en texto plano, listas con guiones, emojis puntuales.

### 🛠️ Herramientas (Tools)
Todas las herramientas se conectan al entorno de Google Sheets.
- **Consultar_Stock_Publico1**: Get Many sobre hoja `stock`. Solo muestra información pública (sin costos).
- **Obtener Clientes**: Get Many sobre hoja `clientes` (uso exclusivo para cálculo de IDs).
- **Obtener El ID por DNI**: Get by Filter sobre `clientes` filtrando por DNI.
- **Herramienta_Verificar_DNI1**: Get by Filter sobre `clientes` filtrando por DNI (verificación).
- **Registrar Cliente1**: Append Row sobre hoja `clientes`. Envía ID_Cliente calculado por el agente.
- **Obtener Turnos**: Get Many sobre hoja `turnos` (uso exclusivo para cálculo de IDs).
- **Obtener Turno Por id**: Get by Filter sobre `turnos` filtrando por ID_Cliente.
- **Agendar_Turno**: Append Row sobre hoja `turnos`. Envía ID_Cliente, Auto, Motivo, Fecha del Turno, Horario, Estado.
- **Herramienta_Cancelar_Turno1**: Update Row sobre `turnos` (cambia Estado a Cancelado).
- **Modificar_turno**: Update Row sobre `turnos` para reprogramación.
- **Consultar Disponibilidad Tool**: Workflow Tool que invoca `SubWorkflow_Consultar_Disponibilidad`.

---

## 🦸‍♂️ Agente: Agente Admin (Asistente de Administración)

**Rol**: Asistente de gestión interna del Taller Mecánico, accesible SOLO para el administrador o mecánico a cargo. El acceso se valida por User ID de Telegram contra lista blanca.
**Modelo**: Mistral Cloud Chat Model (`mistral-large-latest`)
**Memoria**: `Simple Memory1` (Window Buffer Memory), contextWindowLength=30.

### 📝 System Prompt (Reglas Principales)
> - **Panel Interno**: Uso exclusivo interno. No revelar información sensible.
> - **Confirmación**: Confirmar siempre las modificaciones antes de ejecutarlas.
> - **Formato Telegram**: NO usar tablas Markdown. Usar listas con guiones, datos en líneas separadas.
> - **Tono**: Lenguaje directo, técnico y sin exceso de formalidades.

### 🛠️ Herramientas (Tools)
- **Obtener Repuestos**: Get Many sobre hoja `stock`. Muestra información completa (Producto, Código, Precio de costo, Precio de venta, Stock actual).
- **Actualizar Stock1**: Update Row sobre hoja `stock`. Requiere el código exacto del repuesto y la nueva cantidad.
- **Obtener Clientes**: Get Many sobre hoja `clientes` para ver el registro completo.
- **Obtener Turnos**: Get Many sobre hoja `turnos` para visualizar toda la agenda.
