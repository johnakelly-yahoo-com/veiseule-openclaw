---
summary: "Fejlfinding af node-parring, krav til forgrund, tilladelser og værktøjsfejl"
read_when:
  - Node er forbundet, men kamera-/canvas-/skærm-/exec-værktøjer fejler
  - Du har brug for den mentale model for node-parring versus godkendelser
title: "Fejlfinding af node"
---

# Fejlfinding af node

Brug denne side, når en node er synlig i status, men nodeværktøjer fejler.

## Kommandotrin

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Kør derefter nodespecifikke tjek:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

Sunde signaler:

- Node er forbundet og parret til rollen `node`.
- `nodes describe` inkluderer den kapabilitet, du kalder.
- Exec-godkendelser viser forventet tilstand/tilladelsesliste.

## Krav til forgrund

`canvas.*`, `camera.*` og `screen.*` er kun i forgrund på iOS-/Android-noder.

Hurtigt tjek og løsning:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

Hvis du ser `NODE_BACKGROUND_UNAVAILABLE`, så bring node-appen i forgrunden og prøv igen.

## Tilladelsesmatrix

| Kapabilitet                  | iOS                                                                     | Android                                                           | macOS node-app                                     | Typisk fejlkode                |
| ---------------------------- | ----------------------------------------------------------------------- | ----------------------------------------------------------------- | -------------------------------------------------- | ------------------------------ |
| `camera.snap`, `camera.clip` | Kamera (+ mikrofon for kliplyd)                      | Kamera (+ mikrofon for kliplyd)                | Kamera (+ mikrofon for kliplyd) | `*_PERMISSION_REQUIRED`        |
| `screen.record`              | Skærmoptagelse (+ mikrofon valgfri)                  | Prompt for skærmoptagelse (+ mikrofon valgfri) | Skærmoptagelse                                     | `*_PERMISSION_REQUIRED`        |
| `location.get`               | Mens appen bruges eller Altid (afhænger af tilstand) | Forgrunds-/baggrundsplacering afhængigt af tilstand               | Placerings­tilladelse                              | `LOCATION_PERMISSION_REQUIRED` |
| `system.run`                 | n/a (node host path)                                 | n/a (node host path)                           | Exec-godkendelser kræves                           | `SYSTEM_RUN_DENIED`            |

## Parring versus godkendelser

Det er forskellige barrierer:

1. **Enhedsparring**: kan denne node forbinde til gatewayen?
2. **Exec-godkendelser**: kan denne node køre en specifik shell-kommando?

Hurtige tjek:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

Godkend først indholdselementet, hvis parring mangler.
Hvis parring er fin, men `system.run` mislykkes, fix exec godkendelser/allowlist.

## Almindelige node-fejlkoder

- `NODE_BACKGROUND_UNAVAILABLE` → appen er i baggrunden; bring den i forgrunden.
- `CAMERA_DISABLED` → kamerakontakt er deaktiveret i node-indstillinger.
- `*_PERMISSION_REQUIRED` → OS-tilladelse mangler/afvist.
- `LOCATION_DISABLED` → placeringstilstand er slået fra.
- `LOCATION_PERMISSION_REQUIRED` → den anmodede placeringstilstand er ikke givet.
- `LOCATION_BACKGROUND_UNAVAILABLE` → appen er i baggrunden, men der findes kun tilladelsen Mens appen bruges.
- `SYSTEM_RUN_DENIED: approval required` → exec-anmodning kræver eksplicit godkendelse.
- `SYSTEM_RUN_DENIED: allowlist miss` → kommando blokeret af tilladelseslistetilstand.

## Hurtig gendannelsesloop

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

Hvis du stadig sidder fast:

- Godkend enhedsparring igen.
- Genåbn node-appen (forgrund).
- Giv OS-tilladelser igen.
- Genskab/justér politik for exec-godkendelser.

Relateret:

- [/nodes/index](/nodes/index)
- [/nodes/camera](/nodes/camera)
- [/nodes/location-command](/nodes/location-command)
- [/tools/exec-approvals](/tools/exec-approvals)
- [/gateway/pairing](/gateway/pairing)

