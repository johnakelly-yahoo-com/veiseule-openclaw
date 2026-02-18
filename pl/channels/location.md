---
title: "Parsowanie lokalizacji kanału"
---

# Parsowanie lokalizacji kanału

OpenClaw normalizuje udostępnione lokalizacje z kanałów czatu do postaci:

- czytelnego dla człowieka tekstu dołączanego do treści przychodzącej oraz
- ustrukturyzowanych pól w ładunku kontekstu automatycznej odpowiedzi.

Obecnie obsługiwane:

- **Telegram** (pinezki lokalizacji + miejsca/venue + lokalizacje na żywo)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` z `geo_uri`)

## Formatowanie tekstu

Lokalizacje są renderowane jako przyjazne linie bez nawiasów:

- Pinezka:
  - `📍 48.858844, 2.294351 ±12m`
- Nazwane miejsce:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Udostępnianie na żywo:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Jeśli kanał zawiera podpis/komentarz, jest on dołączany w następnej linii:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Pola kontekstu

Gdy obecna jest lokalizacja, do `ctx` dodawane są następujące pola:

- `LocationLat` (liczba)
- `LocationLon` (liczba)
- `LocationAccuracy` (liczba, metry; opcjonalne)
- `LocationName` (ciąg znaków; opcjonalne)
- `LocationAddress` (ciąg znaków; opcjonalne)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## Uwagi dotyczące kanałów

- **Telegram**: miejsca (venues) mapowane są do `LocationName/LocationAddress`; lokalizacje na żywo używają `live_period`.
- **WhatsApp**: `locationMessage.comment` oraz `liveLocationMessage.caption` są dołączane jako linia podpisu.
- **Matrix**: `geo_uri` jest parsowane jako lokalizacja pinezki; wysokość (altitude) jest ignorowana, a `LocationIsLive` jest zawsze false.

