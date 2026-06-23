# Modelo de Datos

## Dominios
- **Stock**: Inventario de repuestos y piezas disponibles en el taller.
- **Clientes**: Información personal de contacto de los usuarios del taller.
- **Turnos**: Reservas de servicio técnico asociadas a un cliente y un vehículo.

## ERD (Entity Relationship Diagram)
- Un `Cliente` puede tener muchos `Turnos` (1:N). La relación se mantiene a través de `ID Cliente`.

## Entidades
### Stock
- **Producto**: Texto
- **Código**: Texto (Identificador único)
- **Precio de Costo**: Numérico (Visibilidad restringida al administrador)
- **Precio a la Venta**: Numérico
- **Stock**: Numérico (Cantidad disponible)

### Clientes
- **ID**: Numérico (Identificador del cliente)
- **Nombre**: Texto
- **Dirección**: Texto
- **DNI**: Numérico (Clave de búsqueda principal)

### Turnos
- **ID Turno**: Numérico (Identificador único)
- **ID Cliente**: Numérico (Clave foránea hacia Clientes)
- **Auto**: Texto (Marca, modelo y año)
- **Diagnóstico**: Texto (Motivo del servicio)
- **Fecha**: Fecha y hora pactada

## Seed data inicial
- El documento requiere un archivo base en Google Sheets llamado "Mecanico" con las tres hojas ("Stock", "Clientes", "Turnos") y sus respectivas cabeceras definidas para funcionar correctamente con la API.
