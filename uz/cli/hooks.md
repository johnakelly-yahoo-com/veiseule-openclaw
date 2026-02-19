---
summary: "43. `openclaw hooks` uchun CLI ma’lumotnomasi (agent hook’lari)"
read_when:
  - 44. Agent hook’larini boshqarmoqchisiz
  - 45. Hook’larni o‘rnatmoqchi yoki yangilamoqchisiz
title: "46. hooks"
---

# 47. `openclaw hooks`

48. Agent hook’larini boshqarish (`/new`, `/reset` kabi buyruqlar va gateway ishga tushishi uchun hodisalarga asoslangan avtomatlashtirishlar).

49. Bog‘liq:

- 50. Hook’lar: [Hooks](/automation/hooks)
- Plugin hooks: [Plugins](/tools/plugin#plugin-hooks)

## List All Hooks

```bash
openclaw hooks list
```

**Variantlar:**

**Options:**

- `--eligible`: Show only eligible hooks (requirements met)
- `--json`: Output as JSON
- `-v, --verbose`: Show detailed information including missing requirements

**Example output:**

```
Hooks (4/4 tayyor)

Tayyor:
  🚀 boot-md ✓ - Gateway ishga tushganda BOOT.md ni ishga tushirish
  📎 bootstrap-extra-files ✓ - Agent bootstrap jarayonida ishchi muhitga qo‘shimcha bootstrap fayllarni qo‘shish
  📝 command-logger ✓ - Barcha buyruq hodisalarini markazlashtirilgan audit fayliga yozish
  💾 session-memory ✓ - /new buyrug‘i berilganda sessiya kontekstini xotiraga saqlash
```

**Example (verbose):**

```bash
openclaw hooks list --verbose
```

Shows missing requirements for ineligible hooks.

**Example (JSON):**

```bash
openclaw hooks list --json
```

Returns structured JSON for programmatic use.

## Get Hook Information

```bash
openclaw hooks info <name>
```

Show detailed information about a specific hook.

**Arguments:**

- `<name>`: Hook name (e.g., `session-memory`)

**Options:**

- `--json`: Output as JSON

**Example:**

```bash
openclaw hooks info session-memory
```

**Output:**

```
💾 session-memory ✓ Tayyor

/new buyrug‘i berilganda sessiya kontekstini xotiraga saqlaydi

Batafsil:
  Manba: openclaw-bundled
  Yo‘l: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Bosh sahifa: https://docs.openclaw.ai/automation/hooks#session-memory
  Hodisalar: command:new

Talablar:
  Config: ✓ workspace.dir
```

## Check Hooks Eligibility

```bash
openclaw hooks check
```

Show summary of hook eligibility status (how many are ready vs. not ready).

**Options:**

- `--json`: Output as JSON

**Example output:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Enable a Hook

```bash
openclaw hooks enable <name>
```

Enable a specific hook by adding it to your config (`~/.openclaw/config.json`).

**Note:** Hooks managed by plugins show `plugin:<id>` in `openclaw hooks list` and
can’t be enabled/disabled here. Enable/disable the plugin instead.

**Arguments:**

- `<name>`: Hook name (e.g., `session-memory`)

**Example:**

```bash
openclaw hooks enable session-memory
```

**Output:**

```
✓ Enabled hook: 💾 session-memory
```

**What it does:**

- Checks if hook exists and is eligible
- Updates `hooks.internal.entries.<name>.enabled = true` in your config
- Saves config to disk

1. **Yoqilgandan so‘ng:**

- Restart the gateway so hooks reload (menu bar app restart on macOS, or restart your gateway process in dev).

## Disable a Hook

```bash
4. openclaw hooks disable <name>
```

5. Konfiguratsiyangizni yangilash orqali ma’lum bir hookni o‘chiring.

**Arguments:**

- `<name>`: Hook name (e.g., `command-logger`)

**Example:**

```bash
11. ⏸ O‘chirilgan hook: 📝 command-logger
```

**Output:**

```
11. ⏸ O‘chirilgan hook: 📝 command-logger
```

12. **O‘chirgandan so‘ng:**

- Restart the gateway so hooks reload

## 14. Hooklarni o‘rnatish

```bash
15. openclaw hooks install <path-or-spec>
```

16. Hooklar to‘plamini mahalliy papka/arxivdan yoki npm’dan o‘rnating.

Npm spetsifikatsiyalari **faqat registry orqali** (paket nomi + ixtiyoriy versiya/tag). Git/URL/file
spetsifikatsiyalari rad etiladi. Xavfsizlik uchun bog‘liqliklarni o‘rnatish `--ignore-scripts` bilan bajariladi.

**What it does:**

- 18. Hooklar to‘plamini `~/.openclaw/hooks/<id>` ichiga nusxalaydi
- 19. O‘rnatilgan hooklarni `hooks.internal.entries.*` da yoqadi
- 20. O‘rnatishni `hooks.internal.installs` ostida qayd etadi

**Options:**

- 22. `-l, --link`: Nusxalash o‘rniga mahalliy katalogni bog‘laydi (uni `hooks.internal.load.extraDirs` ga qo‘shadi)

23. **Qo‘llab-quvvatlanadigan arxivlar:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

24. **Misollar:**

```bash
25. # Mahalliy katalog
openclaw hooks install ./my-hook-pack

# Mahalliy arxiv
openclaw hooks install ./my-hook-pack.zip

# NPM paketi
openclaw hooks install @openclaw/my-hook-pack

# Nusxalamay mahalliy katalogni bog‘lash
openclaw hooks install -l ./my-hook-pack
```

## 26. Hooklarni yangilash

```bash
27. openclaw hooks update <id>
openclaw hooks update --all
```

28. O‘rnatilgan hooklar to‘plamlarini yangilaydi (faqat npm orqali o‘rnatilganlar).

**Options:**

- 30. `--all`: Kuzatilayotgan barcha hooklar to‘plamlarini yangilaydi
- 31. `--dry-run`: Yozmasdan turib nimalar o‘zgarishini ko‘rsatadi

## 32. Biriktirilgan hooklar

### 33. session-memory

34. `/new` buyrug‘ini berganingizda sessiya kontekstini xotiraga saqlaydi.

35. **Yoqish:**

```bash
openclaw hooks enable session-memory
```

37. **Chiqish:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

38. **Qarang:** [session-memory hujjatlari](/automation/hooks#session-memory)

### bootstrap-extra-files

`agent:bootstrap` jarayonida qo‘shimcha bootstrap fayllarni (masalan, monorepo-local `AGENTS.md` / `TOOLS.md`) qo‘shadi.

41. **Yoqish:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**Qarang:** [bootstrap-extra-files documentation](/automation/hooks#bootstrap-extra-files)

### 39. command-logger

40. Barcha buyruq hodisalarini markazlashtirilgan audit fayliga yozadi.

**Yoqish:**

```bash
42. openclaw hooks enable command-logger
```

43. **Chiqish:** `~/.openclaw/logs/commands.log`

44. **Loglarni ko‘rish:**

```bash
45. # So‘nggi buyruqlar
tail -n 20 ~/.openclaw/logs/commands.log

# Chiroyli chiqarish
cat ~/.openclaw/logs/commands.log | jq .

# Amal bo‘yicha filtrlash
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

46. **Qarang:** [command-logger hujjatlari](/automation/hooks#command-logger)

### 2. boot-md

3. Shlyuz ishga tushganda (kanallar ishga tushgandan keyin) `BOOT.md` ni ishga tushiradi.

4. **Hodisalar**: `gateway:startup`

5. **Yoqish**:

```bash
6. openclaw hooks enable boot-md
```

7. **Qarang:** [boot-md hujjatlari](/automation/hooks#boot-md)
