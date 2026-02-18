---
title: "Pag-parse ng Lokasyon ng Channel"
---

# Pag-parse ng lokasyon ng channel

Ini-standardize ng OpenClaw ang mga ibinahaging lokasyon mula sa mga chat channel tungo sa:

- text na madaling basahin na idinadagdag sa inbound body, at
- mga structured field sa auto-reply context payload.

Kasalukuyang sinusuportahan:

- **Telegram** (mga location pin + venue + live location)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` na may `geo_uri`)

## Pag-format ng text

Ang mga lokasyon ay nirerepresenta bilang mga friendly na linya na walang bracket:

- Naka-pin:
  - `📍 48.858844, 2.294351 ±12m`
- Pinangalanang lugar:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- Live na pag-share:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

Kung may caption/komento ang channel, idinadagdag ito sa susunod na linya:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## Mga field ng konteksto

Kapag may lokasyon, idinaragdag ang mga field na ito sa `ctx`:

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number, meters; opsyonal)
- `LocationName` (string; opsyonal)
- `LocationAddress` (string; opsyonal)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## Mga tala sa channel

- **Telegram**: ang mga venue ay minamapa sa `LocationName/LocationAddress`; ang mga live location ay gumagamit ng `live_period`.
- **WhatsApp**: ang `locationMessage.comment` at `liveLocationMessage.caption` ay idinadagdag bilang caption line.
- **Matrix**: ang `geo_uri` ay bina-parse bilang pin location; binabalewala ang altitude at ang `LocationIsLive` ay palaging false.


