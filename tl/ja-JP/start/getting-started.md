---
read_when:
  - Unang setup mula sa simula
  - Gustong malaman ang pinakamabilis na paraan patungo sa gumaganang chat
summary: I-install ang OpenClaw at patakbuhin ang iyong unang chat sa loob ng ilang minuto.
title: Panimula
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Panimula

Layunin: Mula sa wala, makamit ang unang gumaganang chat gamit ang minimal na setup.

<Info>
Pinakamabilis na paraan para mag-chat: Buksan ang Control UI (walang kailangang channel configuration). Patakbuhin ang `openclaw dashboard` at makipag-chat sa browser, o
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway host</Tooltip>buksan ang `http://127.0.0.1:18789/`.
Dokumentasyon: [Dashboard](/web/dashboard) at [Control UI](/web/control-ui).
</Info>

## Mga Kinakailangan

- Node 22 o mas bago

<Tip>
Kung hindi sigurado, patakbuhin ang `node --version` upang suriin ang iyong bersyon ng Node.
</Tip>

## Mabilisang Setup (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    Iba pang mga paraan ng pag-install at mga kinakailangan: [Pag-install](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Awtomatikong iko-configure ng wizard ang authentication, mga setting ng Gateway, at mga opsyonal na channel.
    Tingnan ang [Onboarding Wizard](/start/wizard) para sa mga detalye.
    ```

  
</Step>
  <Step title="Gatewayを確認">    Kung nag-install ka ng serbisyo, dapat ito ay tumatakbo na:


    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Kapag nag-load ang Control UI, handa nang gamitin ang Gateway.
</Check>

## Mga Opsyonal na Beripikasyon at Karagdagang Feature

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">    Kapaki-pakinabang para sa mabilisang pagsubok at pag-troubleshoot.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">    Kinakailangan ang naka-configure na channel.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Mas detalyado pa

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">    Kumpletong sanggunian ng CLI wizard at mga advanced na opsyon.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">    Daloy ng unang pagpapatakbo ng macOS app.
  
</Card>
</Columns>

## Katayuan pagkatapos makumpleto

- Gateway na tumatakbo
- Naka-configure na pagpapatotoo
- Access sa Control UI o nakakonektang channel

## Mga susunod na hakbang

- Kaligtasan at pag-apruba ng DM: [Pagpapares](/channels/pairing)
- Magkonekta ng higit pang channel: [Mga Channel](/channels)
- Advanced na workflow at pagbuo mula sa source: [Setup](/start/setup)
