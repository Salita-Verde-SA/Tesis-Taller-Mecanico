# Funcionalidades

Organizadas por **épica** y luego por **historia de usuario** (formato US-NNN).

## Épica 1: Atención al Cliente y Turnos
### US-001 — Solicitud de Turno
**Como** Cliente
**Quiero** solicitar un turno mediante Telegram
**Para** reservar un arreglo para mi vehículo sin llamar en horario comercial

**Criterios de aceptación**:
- [ ] El sistema identifica si soy cliente existente o me registra.
- [ ] El sistema pide marca, modelo, año y problema del auto.
- [ ] El turno se registra correctamente en la hoja `Turnos`.

### US-002 — Consulta de Repuestos (Cliente)
**Como** Cliente
**Quiero** consultar la disponibilidad de un repuesto
**Para** saber si el taller tiene la pieza que busco

**Criterios de aceptación**:
- [ ] El agente retorna si hay stock del producto consultado.
- [ ] El sistema oculta activamente el "Precio de Costo".

## Épica 2: Administración de Taller
### US-003 — Actualización de Inventario
**Como** Administrador
**Quiero** actualizar la cantidad en stock de un producto
**Para** mantener el inventario sincronizado con el depósito físico

**Criterios de aceptación**:
- [ ] El sistema me solicita confirmación explícita antes de guardar el cambio.
- [ ] La cantidad se actualiza en la hoja `Stock`.

### US-004 — Consulta Administrativa
**Como** Administrador
**Quiero** visualizar todos los turnos agendados y los datos completos de los repuestos (incluyendo precio de costo)
**Para** planificar el trabajo y analizar la rentabilidad
