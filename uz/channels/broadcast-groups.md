---
summary: "Bir nechta agentlarga WhatsApp xabarini translyatsiya qilish"
read_when:
  - Translyatsiya guruhlarini sozlash
  - WhatsApp’da ko‘p agentli javoblarni debug qilish
status: eksperimental
title: "Translyatsiya guruhlari"
---

# Translyatsiya guruhlari

**Holat:** Eksperimental  
**Versiya:** 2026.1.9 da qo‘shilgan

## Umumiy ko‘rinish

Translyatsiya guruhlari bir nechta agentlarga bir xil xabarni bir vaqtning o‘zida qayta ishlash va javob berish imkonini beradi. Bu bitta WhatsApp guruhi yoki DM ichida bir telefon raqamidan foydalangan holda birgalikda ishlaydigan ixtisoslashgan agent jamoalarini yaratish imkonini beradi.

Joriy qamrov: **faqat WhatsApp** (veb kanal).

Translyatsiya guruhlari kanal ruxsat ro‘yxatlari va guruh faollashtirish qoidalaridan keyin baholanadi. WhatsApp guruhlarida bu shuni anglatadiki, OpenClaw odatda qachon javob bersa (masalan, guruh sozlamalaringizga qarab eslatilganda), o‘sha paytda translyatsiyalar amalga oshadi.

## 1. Foydalanish holatlari

### 2. 1. 3. Ixtisoslashgan agent jamoalari

4. Aniq va yo‘naltirilgan vazifalarga ega bir nechta agentlarni ishga tushiring:

```
5. Guruh: "Dasturlash jamoasi"
```

Agentlar:

### - CodeReviewer (kod parchalarini ko‘rib chiqadi)   - DocumentationBot (hujjatlarni yaratadi)

```
  - SecurityAuditor (zaifliklarni tekshiradi)
```

### - TestGenerator (test holatlarini taklif qiladi) 6. Har bir agent bir xil xabarni qayta ishlaydi va o‘zining ixtisoslashgan nuqtai nazarini taqdim etadi.

```
7. 2.
```

### 8. Ko‘p tilli qo‘llab-quvvatlash 9. Guruh: "Xalqaro qo‘llab-quvvatlash"

```
Agentlar:
```

## - Agent_EN (ingliz tilida javob beradi)

### - Agent_DE (nemis tilida javob beradi)

- Agent_ES (ispan tilida javob beradi) 10. 3.

- 11. Sifatni ta’minlash jarayonlari
- 12. Guruh: "Mijozlarni qo‘llab-quvvatlash"

```json
Agentlar:
```

- SupportAgent (javob beradi)

### - QAAgent (sifatni tekshiradi, faqat muammo topilsa javob beradi)

13. 4.

#### 14. Vazifalarni avtomatlashtirish

15. Guruh: "Loyihani boshqarish"

```json
Agentlar:
```

#### - TaskTracker (vazifalar bazasini yangilaydi)

- TimeLogger (sarflangan vaqtni qayd etadi)

```json
  - ReportGenerator (hisobotlar yaratadi)
```

### 16. Konfiguratsiya

```json
17. Asosiy sozlash
```

## 18. Yuqori darajadagi `broadcast` bo‘limini (`bindings` yoniga) qo‘shing.

### 19. Kalitlar — WhatsApp peer ID’lari:

1. 20. guruh chatlari: guruh JID (masalan, `120363403215116621@g.us`)
2. 21. Shaxsiy xabarlar: E.164 telefon raqami (masalan, `+15551234567`)
3. 22. {
       "broadcast": {
       "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
       }
       }
   - 23. **Natija:** OpenClaw ushbu chatda javob berganda, u uchala agentni ishga tushiradi.
   - 24. Qayta ishlash strategiyasi
   - 25. Agentlar xabarlarni qanday qayta ishlashini boshqaring:
4. 26. Parallel (standart)
   - 27. Barcha agentlar bir vaqtning o‘zida qayta ishlaydi:

28) {
    "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
    }
    } 29. Ketma-ket

### 30. Agentlar navbat bilan qayta ishlaydi (bittasi tugamaguncha keyingisi kutadi):

31. {
    "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
    }
    }

- 32. To‘liq misol
- 33. {
      "agents": {
      "list": [
      {
      "id": "code-reviewer",
      "name": "Code Reviewer",
      "workspace": "/path/to/code-reviewer",
      "sandbox": { "mode": "all" }
      },
      {
      "id": "security-auditor",
      "name": "Security Auditor",
      "workspace": "/path/to/security-auditor",
      "sandbox": { "mode": "all" }
      },
      {
      "id": "docs-generator",
      "name": "Documentation Generator",
      "workspace": "/path/to/docs-generator",
      "sandbox": { "mode": "all" }
      }
      ]
      },
      "broadcast": {
      "strategy": "parallel",
      "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
      "120363424282127706@g.us": ["support-en", "support-de"],
      "+15555550123": ["assistant", "logger"]
      }
      }
- 34. Qanday ishlaydi
- **Asboblar ruxsati** (turli allow/deny roʻyxatlari)
- **Xotira/kontekst** (alohida IDENTITY.md, SOUL.md va boshqalar)
- **Guruh kontekst buferi** (kontekst uchun ishlatiladigan soʻnggi guruh xabarlari) har bir peer uchun umumiy, shuning uchun ishga tushirilganda barcha broadcast agentlar bir xil kontekstni koʻradi

Bu har bir agentga quyidagilarga ega boʻlish imkonini beradi:

- Turli shaxsiyatlar
- Turli asboblar ruxsati (masalan, faqat oʻqish vs. oʻqish-yozish)
- Turli modellar (masalan, opus vs. sonnet)
- Turli oʻrnatilgan koʻnikmalar

### Misol: Izolyatsiyalangan sessiyalar

**Alfred konteksti:**

**Alfred konteksti:**

```
Sessiya: agent:alfred:whatsapp:group:120363403215116621@g.us
Tarix: [foydalanuvchi xabari, alfredning oldingi javoblari]
Ish maydoni: /Users/pascal/openclaw-alfred/
Asboblar: read, write, exec
```

**Bärbel konteksti:**

```
Sessiya: agent:baerbel:whatsapp:group:120363403215116621@g.us
Tarix: [foydalanuvchi xabari, baerbelning oldingi javoblari]
Ish maydoni: /Users/pascal/openclaw-baerbel/
Asboblar: faqat read
```

## Eng yaxshi amaliyotlar

### 1. Agentlarni fokusda saqlang

Har bir agentni bitta aniq masʼuliyat bilan loyihalang:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Yaxshi:** Har bir agentning bitta vazifasi bor  
❌ **Yomon:** Bitta umumiy "dev-helper" agent

### 2. Tavsiflovchi nomlardan foydalaning

Har bir agent nima qilishini aniq koʻrsating:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Turli asboblar ruxsatini sozlang

Agentlarga faqat kerakli asboblarni bering:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Faqat oʻqish
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Oʻqish-yozish
    }
  }
}
```

### 4. Ishlashni kuzatib boring

Koʻp agentlar bilan quyidagilarni koʻrib chiqing:

- Tezlik uchun `"strategy": "parallel"` (standart) dan foydalanish
- Broadcast guruhlarini 5–10 agent bilan cheklash
- Oddiy agentlar uchun tezroq modellarni ishlatish

### 5. Nosozliklarni muloyimlik bilan boshqaring

Agentlar mustaqil ravishda xatoga uchraydi. Bitta agentning xatosi boshqalarni bloklamaydi:

```
Xabar → [Agent A ✓, Agent B ✗ xato, Agent C ✓]
Natija: Agent A va C javob beradi, Agent B xatoni log qiladi
```

## Provayderlar

### Provayderlar

26. Hozirda broadcast guruhlari quyidagilar bilan ishlaydi:

- 27. ✅ WhatsApp (joriy etilgan)
- 🚧 Telegram (rejalashtirilgan)
- 🚧 Discord (rejalashtirilgan)
- 🚧 Slack (rejalashtirilgan)

### Yoʻnaltirish

Broadcast guruhlari mavjud routing bilan birga ishlaydi:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Faqat alfred javob beradi (oddiy routing)
- `GROUP_B`: agent1 VA agent2 javob beradi (broadcast)

**Ustuvorlik:** `broadcast` `bindings` ustidan ustun turadi.

## Agentlar javob bermayapti

### Agentlar javob bermayapti

**Tekshiring:**

1. 28. Agent IDlari `agents.list` da mavjud
2. Peer ID formati to‘g‘ri (masalan, `120363403215116621@g.us`)
3. Agentlar deny ro‘yxatlarida emas

**Debug:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Faqat bitta agent javob bermoqda

**Yechim:** Broadcast konfiguratsiyasiga qo‘shing yoki bindings’dan olib tashlang.

**Yechim:** Broadcast konfiguratsiyasiga qo‘shing yoki bindings’dan olib tashlang.

### Unumdorlik muammolari

**Agar ko‘p agentlar bilan sekin bo‘lsa:**

- Guruhdagi agentlar sonini kamaytiring
- Yengilroq modellardan foydalaning (opus o‘rniga sonnet)
- Sandbox ishga tushish vaqtini tekshiring

## Misollar

### 29. Misol 1: Kodni ko‘rib chiqish jamoasi

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Foydalanuvchi yuboradi:** Kod parchasi  
**Javoblar:**

- code-formatter: "Chekinishlar tuzatildi va tur ko‘rsatkichlari qo‘shildi"
- security-scanner: "⚠️ 12-qatorida SQL injection zaifligi"
- test-coverage: "Qamrov 45%, xato holatlar uchun testlar yetishmaydi"
- docs-checker: "`process_data` funksiyasi uchun docstring yetishmaydi"

### Misol 2: Ko‘p tilli qo‘llab-quvvatlash

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Konfiguratsiya sxemasi

### Konfiguratsiya sxemasi

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Maydonlar

- `strategy` (ixtiyoriy): Agentlarni qanday qayta ishlash
  - "parallel" (standart): Barcha agentlar bir vaqtda ishlaydi
  - "sequential": Agentlar massiv tartibida ishlaydi
- [peerId]: WhatsApp guruh JID’i, E.164 raqami yoki boshqa peer ID
  - Qiymat: Xabarlarni qayta ishlashi kerak bo‘lgan agent ID’lari massivi

## Cheklovlar

1. **Maksimal agentlar:** Qattiq limit yo‘q, lekin 10+ agent sekin bo‘lishi mumkin
2. **Umumiy kontekst:** Agentlar bir-birining javoblarini ko‘rmaydi (dizayn bo‘yicha)
3. **Xabarlar tartibi:** Parallel javoblar istalgan tartibda kelishi mumkin
4. **Rate limitlar:** Barcha agentlar WhatsApp rate limitlariga kiradi

## Kelajakdagi yaxshilanishlar

Rejalashtirilgan funksiyalar:

- [ ] Umumiy kontekst rejimi (agentlar bir-birining javoblarini ko‘radi)
- [ ] Agentlarni muvofiqlashtirish (agentlar bir-biriga signal bera oladi)
- [ ] Dinamik agent tanlash (xabar mazmuniga qarab agentlarni tanlash)
- [ ] Agent ustuvorliklari (ba’zi agentlar boshqalardan oldin javob beradi)

## Shuningdek qarang

- [Multi-Agent Configuration](/tools/multi-agent-sandbox-tools)
- [Yo‘naltirish konfiguratsiyasi](/channels/channel-routing)
- [Sessiyalarni boshqarish](/concepts/sessions)

