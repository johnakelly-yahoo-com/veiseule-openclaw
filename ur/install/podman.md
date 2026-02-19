---
summary: "OpenClaw کو ایک rootless Podman کنٹینر میں چلائیں"
read_when:
  - آپ Docker کے بجائے Podman کے ساتھ ایک کنٹینرائزڈ gateway چاہتے ہیں
title: "Podman"
---

# Podman

OpenClaw gateway کو ایک **rootless** Podman کنٹینر میں چلائیں۔ Docker کی طرح اسی امیج کا استعمال کرتا ہے (repo کے [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) سے build کریں)۔

## ضروری تقاضے

- Podman (rootless)
- ایک بار کی سیٹ اپ کے لیے sudo (یوزر بنانا، امیج build کرنا)

## فوری آغاز

**1. ایک بار کی سیٹ اپ** (repo کے روٹ سے؛ یوزر بناتا ہے، امیج build کرتا ہے، launch اسکرپٹ انسٹال کرتا ہے):

```bash
./setup-podman.sh
```

یہ `~openclaw/.openclaw/openclaw.json` میں ایک کم سے کم فائل بھی بناتا ہے ( `gateway.mode="local"` سیٹ کرتا ہے) تاکہ gateway وزرڈ چلائے بغیر شروع ہو سکے۔

بطورِ ڈیفالٹ کنٹینر کو systemd سروس کے طور پر انسٹال نہیں کیا جاتا، آپ اسے دستی طور پر شروع کرتے ہیں (نیچے دیکھیں)۔ پروڈکشن طرز کی سیٹ اپ کے لیے جس میں خودکار آغاز اور ری اسٹارٹس شامل ہوں، اسے systemd Quadlet یوزر سروس کے طور پر انسٹال کریں:

```bash
./setup-podman.sh --quadlet
```

(یا `OPENCLAW_PODMAN_QUADLET=1` سیٹ کریں؛ صرف کنٹینر اور launch اسکرپٹ انسٹال کرنے کے لیے `--container` استعمال کریں۔)

**2. gateway شروع کریں** (دستی طور پر، فوری اسموک ٹیسٹنگ کے لیے):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. آن بورڈنگ وزرڈ** (مثلاً چینلز یا پرووائیڈرز شامل کرنے کے لیے):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

پھر `http://127.0.0.1:18789/` کھولیں اور `~openclaw/.openclaw/.env` سے ٹوکن استعمال کریں (یا وہ ویلیو جو setup کے دوران پرنٹ ہوئی ہو)۔

## Systemd (Quadlet، اختیاری)

اگر آپ نے `./setup-podman.sh --quadlet` (یا `OPENCLAW_PODMAN_QUADLET=1`) چلایا ہے تو ایک [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) یونٹ انسٹال ہو جاتا ہے تاکہ gateway، openclaw یوزر کے لیے systemd یوزر سروس کے طور پر چلے۔ سیٹ اپ کے اختتام پر سروس کو فعال اور شروع کر دیا جاتا ہے۔

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet فائل یہاں موجود ہوتی ہے: `~openclaw/.config/containers/systemd/openclaw.container`۔ پورٹس یا env تبدیل کرنے کے لیے اس فائل (یا اس کی سورس کی گئی `.env`) میں ترمیم کریں، پھر `sudo systemctl --machine openclaw@ --user daemon-reload` چلائیں اور سروس ری اسٹارٹ کریں۔ بوٹ کے وقت، اگر openclaw کے لیے lingering فعال ہو (setup یہ کام کرتا ہے جب loginctl دستیاب ہو)، تو سروس خود بخود شروع ہو جاتی ہے۔

اگر ابتدائی سیٹ اپ میں quadlet استعمال نہیں کیا گیا تھا اور بعد میں شامل کرنا ہو تو دوبارہ چلائیں: `./setup-podman.sh --quadlet`۔

## openclaw یوزر (بغیر لاگ اِن)

`setup-podman.sh` ایک مخصوص سسٹم یوزر `openclaw` بناتا ہے:

- **Shell:** `nologin` — کوئی انٹرایکٹو لاگ اِن نہیں؛ حملے کے امکانات کم کرتا ہے۔

- **Home:** مثلاً `/home/openclaw` — اس میں `~/.openclaw` (config، workspace) اور لانچ اسکرپٹ `run-openclaw-podman.sh` شامل ہوتے ہیں۔

- **Rootless Podman:** صارف کے پاس **subuid** اور **subgid** کی حد (range) ہونی ضروری ہے۔ بہت سی distros صارف بناتے وقت یہ خودکار طور پر تفویض کر دیتی ہیں۔ اگر setup وارننگ دے تو `/etc/subuid` اور `/etc/subgid` میں درج ذیل لائنیں شامل کریں:

  ```text
  openclaw:100000:65536
  ```

  پھر اسی صارف کے طور پر gateway شروع کریں (مثلاً cron یا systemd سے):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** صرف `openclaw` اور root ہی `/home/openclaw/.openclaw` تک رسائی حاصل کر سکتے ہیں۔ config میں ترمیم کے لیے: gateway چلنے کے بعد Control UI استعمال کریں، یا `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`۔

## Environment اور config

- **Token:** `~openclaw/.openclaw/.env` میں `OPENCLAW_GATEWAY_TOKEN` کے طور پر محفوظ ہوتا ہے۔ اگر موجود نہ ہو تو `setup-podman.sh` اور `run-openclaw-podman.sh` اسے خودکار طور پر بنا دیتے ہیں ( `openssl`، `python3`، یا `od` استعمال کرتے ہوئے )۔
- **Optional:** اسی `.env` میں آپ provider keys (مثلاً `GROQ_API_KEY`, `OLLAMA_API_KEY`) اور دیگر OpenClaw env vars سیٹ کر سکتے ہیں۔
- **Host ports:** بطور ڈیفالٹ اسکرپٹ `18789` (gateway) اور `18790` (bridge) میپ کرتا ہے۔ لانچ کرتے وقت **host** پورٹ میپنگ کو `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` اور `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` سے override کریں۔
- **Paths:** Host config اور workspace بطور ڈیفالٹ `~openclaw/.openclaw` اور `~openclaw/.openclaw/workspace` ہوتے ہیں۔ لانچ اسکرپٹ میں استعمال ہونے والے host paths کو `OPENCLAW_CONFIG_DIR` اور `OPENCLAW_WORKSPACE_DIR` سے override کریں۔

## مفید کمانڈز

- **Logs:** quadlet کے ساتھ: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`۔ اسکرپٹ کے ساتھ: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** quadlet کے ساتھ: `sudo systemctl --machine openclaw@ --user stop openclaw.service`۔ اسکرپٹ کے ساتھ: `sudo -u openclaw podman stop openclaw`
- **Start again:** quadlet کے ساتھ: `sudo systemctl --machine openclaw@ --user start openclaw.service`۔ اسکرپٹ کے ساتھ: لانچ اسکرپٹ دوبارہ چلائیں یا `podman start openclaw`
- **Remove container:** `sudo -u openclaw podman rm -f openclaw` — host پر موجود config اور workspace برقرار رہیں گے

## خرابیوں کا حل (Troubleshooting)

- **Permission denied (EACCES) on config or auth-profiles:** container بطور ڈیفالٹ `--userns=keep-id` استعمال کرتا ہے اور اسی host صارف کے uid/gid کے طور پر چلتا ہے جو اسکرپٹ چلا رہا ہو۔ یقینی بنائیں کہ آپ کے host کے `OPENCLAW_CONFIG_DIR` اور `OPENCLAW_WORKSPACE_DIR` اسی صارف کی ملکیت میں ہوں۔
- **Gateway start blocked (missing `gateway.mode=local`):** یقینی بنائیں کہ `~openclaw/.openclaw/openclaw.json` موجود ہو اور اس میں `gateway.mode="local"` سیٹ ہو۔ اگر فائل موجود نہ ہو تو `setup-podman.sh` اسے بنا دیتا ہے۔
- **Rootless Podman fails for user openclaw:** چیک کریں کہ `/etc/subuid` اور `/etc/subgid` میں `openclaw` کے لیے ایک لائن موجود ہو (مثلاً `openclaw:100000:65536`)۔ اگر موجود نہ ہو تو شامل کریں اور سروس دوبارہ شروع کریں۔
- **Container name in use:** لانچ اسکرپٹ `podman run --replace` استعمال کرتا ہے، اس لیے دوبارہ شروع کرنے پر موجودہ container تبدیل ہو جاتا ہے۔ دستی صفائی کے لیے: `podman rm -f openclaw`۔
- **Script not found when running as openclaw:** یقینی بنائیں کہ `setup-podman.sh` چلایا گیا ہو تاکہ `run-openclaw-podman.sh` کو openclaw کے home (مثلاً `/home/openclaw/run-openclaw-podman.sh`) میں کاپی کیا جا سکے۔
- **Quadlet service not found or fails to start:** `.container` فائل میں ترمیم کے بعد `sudo systemctl --machine openclaw@ --user daemon-reload` چلائیں۔ Quadlet کے لیے cgroups v2 درکار ہے: `podman info --format '{{.Host.CgroupsVersion}}'` میں `2` ظاہر ہونا چاہیے۔

## اختیاری: اپنے ذاتی صارف کے طور پر چلائیں

gateway کو اپنے عام صارف کے طور پر چلانے کے لیے (الگ openclaw صارف کے بغیر): image بنائیں، `~/.openclaw/.env` میں `OPENCLAW_GATEWAY_TOKEN` بنائیں، اور container کو `--userns=keep-id` اور اپنے `~/.openclaw` پر mounts کے ساتھ چلائیں۔ لانچ اسکرپٹ openclaw-user فلو کے لیے ڈیزائن کیا گیا ہے؛ سنگل یوزر سیٹ اپ کے لیے آپ اسکرپٹ میں موجود `podman run` کمانڈ کو دستی طور پر چلا سکتے ہیں اور config اور workspace کو اپنے ہوم ڈائریکٹری کی طرف پوائنٹ کر سکتے ہیں۔ زیادہ تر صارفین کے لیے تجویز: `setup-podman.sh` استعمال کریں اور openclaw یوزر کے طور پر چلائیں تاکہ config اور پروسیس الگ تھلگ رہیں۔

