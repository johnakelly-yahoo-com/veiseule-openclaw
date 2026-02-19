---
summary: "Podman के रूटलेस कंटेनर में OpenClaw चलाएँ"
read_when:
  - आप Docker के बजाय Podman के साथ एक कंटेनरीकृत gateway चाहते हैं
title: "Podman"
---

# Podman

OpenClaw gateway को **rootless** Podman कंटेनर में चलाएँ। Docker जैसी ही इमेज का उपयोग करता है (repo से [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) बनाएं)।

## आवश्यकताएँ

- Podman (rootless)
- एक-बार की सेटअप के लिए Sudo (user बनाना, इमेज बनाना)

## त्वरित शुरुआत

**1. एक-बार सेटअप** (repo root से; user बनाता है, इमेज बनाता है, लॉन्च स्क्रिप्ट इंस्टॉल करता है):

```bash
./setup-podman.sh
```

यह एक न्यूनतम `~openclaw/.openclaw/openclaw.json` भी बनाता है (जिसमें `gateway.mode="local"` सेट होता है), ताकि gateway बिना wizard चलाए शुरू हो सके।

डिफ़ॉल्ट रूप से कंटेनर को systemd सेवा के रूप में इंस्टॉल नहीं किया जाता, आप इसे मैन्युअल रूप से शुरू करते हैं (नीचे देखें)। प्रोडक्शन-स्टाइल सेटअप के लिए जिसमें ऑटो-स्टार्ट और रीस्टार्ट शामिल हों, इसे systemd Quadlet user सेवा के रूप में इंस्टॉल करें:

```bash
./setup-podman.sh --quadlet
```

(या `OPENCLAW_PODMAN_QUADLET=1` सेट करें; केवल कंटेनर और लॉन्च स्क्रिप्ट इंस्टॉल करने के लिए `--container` का उपयोग करें।)

**2. gateway शुरू करें** (मैन्युअल, त्वरित स्मोक टेस्टिंग के लिए):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding wizard** (जैसे चैनल या प्रोवाइडर जोड़ने के लिए):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

फिर `http://127.0.0.1:18789/` खोलें और `~openclaw/.openclaw/.env` से टोकन उपयोग करें (या सेटअप द्वारा प्रिंट किया गया मान)।

## Systemd (Quadlet, वैकल्पिक)

यदि आपने `./setup-podman.sh --quadlet` (या `OPENCLAW_PODMAN_QUADLET=1`) चलाया है, तो एक [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) यूनिट इंस्टॉल होती है ताकि gateway openclaw user के लिए systemd user सेवा के रूप में चले। सेवा सेटअप के अंत में सक्षम और प्रारंभ कर दी जाती है।

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet फ़ाइल `~openclaw/.config/containers/systemd/openclaw.container` पर स्थित है। पोर्ट या env बदलने के लिए, उस फ़ाइल (या उसके द्वारा सोर्स की गई `.env`) को संपादित करें, फिर `sudo systemctl --machine openclaw@ --user daemon-reload` चलाएँ और सेवा रीस्टार्ट करें। बूट पर, यदि openclaw के लिए lingering सक्षम है तो सेवा स्वचालित रूप से शुरू होती है (जब loginctl उपलब्ध हो तो सेटअप यह करता है)।

प्रारंभिक सेटअप में यदि quadlet का उपयोग नहीं किया गया था और बाद में जोड़ना हो, तो पुनः चलाएँ: `./setup-podman.sh --quadlet`।

## openclaw user (non-login)

`setup-podman.sh` एक समर्पित सिस्टम user `openclaw` बनाता है:

- **Shell:** `nologin` — कोई इंटरैक्टिव लॉगिन नहीं; अटैक सतह कम करता है।

- **Home:** उदाहरण के लिए `/home/openclaw` — इसमें `~/.openclaw` (कॉन्फ़िग, वर्कस्पेस) और लॉन्च स्क्रिप्ट `run-openclaw-podman.sh` शामिल हैं।

- **Rootless Podman:** उपयोगकर्ता के पास **subuid** और **subgid** रेंज होना आवश्यक है। कई distros उपयोगकर्ता बनाए जाने पर इन्हें स्वतः असाइन कर देते हैं। यदि सेटअप चेतावनी दिखाता है, तो `/etc/subuid` और `/etc/subgid` में निम्न पंक्तियाँ जोड़ें:

  ```text
  openclaw:100000:65536
  ```

  फिर उसी उपयोगकर्ता के रूप में gateway शुरू करें (उदा. cron या systemd से):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** केवल `openclaw` और root ही `/home/openclaw/.openclaw` तक पहुँच सकते हैं। config संपादित करने के लिए: gateway चलने के बाद Control UI का उपयोग करें, या `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json` चलाएँ।

## Environment और config

- **Token:** `~openclaw/.openclaw/.env` में `OPENCLAW_GATEWAY_TOKEN` के रूप में संग्रहीत होता है। `setup-podman.sh` और `run-openclaw-podman.sh` यदि अनुपस्थित हो तो इसे जनरेट करते हैं ( `openssl`, `python3`, या `od` का उपयोग करके )।
- **Optional:** उस `.env` में आप provider keys (उदा. `GROQ_API_KEY`, `OLLAMA_API_KEY`) और अन्य OpenClaw env vars सेट कर सकते हैं।
- **Host ports:** डिफ़ॉल्ट रूप से स्क्रिप्ट `18789` (gateway) और `18790` (bridge) मैप करती है। लॉन्च करते समय **host** पोर्ट मैपिंग को `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` और `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` से ओवरराइड करें।
- **Paths:** Host config और workspace डिफ़ॉल्ट रूप से `~openclaw/.openclaw` और `~openclaw/.openclaw/workspace` होते हैं। लॉन्च स्क्रिप्ट द्वारा उपयोग किए जाने वाले host paths को `OPENCLAW_CONFIG_DIR` और `OPENCLAW_WORKSPACE_DIR` से ओवरराइड करें।

## उपयोगी कमांड्स

- **Logs:** quadlet के साथ: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`। स्क्रिप्ट के साथ: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** quadlet के साथ: `sudo systemctl --machine openclaw@ --user stop openclaw.service`। स्क्रिप्ट के साथ: `sudo -u openclaw podman stop openclaw`
- **Start again:** quadlet के साथ: `sudo systemctl --machine openclaw@ --user start openclaw.service`। स्क्रिप्ट के साथ: लॉन्च स्क्रिप्ट को फिर से चलाएँ या `podman start openclaw`
- **Remove container:** `sudo -u openclaw podman rm -f openclaw` — host पर config और workspace सुरक्षित रहते हैं

## समस्या निवारण

- **Permission denied (EACCES) on config or auth-profiles:** कंटेनर डिफ़ॉल्ट रूप से `--userns=keep-id` उपयोग करता है और स्क्रिप्ट चलाने वाले host उपयोगकर्ता के समान uid/gid से चलता है। सुनिश्चित करें कि आपके host `OPENCLAW_CONFIG_DIR` और `OPENCLAW_WORKSPACE_DIR` उसी उपयोगकर्ता के स्वामित्व में हों।
- **Gateway start blocked (missing `gateway.mode=local`):** सुनिश्चित करें कि `~openclaw/.openclaw/openclaw.json` मौजूद है और उसमें `gateway.mode="local"` सेट है। `setup-podman.sh` यदि यह फ़ाइल अनुपस्थित हो तो उसे बनाता है।
- **Rootless Podman fails for user openclaw:** जाँचें कि `/etc/subuid` और `/etc/subgid` में `openclaw` के लिए एक पंक्ति मौजूद है (उदा. `openclaw:100000:65536`)। यदि अनुपस्थित हो तो इसे जोड़ें और पुनः आरंभ करें।
- **Container name in use:** लॉन्च स्क्रिप्ट `podman run --replace` का उपयोग करती है, इसलिए दोबारा शुरू करने पर मौजूदा कंटेनर बदल दिया जाता है। मैन्युअल रूप से साफ़ करने के लिए: `podman rm -f openclaw`।
- **Script not found when running as openclaw:** सुनिश्चित करें कि `setup-podman.sh` चलाया गया है ताकि `run-openclaw-podman.sh` openclaw के home (उदा. `/home/openclaw/run-openclaw-podman.sh`) में कॉपी हो जाए।
- **Quadlet service not found or fails to start:** `.container` फ़ाइल संपादित करने के बाद `sudo systemctl --machine openclaw@ --user daemon-reload` चलाएँ। Quadlet को cgroups v2 की आवश्यकता होती है: `podman info --format '{{.Host.CgroupsVersion}}'` में `2` दिखना चाहिए।

## वैकल्पिक: अपने स्वयं के उपयोगकर्ता के रूप में चलाएँ

gateway को अपने सामान्य उपयोगकर्ता के रूप में चलाने के लिए (अलग openclaw उपयोगकर्ता के बिना): इमेज बनाएं, `~/.openclaw/.env` में `OPENCLAW_GATEWAY_TOKEN` बनाएं, और कंटेनर को `--userns=keep-id` तथा अपने `~/.openclaw` पर mounts के साथ चलाएँ। लॉन्च स्क्रिप्ट openclaw-user फ्लो के लिए डिज़ाइन की गई है; एकल-उपयोगकर्ता सेटअप के लिए आप स्क्रिप्ट से `podman run` कमांड को मैन्युअली चला सकते हैं, और config तथा workspace को अपने home पर पॉइंट कर सकते हैं। अधिकांश उपयोगकर्ताओं के लिए अनुशंसित: `setup-podman.sh` का उपयोग करें और openclaw उपयोगकर्ता के रूप में चलाएँ ताकि config और प्रक्रिया अलग-थलग रहें।

