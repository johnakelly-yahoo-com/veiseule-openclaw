---
title: "Skills बनाना"
---

# कस्टम Skills बनाना 🛠

OpenClaw को आसानी से extensible होने के लिए डिज़ाइन किया गया है। "Skills" आपके assistant में नई क्षमताएँ जोड़ने का प्राथमिक तरीका हैं।

## Skill क्या है?

एक Skill एक निर्देशिका होती है जिसमें एक `SKILL.md` फ़ाइल होती है (जो LLM को निर्देश और टूल परिभाषाएँ प्रदान करती है) और वैकल्पिक रूप से कुछ स्क्रिप्ट्स या संसाधन शामिल हो सकते हैं।

## चरण-दर-चरण: आपकी पहली Skill

### 1. डायरेक्टरी बनाएँ

Skills आपके workspace में रहती हैं, आमतौर पर `~/.openclaw/workspace/skills/`। अपने skill के लिए एक नया फ़ोल्डर बनाएँ:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. `SKILL.md` परिभाषित करें

उस डायरेक्टरी में एक `SKILL.md` फ़ाइल बनाएँ। यह फ़ाइल metadata के लिए YAML frontmatter और निर्देशों के लिए Markdown का उपयोग करती है।

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. Tools जोड़ें (वैकल्पिक)

आप फ्रंटमैटर में कस्टम टूल्स परिभाषित कर सकते हैं या एजेंट को मौजूदा सिस्टम टूल्स (जैसे `bash` या `browser`) का उपयोग करने का निर्देश दे सकते हैं।

### 4. OpenClaw रिफ्रेश करें

अपने agent से "refresh skills" कहें या gateway को पुनः आरंभ करें। OpenClaw नई डायरेक्टरी खोज लेगा और `SKILL.md` को index करेगा।

## सर्वोत्तम प्रथाएँ

- **संक्षिप्त रहें**: मॉडल को _क्या_ करना है बताएं, न कि AI कैसे बने।
- **सुरक्षा पहले**: यदि आपकी Skill `bash` का उपयोग करती है, तो सुनिश्चित करें कि प्रॉम्प्ट्स अविश्वसनीय उपयोगकर्ता इनपुट से मनमाना कमांड इंजेक्शन की अनुमति न दें।
- **स्थानीय रूप से परीक्षण करें**: परीक्षण के लिए `openclaw agent --message "use my new skill"` का उपयोग करें।

## साझा Skills

आप [ClawHub](https://clawhub.com) पर Skills को ब्राउज़ और योगदान भी कर सकते हैं।


