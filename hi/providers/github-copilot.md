---
title: "GitHub Copilot"
---

# GitHub Copilot

## GitHub Copilot क्या है?

GitHub Copilot, GitHub का AI कोडिंग असिस्टेंट है। यह Copilot तक पहुँच प्रदान करता है।
models for your GitHub account and plan. OpenClaw can use Copilot as a model
provider in two different ways.

## OpenClaw में Copilot उपयोग करने के दो तरीके

### 1. अंतर्निर्मित GitHub Copilot प्रदाता (`github-copilot`)

GitHub टोकन प्राप्त करने के लिए native device-login flow का उपयोग करें, फिर इसे आगे के उपयोग के लिए एक्सचेंज करें।
Copilot API tokens when OpenClaw runs. This is the **default** and simplest path
because it does not require VS Code.

### 2. Copilot Proxy प्लगइन (`copilot-proxy`)

Use the **Copilot Proxy** VS Code extension as a local bridge. OpenClaw talks to
the proxy’s `/v1` endpoint and uses the model list you configure there. Choose
this when you already run Copilot Proxy in VS Code or need to route through it.
आपको plugin enable करना होगा और VS Code extension को चलाए रखना होगा।

GitHub Copilot को एक model provider (`github-copilot`) के रूप में उपयोग करें। लॉगिन कमांड चलती है।
the GitHub device flow, saves an auth profile, and updates your config to use that
profile.

## CLI सेटअप

```bash
openclaw models auth login-github-copilot
```

आपको एक URL पर जाने और एक बार उपयोग होने वाला कोड दर्ज करने के लिए कहा जाएगा। टर्मिनल खुला रखें।
open until it completes.

### वैकल्पिक फ़्लैग्स

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## एक डिफ़ॉल्ट मॉडल सेट करें

```bash
openclaw models set github-copilot/gpt-4o
```

### विन्यास स्निपेट

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## टिप्पणियाँ

- इंटरैक्टिव TTY की आवश्यकता होती है; इसे सीधे टर्मिनल में चलाएँ।
- Copilot मॉडल की उपलब्धता आपके प्लान पर निर्भर करती है; यदि किसी मॉडल को अस्वीकार किया जाता है, तो
  किसी अन्य ID का प्रयास करें (उदाहरण के लिए `github-copilot/gpt-4.1`)।
- लॉगिन, ऑथ प्रोफ़ाइल स्टोर में एक GitHub टोकन सहेजता है और जब OpenClaw चलता है तो उसे
  Copilot API टोकन के लिए एक्सचेंज करता है।
