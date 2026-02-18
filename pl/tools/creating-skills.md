---
title: "Tworzenie Skills"
---

# Tworzenie niestandardowych Skills 🛠

OpenClaw został zaprojektowany tak, aby był łatwo rozszerzalny. „Skills” są podstawowym sposobem dodawania nowych możliwości do Twojego asystenta.

## Czym jest Skill?

Skill to katalog zawierający plik `SKILL.md` (który dostarcza instrukcje i definicje narzędzi dla LLM) oraz opcjonalnie skrypty lub zasoby.

## Krok po kroku: Twój pierwszy Skill

### 1. Utwórz katalog

Skills znajdują się w Twoim obszarze roboczym, zwykle `~/.openclaw/workspace/skills/`. Utwórz nowy folder dla swojego skillu:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. Zdefiniuj `SKILL.md`

Utwórz plik `SKILL.md` w tym katalogu. Ten plik używa frontmatter YAML do metadanych oraz Markdown do instrukcji.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Dodaj narzędzia (opcjonalnie)

Możesz zdefiniować niestandardowe narzędzia we frontmatterze lub poinstruować agenta, aby używał istniejących narzędzi systemowych (takich jak `bash` lub `browser`).

### 4. Odśwież OpenClaw

Poproś agenta o „refresh skills” lub uruchom ponownie gateway. OpenClaw wykryje nowy katalog i zindeksuje `SKILL.md`.

## Najlepsze praktyki

- **Zwięzłość**: Instrukcje powinny mówić modelowi, _co_ ma zrobić, a nie jak ma być AI.
- **Bezpieczeństwo przede wszystkim**: Jeśli Twój skill używa `bash`, upewnij się, że prompty nie pozwalają na dowolne wstrzykiwanie poleceń z niezaufanego wejścia użytkownika.
- **Testuj lokalnie**: Użyj `openclaw agent --message "use my new skill"` do testowania.

## Współdzielone Skills

Możesz także przeglądać i współtworzyć skills w [ClawHub](https://clawhub.com).

