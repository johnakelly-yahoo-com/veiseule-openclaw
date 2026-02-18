---
title: "Node.js"
---

# Node.js

OpenClaw uchun **Node 22 yoki undan yangiroq versiya** talab qilinadi. [o‘rnatish skripti](/install#install-methods) Node’ni avtomatik aniqlaydi va o‘rnatadi — bu sahifa Node’ni o‘zingiz o‘rnatmoqchi bo‘lganingizda va hammasi to‘g‘ri sozlanganiga (versiyalar, PATH, global o‘rnatishlar) ishonch hosil qilish uchun mo‘ljallangan.

## Versiyangizni tekshiring

```bash
node -v
```

Agar natijada `v22.x.x` yoki undan yuqori versiya chiqsa, hammasi joyida. Agar Node o‘rnatilmagan bo‘lsa yoki versiya juda eski bo‘lsa, quyidagi o‘rnatish usullaridan birini tanlang.

## Node’ni o‘rnatish

<Tabs>
  <Tab title="macOS">
    **Homebrew** (tavsiya etiladi):

    ```bash
    brew install node
    ```

    Yoki macOS o‘rnatuvchisini [nodejs.org](https://nodejs.org/) saytidan yuklab oling.

  
</Tab>
  <Tab title="Linux">
    **Ubuntu / Debian:**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL:**

    ```bash
    sudo dnf install nodejs
    ```

    Yoki versiya menejeridan foydalaning (quyida qarang).

  
</Tab>
  <Tab title="Windows">
    **winget** (tavsiya etiladi):

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey:**

    ```powershell
    choco install nodejs-lts
    ```

    Yoki Windows o‘rnatkichini [nodejs.org](https://nodejs.org/) saytidan yuklab oling.

  
</Tab>
</Tabs>

<Accordion title="Versiya menejeridan foydalanish (nvm, fnm, mise, asdf)">
  Versiya menejerlari Node versiyalari orasida oson almashishga imkon beradi. Mashhur variantlar:

- [**fnm**](https://github.com/Schniz/fnm) — tezkor, kross-platforma
- [**nvm**](https://github.com/nvm-sh/nvm) — macOS/Linux’da keng qo‘llaniladi
- [**mise**](https://mise.jdx.dev/) — poliglot (Node, Python, Ruby va boshqalar)

fnm bilan misol:

```bash
fnm install 22
fnm use 22
```

  <Warning>
  Versiya menejeringiz shell ishga tushish faylida (`~/.zshrc` yoki `~/.bashrc`) ishga tushirilganiga ishonch hosil qiling. Aks holda, yangi terminal sessiyalarida `openclaw` topilmasligi mumkin, chunki PATH Node’ning bin katalogini o‘z ichiga olmaydi.
  
</Warning>
</Accordion>

## Nosozliklarni bartaraf etish

### `openclaw: command not found`

Bu deyarli har doim npm’ning global bin katalogi PATH’da yo‘qligini anglatadi.

<Steps>
  <Step title="Global npm prefiksini toping">
    ```bash
    npm prefix -g
    ```
  
</Step>
  <Step title="PATH’da borligini tekshiring">
    ```bash
    echo "$PATH"
    ```

    Natijada `<npm-prefix>/bin` (macOS/Linux) yoki `<npm-prefix>` (Windows) borligini tekshiring.

  
</Step>
  <Step title="Shell ishga tushish fayliga qo‘shing">
    <Tabs>
      <Tab title="macOS / Linux">
        `~/.zshrc` yoki `~/.bashrc` ga qo‘shing:

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        So‘ng yangi terminal oching (yoki zsh’da `rehash` / bash’da `hash -r` ni ishga tushiring).
      
</Tab>
      <Tab title="Windows">
        `npm prefix -g` chiqishini Sozlamalar → Tizim → Atrof-muhit o‘zgaruvchilari orqali tizim PATH’iga qo‘shing.
      
</Tab>
    
</Tabs>

  
</Step>
</Steps>

### `npm install -g` da ruxsat xatolari (Linux)

Agar `EACCES` xatolarini ko‘rsangiz, npm’ning global prefiksini foydalanuvchi yozishi mumkin bo‘lgan katalogga o‘zgartiring:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

`export PATH=...` qatorini doimiy qilish uchun uni `~/.bashrc` yoki `~/.zshrc` faylingizga qo‘shing.

