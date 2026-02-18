---
title: "सुरक्षा"
---

# `openclaw security`

सुरक्षा टूल्स (ऑडिट + वैकल्पिक सुधार)।

संबंधित:

- सुरक्षा मार्गदर्शिका: [सुरक्षा](/gateway/security)

## ऑडिट

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

12. ऑडिट चेतावनी देता है जब कई DM भेजने वाले मुख्य सेशन साझा करते हैं और साझा इनबॉक्स के लिए **secure DM mode** की सिफ़ारिश करता है: `session.dmScope="per-channel-peer"` (या मल्टी-अकाउंट चैनलों के लिए `per-account-channel-peer`)।
13. यह तब भी चेतावनी देता है जब छोटे मॉडल (`<=300B`) बिना सैंडबॉक्सिंग और वेब/ब्राउज़र टूल्स सक्षम होने पर उपयोग किए जाते हैं।
