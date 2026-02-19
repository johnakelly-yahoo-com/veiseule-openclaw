---
summary: "Suporta sa Signal sa pamamagitan ng signal-cli (JSON-RPC + SSE), setup, at modelo ng numero"
read_when:
  - Pagse-set up ng suporta sa Signal
  - Pag-debug ng pagpapadala/pagtanggap sa Signal
title: "Signal"
---

# Signal (signal-cli)

29. Status: external na integrasyon ng CLI. 30. Nakikipag-usap ang Gateway sa `signal-cli` sa pamamagitan ng HTTP JSON-RPC + SSE.

## Mabilis na setup (baguhan)

- Gumamit ng **hiwalay na Signal number** para sa bot (inirerekomenda).
- I-install ang `signal-cli` (kailangan ang Java).
- I-link ang bot device at simulan ang daemon:
- I-configure ang OpenClaw at simulan ang gateway.

## Mabilis na setup (baguhan)

1. Gumamit ng **hiwalay na Signal number** para sa bot (inirerekomenda).
2. I-install ang `signal-cli` (kinakailangan ang Java kung gagamitin mo ang JVM build).
3. Pumili ng isang setup path:
   - **Path A (QR link):** `signal-cli link -n "OpenClaw"` at i-scan gamit ang Signal.
   - **Path B (SMS register):** magrehistro ng dedikadong numero gamit ang captcha + SMS verification.
4. I-configure ang OpenClaw at i-restart ang gateway.
5. Magpadala ng unang DM at aprubahan ang pairing (`openclaw pairing approve signal <CODE>`).

Minimal na config:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Sanggunian ng field:

| Field       | Paglalarawan                                                                                  |
| ----------- | --------------------------------------------------------------------------------------------- |
| `account`   | Numero ng telepono ng bot sa E.164 format (`+15551234567`) |
| `cliPath`   | Path papunta sa `signal-cli` (`signal-cli` kung nasa `PATH`)               |
| `dmPolicy`  | Patakaran sa access ng DM (`pairing` ang inirerekomenda)                   |
| `allowFrom` | Mga numero ng telepono o mga halagang `uuid:&lt;id&gt;` na pinapayagang mag-DM                      |

## Ano ito

- Signal channel sa pamamagitan ng `signal-cli` (hindi embedded libsignal).
- Deterministic routing: ang mga reply ay laging bumabalik sa Signal.
- Ang mga DM ay nagbabahagi ng pangunahing session ng agent; ang mga group ay hiwalay (`agent:<agentId>:signal:group:<groupId>`).

## Ang modelo ng numero (mahalaga)

Bilang default, pinapayagan ang Signal na magsulat ng mga update sa config na na-trigger ng `/config set|unset` (kailangan ang `commands.config: true`).

I-disable gamit ang:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Ang modelo ng numero (mahalaga)

- Kumokonekta ang gateway sa isang **Signal device** (ang `signal-cli` account).
- Kung patatakbuhin mo ang bot sa **iyong personal na Signal account**, i-ignore nito ang sarili mong mga mensahe (proteksyon laban sa loop).
- Para sa “nag-text ako sa bot at nagre-reply ito,” gumamit ng **hiwalay na bot number**.

## Setup path A: i-link ang umiiral na Signal account (QR)

1. I-install ang `signal-cli` (JVM o native build).
2. I-link ang isang bot account:
   - `signal-cli link -n "OpenClaw"` pagkatapos ay i-scan ang QR sa Signal.
3. I-configure ang Signal at simulan ang gateway.

Kung gusto mong ikaw ang mag-manage ng `signal-cli` (mabagal na JVM cold starts, container init, o shared CPUs), patakbuhin ang daemon nang hiwalay at ituro ang OpenClaw dito:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

31. Suporta sa maraming account: gamitin ang `channels.signal.accounts` na may per-account config at opsyonal na `name`. 32. Tingnan ang [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para sa shared na pattern.

## Kontrol sa access (DMs + groups)

Gamitin ito kung gusto mo ng hiwalay na numero para sa bot sa halip na i-link ang umiiral na Signal app account.

1. Kumuha ng numerong kayang tumanggap ng SMS (o voice verification para sa landline).
   - Gumamit ng hiwalay na numero para sa bot upang maiwasan ang mga conflict sa account/session.
2. Ang mga hindi kilalang sender ay tumatanggap ng pairing code; ini-ignore ang mga mensahe hanggang maaprubahan (mag-e-expire ang mga code pagkalipas ng 1 oras).

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Kung gagamit ka ng JVM build (`signal-cli-${VERSION}.tar.gz`), i-install muna ang JRE 25+.
Panatilihing updated ang `signal-cli`; ayon sa upstream, maaaring hindi na gumana ang mga lumang release kapag nagbago ang Signal server APIs.

3. Irehistro at i-verify ang numero:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Kung kailangan ang captcha:

1. Ang outbound text ay hina-hati hanggang `channels.signal.textChunkLimit` (default 4000).
2. Opsyonal na newline chunking: itakda ang `channels.signal.chunkMode="newline"` para hatiin sa mga blank line (mga hangganan ng talata) bago ang length chunking.
3. Suportado ang mga attachment (base64 na kinukuha mula sa `signal-cli`).
4. Default na media cap: `channels.signal.mediaMaxMb` (default 8).

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. **Typing indicators**: Nagpapadala ang OpenClaw ng typing signals sa pamamagitan ng `signal-cli sendTyping` at nire-refresh ang mga ito habang tumatakbo ang reply.

```bash
# Kung pinapatakbo mo ang gateway bilang user systemd service:
systemctl --user restart openclaw-gateway

# Pagkatapos ay i-verify:
openclaw doctor
openclaw channels status --probe
```

5. Gamitin ang `message action=react` kasama ang `channel=signal`.
   - Magpadala ng anumang mensahe sa numero ng bot.
   - Aprubahan ang code sa server: `openclaw pairing approve signal <PAIRING_CODE>`.
   - I-save ang numero ng bot bilang contact sa iyong telepono upang maiwasan ang "Unknown contact".

Mahalaga: ang pagrerehistro ng phone number account gamit ang `signal-cli` ay maaaring mag-de-authenticate ng pangunahing Signal app session para sa numerong iyon. Mas mainam ang hiwalay na numero para sa bot, o gamitin ang QR link mode kung kailangan mong panatilihin ang kasalukuyang setup ng phone app.

Mga sanggunian mula sa upstream:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captcha flow: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Linking flow: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## External daemon mode (httpUrl)

Kung gusto mong ikaw ang mag-manage ng `signal-cli` (mabagal na JVM cold starts, container init, o shared CPUs), patakbuhin ang daemon nang hiwalay at ituro ang OpenClaw dito:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

33. Nilalampasan nito ang auto-spawn at ang startup wait sa loob ng OpenClaw. Para sa mabagal na pagsisimula kapag auto-spawning, itakda ang `channels.signal.startupTimeoutMs`.

## Kontrol sa access (DMs + groups)

DMs:

- Default: `channels.signal.dmPolicy = "pairing"`.
- Ang mga hindi kilalang sender ay tumatanggap ng pairing code; ini-ignore ang mga mensahe hanggang maaprubahan (mag-e-expire ang mga code pagkalipas ng 1 oras).
- Aprubahan sa pamamagitan ng:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Ang pairing ang default na palitan ng token para sa mga Signal DM. Mga detalye: [Pairing](/channels/pairing)
- Ang mga UUID-only sender (mula sa `sourceUuid`) ay ini-store bilang `uuid:<id>` sa `channels.signal.allowFrom`.

Groups:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- Kinokontrol ng `channels.signal.groupAllowFrom` kung sino ang puwedeng mag-trigger sa groups kapag naka-set ang `allowlist`.

## Paano ito gumagana (behavior)

- Tumatakbo ang `signal-cli` bilang daemon; binabasa ng gateway ang mga event sa pamamagitan ng SSE.
- Ang mga papasok na mensahe ay ni-no-normalize sa shared channel envelope.
- Ang mga reply ay laging niruruta pabalik sa parehong numero o group.

## Reference ng configuration (Signal)

- Ang outbound text ay hina-hati hanggang `channels.signal.textChunkLimit` (default 4000).
- Opsyonal na newline chunking: itakda ang `channels.signal.chunkMode="newline"` para hatiin sa mga blank line (mga hangganan ng talata) bago ang length chunking.
- Suportado ang mga attachment (base64 na kinukuha mula sa `signal-cli`).
- Default na media cap: `channels.signal.mediaMaxMb` (default 8).
- Gamitin ang `channels.signal.ignoreAttachments` para laktawan ang pag-download ng media.
- 37. Ang konteksto ng group history ay gumagamit ng `channels.signal.historyLimit` (o `channels.signal.accounts.*.historyLimit`), at babalik sa `messages.groupChat.historyLimit`. Itakda sa `0` para i-disable (default 50).

## Typing + read receipts

- `channels.signal.enabled`: i-enable/i-disable ang channel startup.
- `channels.signal.account`: E.164 para sa bot account.
- `channels.signal.cliPath`: path papunta sa `signal-cli`.

## Reactions (message tool)

- `agents.list[].groupChat.mentionPatterns` (hindi sinusuportahan ng Signal ang native mentions).
- Mga target: sender E.164 o UUID (gamitin ang `uuid:<id>` mula sa pairing output; puwede rin ang bare UUID).
- Ang `messageId` ay ang Signal timestamp para sa mensaheng nirereact-an.
- Nangangailangan ang group reactions ng `targetAuthor` o `targetAuthorUuid`.

Mga halimbawa:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Config:

- `channels.signal.actions.reactions`: i-enable/i-disable ang reaction actions (default true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - Ang `off`/`ack` ay nagdi-disable ng agent reactions (mag-e-error ang message tool na `react`).
  - Ang `minimal`/`extensive` ay nag-e-enable ng agent reactions at nagse-set ng guidance level.
- Mga override kada account: `channels.signal.accounts.<id>40. .actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Mga delivery target (CLI/cron)

- DMs: `signal:+15551234567` (o plain E.164).
- UUID DMs: `uuid:<id>` (o bare UUID).
- Groups: `signal:group:<groupId>`.
- Mga username: `username:<name>` (kung suportado ng iyong Signal account).

## Pag-troubleshoot

Patakbuhin muna ang ladder na ito:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Pagkatapos, kumpirmahin ang DM pairing state kung kailangan:

```bash
openclaw pairing list signal
```

Mga karaniwang failure:

- Maaabot ang daemon pero walang reply: i-verify ang mga setting ng account/daemon (`httpUrl`, `account`) at receive mode.
- Ini-ignore ang mga DM: pending ang pairing approval ng sender.
- Ini-ignore ang mga group message: hinaharangan ng group sender/mention gating ang delivery.
- Mga error sa pag-validate ng config matapos ang pag-edit: patakbuhin ang `openclaw doctor --fix`.
- Walang Signal sa diagnostics: tiyaking `channels.signal.enabled: true`.

Mga dagdag na pagsusuri:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Para sa triage flow: [/channels/troubleshooting](/channels/troubleshooting).

## Mga tala sa seguridad

- `signal-cli` ay nag-iimbak ng mga account key nang lokal (karaniwan sa `~/.local/share/signal-cli/data/`).
- Mag-back up ng estado ng Signal account bago ang paglipat o muling pagbuo ng server.
- Panatilihin ang `channels.signal.dmPolicy: "pairing"` maliban kung tahasan mong nais ang mas malawak na DM access.
- Kailangan lamang ang SMS verification para sa mga daloy ng pagpaparehistro o pagbawi, ngunit ang pagkawala ng kontrol sa numero/account ay maaaring magpahirap sa muling pagpaparehistro.

## Reference ng configuration (Signal)

Buong configuration: [Configuration](/gateway/configuration)

Mga opsyon ng provider:

- `channels.signal.enabled`: i-enable/i-disable ang channel startup.
- `channels.signal.account`: E.164 para sa bot account.
- `channels.signal.cliPath`: path papunta sa `signal-cli`.
- `channels.signal.httpUrl`: buong daemon URL (inu-override ang host/port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: daemon bind (default 127.0.0.1:8080).
- `channels.signal.autoStart`: auto-spawn ng daemon (default true kung hindi naka-set ang `httpUrl`).
- `channels.signal.startupTimeoutMs`: startup wait timeout sa ms (cap 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: laktawan ang pag-download ng attachment.
- `channels.signal.ignoreStories`: i-ignore ang stories mula sa daemon.
- `channels.signal.sendReadReceipts`: ipasa ang read receipts.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing).
- 42. `channels.signal.allowFrom`: DM allowlist (E.164 o `uuid:<id>`). `open` ay nangangailangan ng `"*"`. 44. Walang username ang Signal; gumamit ng phone/UUID ids.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (default: allowlist).
- `channels.signal.groupAllowFrom`: group sender allowlist.
- `channels.signal.historyLimit`: max na bilang ng group messages na isasama bilang context (0 ay nagdi-disable).
- 45. `channels.signal.dmHistoryLimit`: DM history limit sa bilang ng user turns. Mga override kada user: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: outbound chunk size (chars).
- `channels.signal.chunkMode`: `length` (default) o `newline` para hatiin sa mga blank line (mga hangganan ng talata) bago ang length chunking.
- `channels.signal.mediaMaxMb`: inbound/outbound media cap (MB).

Mga kaugnay na global option:

- `agents.list[].groupChat.mentionPatterns` (hindi sinusuportahan ng Signal ang native mentions).
- `messages.groupChat.mentionPatterns` (global fallback).
- `messages.responsePrefix`.

