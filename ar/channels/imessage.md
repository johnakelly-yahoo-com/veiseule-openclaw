---
summary: "دعم iMessage القديم عبر imsg ‏(JSON-RPC عبر stdio). يُنصَح بالإعدادات الجديدة باستخدام BlueBubbles."
read_when:
  - إعداد دعم iMessage
  - استكشاف أخطاء إرسال/استقبال iMessage
title: "iMessage"
---

# iMessage (قديم: imsg)

<Warning>
**موصى به:** استخدم [BlueBubbles](/channels/bluebubbles) لإعدادات iMessage الجديدة.

قناة `imsg` هي تكامل خارجي قديم عبر CLI وقد تتم إزالتها في إصدار مستقبلي. 
</Warning>

الحالة: تكامل خارجي قديم عبر CLI. يقوم Gateway بتشغيل `imsg rpc` ‏(JSON-RPC عبر stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    المسار المفضّل لـ iMessage في عمليات الإعداد الجديدة.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    رسائل iMessage الخاصة (DMs) تستخدم وضع الاقتران افتراضيًا.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">توجيه حتمي: تعود الردود دائمًا إلى iMessage.
</Card>
</CardGroup>

## الإعداد (المسار السريع)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="إعداد OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="بدء gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="الموافقة على أول اقتران عبر رسالة خاصة (dmPolicy الافتراضي)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            تنتهي صلاحية طلبات الاقتران بعد ساعة واحدة.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">يمكن أن يشير `channels.imessage.cliPath` إلى أي أمر يوكّل stdin/stdout (على سبيل المثال، سكربت غلاف يتصل عبر SSH بجهاز Mac آخر ويشغّل `imsg rpc`).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    الإعداد الموصى به عند تمكين المرفقات:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    إذا لم يتم تعيين `remoteHost`، يحاول OpenClaw اكتشافه تلقائيًا عبر تحليل أمر SSH في سكربت الغلاف لديك.
    ```

  
</Tab>
</Tabs>

## المتطلبات والأذونات (macOS)

- macOS مع تسجيل الدخول إلى Messages.
- Full Disk Access لـ OpenClaw + `imsg` (الوصول إلى قاعدة بيانات Messages).
- يتطلب إذن Automation لإرسال الرسائل عبر Messages.app.

<Tip>
يمنح macOS أذونات TCC لكل تطبيق/سياق عملية. وافق على المطالبات في السياق نفسه الذي يشغّل `imsg` (مثل Terminal/iTerm، أو جلسة LaunchAgent، أو عملية أُطلقت عبر SSH).

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## التحكم في الوصول والتوجيه

<Tabs>
  <Tab title="DM policy">
    يتحكم `channels.imessage.dmPolicy` في الرسائل الخاصة المباشرة:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (الافتراضي: قائمة السماح).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: قائمة سماح مرسلي المجموعات.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - تستخدم الرسائل الخاصة (DMs) التوجيه المباشر؛ بينما تستخدم المجموعات توجيه المجموعات.
    - مع الإعداد الافتراضي `session.dmScope=main`، يتم دمج رسائل iMessage الخاصة ضمن الجلسة الرئيسية للوكيل.
    عزل الجلسات (مفتاح جلسة `agent:<agentId>:imessage:group:<chat_id>` منفصل)<agentId>المجموعات:<chat_id>`).
    - يتم توجيه الردود مرة أخرى إلى iMessage باستخدام بيانات القناة/الهدف الأصلية.

    ```
    إذا وصل خيط متعدد المشاركين مع `is_group=false`، فلا يزال بإمكانك عزله عبر `chat_id` باستخدام `channels.imessage.groups` (انظر «الخيوط الشبيهة بالمجموعات» أدناه).
    ```

  
</Tab>
</Tabs>

## أنماط النشر

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">إذا أردت أن يرسل البوت من **هوية iMessage منفصلة** (وإبقاء Messages الشخصية نظيفة)، فاستخدم Apple ID مخصّصًا + مستخدم macOS مخصّصًا.

    ```
    افتح Messages ضمن مستخدم macOS هذا وسجّل الدخول إلى iMessage باستخدام Apple ID الخاص بالبوت.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    البنية الشائعة:


    ```
    إذا كان Gateway يعمل على مضيف/آلة افتراضية Linux لكن يجب أن يعمل iMessage على Mac، فإن Tailscale هو الجسر الأبسط: يتواصل Gateway مع الـ Mac عبر tailnet، ويشغّل `imsg` عبر SSH، ويجلب المرفقات عبر SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    استخدم مفاتيح SSH بحيث يكون كل من SSH و SCP بدون تفاعل.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">قناة iMessage مدعومة بواسطة `imsg` على macOS.

    ```
    يمكن لكل حساب تجاوز حقول مثل `cliPath` و `dbPath` و `allowFrom` و `groupPolicy` و `mediaMaxMb` وإعدادات السجل.
    ```

  
</Accordion>
</AccordionGroup>

## الوسائط، التقسيم، وأهداف التسليم

<AccordionGroup>
  <Accordion title="Attachments and media">تُقيَّد تحميلات الوسائط بواسطة `channels.imessage.mediaMaxMb` (الافتراضي 16).
</Accordion>

  <Accordion title="Outbound chunking">يُجزَّأ النص الصادر إلى `channels.imessage.textChunkLimit` (الافتراضي 4000).
</Accordion>

  <Accordion title="Addressing formats">
    الأهداف الصريحة المفضلة:


    ```
    - `chat_id:123` (موصى به لتوجيه مستقر)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    الأهداف عبر المعرّفات مدعومة أيضًا:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## كتابات التهيئة

افتراضيًا، يُسمح لـ iMessage بكتابة تحديثات التهيئة المُحفَّزة بواسطة `/config set|unset` (يتطلب `commands.config: true`).

للتعطيل:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## استكشاف الأخطاء وإصلاحها

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    تحقّق من الملف التنفيذي ودعم RPC:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    إذا أشار الفحص إلى أن RPC غير مدعوم، فقم بتحديث `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">ملاحظات:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: الاقتران).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">قائمة التحقق:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">الموافقة عبر:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    أعد التشغيل في نافذة طرفية GUI تفاعلية ضمن نفس سياق المستخدم/الجلسة ووافق على المطالبات:


```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    ابدأ تشغيل Gateway ووافق على أي مطالبات من macOS ‏(Automation + Full Disk Access).
    ```

  
</Accordion>
</AccordionGroup>

## مراجع التهيئة

- مرجع التهيئة (iMessage)
- التهيئة الكاملة: [التهيئة](/gateway/configuration)
- التفاصيل: [الاقتران](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
