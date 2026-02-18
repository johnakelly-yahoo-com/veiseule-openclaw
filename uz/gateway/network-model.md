---
title: "Tarmoq modeli"
---

Ko‘pgina amallar Gateway (`openclaw gateway`) orqali, ya’ni uzoq muddat ishlaydigan yagona xizmat orqali amalga oshiriladi.
process that owns channel connections and the WebSocket control plane.

## Asosiy qoidalar

- One Gateway per host is recommended. It is the only process allowed to own the WhatsApp Web session. Qutqaruv botlari yoki qat’iy izolyatsiya uchun, izolyatsiyalangan profillar va portlar bilan bir nechta shlyuzlarni ishga tushiring. See [Multiple gateways](/gateway/multiple-gateways).
- Avval loopback: Gateway WS sukut bo‘yicha `ws://127.0.0.1:18789` manziliga sozlangan. Wizard odatda, hatto loopback uchun ham, gateway token yaratadi. Tailnet orqali kirish uchun `openclaw gateway --bind tailnet --token ...` ni ishga tushiring, chunki loopback’dan tashqari ulanishlar uchun tokenlar talab qilinadi.
- Tugunlar zaruratga ko‘ra LAN, tailnet yoki SSH orqali Gateway WS’ga ulanadi. Eski TCP ko‘prigi eskirgan (deprecated).
- Canvas host is an HTTP file server on `canvasHost.port` (default `18793`) serving `/__openclaw__/canvas/` for node WebViews. See [Gateway configuration](/gateway/configuration) (`canvasHost`).
- Remote use is typically SSH tunnel or tailnet VPN. See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery).
