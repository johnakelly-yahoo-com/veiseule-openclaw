---
title: IRC
description: Połącz OpenClaw z kanałami IRC i wiadomościami prywatnymi.
---

Używaj IRC, gdy chcesz mieć OpenClaw w klasycznych kanałach (`#room`) oraz w wiadomościach prywatnych.
IRC jest dostarczany jako wtyczka rozszerzenia, ale konfiguruje się go w głównym pliku konfiguracyjnym w sekcji `channels.irc`.

## Szybki start

1. Włącz konfigurację IRC w `~/.openclaw/openclaw.json`.
2. Ustaw co najmniej:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Uruchom/zrestartuj gateway:

```bash
openclaw gateway run
```

## Domyślne ustawienia bezpieczeństwa

- `channels.irc.dmPolicy` domyślnie ma wartość `"pairing"`.
- `channels.irc.groupPolicy` domyślnie ma wartość `"allowlist"`.
- Przy `groupPolicy="allowlist"` ustaw `channels.irc.groups`, aby zdefiniować dozwolone kanały.
- Używaj TLS (`channels.irc.tls=true`), chyba że celowo akceptujesz nieszyfrowany transfer.

## Kontrola dostępu

Istnieją dwie oddzielne „bramki” dla kanałów IRC:

1. **Dostęp do kanału** (`groupPolicy` + `groups`): czy bot w ogóle akceptuje wiadomości z danego kanału.
2. **Dostęp nadawcy** (`groupAllowFrom` / per-channel `groups["#channel"].allowFrom`): kto może wywołać bota w obrębie tego kanału.

Klucze konfiguracyjne:

- Allowlista DM (dostęp nadawców w DM): `channels.irc.allowFrom`
- Allowlista nadawców w kanałach (dostęp nadawców w kanałach): `channels.irc.groupAllowFrom`
- Kontrola per kanał (kanał + nadawca + reguły wzmianek): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` pozwala na nieskonfigurowane kanały (**nadal domyślnie wymagane jest wspomnienie**).

Wpisy na liście dozwolonych mogą używać formatu nick lub `nick!user@host`.

### Częsty błąd: `allowFrom` dotyczy DM, a nie kanałów

Jeśli widzisz logi takie jak:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…oznacza to, że nadawca nie był dozwolony dla wiadomości **grupowych/kanałowych**. Napraw to, wykonując jedno z poniższych:

- ustaw `channels.irc.groupAllowFrom` (globalnie dla wszystkich kanałów), lub
- ustaw per-kanałowe allowlisty nadawców: `channels.irc.groups["#channel"].allowFrom`

Przykład (pozwól każdemu w `#tuirc-dev` rozmawiać z botem):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Wyzwalanie odpowiedzi (wzmianki)

Nawet jeśli kanał jest dozwolony (przez `groupPolicy` + `groups`) i nadawca jest dozwolony, OpenClaw domyślnie wymaga **wzmianki** w kontekstach grupowych.

Oznacza to, że możesz zobaczyć logi takie jak `drop channel … (missing-mention)` chyba że wiadomość zawiera wzorzec wzmianki pasujący do bota.

Aby bot odpowiadał w kanale IRC **bez konieczności używania wzmianki**, wyłącz wymóg wzmianki dla tego kanału:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Lub aby zezwolić na **wszystkie** kanały IRC (bez listy dozwolonych per kanał) i nadal odpowiadać bez wzmianek:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Uwaga dotycząca bezpieczeństwa (zalecane dla kanałów publicznych)

Jeśli ustawisz `allowFrom: ["*"]` na publicznym kanale, każdy może wysyłać polecenia do bota.
Aby zmniejszyć ryzyko, ogranicz narzędzia dla tego kanału.

### Te same narzędzia dla wszystkich na kanale

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Różne narzędzia w zależności od nadawcy (właściciel ma większe uprawnienia)

Użyj `toolsBySender`, aby zastosować bardziej restrykcyjną politykę dla `"*"` oraz mniej restrykcyjną dla swojego nicka:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Uwagi:

- Klucze `toolsBySender` mogą być nickiem (np. `"eigen"`) lub pełną maską hosta (`"eigen!~eigen@174.127.248.171"`) dla silniejszego dopasowania tożsamości.
- Obowiązuje pierwsza pasująca polityka nadawcy; `"*"` jest domyślnym wzorcem zastępczym.

Więcej informacji o dostępie grupowym vs. wymogu wzmianki (oraz o tym, jak na siebie wpływają) znajdziesz tutaj: [/channels/groups](/channels/groups).

## NickServ

Aby uwierzytelnić się w NickServ po połączeniu:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Opcjonalna jednorazowa rejestracja przy połączeniu:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Wyłącz `register` po zarejestrowaniu nicka, aby uniknąć ponawianych prób REGISTER.

## Zmienne środowiskowe

Domyślne konto obsługuje:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (rozdzielone przecinkami)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Rozwiązywanie problemów

- Jeśli bot łączy się, ale nigdy nie odpowiada na kanałach, sprawdź `channels.irc.groups` **oraz** czy wymóg wzmianki nie odrzuca wiadomości (`missing-mention`). Jeśli chcesz, aby odpowiadał bez pingów, ustaw `requireMention:false` dla danego kanału.
- Jeśli logowanie się nie powiedzie, sprawdź dostępność nicka oraz hasło serwera.
- Jeśli TLS nie działa w niestandardowej sieci, sprawdź host/port oraz konfigurację certyfikatu.
