---
title: "Presensya"
---

# Presensya

Ang “presence” ng OpenClaw ay isang magaan at best‑effort na view ng:

- ang mismong **Gateway**, at
- **mga client na nakakonekta sa Gateway** (mac app, WebChat, CLI, atbp.)

Pangunahing ginagamit ang presence para i-render ang **Instances** tab ng macOS app at
para magbigay ng mabilis na visibility para sa operator.

## Presence fields (kung ano ang lumalabas)

Ang mga presence entry ay mga structured object na may mga field tulad ng:

- `instanceId` (opsyonal pero lubos na inirerekomenda): stable na identidad ng client (karaniwang `connect.client.instanceId`)
- `host`: human‑friendly na host name
- `ip`: best‑effort na IP address
- `version`: string ng bersyon ng client
- `deviceFamily` / `modelIdentifier`: mga hardware hint
- `mode`: `ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, ...
- `lastInputSeconds`: “mga segundo mula sa huling user input” (kung alam)
- `reason`: `self`, `connect`, `node-connected`, `periodic`, ...
- `ts`: timestamp ng huling update (ms mula epoch)

## Mga producer (kung saan nanggagaling ang presence)

Ang mga presence entry ay ginagawa ng maraming source at **pinagsasama**.

### 1. Gateway self entry

Palaging nagse-seed ang Gateway ng isang “self” entry sa startup para makita ng mga UI ang host ng gateway
kahit wala pang nakakonektang client.

### 2. WebSocket connect

Bawat WS client ay nagsisimula sa isang `connect` na kahilingan. Kapag matagumpay ang handshake ang
Gateway upserts a presence entry for that connection.

#### Bakit hindi lumalabas ang mga one‑off na CLI command

Ang CLI ay madalas kumonekta para sa maiikli, minsanang mga utos. Upang maiwasan ang labis na pag-spam sa
Instances list, `client.mode === "cli"` is **not** turned into a presence entry.

### 3. `system-event` beacons

Maaaring magpadala ang mga client ng mas detalyadong pana-panahong beacon sa pamamagitan ng `system-event` na method. Ang mac
app uses this to report host name, IP, and `lastInputSeconds`.

### 4. Mga node connect (role: node)

Kapag kumonek ang isang node sa Gateway WebSocket gamit ang `role: node`, nag-a-upsert ang Gateway
ng isang presence entry para sa node na iyon (parehong flow gaya ng ibang WS client).

## Mga patakaran sa merge + dedupe (bakit mahalaga ang `instanceId`)

Ang mga presence entry ay ini-store sa iisang in‑memory map:

- Ang mga entry ay naka-key sa isang **presence key**.
- Ang pinakamainam na key ay isang stable na `instanceId` (mula sa `connect.client.instanceId`) na nananatili kahit mag-restart.
- Case‑insensitive ang mga key.

Kung muling kumonek ang isang client nang walang stable na `instanceId`, maaari itong lumabas bilang
isang **duplicate** na row.

## TTL at bounded size

Sinasadyang ephemeral ang presence:

- **TTL:** ang mga entry na mas luma sa 5 minuto ay tinatanggal
- **Max entries:** 200 (pinakamatatanda ang unang tinatanggal)

Pinananatiling sariwa nito ang listahan at iniiwasan ang walang hangganang paglaki ng memory.

## Paalaala sa remote/tunnel (mga loopback IP)

Kapag kumokonekta ang isang client sa pamamagitan ng SSH tunnel / lokal na port forward, maaaring ang Gateway ay
see the remote address as `127.0.0.1`. To avoid overwriting a good client‑reported
IP, loopback remote addresses are ignored.

## Mga consumer

### macOS Instances tab

Ine-render ng macOS app ang output ng `system-presence` at nag-a-apply ng maliit na status
indicator (Active/Idle/Stale) batay sa edad ng huling update.

## Mga tip sa pag-debug

- Para makita ang raw na listahan, tawagin ang `system-presence` laban sa Gateway.
- Kung nakakakita ka ng mga duplicate:
  - kumpirmahin na nagpapadala ang mga client ng stable na `client.instanceId` sa handshake
  - kumpirmahin na ang mga periodic beacon ay gumagamit ng parehong `instanceId`
  - tingnan kung nawawala ang `instanceId` sa entry na galing sa koneksyon (inaasahan ang mga duplicate)
