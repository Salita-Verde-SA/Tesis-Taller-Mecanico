# Actores y Roles

## Actores del sistema
| Actor | Descripción | Cómo interactúa |
|---|---|---|
| Cliente (Público) | Usuario estándar que busca reservar un turno o hacer consultas. | Mediante chat en lenguaje natural con el bot de Telegram. |
| Administrador (Privado) | Mecánico o encargado del taller, con permisos elevados. | Mediante chat en lenguaje natural con el bot, validado por su User ID. |

## RBAC — Matriz de permisos
| Rol | Recurso (Hoja) | Permisos (CRUD) |
|---|---|---|
| Cliente | Turnos | Crear (C) |
| Cliente | Clientes | Leer su propio registro (R), Crear (C) |
| Cliente | Stock | Leer sin precio de costo (R) |
| Administrador | Turnos | Leer lista completa (R) |
| Administrador | Clientes | Leer lista completa (R) |
| Administrador | Stock | Leer completo (R), Actualizar (U) |

## Rutas públicas
- Todas las interacciones iniciales con el Bot de Telegram están abiertas al público. Los usuarios no autorizados son derivados automáticamente al Agente de Servicio Técnico (público).
