---
title: "apply_patch ٹول"
---

# apply_patch ٹول

فائل کی تبدیلیوں کو ایک منظم پیچ فارمیٹ کے ذریعے لاگو کریں۔ یہ کثیر فائل کے لیے مثالی ہے۔
or multi-hunk edits where a single `edit` call would be brittle.

یہ ٹول ایک واحد `input` اسٹرنگ قبول کرتا ہے جو ایک یا زیادہ فائل آپریشنز کو لپیٹتی ہے:

```
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## پیرامیٹرز

- `input` (required): مکمل پیچ مواد، بشمول `*** Begin Patch` اور `*** End Patch`۔

## نوٹس

- راستے ورک اسپیس روٹ کے نسبت حل کیے جاتے ہیں۔
- فائلوں کا نام تبدیل کرنے کے لیے `*** Update File:` ہنک کے اندر `*** Move to:` استعمال کریں۔
- ضرورت پڑنے پر `*** End of File` صرف EOF پر اندراج کو نشان زد کرتا ہے۔
- Experimental and disabled by default. Enable with `tools.exec.applyPatch.enabled`.
- OpenAI-only (including OpenAI Codex). Optionally gate by model via
  `tools.exec.applyPatch.allowModels`.
- کنفیگ صرف `tools.exec` کے تحت ہے۔

## Example

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```


