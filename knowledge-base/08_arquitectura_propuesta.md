# Arquitectura Propuesta

## Patrones aplicados
| Patrón | Dónde se usa | Por qué |
|---|---|---|
| Tool-Using Agent (ReAct) | Agentes Mistral | Permite descomponer intenciones complejas en acciones discretas con las bases de datos. |
| Principio de Privilegio Mínimo | Nodos de herramientas | El Agente Público no tiene acceso a operaciones de actualización de stock ni a leer costos. |
| Enrutamiento por Atributo | Nodo Switch post-Trigger | Evita ataques de inyección al basar el acceso admin en el ID numérico duro de Telegram y no en el texto. |

## Estructura del Workflow (n8n)
El workflow sigue una estructura de tubería (pipeline) bifurcada:
1. `Telegram Trigger`
2. `Set/Edit Fields` (Extracción de `from.id` y asignación de boolean `admin`)
3. `Switch` (Bifurcación basada en la variable `admin`)
   - **Rama Pública**:
     - `AI Agent` (Prompt Público) + Google Sheets Tools (Lectura limitada, Creación de Turno/Cliente).
   - **Rama Privada**:
     - `AI Agent` (Prompt Privado) + Google Sheets Tools (Lectura total, Actualización de Stock).
4. `Telegram Send Chat Action` (typing...)
5. `Telegram Send Message` (Respuesta final).

## Seguridad
- **Autenticación (Admin)**: Basada en el `User ID` numérico de Telegram, configurado manualmente en el workflow de n8n.
- **Autorización (Google Sheets)**: OAuth2, otorgando acceso únicamente a nivel de aplicación n8n sobre la hoja.
- **Validación de input**: Dinámica a cargo del modelo Mistral mediante la inyección `$fromAI()`.
