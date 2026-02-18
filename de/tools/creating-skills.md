---
title: "Skills erstellen"
---

# Eigene Skills erstellen 🛠

OpenClaw ist darauf ausgelegt, leicht erweiterbar zu sein. „Skills“ sind der primäre Weg, Ihrem Assistenten neue Fähigkeiten hinzuzufügen.

## Was ist ein Skill?

Ein Skill ist ein Verzeichnis, das eine `SKILL.md`-Datei enthält (die Anweisungen und Werkzeugdefinitionen für das LLM bereitstellt) und optional einige Skripte oder Ressourcen.

## Schritt für Schritt: Ihr erster Skill

### 1. Verzeichnis erstellen

Skills befinden sich in Ihrem Workspace, üblicherweise `~/.openclaw/workspace/skills/`. Erstellen Sie einen neuen Ordner für Ihren Skill:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Die `SKILL.md` definieren

Erstellen Sie in diesem Verzeichnis eine `SKILL.md`-Datei. Diese Datei verwendet YAML-Frontmatter für Metadaten und Markdown für Anweisungen.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Werkzeuge hinzufügen (optional)

Sie können benutzerdefinierte Werkzeuge im Frontmatter definieren oder den Agenten anweisen, vorhandene Systemwerkzeuge zu verwenden (wie `bash` oder `browser`).

### 4. OpenClaw aktualisieren

Bitten Sie Ihren Agenten, „Skills zu aktualisieren“, oder starten Sie das Gateway neu. OpenClaw erkennt das neue Verzeichnis und indiziert die `SKILL.md`.

## Bewährte Methoden

- **Seien Sie präzise**: Weisen Sie das Modell an, _was_ zu tun ist, nicht, wie es ein KI-Modell sein soll.
- **Sicherheit zuerst**: Wenn Ihr Skill `bash` verwendet, stellen Sie sicher, dass die Prompts keine beliebige Befehlsinjektion aus nicht vertrauenswürdigen Benutzereingaben zulassen.
- **Lokal testen**: Verwenden Sie `openclaw agent --message "use my new skill"` zum Testen.

## Geteilte Skills

Sie können Skills auch auf [ClawHub](https://clawhub.com) durchsuchen und dazu beitragen.

