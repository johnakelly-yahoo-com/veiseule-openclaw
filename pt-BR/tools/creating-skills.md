---
title: "Criando Skills"
---

# Criando Skills Personalizadas 🛠

O OpenClaw foi projetado para ser facilmente extensível. As "Skills" são a principal forma de adicionar novas capacidades ao seu assistente.

## O que é uma Skill?

Uma skill é um diretório que contém um arquivo `SKILL.md` (que fornece instruções e definições de ferramentas para o LLM) e, opcionalmente, alguns scripts ou recursos.

## Passo a passo: sua primeira Skill

### 1. Crie o diretório

As Skills ficam no seu workspace, geralmente em `~/.openclaw/workspace/skills/`. Crie uma nova pasta para sua skill:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Defina o `SKILL.md`

Crie um arquivo `SKILL.md` nesse diretório. Esse arquivo usa frontmatter YAML para metadados e Markdown para instruções.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Adicione ferramentas (opcional)

Você pode definir ferramentas personalizadas no frontmatter ou instruir o agente a usar ferramentas do sistema existentes (como `bash` ou `browser`).

### 4. Atualize o OpenClaw

Peça ao seu agente para "atualizar as skills" ou reinicie o gateway. O OpenClaw irá descobrir o novo diretório e indexar o `SKILL.md`.

## Boas práticas

- **Seja conciso**: Instrua o modelo sobre _o que_ fazer, não sobre como ser uma IA.
- **Segurança em primeiro lugar**: Se sua skill usa `bash`, garanta que os prompts não permitam injeção arbitrária de comandos a partir de entradas de usuários não confiáveis.
- **Teste localmente**: Use `openclaw agent --message "use my new skill"` para testar.

## Skills compartilhadas

Você também pode explorar e contribuir com skills no [ClawHub](https://clawhub.com).


