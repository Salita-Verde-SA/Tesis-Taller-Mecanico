# Reglas de Negocio

Cada regla tiene un código único `RN-{DOMINIO}-{NN}` para trazabilidad.

## Dominio: Autenticación (RN-AU)
- **RN-AU-01**: El acceso al panel administrativo se valida comparando el campo `from.id` de Telegram contra una lista predefinida (hardcodeada) de administradores autorizados.
- **RN-AU-02**: Todo usuario cuyo ID no coincida con la lista de administradores es enrutado forzosamente al Agente de Servicio Técnico (Público).

## Dominio: Turnos (RN-TU)
- **RN-TU-01**: El registro de un turno nuevo requiere identificar al cliente por su DNI en la base de datos.
- **RN-TU-02**: Si el DNI no existe en la hoja "Clientes", el sistema debe solicitar al usuario los datos e invocar la herramienta de creación de cliente antes de registrar el turno.
- **RN-TU-03**: El registro de un turno exige los detalles del vehículo (marca, modelo, año), diagnóstico y fecha pactada.

## Dominio: Stock (RN-ST)
- **RN-ST-01**: El precio de costo de los repuestos está estrictamente oculto para los usuarios no administrativos.
- **RN-ST-02**: Cualquier operación de actualización de inventario ejecutada por un administrador requiere confirmación explícita (e.g., "¿Confirma que desea cambiar el stock...?") antes de modificar la base de datos.
