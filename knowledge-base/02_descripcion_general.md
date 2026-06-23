# Descripción General

## Stack tecnológico
| Capa | Tecnologías | Versión mínima |
|---|---|---|
| Orquestación / Flujos | n8n | 1.0+ (Soporte LangChain) |
| Interfaz de Usuario | Telegram Bot API | N/A |
| Inteligencia Artificial | Mistral Cloud (mistral-small-latest) | N/A |
| Persistencia de Datos | Google Sheets | N/A |
| Framework de IA | LangChain | N/A |

## Arquitectura general
El sistema utiliza una arquitectura basada en eventos orquestada por n8n. Un `Telegram Trigger` recibe los mensajes, que son enrutados por un nodo `Switch` hacia dos ramas distintas (Agente Público o Agente Administrativo) basándose en el ID del usuario. Cada rama utiliza un `AI Agent` (LangChain) respaldado por Mistral Cloud, el cual tiene acceso a herramientas específicas (Google Sheets Tools) para interactuar con la base de datos. La respuesta se envía de vuelta al usuario a través de nodos de Telegram.

## Integraciones externas
| Servicio | Propósito | Tipo (REST/webhook/SDK) |
|---|---|---|
| Telegram API | Interfaz conversacional con el usuario. | Webhook (Telegram Trigger) y REST (Send) |
| Mistral Cloud | Razonamiento LLM y procesamiento de lenguaje natural. | API (nodo nativo) |
| Google Sheets API | Persistencia y almacenamiento de base de datos. | OAuth2 (nodo nativo) |
