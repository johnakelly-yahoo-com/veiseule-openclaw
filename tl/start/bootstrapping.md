---
title: "Bootstrapping ng Agent"
sidebarTitle: "Pag-bootstrapping"
---

# Bootstrapping ng Agent

Ang Bootstrapping ay ang **unang pagtakbo** na ritwal na naghahanda ng workspace ng agent at
collects identity details. It happens after onboarding, when the agent starts
for the first time.

## Ano ang ginagawa ng bootstrapping

Sa unang takbo ng agent, bina-bootstrap ng OpenClaw ang workspace (default
`~/.openclaw/workspace`):

- Naglalagay ng paunang laman sa `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
- Nagpapatakbo ng maikling ritwal ng Q&A (isang tanong sa bawat pagkakataon).
- Nagsusulat ng identity + mga preference sa `IDENTITY.md`, `USER.md`, `SOUL.md`.
- Inaalis ang `BOOTSTRAP.md` kapag tapos na upang isang beses lang itong tumakbo.

## Saan ito tumatakbo

Laging tumatakbo ang Bootstrapping sa **host ng gateway**. Kung kumokonekta ang macOS app sa
a remote Gateway, the workspace and bootstrapping files live on that remote
machine.

<Note>
Kapag tumatakbo ang Gateway sa ibang makina, i-edit ang mga workspace file sa host ng Gateway
(halimbawa, `user@gateway-host:~/.openclaw/workspace`).
</Note>

## Kaugnay na docs

- Onboarding ng macOS app: [Onboarding](/start/onboarding)
- Layout ng workspace: [Agent workspace](/concepts/agent-workspace)

