---
title: "Análisis de ubicación del canal"
---

# Análisis de ubicación del canal

OpenClaw normaliza las ubicaciones compartidas desde los canales de chat en:

- texto legible para humanos añadido al cuerpo entrante, y
- campos estructurados en la carga útil de contexto de respuesta automática.

Actualmente compatible con:

- **Telegram** (pines de ubicación + lugares + ubicaciones en vivo)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` con `geo_uri`)

## Formato de texto

Las ubicaciones se representan como líneas amigables sin corchetes:

- Pin:
  - `📍 48.858844, 2.294351 ±12m`
- Lugar con nombre:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Compartir en vivo:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Si el canal incluye un pie de foto/comentario, se añade en la siguiente línea:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Campos de contexto

Cuando hay una ubicación presente, estos campos se añaden a `ctx`:

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number, metros; opcional)
- `LocationName` (string; opcional)
- `LocationAddress` (string; opcional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## Notas por canal

- **Telegram**: los lugares se asignan a `LocationName/LocationAddress`; las ubicaciones en vivo usan `live_period`.
- **WhatsApp**: `locationMessage.comment` y `liveLocationMessage.caption` se añaden como la línea de pie de foto.
- **Matrix**: `geo_uri` se analiza como una ubicación de pin; la altitud se ignora y `LocationIsLive` siempre es false.

