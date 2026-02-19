---
summary: "OpenClaw ကို rootless Podman container ထဲတွင် လုပ်ဆောင်ရန်"
read_when:
  - Docker အစား Podman ကို အသုံးပြုထားသော containerized gateway တစ်ခုကို အသုံးပြုလိုသည်
title: "Podman"
---

# Podman

OpenClaw gateway ကို **rootless** Podman container တစ်ခုအဖြစ် လုပ်ဆောင်ပါ။ Docker နှင့် တူညီသော image ကို အသုံးပြုသည် (repo ရှိ [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile) မှ build လုပ်ပါ)။

## လိုအပ်ချက်များ

- Podman (rootless)
- တစ်ကြိမ်တည်း setup အတွက် Sudo (user ဖန်တီးခြင်း၊ image build လုပ်ခြင်း)

## အမြန်စတင်ရန်

**1. တစ်ကြိမ်တည်း setup** (repo root မှ run လုပ်ပါ; user ဖန်တီးခြင်း၊ image build လုပ်ခြင်း၊ launch script ထည့်သွင်းခြင်း)

```bash
./setup-podman.sh
```

ဤလုပ်ဆောင်မှုသည် minimal `~openclaw/.openclaw/openclaw.json` ဖိုင်ကိုလည်း ဖန်တီးပေးသည် (`gateway.mode="local"` ဟု သတ်မှတ်ထားသည်) ထို့ကြောင့် wizard ကို မလုပ်ဆောင်ဘဲ gateway ကို စတင်နိုင်ပါသည်။

မူလအခြေအနေတွင် container ကို systemd service အဖြစ် **မတပ်ဆင်ထားပါ**၊ သင်ကိုယ်တိုင် စတင်ရပါမည် (အောက်တွင် ကြည့်ပါ)။ auto-start နှင့် restart များပါဝင်သော production ပုံစံ setup အတွက် systemd Quadlet user service အဖြစ် ထည့်သွင်းပါ:

```bash
./setup-podman.sh --quadlet
```

(သို့မဟုတ် `OPENCLAW_PODMAN_QUADLET=1` ဟု သတ်မှတ်ပါ; `--container` ကို အသုံးပြုပါက container နှင့် launch script ကိုသာ ထည့်သွင်းမည်ဖြစ်သည်။)

**2. gateway စတင်ရန်** (လျင်မြန်သော smoke testing အတွက် manual)

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Onboarding wizard** (ဥပမာ channels သို့မဟုတ် providers များ ထည့်ရန်):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

ထို့နောက် `http://127.0.0.1:18789/` ကို ဖွင့်ပြီး `~openclaw/.openclaw/.env` ထဲရှိ token (သို့မဟုတ် setup ပြုလုပ်စဉ် ပုံနှိပ်ပြသသော တန်ဖိုး) ကို အသုံးပြုပါ။

## Systemd (Quadlet, optional)

`./setup-podman.sh --quadlet` (သို့မဟုတ် `OPENCLAW_PODMAN_QUADLET=1`) ကို run လုပ်ထားပါက [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) unit တစ်ခုကို ထည့်သွင်းမည်ဖြစ်ပြီး gateway ကို openclaw user အတွက် systemd user service အဖြစ် လုပ်ဆောင်စေပါသည်။ Setup အဆုံးတွင် service ကို enable လုပ်ပြီး start လုပ်ထားပါသည်။

- **Start:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Stop:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Status:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Logs:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadlet ဖိုင်သည် `~openclaw/.config/containers/systemd/openclaw.container` တွင်ရှိသည်။ port များ သို့မဟုတ် env ကို ပြောင်းလဲလိုပါက ထိုဖိုင်ကို (သို့မဟုတ် ၎င်းမှ source လုပ်ထားသော `.env`) ကို ပြင်ဆင်ပြီးနောက် `sudo systemctl --machine openclaw@ --user daemon-reload` ကို chạy လုပ်ကာ service ကို restart လုပ်ပါ။ boot လုပ်သောအခါ openclaw အတွက် lingering ကို enable လုပ်ထားပါက (setup တွင် loginctl ရရှိနိုင်ပါက အလိုအလျောက် လုပ်ဆောင်ပေးသည်) service သည် အလိုအလျောက် start လုပ်ပါမည်။

quadlet ကို မသုံးဘဲ initial setup ပြုလုပ်ထားပြီးနောက် **နောက်မှ** ထည့်လိုပါက `./setup-podman.sh --quadlet` ကို ပြန်လည် chạy လုပ်ပါ။

## openclaw အသုံးပြုသူ (login မဝင်နိုင်သော user)

`setup-podman.sh` သည် သီးသန့် system user `openclaw` ကို ဖန်တီးပေးပါသည်။

- **Shell:** `nologin` — interactive login မရပါ; attack surface ကို လျှော့ချပေးသည်။

- **Home:** ဥပမာ `/home/openclaw` — `~/.openclaw` (config, workspace) နှင့် launch script `run-openclaw-podman.sh` ကို သိမ်းဆည်းထားသည်။

- **Rootless Podman:** အသုံးပြုသူတွင် **subuid** နှင့် **subgid** range ရှိရပါမည်။ distribution များစွာတွင် user ဖန်တီးချိန်၌ အလိုအလျောက် သတ်မှတ်ပေးပါသည်။ setup မှ warning ပြပါက `/etc/subuid` နှင့် `/etc/subgid` တွင် အောက်ပါလိုင်းများကို ထည့်ပါ။

  ```text
  openclaw:100000:65536
  ```

  ထို့နောက် ထို user အဖြစ် gateway ကို စတင်ပါ (ဥပမာ cron သို့မဟုတ် systemd မှ):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** `/home/openclaw/.openclaw` ကို `openclaw` နှင့် root သာ ဝင်ရောက်နိုင်သည်။ config ကို ပြင်ဆင်ရန်: gateway chạy နေစဉ် Control UI ကို အသုံးပြုပါ၊ သို့မဟုတ် `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json` ကို အသုံးပြုပါ။

## Environment နှင့် config

- **Token:** `~openclaw/.openclaw/.env` တွင် `OPENCLAW_GATEWAY_TOKEN` အဖြစ် သိမ်းဆည်းထားသည်။ `setup-podman.sh` နှင့် `run-openclaw-podman.sh` သည် မရှိသေးပါက ၎င်းကို ဖန်တီးပေးသည် (`openssl`, `python3`, သို့မဟုတ် `od` ကို အသုံးပြုသည်)။
- **Optional:** ထို `.env` တွင် provider key များ (ဥပမာ `GROQ_API_KEY`, `OLLAMA_API_KEY`) နှင့် အခြား OpenClaw env var များကို သတ်မှတ်နိုင်သည်။
- **Host ports:** default အားဖြင့် script သည် `18789` (gateway) နှင့် `18790` (bridge) ကို map လုပ်သည်။ launch လုပ်ချိန်တွင် `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` နှင့် `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` ဖြင့် **host** port mapping ကို override လုပ်နိုင်သည်။
- **Paths:** Host config နှင့် workspace ၏ default တန်ဖိုးများမှာ `~openclaw/.openclaw` နှင့် `~openclaw/.openclaw/workspace` ဖြစ်သည်။ launch script အသုံးပြုသော host path များကို `OPENCLAW_CONFIG_DIR` နှင့် `OPENCLAW_WORKSPACE_DIR` ဖြင့် override လုပ်နိုင်သည်။

## အသုံးဝင်သော command များ

- **Logs:** quadlet ဖြင့်: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`။ script ဖြင့်: `sudo -u openclaw podman logs -f openclaw`
- **Stop:** quadlet ဖြင့်: `sudo systemctl --machine openclaw@ --user stop openclaw.service`။ script ဖြင့်: `sudo -u openclaw podman stop openclaw`
- **Start again:** quadlet ဖြင့်: `sudo systemctl --machine openclaw@ --user start openclaw.service`။ script ဖြင့်: launch script ကို ပြန်လည် chạy လုပ်ပါ သို့မဟုတ် `podman start openclaw` ကို အသုံးပြုပါ။
- **Remove container:** `sudo -u openclaw podman rm -f openclaw` — host ပေါ်ရှိ config နှင့် workspace ကို ဆက်လက် ထိန်းသိမ်းထားမည်။

## ပြဿနာဖြေရှင်းခြင်း

- **Permission denied (EACCES) on config or auth-profiles:** container သည် default အနေဖြင့် `--userns=keep-id` ကို အသုံးပြုပြီး script ကို chạy လုပ်သော host user ၏ uid/gid နှင့် တူညီသောအဖြစ် chạy လုပ်သည်။ သင်၏ host `OPENCLAW_CONFIG_DIR` နှင့် `OPENCLAW_WORKSPACE_DIR` ကို ထို user ၏ ပိုင်ဆိုင်မှုဖြစ်ကြောင်း သေချာစေပါ။
- **Gateway စတင်မှု ပိတ်ဆို့ထားသည် (`gateway.mode=local` မရှိခြင်းကြောင့်):** `~openclaw/.openclaw/openclaw.json` ဖိုင်ရှိနေပြီး `gateway.mode="local"` ဟု သတ်မှတ်ထားကြောင်း သေချာပါစေ။ ဖိုင်မရှိပါက `setup-podman.sh` သည် ဤဖိုင်ကို ဖန်တီးပေးပါသည်။
- **Rootless Podman သည် openclaw အသုံးပြုသူအတွက် မအောင်မြင်ပါ:** `/etc/subuid` နှင့် `/etc/subgid` တွင် `openclaw` အတွက် လိုင်းတစ်ကြောင်း (ဥပမာ `openclaw:100000:65536`) ပါရှိကြောင်း စစ်ဆေးပါ။ မရှိပါက ထည့်သွင်းပြီး ပြန်လည်စတင်ပါ။
- **Container အမည်ကို အသုံးပြုနေပြီးသားဖြစ်သည်:** စတင်ရေး script သည် `podman run --replace` ကို အသုံးပြုသောကြောင့် ထပ်မံစတင်သောအခါ ရှိပြီးသား container ကို အစားထိုးမည်ဖြစ်သည်။ လက်ဖြင့် ဖယ်ရှားရန်: `podman rm -f openclaw`။
- **openclaw အဖြစ် လည်ပတ်ရာတွင် Script မတွေ့ပါ:** `setup-podman.sh` ကို လည်ပတ်ထားပြီး `run-openclaw-podman.sh` ကို openclaw ၏ home (ဥပမာ `/home/openclaw/run-openclaw-podman.sh`) သို့ ကူးယူထားကြောင်း သေချာပါစေ။
- **Quadlet service မတွေ့ပါ သို့မဟုတ် စတင်ရန် မအောင်မြင်ပါ:** `.container` ဖိုင်ကို ပြင်ဆင်ပြီးနောက် `sudo systemctl --machine openclaw@ --user daemon-reload` ကို လည်ပတ်ပါ။ Quadlet သည် cgroups v2 ကို လိုအပ်သည်: `podman info --format '{{.Host.CgroupsVersion}}'` သည် `2` ဟု ပြသသင့်သည်။

## ရွေးချယ်နိုင်သည်: ကိုယ်ပိုင် အသုံးပြုသူအဖြစ် လည်ပတ်ရန်

gateway ကို ပုံမှန် အသုံးပြုသူအဖြစ် (သီးသန့် openclaw အသုံးပြုသူ မလိုဘဲ) လည်ပတ်လိုပါက: image ကို build လုပ်ပါ၊ `~/.openclaw/.env` ကို `OPENCLAW_GATEWAY_TOKEN` ဖြင့် ဖန်တီးပါ၊ ထို့နောက် container ကို `--userns=keep-id` နှင့် သင့် `~/.openclaw` သို့ mount များဖြင့် လည်ပတ်ပါ။ စတင်ရေး script သည် openclaw-user လုပ်ငန်းစဉ်အတွက် ဒီဇိုင်းပြုလုပ်ထားခြင်းဖြစ်သည်။ တစ်ဦးတည်း အသုံးပြုမှုအတွက် setup တွင် script ထဲရှိ `podman run` command ကို လက်ဖြင့် လည်ပတ်ပြီး config နှင့် workspace ကို သင့် home သို့ ညွှန်ပြနိုင်သည်။ အသုံးပြုသူအများစုအတွက် အကြံပြုချက်: `setup-podman.sh` ကို အသုံးပြုပြီး config နှင့် process ကို သီးခြားခွဲထားနိုင်ရန် openclaw အသုံးပြုသူအဖြစ် လည်ပတ်ပါ။
