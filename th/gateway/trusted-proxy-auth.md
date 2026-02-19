---
summary: "มอบหมายการยืนยันตัวตนของ gateway ให้กับ reverse proxy ที่เชื่อถือได้ (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - การรัน OpenClaw หลัง identity-aware proxy
  - การตั้งค่า Pomerium, Caddy หรือ nginx พร้อม OAuth ไว้หน้า OpenClaw
  - การแก้ไขข้อผิดพลาด WebSocket `1008 unauthorized` กับการตั้งค่า reverse proxy
---

# Trusted Proxy Auth

> ⚠️ **ฟีเจอร์ที่มีความอ่อนไหวด้านความปลอดภัย** โหมดนี้จะมอบหมายการยืนยันตัวตนทั้งหมดให้กับ reverse proxy ของคุณ การตั้งค่าที่ไม่ถูกต้องอาจทำให้ Gateway ของคุณถูกเข้าถึงโดยไม่ได้รับอนุญาต โปรดอ่านหน้านี้อย่างละเอียดก่อนเปิดใช้งาน

## ควรใช้เมื่อใด

ใช้โหมด auth แบบ `trusted-proxy` เมื่อ:

- คุณรัน OpenClaw หลัง **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- proxy ของคุณจัดการการยืนยันตัวตนทั้งหมดและส่งข้อมูลตัวตนผู้ใช้ผ่าน headers
- คุณอยู่ในสภาพแวดล้อม Kubernetes หรือคอนเทนเนอร์ที่ proxy เป็นเส้นทางเดียวไปยัง Gateway
- คุณพบข้อผิดพลาด WebSocket `1008 unauthorized` เพราะเบราว์เซอร์ไม่สามารถส่ง token ใน payload ของ WS ได้

## ไม่ควรใช้เมื่อใด

- หาก proxy ของคุณไม่ได้ทำการยืนยันตัวตนผู้ใช้ (เป็นเพียงตัวยุติ TLS หรือ load balancer)
- หากมีเส้นทางใด ๆ ไปยัง Gateway ที่ข้าม proxy ได้ (ช่องโหว่ไฟร์วอลล์ การเข้าถึงเครือข่ายภายใน)
- หากคุณไม่แน่ใจว่า proxy ของคุณลบ/เขียนทับ forwarded headers อย่างถูกต้องหรือไม่
- หากคุณต้องการการเข้าถึงแบบผู้ใช้คนเดียวส่วนตัวเท่านั้น (พิจารณาใช้ Tailscale Serve + loopback เพื่อการตั้งค่าที่ง่ายกว่า)

## การทำงาน

1. พร็อกซีแบบย้อนกลับของคุณทำการยืนยันตัวตนผู้ใช้ (OAuth, OIDC, SAML เป็นต้น)
2. พร็อกซีเพิ่ม header ที่มีข้อมูลตัวตนผู้ใช้ที่ยืนยันแล้ว (เช่น `x-forwarded-user: nick@example.com`)
3. OpenClaw ตรวจสอบว่าคำขอมาจาก **IP ของพร็อกซีที่เชื่อถือได้** (กำหนดค่าไว้ใน `gateway.trustedProxies`)
4. OpenClaw ดึงข้อมูลตัวตนผู้ใช้จาก header ที่กำหนดค่าไว้
5. หากทุกอย่างถูกต้อง คำขอจะได้รับการอนุญาต

## การกำหนดค่า

```json5
{
  gateway: {
    // Must bind to network interface (not loopback)
    bind: "lan",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### เอกสารอ้างอิงการกำหนดค่า

| ฟิลด์                                       | จำเป็น | คำอธิบาย                                                                                                           |
| ------------------------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------ |
| `gateway.trustedProxies`                    | ใช่    | รายการอาร์เรย์ของ IP พร็อกซีที่เชื่อถือได้ คำขอจาก IP อื่นจะถูกปฏิเสธ                                              |
| `gateway.auth.mode`                         | ใช่    | ต้องเป็นค่า `"trusted-proxy"`                                                                                      |
| `gateway.auth.trustedProxy.userHeader`      | ใช่    | ชื่อ header ที่มีข้อมูลตัวตนผู้ใช้ที่ยืนยันแล้ว                                                                    |
| `gateway.auth.trustedProxy.requiredHeaders` | ไม่    | header เพิ่มเติมที่ต้องมีอยู่เพื่อให้คำขอได้รับความเชื่อถือ                                                        |
| `gateway.auth.trustedProxy.allowUsers`      | ไม่    | รายการอนุญาต (allowlist) ของตัวตนผู้ใช้ หากเว้นว่างหมายถึงอนุญาตผู้ใช้ที่ยืนยันตัวตนแล้วทั้งหมด |

## ตัวอย่างการตั้งค่าพร็อกซี

### Pomerium

Pomerium ส่งข้อมูลตัวตนผ่าน `x-pomerium-claim-email` (หรือ claim header อื่น ๆ) และส่ง JWT ผ่าน `x-pomerium-jwt-assertion`

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

ตัวอย่างการตั้งค่า Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy พร้อม OAuth

Caddy ที่ใช้ปลั๊กอิน `caddy-security` สามารถยืนยันตัวตนผู้ใช้และส่งต่อ identity header ได้

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

ตัวอย่าง Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy ทำการยืนยันตัวตนผู้ใช้และส่งข้อมูลตัวตนผ่าน `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

ตัวอย่างการตั้งค่า nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik พร้อม Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## รายการตรวจสอบด้านความปลอดภัย

ก่อนเปิดใช้งาน trusted-proxy auth ให้ตรวจสอบว่า:

- [ ] **Proxy เป็นเส้นทางเดียวเท่านั้น**: พอร์ตของ Gateway ถูกป้องกันด้วยไฟร์วอลล์จากทุกอย่าง ยกเว้น proxy ของคุณ
- [ ] **trustedProxies มีเฉพาะที่จำเป็น**: ระบุเฉพาะ IP ของ proxy จริง ๆ เท่านั้น ไม่ใช่ทั้ง subnet
- [ ] **Proxy ลบและเขียนทับ headers**: Proxy ของคุณต้องเขียนทับ (ไม่ใช่เพิ่มต่อท้าย) `x-forwarded-*` headers ที่มาจากไคลเอนต์
- [ ] **TLS termination**: Proxy ของคุณจัดการ TLS และผู้ใช้เชื่อมต่อผ่าน HTTPS
- [ ] **ตั้งค่า allowUsers แล้ว** (แนะนำ): จำกัดเฉพาะผู้ใช้ที่รู้จัก แทนที่จะอนุญาตทุกคนที่ผ่านการยืนยันตัวตน

## การตรวจสอบความปลอดภัย

`openclaw security audit` จะแจ้งเตือน trusted-proxy auth ด้วยระดับความรุนแรง **critical**. นี่เป็นความตั้งใจ — เพื่อเตือนว่าคุณกำลังมอบความรับผิดชอบด้านความปลอดภัยให้กับการตั้งค่า proxy ของคุณ.

การตรวจสอบจะเช็กสิ่งต่อไปนี้:

- ไม่มีการตั้งค่า `trustedProxies`
- ไม่มีการตั้งค่า `userHeader`
- `allowUsers` ว่าง (อนุญาตผู้ใช้ที่ยืนยันตัวตนแล้วทุกคน)

## การแก้ไขปัญหา

### "trusted_proxy_untrusted_source"

คำขอไม่ได้มาจาก IP ที่อยู่ใน `gateway.trustedProxies`. ตรวจสอบ:

- IP ของ proxy ถูกต้องหรือไม่? (IP ของ Docker container อาจมีการเปลี่ยนแปลง)
- มี load balancer อยู่หน้า proxy ของคุณหรือไม่?
- ใช้ `docker inspect` หรือ `kubectl get pods -o wide` เพื่อค้นหา IP จริง

### "trusted_proxy_user_missing"

header ผู้ใช้ว่างเปล่าหรือไม่มีอยู่. ตรวจสอบ:

- Proxy ของคุณถูกตั้งค่าให้ส่งต่อ identity headers หรือไม่?
- ชื่อ header ถูกต้องหรือไม่? (ไม่สนใจตัวพิมพ์เล็กใหญ่ แต่การสะกดต้องถูกต้อง)
- ผู้ใช้ได้ยืนยันตัวตนที่ proxy จริงหรือไม่?

### "trusted_proxy_missing_header_\*"

ไม่มี header ที่จำเป็น. ตรวจสอบ:

- การตั้งค่า proxy ของคุณสำหรับ headers เหล่านั้นโดยเฉพาะ
- มีการลบ headers ที่ใดที่หนึ่งในลำดับการส่งต่อหรือไม่

### trusted_proxy_user_not_allowed

ผู้ใช้ได้รับการยืนยันตัวตนแล้ว แต่ไม่อยู่ใน `allowUsers` ให้เพิ่มผู้ใช้เข้าไป หรือเอา allowlist ออก

### WebSocket ยังล้มเหลว

ตรวจสอบให้แน่ใจว่า proxy ของคุณ:

- รองรับการอัปเกรด WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- ส่งต่อ identity headers ในคำขออัปเกรด WebSocket (ไม่ใช่แค่ HTTP เท่านั้น)
- ไม่มีเส้นทางการยืนยันตัวตนแยกต่างหากสำหรับการเชื่อมต่อ WebSocket

## การย้ายจาก Token Auth

หากคุณกำลังย้ายจาก token auth ไปเป็น trusted-proxy:

1. ตั้งค่า proxy ของคุณให้ยืนยันตัวตนผู้ใช้และส่งต่อ headers
2. ทดสอบการตั้งค่า proxy แยกต่างหาก (curl พร้อม headers)
3. อัปเดตการตั้งค่า OpenClaw ให้ใช้ trusted-proxy auth
4. รีสตาร์ท Gateway
5. ทดสอบการเชื่อมต่อ WebSocket จาก Control UI
6. รัน `openclaw security audit` และตรวจสอบผลลัพธ์

## ที่เกี่ยวข้อง

- [Security](/gateway/security) — คู่มือความปลอดภัยฉบับเต็ม
- [Configuration](/gateway/configuration) — เอกสารอ้างอิงการตั้งค่า
- [Remote Access](/gateway/remote) — รูปแบบการเข้าถึงระยะไกลอื่น ๆ
- [Tailscale](/gateway/tailscale) — ทางเลือกที่ง่ายกว่าสำหรับการเข้าถึงเฉพาะ tailnet

