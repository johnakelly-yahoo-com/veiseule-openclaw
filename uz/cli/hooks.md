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
- Plagin ilgaklari: [Plugins](/tools/plugin#plugin-hooks)

## Barcha ilgaklarni ro‘yxatlash

```bash
openclaw hooks list
```

Workspace, boshqariladigan va birga kelgan kataloglardan aniqlangan barcha ilgaklarni ro‘yxatlaydi.

**Variantlar:**

- `--eligible`: Faqat mos ilgaklarni ko‘rsatish (talablar bajarilgan)
- `--json`: Natijani JSON formatida chiqarish
- `-v, --verbose`: Yetishmayotgan talablar bilan birga batafsil ma’lumotni ko‘rsatish

**Namuna chiqishi:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**Namuna (batafsil):**

```bash
openclaw hooks list --verbose
```

Mos kelmaydigan ilgaklar uchun yetishmayotgan talablarni ko‘rsatadi.

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

**Variantlar:**

- `--json`: Natijani JSON formatida chiqarish

**Example:**

```bash
openclaw hooks info session-memory
```

**Output:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Check Hooks Eligibility

```bash
openclaw hooks check
```

Show summary of hook eligibility status (how many are ready vs. not ready).

**Variantlar:**

- `--json`: Natijani JSON formatida chiqarish

**Namuna chiqishi:**

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

6. **Argumentlar:**

- `<name>`: Hook name (e.g., `command-logger`)

8. **Misol:**

```bash
9. openclaw hooks disable command-logger
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

17. **Nima qiladi:**

- 18. Hooklar to‘plamini `~/.openclaw/hooks/<id>` ichiga nusxalaydi
- 19. O‘rnatilgan hooklarni `hooks.internal.entries.*` da yoqadi
- 20. O‘rnatishni `hooks.internal.installs` ostida qayd etadi

21. **Parametrlar:**

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

29. **Parametrlar:**

- 30. `--all`: Kuzatilayotgan barcha hooklar to‘plamlarini yangilaydi
- 31. `--dry-run`: Yozmasdan turib nimalar o‘zgarishini ko‘rsatadi

## 32. Biriktirilgan hooklar

### 33. session-memory

34. `/new` buyrug‘ini berganingizda sessiya kontekstini xotiraga saqlaydi.

35. **Yoqish:**

```bash
36. openclaw hooks enable session-memory
```

37. **Chiqish:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

38. **Qarang:** [session-memory hujjatlari](/automation/hooks#session-memory)

### 39. command-logger

40. Barcha buyruq hodisalarini markazlashtirilgan audit fayliga yozadi.

41. **Yoqish:**

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

### 47. soul-evil

48. Tozalash oynasi vaqtida yoki tasodifiy ehtimol bilan kiritilgan `SOUL.md` mazmunini `SOUL_EVIL.md` bilan almashtiradi.

49. **Yoqish:**

```bash
50. openclaw hooks enable soul-evil
```

1. **Qarang:** [SOUL Evil Hook](/hooks/soul-evil)

### 2. boot-md

3. Shlyuz ishga tushganda (kanallar ishga tushgandan keyin) `BOOT.md` ni ishga tushiradi.

4. **Hodisalar**: `gateway:startup`

5. **Yoqish**:

```bash
6. openclaw hooks enable boot-md
```

7. **Qarang:** [boot-md hujjatlari](/automation/hooks#boot-md)
