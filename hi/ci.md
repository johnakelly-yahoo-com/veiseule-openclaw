---
title: CI Pipeline
description: OpenClaw CI pipeline कैसे काम करती है
---

# CI Pipeline

`main` पर हर push और हर pull request पर CI चलता है। यह स्मार्ट स्कोपिंग का उपयोग करता है ताकि जब केवल docs या native code में बदलाव हुआ हो तो महंगे jobs को छोड़ा जा सके।

## Job अवलोकन

| Job               | उद्देश्य                                                                       | यह कब चलता है               |
| ----------------- | ------------------------------------------------------------------------------ | --------------------------- |
| `docs-scope`      | केवल docs में हुए बदलाव का पता लगाना                                           | हमेशा                       |
| `changed-scope`   | कौन से क्षेत्र बदले हैं (node/macos/android) इसका पता लगाना | Non-docs PRs                |
| `check`           | TypeScript types, lint, format                                                 | Non-docs बदलाव              |
| `check-docs`      | Markdown lint + टूटी हुई लिंक की जाँच                                          | Docs में बदलाव              |
| `code-analysis`   | LOC सीमा जाँच (1000 पंक्तियाँ)                              | केवल PRs                    |
| `secrets`         | लीक हुए secrets का पता लगाना                                                   | हमेशा                       |
| `build-artifacts` | dist को एक बार build करना, अन्य jobs के साथ साझा करना                          | Non-docs, node बदलाव        |
| `release-check`   | npm pack की सामग्री को सत्यापित करना                                           | Build के बाद                |
| `checks`          | Node/Bun tests + protocol जाँच                                                 | Non-docs, node बदलाव        |
| `checks-windows`  | Windows-विशिष्ट tests                                                          | Non-docs, node बदलाव        |
| `macos`           | Swift lint/build/test + TS tests                                               | macos बदलाव वाले PRs        |
| `android`         | Gradle बिल्ड + टेस्ट                                                           | गैर-डॉक्स, Android परिवर्तन |

## फेल-फास्ट क्रम

जॉब्स को इस तरह क्रमबद्ध किया जाता है कि सस्ते जाँच पहले विफल हों, उसके बाद महंगे जॉब्स चलें:

1. `docs-scope` + `code-analysis` + `check` (समानांतर, ~1-2 मिनट)
2. `build-artifacts` (उपरोक्त पर निर्भर)
3. `checks`, `checks-windows`, `macos`, `android` (बिल्ड पर निर्भर)

## रनर्स

| रनर                             | जॉब्स                                     |
| ------------------------------- | ----------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | अधिकांश Linux जॉब्स                       |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                          |
| `macos-latest`                  | `macos`, `ios`                            |
| `ubuntu-latest`                 | स्कोप डिटेक्शन (हल्का) |

## स्थानीय समकक्ष

```bash
pnpm check          # types + lint + format
pnpm test           # vitest tests
pnpm check:docs     # docs format + lint + broken links
pnpm release:check  # validate npm pack
```
