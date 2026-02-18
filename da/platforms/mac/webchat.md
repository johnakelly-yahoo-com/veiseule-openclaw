---
title: "WebChat"
---

# WebChat (macOS-app)

MacOS menulinjen app indlejrer WebChat UI som en indfødt SwiftUI visning. It
forbinder til Gateway og standard \*\* main session\*\* for den valgte
agent (med en session switcher for andre sessioner).

- **Lokal tilstand**: forbinder direkte til den lokale Gateway WebSocket.
- **Fjern-tilstand**: videresender Gateway-kontrolporten over SSH og bruger den
  tunnel som dataplan.

## Start & fejlfinding

- Manuelt: Lobster-menu → “Open Chat”.

- Automatisk åbning til test:

  ```bash
  dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
  ```

- Logfiler: `./scripts/clawlog.sh` (undersystem `bot.molt`, kategori `WebChatSwiftUI`).

## Sådan er det koblet sammen

- Dataplan: Gateway WS-metoder `chat.history`, `chat.send`, `chat.abort`,
  `chat.inject` samt events `chat`, `agent`, `presence`, `tick`, `health`.
- Session: standard er den primære session (`main`, eller `global` når omfanget er
  global). Brugergrænsefladen kan skifte mellem sessioner.
- Introduktion bruger en dedikeret session for at holde førstegangsopsætning adskilt.

## Sikkerhedsflade

- Fjern-tilstand videresender kun Gateway WebSocket-kontrolporten over SSH.

## Kendte begrænsninger

- UI’et er optimeret til chatsessioner (ikke en fuld browser-sandbox).

