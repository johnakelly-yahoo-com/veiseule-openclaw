---
title: "channels/location.md"
---

# Kanal-Standort-Parsing

OpenClaw normalisiert geteilte Standorte aus Chat-Kanälen zu:

- menschenlesbarem Text, der an den eingehenden Nachrichtentext angehängt wird, und
- strukturierten Feldern im Kontext-Payload der automatischen Antwort.

Derzeit unterstützt:

- **Telegram** (Standort-Pins + Orte + Live-Standorte)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` mit `geo_uri`)

## Textformatierung

Standorte werden als freundliche Zeilen ohne Klammern dargestellt:

- Pin:
  - `📍 48.858844, 2.294351 ±12m`
- Benannter Ort:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Live-Freigabe:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Wenn der Kanal eine Bildunterschrift/einen Kommentar enthält, wird dieser in der nächsten Zeile angehängt:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Kontextfelder

Wenn ein Standort vorhanden ist, werden diese Felder zu `ctx` hinzugefügt:

- `LocationLat` (Zahl)
- `LocationLon` (Zahl)
- `LocationAccuracy` (Zahl, Meter; optional)
- `LocationName` (Zeichenkette; optional)
- `LocationAddress` (Zeichenkette; optional)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (Boolean)

## Kanalhinweise

- **Telegram**: Orte werden auf `LocationName/LocationAddress` abgebildet; Live-Standorte verwenden `live_period`.
- **WhatsApp**: `locationMessage.comment` und `liveLocationMessage.caption` werden als Beschriftungszeile angehängt.
- **Matrix**: `geo_uri` wird als Pin-Standort geparst; die Höhe wird ignoriert und `LocationIsLive` ist immer false.


