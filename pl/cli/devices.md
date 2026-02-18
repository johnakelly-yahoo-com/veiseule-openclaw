---
title: "urządzenia"
---

# `openclaw devices`

Zarządzaj żądaniami parowania urządzeń oraz tokenami o zakresie urządzenia.

## Polecenia

### `openclaw devices list`

Wyświetla listę oczekujących żądań parowania oraz sparowanych urządzeń.

```
openclaw devices list
openclaw devices list --json
```

### `openclaw devices approve <requestId>`

Zatwierdza oczekujące żądanie parowania urządzenia.

```
openclaw devices approve <requestId>
```

### `openclaw devices reject <requestId>`

Odrzuca oczekujące żądanie parowania urządzenia.

```
openclaw devices reject <requestId>
```

### `openclaw devices rotate --device <id> --role <role> [--scope <scope...>]`

Obraca token urządzenia dla określonej roli (opcjonalnie aktualizując zakresy).

```
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `openclaw devices revoke --device <id> --role <role>`

Cofa token urządzenia dla określonej roli.

```
openclaw devices revoke --device <deviceId> --role node
```

## Wspólne opcje

- `--url <url>`: Adres URL WebSocket Gateway (domyślnie `gateway.remote.url` po skonfigurowaniu).
- `--token <token>`: Token Gateway (jeśli wymagany).
- `--password <password>`: Hasło Gateway (uwierzytelnianie hasłem).
- `--timeout <ms>`: Limit czasu RPC.
- `--json`: Wyjście JSON (zalecane do skryptów).

Uwaga: gdy ustawisz `--url`, CLI nie korzysta z konfiguracji ani poświadczeń środowiskowych.
Przekaż jawnie `--token` lub `--password`. Brak jawnych poświadczeń jest błędem.

## Uwagi

- Rotacja tokenu zwraca nowy token (wrażliwy). Traktuj go jak sekret.
- Te polecenia wymagają zakresu `operator.pairing` (lub `operator.admin`).


