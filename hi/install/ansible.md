---
title: "Ansible"
---

# Ansible स्थापना

प्रोडक्शन सर्वरों पर OpenClaw को परिनियोजित करने का अनुशंसित तरीका **[openclaw-ansible](https://github.com/openclaw/openclaw-ansible)** के माध्यम से है — यह एक स्वचालित इंस्टॉलर है, जिसकी आर्किटेक्चर सुरक्षा-प्रथम दृष्टिकोण पर आधारित है।

## त्वरित प्रारंभ

एक-कमांड स्थापना:

```bash
curl -fsSL https://raw.githubusercontent.com/openclaw/openclaw-ansible/main/install.sh | bash
```

> **📦 पूर्ण मार्गदर्शिका: [github.com/openclaw/openclaw-ansible](https://github.com/openclaw/openclaw-ansible)**
>
> यह पेज एक त्वरित ओवरव्यू है। Note: गेटवे **सीधे होस्ट पर** चलता है (Docker में नहीं), लेकिन एजेंट सैंडबॉक्स isolation के लिए Docker का उपयोग करते हैं।

## आपको क्या मिलेगा

- 🔒 **फ़ायरवॉल-प्रथम सुरक्षा**: UFW + Docker आइसोलेशन (केवल SSH + Tailscale सुलभ)
- 🔐 **Tailscale VPN**: सेवाओं को सार्वजनिक रूप से उजागर किए बिना सुरक्षित दूरस्थ एक्सेस
- 🐳 **Docker**: आइसोलेटेड sandbox कंटेनर, केवल localhost बाइंडिंग
- 🛡️ **Defense in depth**: 4-स्तरीय सुरक्षा आर्किटेक्चर
- 🚀 **एक-कमांड सेटअप**: मिनटों में पूर्ण परिनियोजन
- 🔧 **Systemd एकीकरण**: सुदृढ़ीकरण के साथ बूट पर स्वतः प्रारंभ

## आवश्यकताएँ

- **OS**: Debian 11+ या Ubuntu 20.04+
- **एक्सेस**: Root या sudo विशेषाधिकार
- **नेटवर्क**: पैकेज स्थापना के लिए इंटरनेट कनेक्शन
- **Ansible**: 2.14+ (त्वरित-प्रारंभ स्क्रिप्ट द्वारा स्वतः स्थापित)

## क्या-क्या स्थापित होता है

Ansible प्लेबुक निम्नलिखित को स्थापित और विन्यस्त करता है:

1. **Tailscale** (सुरक्षित दूरस्थ एक्सेस के लिए mesh VPN)
2. **UFW फ़ायरवॉल** (केवल SSH + Tailscale पोर्ट)
3. **Docker CE + Compose V2** (एजेंट sandbox के लिए)
4. **Node.js 22.x + pnpm** (रनटाइम निर्भरताएँ)
5. **OpenClaw** (होस्ट-आधारित, कंटेनराइज़्ड नहीं)
6. **Systemd सेवा** (सुरक्षा सुदृढ़ीकरण के साथ स्वतः प्रारंभ)

Note: Gateway **सीधे host पर चलता है** (Docker में नहीं), लेकिन agent sandboxes isolation के लिए Docker का उपयोग करते हैं। **सिर्फ पोर्ट 22** (SSH) खुला होना चाहिए।

## पोस्ट-इंस्टॉल सेटअप

स्थापना पूर्ण होने के बाद, openclaw उपयोगकर्ता में स्विच करें:

```bash
sudo -i -u openclaw
```

पोस्ट-इंस्टॉल स्क्रिप्ट आपको निम्न के माध्यम से मार्गदर्शन करेगी:

1. **ऑनबोर्डिंग विज़ार्ड**: OpenClaw सेटिंग्स का विन्यास
2. **प्रदाता लॉगिन**: WhatsApp/Telegram/Discord/Signal से कनेक्ट करें
3. **Gateway परीक्षण**: स्थापना का सत्यापन
4. **Tailscale सेटअप**: अपने VPN mesh से कनेक्ट करें

### त्वरित कमांड

```bash
# Check service status
sudo systemctl status openclaw

# View live logs
sudo journalctl -u openclaw -f

# Restart gateway
sudo systemctl restart openclaw

# Provider login (run as openclaw user)
sudo -i -u openclaw
openclaw channels login
```

## सुरक्षा आर्किटेक्चर

### 4-स्तरीय रक्षा

1. **फ़ायरवॉल (UFW)**: केवल SSH (22) + Tailscale (41641/udp) सार्वजनिक रूप से खुले
2. **VPN (Tailscale)**: Gateway केवल VPN mesh के माध्यम से सुलभ
3. **Docker आइसोलेशन**: DOCKER-USER iptables चेन बाहरी पोर्ट एक्सपोज़र को रोकती है
4. **Systemd सुदृढ़ीकरण**: NoNewPrivileges, PrivateTmp, अप्रिविलेज्ड उपयोगकर्ता

### सत्यापन

बाहरी आक्रमण सतह का परीक्षण करें:

```bash
nmap -p- YOUR_SERVER_IP
```

बाकी सभी सर्विसेज़ (gateway, Docker) लॉक डाउन रहती हैं। Docker **agent sandboxes** (isolated tool execution) के लिए इंस्टॉल होता है, गेटवे खुद चलाने के लिए नहीं।

### Docker उपलब्धता

गेटवे केवल localhost पर bind होता है और Tailscale VPN के ज़रिए एक्सेस किया जा सकता है। Ansible installer, OpenClaw को मैनुअल अपडेट्स के लिए सेटअप करता है।

sandbox विन्यास के लिए [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) देखें।

## मैनुअल स्थापना

यदि आप ऑटोमेशन पर मैनुअल नियंत्रण पसंद करते हैं:

```bash
# 1. Install prerequisites
sudo apt update && sudo apt install -y ansible git

# 2. Clone repository
git clone https://github.com/openclaw/openclaw-ansible.git
cd openclaw-ansible

# 3. Install Ansible collections
ansible-galaxy collection install -r requirements.yml

# 4. Run playbook
./run-playbook.sh

# Or run directly (then manually execute /tmp/openclaw-setup.sh after)
# ansible-playbook playbook.yml --ask-become-pass
```

## OpenClaw अपडेट करना

स्टैंडर्ड अपडेट फ्लो के लिए देखें [Updating](/install/updating)। ⚠️ **Gateway runtime के लिए अनुशंसित नहीं** (WhatsApp/Telegram bugs)।

Ansible प्लेबुक को पुनः चलाने के लिए (उदाहरण: विन्यास परिवर्तन हेतु):

```bash
cd openclaw-ansible
./run-playbook.sh
```

टिप्पणी: यह idempotent है और इसे कई बार चलाना सुरक्षित है।

## समस्या-निवारण

### फ़ायरवॉल मेरा कनेक्शन ब्लॉक कर रहा है

यदि आप लॉक आउट हो गए हैं:

- पहले सुनिश्चित करें कि आप Tailscale VPN के माध्यम से एक्सेस कर सकते हैं
- SSH एक्सेस (पोर्ट 22) हमेशा अनुमत है
- Gateway डिज़ाइन के अनुसार **केवल** Tailscale के माध्यम से सुलभ है

### सेवा प्रारंभ नहीं हो रही

```bash
# Check logs
sudo journalctl -u openclaw -n 100

# Verify permissions
sudo ls -la /opt/openclaw

# Test manual start
sudo -i -u openclaw
cd ~/openclaw
pnpm start
```

### Docker sandbox से संबंधित समस्याएँ

```bash
# Verify Docker is running
sudo systemctl status docker

# Check sandbox image
sudo docker images | grep openclaw-sandbox

# Build sandbox image if missing
cd /opt/openclaw/openclaw
sudo -u openclaw ./scripts/sandbox-setup.sh
```

### प्रदाता लॉगिन विफल हो रहा है

सुनिश्चित करें कि आप `openclaw` उपयोगकर्ता के रूप में चल रहे हैं:

```bash
sudo -i -u openclaw
openclaw channels login
```

## उन्नत विन्यास

विस्तृत सुरक्षा आर्किटेक्चर और समस्या-निवारण के लिए:

- [Security Architecture](https://github.com/openclaw/openclaw-ansible/blob/main/docs/security.md)
- [Technical Details](https://github.com/openclaw/openclaw-ansible/blob/main/docs/architecture.md)
- [Troubleshooting Guide](https://github.com/openclaw/openclaw-ansible/blob/main/docs/troubleshooting.md)

## संबंधित

- [openclaw-ansible](https://github.com/openclaw/openclaw-ansible) — पूर्ण परिनियोजन मार्गदर्शिका
- [Docker](/install/docker) — कंटेनराइज़्ड Gateway सेटअप
- [Sandboxing](/gateway/sandboxing) — एजेंट sandbox विन्यास
- [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) — प्रति-एजेंट आइसोलेशन
