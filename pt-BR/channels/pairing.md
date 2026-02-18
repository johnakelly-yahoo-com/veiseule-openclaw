---
title: "Pareamento"
---

# Pareamento

“Pareamento” é a etapa explícita de **aprovação do proprietário** do OpenClaw.
Ela é usada em dois lugares:

1. **Pareamento de DM** (quem tem permissão para falar com o bot)
2. **Pareamento de nós** (quais dispositivos/nós podem entrar na rede do gateway)

Contexto de segurança: [Security](/gateway/security)

## 1. Pareamento de DM (acesso a chat de entrada)

Quando um canal é configurado com a política de DM `pairing`, remetentes desconhecidos recebem um código curto e a mensagem **não é processada** até que você aprove.

As políticas padrão de DM estão documentadas em: [Security](/gateway/security)

Códigos de pareamento:

- 8 caracteres, maiúsculos, sem caracteres ambíguos (`0O1I`).
- **Expiram após 1 hora**. O bot só envia a mensagem de pareamento quando uma nova solicitação é criada (aproximadamente uma vez por hora por remetente).
- Solicitações de pareamento de DM pendentes são limitadas a **3 por canal** por padrão; solicitações adicionais são ignoradas até que uma expire ou seja aprovada.

### Aprovar um remetente

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Canais suportados: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Onde o estado fica armazenado

Armazenado em `~/.openclaw/credentials/`:

- Solicitações pendentes: `<channel>-pairing.json`
- Armazenamento da lista de permissões aprovada: `<channel>-allowFrom.json`

Trate-os como sensíveis (eles controlam o acesso ao seu assistente).

## 2. Pareamento de dispositivos de nó (iOS/Android/macOS/nós headless)

Os nós se conectam ao Gateway como **dispositivos** com `role: node`. O Gateway
cria uma solicitação de pareamento de dispositivo que deve ser aprovada.

### Pareamento via Telegram (recomendado para iOS)

Se você usa o plugin `device-pair`, pode fazer o pareamento inicial do dispositivo totalmente pelo Telegram:

1. No Telegram, envie uma mensagem para o seu bot: `/pair`
2. O bot responde com duas mensagens: uma mensagem de instruções e uma mensagem separada com o **código de configuração** (fácil de copiar/colar no Telegram).
3. No seu celular, abra o app OpenClaw para iOS → Settings → Gateway.
4. Cole o código de configuração e conecte-se.
5. De volta ao Telegram: `/pair approve`

O código de configuração é um payload JSON codificado em base64 que contém:

- `url`: a URL WebSocket do Gateway (`ws://...` ou `wss://...`)
- `token`: um token de pareamento de curta duração

Treat the setup code like a password while it is valid.

### Aprovar um dispositivo de nó

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Armazenamento do estado de pareamento de nós

Armazenado em `~/.openclaw/devices/`:

- `pending.json` (curta duração; solicitações pendentes expiram)
- `paired.json` (dispositivos pareados + tokens)

### Notas

- A API legada `node.pair.*` (CLI: `openclaw nodes pending/approve`) é um
  armazenamento de pareamento separado, de propriedade do gateway. Nós WS ainda exigem pareamento de dispositivo.

## Documentos relacionados

- Modelo de segurança + prompt injection: [Security](/gateway/security)
- Atualizando com segurança (run doctor): [Updating](/install/updating)
- Configurações de canais:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (legado): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)
