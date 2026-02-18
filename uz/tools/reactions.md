---
title: "Reaksiyalar"
---

# Reaksiya vositalari

Kanallar o‘rtasida umumiy reaksiya semantikasi:

- Reaksiya qo‘shishda `emoji` majburiy.
- `emoji=""` qo‘llab-quvvatlanganda botning reaksiya(lar)ini olib tashlaydi.
- `remove: true` qo‘llab-quvvatlanganda ko‘rsatilgan emojini olib tashlaydi (`emoji` talab qilinadi).

Kanal bo‘yicha izohlar:

- **Discord/Slack**: bo‘sh `emoji` xabardagi botning barcha reaksiyalarini olib tashlaydi; `remove: true` faqat shu emojini olib tashlaydi.
- **Google Chat**: bo‘sh `emoji` xabardagi ilovaning reaksiyalarini olib tashlaydi; `remove: true` faqat shu emojini olib tashlaydi.
- **Telegram**: bo‘sh `emoji` botning reaksiyalarini olib tashlaydi; `remove: true` ham reaksiyalarni olib tashlaydi, biroq vosita tekshiruvi uchun baribir bo‘sh bo‘lmagan `emoji` talab qilinadi.
- **WhatsApp**: bo‘sh `emoji` bot reaksiyasini olib tashlaydi; `remove: true` bo‘sh emoji sifatida qabul qilinadi (baribir `emoji` talab qilinadi).
- **Signal**: kiruvchi reaksiya bildirishnomalari `channels.signal.reactionNotifications` yoqilganda tizim hodisalarini chiqaradi.

