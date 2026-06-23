# Flujos Principales

Cada flujo se documenta extremo a extremo, mostrando interacciones entre componentes.

## Flujo 1: Registro de Turno Nuevo
**Disparador**: Mensaje del usuario por Telegram pidiendo turno.
**Actor**: Cliente (Público)

**Pasos**:
1. El usuario envía mensaje. Nodo `Telegram Trigger` lo recibe.
2. Nodo `Switch` detecta que el User ID no es admin; enruta a `Agente de Servicio Técnico`.
3. El Agente solicita el DNI del usuario.
4. El Agente invoca herramienta `Obtener Clientes`. Retorna nulo.
5. El Agente solicita nombre y dirección, e invoca `Registrar Cliente`.
6. El Agente solicita datos del auto (marca, modelo, año) y problema.
7. El Agente invoca `Registrar Turno` y confirma con el usuario.

**Diagrama de secuencia** (ASCII):
```text
Cliente → Telegram API → n8n (Trigger) → Switch → Agente Público (LLM)
                                                      ↓ (Tools)
                                                 Google Sheets (DNI, Registrar)
```

## Flujo 2: Actualización de Stock
**Disparador**: Mensaje de actualización de stock.
**Actor**: Administrador (Privado)

**Pasos**:
1. Administrador envía mensaje (ej. "Actualizar pastillas freno a 10").
2. Nodo `Switch` detecta ID autorizado; enruta a `Agente Administrativo`.
3. Agente solicita confirmación: "¿Confirma cambiar el stock de Pastillas a 10?".
4. Administrador responde afirmativamente.
5. Agente invoca herramienta `Actualizar Stock`.
6. Google Sheets se actualiza. Agente informa éxito.
