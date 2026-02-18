---
title: "ایجنٹ بوٹ اسٹرَیپنگ"
sidebarTitle: "ابتدائی ترتیب (Bootstrapping)"
---

# ایجنٹ بوٹ اسٹرَیپنگ

Bootstrapping ایک **پہلی بار چلنے** کی رسم ہے جو ایجنٹ کے ورک اسپیس کو تیار کرتی ہے اور
collects identity details. It happens after onboarding, when the agent starts
for the first time.

## بوٹ اسٹرَیپنگ کیا کرتا ہے

ایجنٹ کے پہلی بار چلنے پر، OpenClaw ورک اسپیس کو بوٹ اسٹرَیپ کرتا ہے (بطورِ طے شدہ
`~/.openclaw/workspace`):

- `AGENTS.md`، `BOOTSTRAP.md`، `IDENTITY.md`، `USER.md` کو بیج فراہم کرتا ہے۔
- ایک مختصر سوال و جواب کا عمل چلاتا ہے (ایک وقت میں ایک سوال)۔
- شناخت اور ترجیحات کو `IDENTITY.md`، `USER.md`، `SOUL.md` میں لکھتا ہے۔
- مکمل ہونے پر `BOOTSTRAP.md` کو ہٹا دیتا ہے تاکہ یہ صرف ایک بار ہی چلے۔

## یہ کہاں چلتا ہے

Bootstrapping ہمیشہ **gateway host** پر چلتا ہے۔ اگر macOS ایپ کنیکٹ ہوتی ہے تو
a remote Gateway, the workspace and bootstrapping files live on that remote
machine.

<Note>
جب Gateway کسی دوسری مشین پر چل رہا ہو، تو ورک اسپیس فائلوں میں ترمیم گیٹ وے ہوسٹ
پر کریں (مثال کے طور پر، `user@gateway-host:~/.openclaw/workspace`)۔
</Note>

## متعلقہ دستاویزات

- macOS ایپ آن بورڈنگ: [Onboarding](/start/onboarding)
- ورک اسپیس لے آؤٹ: [Agent workspace](/concepts/agent-workspace)


