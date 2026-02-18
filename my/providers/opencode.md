---
title: "OpenCode Zen"
---

# OpenCode Zen

OpenCode Zen သည် coding agents များအတွက် OpenCode အဖွဲ့မှ အကြံပြုထားသော **ရွေးချယ်ထားသည့် မော်ဒယ်များ စာရင်း** ဖြစ်သည်။
It is an optional, hosted model access path that uses an API key and the `opencode` provider.
Zen is currently in beta.

## CLI တပ်ဆင်ခြင်း

```bash
openclaw onboard --auth-choice opencode-zen
# or non-interactive
openclaw onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## Config အပိုင်းနမူနာ

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-6" } } },
}
```

## မှတ်ချက်များ

- `OPENCODE_ZEN_API_KEY` ကိုလည်း ပံ့ပိုးထားပါသည်။
- Zen သို့ ဝင်ရောက်အကောင့်ဖွင့်ပြီး billing အချက်အလက်များ ထည့်သွင်းကာ သင်၏ API key ကို ကူးယူပါ။
- OpenCode Zen သည် တောင်းဆိုမှုတစ်ခုချင်းစီအလိုက် အခကြေးငွေကောက်ခံပါသည်။ အသေးစိတ်အချက်အလက်များအတွက် OpenCode dashboard ကို စစ်ဆေးပါ။

