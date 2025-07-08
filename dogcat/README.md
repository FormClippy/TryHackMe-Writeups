## ✨ บทนำ
ห้อง Dogcat เป็นห้อง CTF ที่เน้นการโจมตีผ่านช่องโหว่ Local File Inclusion (LFI) และการยกระดับสิทธิ์ในระบบ โดยมีเป้าหมายหลักคือการเจาะระบบและอ่านไฟล์ flag ที่ซ่อนอยู่

## 🔍 เป้าหมายของโจทย์
- **เจาะระบบเว็บเซิร์ฟเวอร์**
  - ใช้ช่องโหว่ Local File Inclusion (LFI) เพื่อเข้าถึงระบบ
  - ยกระดับสิทธิ์จากผู้ใช้เว็บ (`www-data`) เป็นผู้ดูแลระบบ (`root`)
- **ค้นหาและอ่านไฟล์ Flag ทั้ง 4 ใบ**

# 🧠 TryHackMe - Dogcat

> 🟡 หมวด: Web Security / Privilege Escalation  
> 🧩 ความยาก: Medium  
> 🕵️‍♂️ โหมด: Capture The Flag  
> 🔗 URL: [Dogcat](https://tryhackme.com/room/dogcat)
> 👨‍💻 ผู้ทำ: Thanyakorn

---

## 🛰️ 1. ข้อมูลเบื้องต้น (Target Info)
- IP เครื่องเป้าหมาย: `10.10.85.123`
- Web servers: Apache Web Server

## 🚀 ขั้นตอนการโจมมตี 

### 📌 เริ่มต้นเจาะระบบด้วย LFI (Local File Inclusion)

1. อย่างแรกเลย เข้าไปที่หน้าเว็บด้วย IP ที่โจทย์ให้แล้วเราจะเจอกับหน้าเว็บหลักที่ดูเหมือนเป็นเว็บเกี่ยวกับหมาแมว (Dog / Cat)

![dogcat](images/1.png)

2. จากที่โจทย์บอกว่าเว็บนี้เขียนด้วย PHP และเราสงสัยว่าอาจจะมี LFI เราเลยลองยิง Payload เพื่อทดสอบ LFI แบบ Directory Traversal ด้วย URL นี้:

```bash
http://10.10.85.123/?view=dog/../../../../etc/passwd
```

![dogcat](images/2.png)

- ใช้ ../../../../ เพื่อพยายามออกจากโฟลเดอร์ /dog/ แล้วเข้าไปอ่านไฟล์ /etc/passwd
- แต่ปรากฏว่าเจอ Error ขึ้นมาว่า:
> failed to open stream: No such file or directory in /var/www/html/index.php

3. จาก Error นี้ทำให้เรารู้ว่า Web กำลังเรียกใช้ไฟล์ `index.php` และมีการ include ไฟล์ที่รับมาจาก user
ซึ่งแปลว่าแอปนี้น่าจะใช้พวก `include()` หรือ `require()` แล้วเอาค่าจากพารามิเตอร์ `view` ไปต่อ path

4. ทีนี้เราสงสัยว่าทำไมมันถึงเปิดไฟล์ไม่ได้ ทั้งที่ `/etc/passwd` ก็น่าจะมี
→ อาจจะเพราะ Web เติม `.php` ต่อท้ายให้อัตโนมัติ เช่น `/etc/passwd.php` (ซึ่งไม่มีจริงเลย Error)

5. เพื่อพิสูจน์ว่า Web นี้กำลังทำงานยังไง และเข้าใจ Logic ด้านหลัง
เราจึงต้องพยายาม อ่าน source code ของ index.php

### 🕵️‍♂️ ใช้เทคนิค php://filter เพื่อดู Source Code

6. ถ้าเราจะอ่านไฟล์ PHP ตรง ๆ มันจะถูก execute ทันที (Run ไม่ได้อ่าน)
ดังนั้นต้องใช้ `php://filter` ที่เอาไว้ "กรองไฟล์ก่อนแสดงผล" และแปลงให้อยู่ในรูปแบบ Base64 แทน

7. ยิง Payload นี้ไป:

```bash
http://10.10.85.123/?view=php://filter/convert.base64-encode/resource=dog/../index
```

- `php://filter`: stream wrapper ของ PHP
- `convert.base64-encode`: ให้ผลลัพธ์เป็น Base64
- `dog/../index`: ย้อนกลับมาจาก `/dog/` แล้วเข้าไปยัง `index` (จริง ๆ ก็คือ index.php)

![dogcat](images/3.png)

8. เมื่อยิง URL นี้แล้ว เราจะได้เนื้อหาแบบ Base64 กลับมา

   → ให้ copy ข้อความที่ได้แล้วเอาไปวางในเว็บ [CyberChef](https://gchq.github.io/CyberChef/)

9. ตั้งค่า CyberChef ให้ใช้ Recipe เป็น `From Base64` ก็จะได้ Source code ของ `index.php` ออกมา

![Cyberchef](images/4.png)

### 🔍 วิเคราะห์ index.php ที่ถูกแปลงจาก Base64

10. หลังจากยิง Payload `php://filter` แล้ว เราจะได้ข้อความยาว ๆ ที่ถูกเข้ารหัส Base64 กลับมา

11. เมื่อดูจากโค้ดที่ได้มา จะเห็นว่า:
    - มีการใช้ตัวแปร $ext เพื่อเก็บ "นามสกุลของไฟล์"
    - ตัวเว็บจะ เติม .php เข้าไปให้โดยอัตโนมัติ เสมอ
    - ดูเหมือนว่า View ที่เราส่งเข้าไป (เช่น dog หรือ cat) จะเป็น input หลักที่เว็บนำไปใช้ในการ include ไฟล์
      
   ตัวอย่างเช่น:

```bash
include($_GET["view"] . $ext);
```
  
