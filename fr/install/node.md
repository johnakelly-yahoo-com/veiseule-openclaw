---
title: "Node.js"---

# Node.js

La base d’exécution d’OpenClaw est **Node 22+**. Le [script d'installation](/install#install-methods) détectera et installera automatiquement Node — cette page est réservée lorsque vous voulez configurer Node vous-même et vous assurer que tout est bien branché (versions, PATH, installations globales).

## Vérifiez votre version

```bash
node -v
```

Si cela affiche `v22.x.x` ou supérieur, vous êtes bon. Si Node n'est pas installé ou que la version est trop ancienne, choisissez une méthode d'installation ci-dessous.

## Install Node

<Tabs>
  <Tab title="macOS">
    **Homebrew** (recommandé) :

    ```bash
    brew install node
    ```

    Ou téléchargez l’installateur macOS depuis [nodejs.org](https://nodejs.org/).

  </Tab>
  <Tab title="Linux">
    **Ubuntu / Debian :**

    ```bash
    curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
    sudo apt-get install -y nodejs
    ```

    **Fedora / RHEL :**

    ```bash
    sudo dnf install nodejs
    ```

    Ou utilisez un gestionnaire de versions (voir ci-dessous).

  </Tab>
  <Tab title="Windows">
    **winget** (recommandé) :

    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```

    **Chocolatey:**

    ```powershell
    choco install nodejs-lts
    ```

    Ou téléchargez l’installateur Windows depuis [nodejs.org](https://nodejs.org/).

  </Tab>
</Tabs>

<Accordion title="Using a version manager (nvm, fnm, mise, asdf)">
  Les gestionnaires de versions vous permettent de basculer facilement entre les versions de Node. Options populaires :

- [**fnm**](https://github.com/Schniz/fnm) — fast, cross-platform
- [**nvm**](https://github.com/nvm-sh/nvm) — largement utilisé sur macOS/Linux
- [**mise**](https://mise.jdx.dev/) — polygotte (Node, Python, Ruby, etc.)

Exemple avec fnm :

```bash
fnm install 22
fnm use 22
```

  <Warning>
  Assurez-vous que votre gestionnaire de versions est initialisé dans le fichier de démarrage de votre shell (`~/.zshrc` ou `~/.bashrc`). Sinon, `openclaw` peut ne pas être trouvé dans les nouvelles sessions de terminal, car le PATH n’inclura pas le répertoire bin de Node.
  </Warning>
</Accordion>

## Problemes courants

### `openclaw: command not found`

Cela signifie presque toujours que le répertoire global npm n’est pas présent dans votre PATH.

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

    Recherchez `<npm-prefix>/bin` (macOS/Linux) ou `<npm-prefix>` (Windows) dans la sortie.

  </Step>
  <Step title="Add it to your shell startup file">
    <Tabs>
      <Tab title="macOS / Linux">
        Ajoutez ceci à `~/.zshrc` ou `~/.bashrc` :

        ```bash
        export PATH="$(npm prefix -g)/bin:$PATH"
        ```

        Ouvrez ensuite un nouveau terminal (ou exécutez `rehash` dans zsh / `hash -r` dans bash).
      </Tab>
      <Tab title="Windows">
        Ajoutez la sortie de `npm prefix -g` à votre PATH système via Paramètres → Système → Variables d’environnement.
      </Tab>
    </Tabs>

  </Step>
</Steps>

### Permission errors on `npm install -g` (Linux)

Si vous voyez des erreurs `EACCES`, changez le préfixe global npm vers un répertoire accessible en écriture par l’utilisateur :

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Ajoutez la ligne `export PATH=...` à votre `~/.bashrc` ou `~/.zshrc` pour la rendre permanente.
