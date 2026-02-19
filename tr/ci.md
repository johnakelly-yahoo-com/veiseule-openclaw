---
title: CI Pipeline
description: OpenClaw CI pipeline nasıl çalışır
---

# CI Pipeline

CI, `main` dalına yapılan her push işleminde ve her pull request'te çalışır. Yalnızca dokümantasyon veya native kod değiştiğinde maliyetli işleri atlamak için akıllı kapsam belirleme kullanır.

## Job Genel Bakış

| Job               | Amaç                                                                            | Ne zaman çalışır                           |
| ----------------- | ------------------------------------------------------------------------------- | ------------------------------------------ |
| `docs-scope`      | Yalnızca dokümantasyon değişikliklerini tespit eder                             | Her zaman                                  |
| `changed-scope`   | Hangi alanların değiştiğini tespit eder (node/macos/android) | Dokümantasyon dışı PR'lar                  |
| `check`           | TypeScript tipleri, lint, format                                                | Dokümantasyon dışı değişiklikler           |
| `check-docs`      | Markdown lint + bozuk bağlantı kontrolü                                         | Dokümantasyon değiştiğinde                 |
| `code-analysis`   | LOC eşik kontrolü (1000 satır)                               | Yalnızca PR'lar                            |
| `secrets`         | Sızdırılmış gizli bilgileri tespit eder                                         | Her zaman                                  |
| `build-artifacts` | dist’i bir kez derler, diğer job’larla paylaşır                                 | Dokümantasyon dışı, node değişiklikleri    |
| `release-check`   | npm pack içeriğini doğrular                                                     | Build sonrasında                           |
| `checks`          | Node/Bun testleri + protokol kontrolü                                           | Dokümantasyon dışı, node değişiklikleri    |
| `checks-windows`  | Windows'a özel testler                                                          | Dokümantasyon dışı, node değişiklikleri    |
| `macos`           | Swift lint/build/test + TS testleri                                             | macos değişiklikleri içeren PR'lar         |
| `android`         | Gradle derleme + testler                                                        | Dokümantasyon dışı, android değişiklikleri |

## Fail-Fast Sıralaması

İşler, pahalı olanlar çalışmadan önce ucuz kontroller başarısız olacak şekilde sıralanır:

1. `docs-scope` + `code-analysis` + `check` (paralel, ~1-2 dk)
2. `build-artifacts` (yukarıdakilere bağlı)
3. `checks`, `checks-windows`, `macos`, `android` (derlemeye bağlı)

## Çalıştırıcılar

| Çalıştırıcı                     | İşler                                     |
| ------------------------------- | ----------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | Çoğu Linux işi                            |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                          |
| `macos-latest`                  | `macos`, `ios`                            |
| `ubuntu-latest`                 | Kapsam tespiti (hafif) |

## Yerel Eşdeğerler

```bash
pnpm check          # types + lint + format
pnpm test           # vitest testleri
pnpm check:docs     # doküman formatı + lint + bozuk bağlantılar
pnpm release:check  # npm pack doğrulama
```

