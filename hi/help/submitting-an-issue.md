---
title: "इश्यू सबमिट करना"
---

## इश्यू सबमिट करना

स्पष्ट और संक्षिप्त मुद्दे निदान और सुधार की प्रक्रिया को तेज करते हैं। बग, रिग्रेशन या फीचर की कमी के लिए निम्नलिखित शामिल करें:

### क्या शामिल करें

- [ ] शीर्षक: क्षेत्र एवं लक्षण
- [ ] न्यूनतम पुनरुत्पादन (repro) चरण
- [ ] अपेक्षित बनाम वास्तविक
- [ ] प्रभाव एवं गंभीरता
- [ ] परिवेश: OS, रनटाइम, संस्करण, विन्यास
- [ ] प्रमाण: संपादित (redacted) लॉग, स्क्रीनशॉट (गैर‑PII)
- [ ] दायरा: नया, रिग्रेशन, या लंबे समय से मौजूद
- [ ] कोड शब्द: अपने इश्यू में lobster-biscuit
- [ ] मौजूदा इश्यू के लिए कोडबेस एवं GitHub खोजा
- [ ] हाल ही में ठीक/सुलझाया नहीं गया है, इसकी पुष्टि (विशेषकर सुरक्षा)
- [ ] दावे प्रमाण या पुनरुत्पादन द्वारा समर्थित

संक्षिप्त रहें। संक्षिप्तता > पूर्ण व्याकरण।

मान्यकरण (PR से पहले चलाएँ/ठीक करें):

- `pnpm lint`
- `pnpm check`
- `pnpm build`
- `pnpm test`
- यदि प्रोटोकॉल कोड: `pnpm protocol:check`

### टेम्पलेट्स

#### बग रिपोर्ट

```md
- [ ] Minimal repro
- [ ] Expected vs actual
- [ ] Environment
- [ ] Affected channels, where not seen
- [ ] Logs/screenshots (redacted)
- [ ] Impact/severity
- [ ] Workarounds

### Summary

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact

### Workarounds
```

#### सुरक्षा इश्यू

```md
### Summary

### Impact

### Versions

### Repro Steps (safe to share)

### Mitigation/workaround

### Evidence (redacted)
```

_सार्वजनिक रूप से गोपनीय जानकारी/एक्सप्लॉइट विवरण साझा करने से बचें। संवेदनशील मुद्दों के लिए विवरण न्यूनतम रखें और निजी प्रकटीकरण का अनुरोध करें।_

#### रिग्रेशन रिपोर्ट

```md
### Summary

### Last Known Good

### First Known Bad

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact
```

#### फीचर अनुरोध

```md
### Summary

### Problem

### Proposed Solution

### Alternatives

### Impact

### Evidence/examples
```

#### एन्हांसमेंट

```md
### Summary

### Current vs Desired Behavior

### Rationale

### Alternatives

### Evidence/examples
```

#### जांच

```md
### Summary

### Symptoms

### What Was Tried

### Environment

### Logs/Evidence

### Impact
```

### फिक्स PR सबमिट करना

PR से पहले Issue बनाना वैकल्पिक है। यदि छोड़ रहे हैं, तो विवरण PR में शामिल करें। PR को केंद्रित रखें, issue नंबर उल्लेख करें, परीक्षण जोड़ें या उनकी अनुपस्थिति का कारण बताएं, व्यवहार में बदलाव/जोखिम को दस्तावेज़ करें, प्रमाण के रूप में संपादित (redacted) लॉग/स्क्रीनशॉट शामिल करें, और सबमिट करने से पहले उचित वैलिडेशन चलाएँ।

