# 🤖 Arquitectura de Agentes AI (Taller Mecánico)

Este documento detalla la arquitectura de los agentes inteligentes configurados en n8n para el Taller Mecánico, sus responsabilidades, configuración y herramientas conectadas a Google Sheets.

## 🔀 Enrutamiento & Orquestación (Telegram)

El flujo se dispara mediante un **Telegram Trigger**.
Los mensajes entrantes pasan por un nodo **Switch**, el cual determina la ruta del mensaje:
- Si el mensaje es exactamente `ADMIN`, se deriva al **Agente Administrador** (`AI Agent1`).
- Si el mensaje es `/start`, se envía un **Mensaje de Bienvenida** estático.
- Cualquier otro mensaje se deriva al **Agente de Clientes** (`Servicio Tecnico`).

Las respuestas generadas por los agentes retornan a Telegram mediante los nodos finales **Responder Cliente** y **Responder Admin**.

---

## 🦸‍♂️ Agente: Servicio Tecnico (Asistente de Clientes)

**Rol**: Asistente virtual del Taller Mecánico. Atiende a los clientes por Telegram de forma amable, profesional y eficiente. Gestiona consultas de stock público, registro de clientes y agendamiento de turnos.
**Modelo**: Mistral Cloud Chat Model
**Memoria**: `Simple Memory2` (Window Buffer Memory), separada por `chat_id` para mantener el contexto individual de cada cliente.

### 📝 System Prompt (Reglas Principales)
> - **Horarios**: Lunes a Sábados, de 09:00 a 18:00 hs. Domingos cerrado.
> - **Bloques Exactos**: Los turnos duran 1 hora en punto (09:00, 10:00... hasta 17:00).
> - **Límite**: Un cliente NO puede tener más de 3 turnos agendados en total.
> - **Gestión de IDs (Relacional)**: El agente nunca genera IDs ni pide IDs al registrar clientes. En su lugar, usa el DNI para verificar al cliente y obtener su `ID_Cliente` interno. A partir de ahí, TODA operación de turnos usa el `ID_Cliente`.

### 🛠️ Herramientas (Tools)
Todas las herramientas se conectan al entorno de Google Sheets.
- **Verificar_Identidad**: Busca en la hoja `clientes` filtrando por la columna `DNI` para recuperar los datos del cliente (incluyendo el vital `ID_Cliente`).
- **Registrar Cliente**: Agrega una nueva fila en la hoja `clientes` (Nombre, Apellido, DNI, Dirección, Teléfono). No envía el `ID_Cliente` (se delega a Sheets).
- **Obtener Turnos**: Busca en la hoja `turnos` filtrando por la columna `ID_Cliente` para validar el límite de turnos vigentes.
- **Agendar_Turno**: Agrega una fila en la hoja `turnos`. Requiere enviar `ID_Cliente`, `Auto`, `Motivo`, `Fecha del Turno` y `Horario`.
- **Consultar_Stock_Publico**: Lee la hoja `stock` para informar repuestos disponibles y el `Precio a la Venta`. Tiene explícitamente prohibido mostrar precios de costo.

---

## 🦸‍♂️ Agente: AI Agent1 (Asistente de Administración)

**Rol**: Asistente de gestión interna del Taller Mecánico, accesible SOLO para el administrador o mecánico a cargo. Ayuda a gestionar el stock, los turnos y los clientes de forma rápida mediante comandos.
**Modelo**: Mistral Cloud Chat Model
**Memoria**: `Simple Memory` (Window Buffer Memory).

### 📝 System Prompt (Reglas Principales)
> - **Panel Interno**: Uso exclusivo interno. No revelar esta información sensible en otros contextos.
> - **Confirmación**: Debe confirmar siempre las modificaciones antes de ejecutarlas (ej: "¿Confirmás actualizar el stock de X a Y?").
> - **Tono**: Lenguaje directo, técnico y sin exceso de formalidades.

### 🛠️ Herramientas (Tools)
- **Obtener Repuestos1**: Lee la hoja `stock`. Muestra información completa incluyendo `Código`, `Precio de costo`, `Precio de venta` y `Stock actual`.
- **Actualizar Stock**: Modifica la hoja `stock`. Requiere el ID o código exacto del repuesto y la nueva cantidad.
- **Obtener Clientes1**: Lee la hoja `clientes` para ver el registro completo.
- **Obtener Turnos1**: Lee la hoja `turnos` para visualizar toda la agenda, diagnósticos, presupuestos y estado de las reparaciones.
