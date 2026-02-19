---
summary: "Hvordan Gateway, noder og canvas-vært forbinder."
read_when:
  - Du ønsker et kort overblik over Gatewayens netværksmodel
title: "Netværksmodel"
---

De fleste handlinger går gennem Gateway (`openclaw gateway`), en enkelt langvarigt kørende
proces, der ejer kanalforbindelser og WebSocket-kontrolplanet.

## Grundregler

- En Gateway pr. vært anbefales. Det er den eneste proces tilladt at eje WhatsApp Web session. For redning bots eller streng isolation, køre flere gateways med isolerede profiler og havne. Se [Flere gateways](/gateway/multiple-gateways).
- Loopback først: Gateway WS standard `ws://127.0.0.1:18789`. Guiden genererer en gateway token som standard, selv for loopback. For tailnet adgang, køre `openclaw gateway --bind tailnet --token ...` fordi tokens er påkrævet for ikke-loopback bind.
- Knuder opretter forbindelse til Gateway WS over LAN, tailnet, eller SSH efter behov. Den ældre TCP-bro er forældet.
- Canvas host leveres af Gateway HTTP-serveren på **samme port** som Gateway (standard `18789`):
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    Når `gateway.auth` er konfigureret, og Gateway binder ud over loopback, er disse ruter beskyttet af Gateway-auth (loopback-forespørgsler er undtaget). Se [Gateway configuration](/gateway/configuration) (`canvasHost`, `gateway`).
- Fjernbrug er typisk SSH tunnel eller tailnet VPN. Se [Fjernadgang] (/gateway/remote) og [Discovery](/gateway/discovery).
