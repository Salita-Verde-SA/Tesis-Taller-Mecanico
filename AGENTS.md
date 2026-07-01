# 🤖 Arquitectura de Agentes AI (Taller Mecánico) + Reglas del asistente

## Rol del asistente

Sos el asistente de desarrollo del Taller Mecánico. Tu responsabilidad es mantener el flujo de n8n y la documentación sincronizados.

## Source of Truth — Flujo de n8n

Los archivos JSON en este repositorio son la fuente de verdad definitiva del flujo de n8n:
- `Taller Mecanico.json` — Workflow principal

## Documentación de referencia (tesis)

La carpeta `docs/` contiene el informe de tesis. **No se modifica, no se reescribe, no se toca.** El archivo `docs/Tesis_Taller_Mecanico_V2.md` es la especificación funcional del sistema en formato Markdown legible para el agente. TODO el trabajo en el flujo de n8n debe alinearse con lo que describe ese documento.

### Reglas de sincronización

1. **No modificar docs/**: La carpeta `docs/` contiene el informe de tesis final. No se edita, no se mueve, no se toca bajo ningún concepto.
2. **Alineación con la tesis**: Cualquier cambio en el flujo de n8n debe ser coherente con lo especificado en `docs/Tesis_Taller_Mecanico_V2.md`. Si un cambio se desvía de la tesis, requiere justificación explícita.
3. **Importación (repo → n8n)**: Si el flujo en n8n está desactualizado respecto a los JSON del repo, importalos a n8n.
4. **Exportación (n8n → repo)**: Si hacés cambios directamente en n8n (nodos, conexiones, config), exportá el workflow actualizado a su JSON correspondiente y commitealo.
5. **Verificación**: Antes de considerar una tarea completa, verificá que los JSON del repo reflejen el estado real del flujo en n8n.
6. **Credenciales vacías en JSON**: Todos los bloques `credentials` en los JSON del repo deben contener solo el `"name"` de la credencial, sin `"id"`. Ejemplo: `"credentials": {"telegramApi": {"name": "Telegram account"}}`. n8n matchea automáticamente por nombre al importar. Nunca incluir IDs, ya que cambian entre instancias o al recrear credenciales.

---

## 🔀 Enrutamiento & Orquestación (Telegram)

El flujo se dispara mediante un **Telegram Trigger**.
Los mensajes entrantes pasan por un nodo **Switch**, el cual determina la ruta del mensaje:
- Si el mensaje es exactamente `ADMIN`, se deriva al **Agente Administrador** (`Agente Administrador`).
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

## 🦸‍♂️ Agente: Agente Administrador (Asistente de Administración)

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
