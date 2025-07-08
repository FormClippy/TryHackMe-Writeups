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
### 🕵️‍♂️ 1. สำรวจช่องโหว่ LFI ผ่านพารามิเตอร์ view
📌 เริ่มต้นเจาะระบบด้วย LFI (Local File Inclusion)

1. อย่างแรกเลย เข้าไปที่หน้าเว็บด้วย IP ที่โจทย์ให้แล้วเราจะเจอกับหน้าเว็บหลักที่ดูเหมือนเป็นเว็บเกี่ยวกับหมาแมว (Dog / Cat)

![dogcat](images/1.png)

2. จากที่โจทย์บอกว่าเว็บนี้เขียนด้วย PHP และเราสงสัยว่าอาจจะมี LFI เราเลยลองยิง Payload เพื่อทดสอบ LFI แบบ Directory Traversal ด้วย URL นี้:

```bash
http://10.10.85.123/?view=dog/../../../../etc/passwd
```
- ใช้ ../../../../ เพื่อพยายามออกจากโฟลเดอร์ /dog/ แล้วเข้าไปอ่านไฟล์ /etc/passwd
- แต่ปรากฏว่าเจอ Error ขึ้นมาว่า:
> failed to open stream: No such file or directory in /var/www/html/index.php


