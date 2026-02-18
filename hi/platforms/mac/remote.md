---
title: "दूरस्थ नियंत्रण"
---

# Remote OpenClaw (macOS ⇄ दूरस्थ होस्ट)

यह फ्लो macOS ऐप को किसी अन्य होस्ट (डेस्कटॉप/सर्वर) पर चल रहे OpenClaw gateway के लिए पूर्ण रिमोट कंट्रोल के रूप में कार्य करने देता है। यह ऐप की **Remote over SSH** (remote run) सुविधा है। सभी फीचर्स—health checks, Voice Wake forwarding, और Web Chat—_Settings → General_ से उसी रिमोट SSH कॉन्फ़िगरेशन का पुन: उपयोग करते हैं।

## मोड्स

- **Local (this Mac)**: सब कुछ लैपटॉप पर चलता है। SSH शामिल नहीं है।
- **Remote over SSH (default)**: OpenClaw कमांड्स रिमोट होस्ट पर निष्पादित किए जाते हैं। mac ऐप `-o BatchMode` के साथ आपकी चुनी हुई identity/key और एक लोकल port-forward के माध्यम से SSH कनेक्शन खोलता है।
- **Remote direct (ws/wss)**: कोई SSH टनल नहीं। mac ऐप सीधे gateway URL से कनेक्ट होता है (उदाहरण के लिए, Tailscale Serve या पब्लिक HTTPS reverse proxy के माध्यम से)।

## रिमोट ट्रांसपोर्ट्स

Remote मोड दो ट्रांसपोर्ट का समर्थन करता है:

- **SSH tunnel** (default): gateway पोर्ट को localhost पर फ़ॉरवर्ड करने के लिए `ssh -N -L ...` का उपयोग करता है। टनल loopback होने के कारण gateway को node का IP `127.0.0.1` के रूप में दिखाई देगा।
- **Direct (ws/wss)**: सीधे gateway URL से कनेक्ट करता है। gateway को वास्तविक client IP दिखाई देता है।

## रिमोट होस्ट पर आवश्यक पूर्व-शर्तें

1. Node + pnpm इंस्टॉल करें और OpenClaw CLI (`pnpm install && pnpm build && pnpm link --global`) को बिल्ड/इंस्टॉल करें।
2. सुनिश्चित करें कि `openclaw` non-interactive shells के लिए PATH पर है (आवश्यक होने पर `/usr/local/bin` या `/opt/homebrew/bin` में symlink करें)।
3. Key auth के साथ Open SSH। LAN के बाहर स्थिर कनेक्टिविटी के लिए हम **Tailscale** IPs की सिफारिश करते हैं।

## macOS app setup

1. _Settings → General_ खोलें।
2. **OpenClaw runs** के अंतर्गत **Remote over SSH** चुनें और सेट करें:
   - **Transport**: **SSH tunnel** या **Direct (ws/wss)**।
   - **SSH target**: `user@host` (वैकल्पिक `:port`)।
     - यदि Gateway उसी LAN पर है और Bonjour का विज्ञापन करता है, तो इस फ़ील्ड को स्वतः भरने के लिए खोजी गई सूची से उसे चुनें।
   - **Gateway URL** (केवल Direct): `wss://gateway.example.ts.net` (या local/LAN के लिए `ws://...`)।
   - **Identity file** (advanced): आपकी कुंजी का पथ।
   - **Project root** (advanced): कमांड्स के लिए उपयोग किया जाने वाला दूरस्थ checkout पथ।
   - **CLI path** (advanced): वैकल्पिक रूप से चलाने योग्य `openclaw` entrypoint/binary का पथ (विज्ञापित होने पर स्वतः भरा जाता है)।
3. Hit **Test remote**. Success indicates the remote `openclaw status --json` runs correctly. Failures usually mean PATH/CLI issues; exit 127 means the CLI isn’t found remotely.
4. स्वास्थ्य जाँच और Web Chat अब इस SSH टनल के माध्यम से स्वतः चलेंगे।

## Web Chat

- **SSH tunnel**: Web Chat फ़ॉरवर्ड किए गए WebSocket कंट्रोल पोर्ट (default 18789) के माध्यम से Gateway से कनेक्ट करता है।
- **Direct (ws/wss)**: Web Chat सीधे कॉन्फ़िगर किए गए Gateway URL से कनेक्ट करता है।
- अब कोई अलग WebChat HTTP सर्वर नहीं है।

## Permissions

- The remote host needs the same TCC approvals as local (Automation, Accessibility, Screen Recording, Microphone, Speech Recognition, Notifications). Run onboarding on that machine to grant them once.
- नोड्स अपनी अनुमति स्थिति `node.list` / `node.describe` के माध्यम से विज्ञापित करते हैं ताकि एजेंट जान सकें कि क्या उपलब्ध है।

## Security notes

- दूरस्थ होस्ट पर loopback binds को प्राथमिकता दें और SSH या Tailscale के माध्यम से कनेक्ट करें।
- यदि आप Gateway को non-loopback इंटरफ़ेस से bind करते हैं, तो token/password प्रमाणीकरण आवश्यक करें।
- [Security](/gateway/security) और [Tailscale](/gateway/tailscale) देखें।

## WhatsApp login flow (remote)

- Run `openclaw channels login --verbose` **on the remote host**. Scan the QR with WhatsApp on your phone.
- Re-run login on that host if auth expires. Health check will surface link problems.

## Troubleshooting

- **exit 127 / not found**: `openclaw` isn’t on PATH for non-login shells. Add it to `/etc/paths`, your shell rc, or symlink into `/usr/local/bin`/`/opt/homebrew/bin`.
- **Health probe failed**: SSH पहुँच, PATH, और यह कि Baileys लॉग इन है (`openclaw status --json`)—इनकी जाँच करें।
- **Web Chat stuck**: पुष्टि करें कि Gateway दूरस्थ होस्ट पर चल रहा है और फ़ॉरवर्ड किया गया पोर्ट Gateway WS पोर्ट से मेल खाता है; UI को स्वस्थ WS कनेक्शन चाहिए।
- **Node IP shows 127.0.0.1**: expected with the SSH tunnel. Switch **Transport** to **Direct (ws/wss)** if you want the gateway to see the real client IP.
- **Voice Wake**: remote मोड में ट्रिगर वाक्यांश स्वतः फ़ॉरवर्ड हो जाते हैं; अलग फ़ॉरवर्डर की आवश्यकता नहीं।

## Notification sounds

स्क्रिप्ट्स से प्रति नोटिफ़िकेशन ध्वनियाँ `openclaw` और `node.invoke` के साथ चुनें, उदाहरण के लिए:

```bash
openclaw nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

अब ऐप में कोई वैश्विक “default sound” टॉगल नहीं है; कॉलर प्रत्येक अनुरोध के लिए एक ध्वनि (या कोई नहीं) चुनते हैं।
