---
title: "CLI خودکاری"
sidebarTitle: "CLI آٹومیشن"
---

# CLI خودکاری

`--non-interactive` کا استعمال کر کے `openclaw onboard` کو خودکار بنائیں۔

<Note>
`--json` غیر تعاملی موڈ کی نشاندہی نہیں کرتا۔ اسکرپٹس کے لیے `--non-interactive` (اور `--workspace`) استعمال کریں۔
</Note>

## بنیادی غیر تعاملی مثال

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

مشین کے قابلِ مطالعہ خلاصے کے لیے `--json` شامل کریں۔

## فراہم کنندہ کے مطابق مثالیں

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

## ایک اور ایجنٹ شامل کریں

Use `openclaw agents add <name>` to create a separate agent with its own workspace,
sessions, and auth profiles. Running without `--workspace` launches the wizard.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

یہ کن چیزوں کو سیٹ کرتا ہے:

- `agents.list[].name`
- `agents.list[].workspace`
- `agents.list[].agentDir`

نوٹس:

- ڈیفالٹ ورک اسپیسز `~/.openclaw/workspace-<agentId>` کی پیروی کرتے ہیں۔
- آنے والے پیغامات کو روٹ کرنے کے لیے `bindings` شامل کریں (یہ کام ویزارڈ بھی کر سکتا ہے)۔
- غیر تعاملی فلیگز: `--model`, `--agent-dir`, `--bind`, `--non-interactive`۔

## متعلقہ دستاویزات

- آن بورڈنگ ہب: [Onboarding Wizard (CLI)](/start/wizard)
- مکمل حوالہ: [CLI Onboarding Reference](/start/wizard-cli-reference)
- کمانڈ حوالہ: [`openclaw onboard`](/cli/onboard)
