# ManyPay Documentation

เอกสารคู่มือการใช้งาน ManyPay API

---

## ลิงก์ที่อยู่ API URL

```
กรุณาติดต่อเพื่อรับที่อยู่
```

---

## 📌 ใช้งาน API

### 1. สร้าง QR โค้ดสำหรับชำระเงิน

**ส่งข้อมูลไปที่:** `POST /createpayment`

> **สำคัญ:**
> - ✅ สามารถใช้ `referenceNo` ซ้ำได้หาก QR โค้ด **ยังชำระเงินไม่สำเร็จ**
> - ❌ ไม่สามารใช้ `referenceNo` ซ้ำได้หาก QR โค้ด **ชำระเงินแล้ว**

**ข้อมูลที่ต้องส่ง (Body เป็น Json):**
```json
{
  "token": "xxx",                              // โทเค็นที่ได้มาจากทางหลายบาท
  "amount": "1",                               // จำนวนเงินที่ต้องการให้ลูกค้าชำระ
  "referenceNo": "12345abcdxxxxxx",            // Ref **สำคัญ 6 ตัวหลังสุดต้องระบุตามที่ทางหลายบาทได้แจ้งไว้", ห้ามเกิน 15 ตัวรวมทั้งหมด
  "verifykey": "someencryptedmessage",         // (ไม่จำเป็น) เอาไว้ใส่ตัวเข้ารหัสเพื่อยืนยันข้อมูล Webhook ที่ส่งมา (ทางลูกค้าต้องทำการสร้างระบบยืนยันข้อมูล Webhook เอง), ห้ามเกิน 150 ตัว
  "etc1": "otherinfo1",                        // (ไม่จำเป็น) ข้อมูลอื่น ๆ 1
  "etc2": "otherinfo2",                        // (ไม่จำเป็น) ข้อมูลอื่น ๆ 2
  "etc3": "otherinfo3",                        // (ไม่จำเป็น) ข้อมูลอื่น ๆ 3
  "etc4": "otherinfo4",                        // (ไม่จำเป็น) ข้อมูลอื่น ๆ 4
  "backgroundUrl": "https://example.com/test"  // (ไม่จำเป็น) ลิงก์ URL ไว้ส่ง Webhook, ทางลูกค้าไม่จำเป็นต้องใส่เนื่องจากระบบล็อกลิงก์ไว้แล้ว
}
```

**ตอบกลับ (Body เป็น Json) code 200:**
```json
{
  "ffpReferenceNo": "someffpref",
  "referenceNo": "12345abcdxxxxxx",
  "qrcode": "xxxxxx", // สำคัญ ข้อมูลนำไปแสดงเป็น QR โค้ด
  "resultCode": "00",
  "resultMessage": "Success"
}
```

**Webhook หลังลูกค้าชำระเงินแล้ว (ส่ง POST เป็น Json ไปยังข้อมูลที่ระบุไว้ใน `backgroundUrl`):**
```json
{
  "amount": 1,
  "retryFlag": "N",
  "referenceNo": "12345abcdxxxxxx",
  "ffpReferenceNo": "someffpref",
  "currencyCode": "764",
  "resultCode": "00",
  "resultMessage": null,
  "token": null,
  "totalAmount": 1,
  "fee": 0.01,
  "vat": 0.0007,
  "customerName": "mr. realname surname",
  "merchantDefined1": "someencryptedmessage", // จาก verifykey
  "merchantDefined2": "otherinfo1",
  "merchantDefined3": "otherinfo2",
  "merchantDefined4": "otherinfo3",
  "merchantDefined5": "otherinfo4",
  "date": "01012025",
  "time": "000000",
  "paymentType": "Q",
  "backgroundUrlSend": "https://example.com/test"
}
```

---

### 2. ตรวจสอบชำระเงินด้วย Ref

**ส่งข้อมูลไปที่:** `POST /checkref`

**ข้อมูลที่ต้องส่ง (Body เป็น Json):**
```json
{
  "token": "xxx", // โทเค็นที่ได้มาจากทางหลายบาท
  "ref": "12345abcdxxxxxx"
}
```

**ตอบกลับ (Body เป็น Json) code 200:**
```json
{
  "resultCode": "00",
  "txn": {
    "date": "01012025",
    "amountPerMonth": ".00",
    "paymentType": "Q",
    "currency": "THB",
    "merchantDefined1": "someencryptedmessage",
    "merchantDefined2": "otherinfo1",
    "merchantDefined3": "otherinfo2",
    "merchantDefined4": "otherinfo3",
    "merchantDefined5": "otherinfo4",
    "ffpReferenceNo": "someffpref",
    "amount": "1.00",
    "referenceNo": "12345abcdxxxxxx",
    "resultMessage": "Error",
    "customerName": "mr. realname surname",
    "totalAmount": "1.00",
    "authorizeCode": "000000",
    "time": "000000",
    "currencyCode": "764",
    "status": "S" // หากเป็น "S" แปลว่าชำระเงินสำเร็จ
  },
  "resultMessage": "Success"
}
```