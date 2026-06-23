## Why

Actualmente n8n se ejecuta manualmente mediante la consola y el túnel (ngrok o localtunnel) se debe levantar de forma independiente. Esto genera fricción y es propenso a errores. Empaquetar la infraestructura en Docker Compose asegura que n8n y su túnel HTTPS arranquen simultáneamente con la configuración correcta, unificando las variables de entorno en un archivo `.env`.

## What Changes

- Creación de `docker-compose.yml` con servicios para `n8n` y `ngrok`.
- Configuración de red interna de Docker para que ngrok apunte directamente al contenedor de n8n.
- Creación de un archivo `.env.example` con las variables requeridas (dominio de ngrok, authtoken, etc.).
- Montaje de volúmenes para persistir la base de datos de n8n.

## Capabilities

### New Capabilities
- `docker-infrastructure`: Orquestación automatizada de servicios mediante Docker Compose.
- `environment-configuration`: Centralización de secretos y configuración mediante archivo `.env`.

### Modified Capabilities
_(ninguna)_

## Impact

- **Infraestructura**: Cambia el modo de ejecución de manual a orquestado.
- **Desarrollo**: Simplifica el arranque con un solo comando `docker-compose up -d`.
- **Archivos**: Agrega archivos de configuración en la raíz del proyecto.
