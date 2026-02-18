---
summary: "CLI حوالہ برائے `openclaw devices` (ڈیوائس جوڑی بنانا + ٹوکن کی گردش/منسوخی)"
read_when:
  - آپ ڈیوائس جوڑی بنانے کی درخواستوں کی منظوری دے رہے ہوں
  - آپ کو ڈیوائس ٹوکنز کی گردش یا منسوخی درکار ہو
title: "ڈیوائسز"
---

# `openclaw devices`

ڈیوائس جوڑی بنانے کی درخواستوں اور ڈیوائس-محدود ٹوکنز کا انتظام کریں۔

## کمانڈز

### `openclaw devices list`

زیرِ التواء جوڑی بنانے کی درخواستیں اور جوڑی شدہ ڈیوائسز کی فہرست دکھائیں۔

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices approve <requestId>`

زیرِ التواء ڈیوائس جوڑی بنانے کی درخواست منظور کریں۔

```
openclaw devices approve <requestId>
```

### `openclaw devices reject <requestId>`

زیرِ التواء ڈیوائس جوڑی بنانے کی درخواست مسترد کریں۔

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`

کسی مخصوص کردار کے لیے ڈیوائس ٹوکن کی گردش کریں (اختیاری طور پر اسکوپس اپ ڈیٹ کرتے ہوئے)۔

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --role <role>`

کسی مخصوص کردار کے لیے ڈیوائس ٹوکن منسوخ کریں۔

```
openclaw devices revoke --device <deviceId> --role node
```

## عام اختیارات

- `--url <url>`: Gateway WebSocket URL (کنفیگ ہونے پر بطورِ طے شدہ `gateway.remote.url`)۔
- `--token <token>`: Gateway ٹوکن (اگر درکار ہو)۔
- `--password <password>`: Gateway پاس ورڈ (پاس ورڈ تصدیق)۔
- `--timeout <ms>`: RPC ٹائم آؤٹ۔
- `--json`: JSON آؤٹ پٹ (اسکرپٹنگ کے لیے سفارش کردہ)۔

نوٹ: جب آپ `--url` سیٹ کرتے ہیں تو CLI کنفیگ یا ماحولیاتی اسناد کی طرف رجوع نہیں کرتا۔
Pass `--token` or `--password` explicitly. Missing explicit credentials is an error.

## نوٹس

- ٹوکن روٹیشن ایک نیا ٹوکن (حساس) واپس کرتی ہے۔ اسے راز کی طرح محفوظ رکھیں۔
- ان کمانڈز کے لیے `operator.pairing` (یا `operator.admin`) اسکوپ درکار ہے۔
