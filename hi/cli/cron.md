---
summary: "`openclaw cron` के लिए CLI संदर्भ (अनुसूचित करें और पृष्ठभूमि जॉब्स चलाएँ)"
read_when:
  - आपको अनुसूचित जॉब्स और वेकअप्स चाहिए
  - आप cron निष्पादन और लॉग्स का डिबग कर रहे हैं
title: "cron"
---

# `openclaw cron`

Gateway शेड्यूलर के लिए cron जॉब्स प्रबंधित करें।

संबंधित:

- क्रॉन जॉब्स: [Cron jobs](/automation/cron-jobs)

सुझाव: पूर्ण कमांड सतह के लिए `openclaw cron --help` चलाएँ।

नोट: अलग-थलग `cron add` जॉब्स डिफ़ॉल्ट रूप से `--announce` डिलीवरी का उपयोग करते हैं। उन्हें बनाए रखने के लिए `--no-deliver` का उपयोग करें।
output internal. `--deliver` remains as a deprecated alias for `--announce`.

नोट: एक-बार चलने वाले (`--at`) जॉब्स डिफ़ॉल्ट रूप से सफल होने के बाद हट जाते हैं। उन्हें बनाए रखने के लिए `--keep-after-run` का उपयोग करें।

टिप्पणी: आवर्ती जॉब्स अब लगातार त्रुटियों के बाद घातीय पुनःप्रयास बैकऑफ का उपयोग करते हैं (30s → 1m → 5m → 15m → 60m), फिर अगली सफल रन के बाद सामान्य शेड्यूल पर लौट आते हैं।

## सामान्य संपादन

संदेश बदले बिना डिलीवरी सेटिंग्स अपडेट करें:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

पृथक जॉब के लिए डिलीवरी अक्षम करें:

```bash
openclaw cron edit <job-id> --no-deliver
```

किसी विशिष्ट चैनल में घोषणा करें:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```
