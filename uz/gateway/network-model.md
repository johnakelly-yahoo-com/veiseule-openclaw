---
summary: "Gateway, node’lar va canvas xosti qanday ulanadi."
read_when:
  - You want a concise view of the Gateway networking model
title: "Tarmoq modeli"
---

Most operations flow through the Gateway (`openclaw gateway`), a single long-running
process that owns channel connections and the WebSocket control plane.

## Asosiy qoidalar

- One Gateway per host is recommended. It is the only process allowed to own the WhatsApp Web session. Qutqaruv botlari yoki qat’iy izolyatsiya uchun, izolyatsiyalangan profillar va portlar bilan bir nechta shlyuzlarni ishga tushiring. See [Multiple gateways](/gateway/multiple-gateways).
- Loopback first: the Gateway WS defaults to `ws://127.0.0.1:18789`. The wizard generates a gateway token by default, even for loopback. For tailnet access, run `openclaw gateway --bind tailnet --token ...` because tokens are required for non-loopback binds.
- Nodes connect to the Gateway WS over LAN, tailnet, or SSH as needed. The legacy TCP bridge is deprecated.
- Canvas xosti Gateway HTTP serveri tomonidan Gateway bilan **bir xil portda** (standart `18789`) xizmat ko‘rsatiladi:
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    `gateway.auth` sozlanganda va Gateway loopback’dan tashqariga bog‘langanda, ushbu marshrutlar Gateway auth bilan himoyalanadi (loopback so‘rovlari bundan mustasno). [Gateway configuration](/gateway/configuration) (`canvasHost`, `gateway`) ga qarang.
- Remote use is typically SSH tunnel or tailnet VPN. See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery).
