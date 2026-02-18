---
title: "Kompaktering"
---

# Kontextfönster & kompaktering

Varje modell har ett **sammanhangsfönster** (max polletter det kan se). Långsiktiga chattar samlar meddelanden och verktygsresultat; när fönstret är tätt, OpenClaw **kompakter** äldre historik för att hålla sig inom gränserna.

## Vad kompaktering är

Komprimering **sammanfattar äldre konversation** i en kompakt sammanfattande post och håller de senaste meddelandena intakta. Sammanfattningen lagras i sessionshistoriken, så framtida förfrågningar användning:

- Kompakteringssammanfattningen
- Senaste meddelanden efter kompakteringspunkten

Kompaktering **består** i sessionens JSONL-historik.

## Konfiguration

Se [Kompakteringskonfig & lägen](/concepts/compaction) för inställningarna `agents.defaults.compaction`.

## Autokompaktering (på som standard)

När en session närmar sig eller överskrider modellens kontextfönster utlöser OpenClaw autokompaktering och kan försöka om den ursprungliga begäran med den kompakterade kontexten.

Du ser:

- `🧹 Auto-compaction complete` i utförligt läge
- `/status` som visar `🧹 Compactions: <count>`

Innan komprimering kan OpenClaw köra en **tyst minne flush** sväng för att lagra
hållbara anteckningar till disk. Se [Memory](/concepts/memory) för detaljer och konfiguration.

## Manuell kompaktering

Använd `/compact` (valfritt med instruktioner) för att tvinga en kompakteringskörning:

```
/compact Focus on decisions and open questions
```

## Källa för kontextfönster

Kontextfönstret är modellspecifikt. OpenClaw använder modelldefinitionen från den konfigurerade leverantörskatalogen för att bestämma gränser.

## Kompaktering vs beskärning

- **Kompaktering**: sammanfattar och **består** i JSONL.
- **Sessionsbeskärning**: trimmar endast gamla **verktygsresultat**, **i minnet**, per begäran.

Se [/concepts/session-pruning](/concepts/session-pruning) för detaljer om beskärning.

## Tips

- Använd `/compact` när sessioner känns stela eller kontexten är uppblåst.
- Stora verktygsutdata trunkeras redan; beskärning kan ytterligare minska ansamling av verktygsresultat.
- Om du behöver ett helt nytt blad, `/new` eller `/reset` startar ett nytt sessions-ID.

