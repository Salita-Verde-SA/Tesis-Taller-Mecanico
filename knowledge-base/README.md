# Asistente Inteligente para Taller Mecánico — Base de Conocimiento

Base de conocimiento generada a partir del Trabajo Final de Carrera (UTN).

## Índice de Archivos

| Archivo | Contenido |
|---------|-----------|
| [01_vision_y_objetivos.md](01_vision_y_objetivos.md) | Propósito, objetivos, métricas y alcance del MVP del bot. |
| [02_descripcion_general.md](02_descripcion_general.md) | Stack basado en n8n, Mistral Cloud, LangChain y Google Sheets. |
| [03_actores_y_roles.md](03_actores_y_roles.md) | Matriz RBAC para Clientes y Administradores (Mecánico). |
| [04_modelo_de_datos.md](04_modelo_de_datos.md) | Estructura de la base de datos (Stock, Clientes, Turnos). |
| [05_reglas_de_negocio.md](05_reglas_de_negocio.md) | Lógicas de validación de identidad y confirmación de acciones. |
| [06_funcionalidades.md](06_funcionalidades.md) | Épicas de usuario para reserva y actualización de inventario. |
| [07_flujos_principales.md](07_flujos_principales.md) | Flujos paso a paso de registro de turnos y modificación de stock. |
| [08_arquitectura_propuesta.md](08_arquitectura_propuesta.md) | Pipeline en n8n, nodo Switch, segregación de privilegios. |
| [09_decisiones_y_supuestos.md](09_decisiones_y_supuestos.md) | Justificación de low-code (n8n), LLM open weights (Mistral) y Google Sheets. |
| [10_preguntas_abiertas.md](10_preguntas_abiertas.md) | Inconsistencias (colisiones de turno) y próximos pasos. |

## Quick Start para Desarrolladores

1. Entender el dominio → [01](01_vision_y_objetivos.md), [03](03_actores_y_roles.md)
2. Entender los datos → [04](04_modelo_de_datos.md)
3. Entender las reglas → [05](05_reglas_de_negocio.md)
4. Entender la arquitectura → [02](02_descripcion_general.md), [08](08_arquitectura_propuesta.md)
5. Implementar → [07](07_flujos_principales.md), [06](06_funcionalidades.md)
6. Antes de codificar → [10](10_preguntas_abiertas.md)

## Resumen Ejecutivo

Sistema automatizado de gestión operativa para un taller mecánico, implementado en n8n mediante agentes conversacionales LangChain impulsados por Mistral. Utiliza Telegram como interfaz y Google Sheets como base de datos, separando las capacidades públicas (reservas) de las privadas (control de stock).
