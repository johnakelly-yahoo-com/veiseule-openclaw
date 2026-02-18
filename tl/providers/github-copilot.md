---
title: "GitHub Copilot"
---

# GitHub Copilot

## Ano ang GitHub Copilot?

Ang GitHub Copilot ay ang AI coding assistant ng GitHub. Nagbibigay ito ng access sa Copilot
models for your GitHub account and plan. OpenClaw can use Copilot as a model
provider in two different ways.

## Dalawang paraan para gamitin ang Copilot sa OpenClaw

### 1. Built-in na GitHub Copilot provider (`github-copilot`)

Gamitin ang native device-login flow upang makakuha ng GitHub token, pagkatapos ay ipagpalit ito para sa
Copilot API tokens when OpenClaw runs. This is the **default** and simplest path
because it does not require VS Code.

### 2. Copilot Proxy plugin (`copilot-proxy`)

Gamitin ang **Copilot Proxy** VS Code extension bilang lokal na tulay. Nakikipag-ugnayan ang OpenClaw sa
the proxy’s `/v1` endpoint and uses the model list you configure there. Choose
this when you already run Copilot Proxy in VS Code or need to route through it.
You must enable the plugin and keep the VS Code extension running.

Gamitin ang GitHub Copilot bilang model provider (`github-copilot`). Pinapatakbo ng login command ang
the GitHub device flow, saves an auth profile, and updates your config to use that
profile.

## Pag-set up ng CLI

```bash
openclaw models auth login-github-copilot
```

Hihilingin sa iyong bumisita sa isang URL at maglagay ng one-time code. Panatilihing bukas ang terminal
open until it completes.

### Mga opsyonal na flag

```bash
openclaw models auth login-github-copilot --profile-id github-copilot:work
openclaw models auth login-github-copilot --yes
```

## Magtakda ng default na model

```bash
openclaw models set github-copilot/gpt-4o
```

### Snippet ng config

```json5
{
  agents: { defaults: { model: { primary: "github-copilot/gpt-4o" } } },
}
```

## Mga tala

- Nangangailangan ng interactive TTY; patakbuhin ito nang direkta sa isang terminal.
- Ang availability ng Copilot model ay depende sa iyong plano; kung may model na ma-reject, subukan ang ibang ID (halimbawa `github-copilot/gpt-4.1`).
- Ang login ay nag-iimbak ng GitHub token sa auth profile store at ine-exchange ito para sa isang Copilot API token kapag tumatakbo ang OpenClaw.
