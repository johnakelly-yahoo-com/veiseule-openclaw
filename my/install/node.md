---
title: "Node.js"---

# Node.js

OpenClaw သည် **Node 22 သို့မဟုတ် ထို့ထက် အသစ်သောဗားရှင်း** ကို လိုအပ်ပါသည်။ [installer script](/install#install-methods) သည် Node ကို အလိုအလျောက် စစ်ဆေးပြီး ထည့်သွင်းပေးမည်ဖြစ်သည် — ဤစာမျက်နှာမှာတော့ Node ကို ကိုယ်တိုင် စီစဉ်တပ်ဆင်လိုပြီး (versions, PATH, global installs စသည်) အားလုံး မှန်ကန်စွာ ချိတ်ဆက်ထားကြောင်း သေချာစေရန် အတွက် ဖြစ်ပါသည်။

## သင့်ဗားရှင်းကို စစ်ဆေးပါ

```bash
node -v
```

ဤအမိန့်၏ရလဒ်သည် `v22.x.x` သို့မဟုတ် ထို့ထက် မြင့်ပါက အဆင်ပြေပါသည်။ Node မထည့်သွင်းရသေးပါက သို့မဟုတ် ဗားရှင်းအဟောင်း ဖြစ်နေပါက အောက်ပါ install နည်းလမ်းများထဲမှ တစ်ခုကို ရွေးချယ်ပါ။

## Node ကို ထည့်သွင်းခြင်း

<Tabs>
  <Tab title="macOS">
    **Homebrew** (အကြံပြု):

    ```bash
    brew install node
    ```

    သို့မဟုတ် [nodejs.org](https://nodejs.org/) မှ macOS installer ကို ဒေါင်းလုဒ်လုပ်နိုင်ပါသည်။

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

    သို့မဟုတ် version manager ကို အသုံးပြုနိုင်ပါသည် (အောက်တွင် ကြည့်ပါ)။

  </Tab>
  <Tab title="Windows">
    **winget** (အကြံပြု):

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey:**

    ```powershell
    choco install nodejs-lts
    ```

    သို့မဟုတ် [nodejs.org](https://nodejs.org/) မှ Windows installer ကို ဒေါင်းလုဒ်လုပ်နိုင်ပါသည်။

  </Tab>
</Tabs>

<Accordion title="Using a version manager (nvm, fnm, mise, asdf)">
  Version manager များသည် Node ဗားရှင်းများအကြားကို လွယ်ကူစွာ ပြောင်းလဲအသုံးပြုနိုင်ရန် ကူညီပေးပါသည်။ လူသုံးများသော ရွေးချယ်စရာများမှာ —

- [**fnm**](https://github.com/Schniz/fnm) — မြန်ဆန်ပြီး cross-platform
- [**nvm**](https://github.com/nvm-sh/nvm) — macOS/Linux တွင် အကျယ်ပြန့် အသုံးပြုကြသည်
- [**mise**](https://mise.jdx.dev/) — polyglot (Node, Python, Ruby စသည်)

Example with fnm:

```bash
fnm install 22
fnm use 22
```

  <Warning>
  သင့် shell startup ဖိုင် (`~/.zshrc` သို့မဟုတ် `~/.bashrc`) ထဲတွင် version manager ကို initialize လုပ်ထားကြောင်း သေချာပါစေ။ မလုပ်ထားပါက terminal session အသစ်များတွင် PATH ထဲ၌ Node ၏ bin directory မပါဝင်နိုင်သဖြင့် `openclaw` ကို မတွေ့နိုင်ပါ။
  </Warning>
</Accordion>

## ပြဿနာဖြေရှင်းခြင်း

### `openclaw: command not found`

ဤအခြေအနေသည် အများအားဖြင့် npm ၏ global bin directory သည် PATH ထဲတွင် မပါဝင်ခြင်းကြောင့် ဖြစ်ပါသည်။

<Steps>
  <Step title="Find your global npm prefix">
    ```bash
    npm prefix -g
    ```
  </Step>
  <Step title="Check if it's on your PATH">
    ```bash
    echo "$PATH"
    ```

    ရလဒ်ထဲတွင် `<npm-prefix>/bin` (macOS/Linux) သို့မဟုတ် `<npm-prefix>` (Windows) ကို ရှာပါ။

  </Step>
  <Step title="Add it to your shell startup file">
    <Tabs>
      <Tab title="macOS / Linux">
        `~/.zshrc` သို့မဟုတ် `~/.bashrc` ထဲသို့ ထည့်ပါ —

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        ထို့နောက် terminal အသစ်တစ်ခုကို ဖွင့်ပါ (သို့မဟုတ် zsh တွင် `rehash` / bash တွင် `hash -r` ကို လည်ပတ်ပါ)။
      </Tab>
      <Tab title="Windows">
        `npm prefix -g` ၏ ရလဒ်ကို Settings → System → Environment Variables မှတဆင့် system PATH ထဲသို့ ထည့်ပါ။
      </Tab>
    </Tabs>

  </Step>
</Steps>

### `npm install -g` တွင် Permission errors (Linux)

`EACCES` အမှားများကို တွေ့ပါက npm ၏ global prefix ကို အသုံးပြုသူရေးခွင့်ရှိသော directory သို့ ပြောင်းလဲပါ —

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

အမြဲတမ်း အသက်ဝင်စေရန် `export PATH=...` လိုင်းကို သင့် `~/.bashrc` သို့မဟုတ် `~/.zshrc` ထဲသို့ ထည့်ပါ။