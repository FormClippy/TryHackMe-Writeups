# 🧠 TryHackMe - Revenge

> 🟡 หมวด: Web / Privilege Escalation
 
> 🧩 ความยาก: Medium

> 🕵️‍♂️ โหมด: CTF แบบ Capture Flag

> 🧩 URL: [Revenge](https://tryhackme.com/room/revenge)

> 👨‍💻 ผู้ทำ: Thanyakorn]



---
## ข้อมูลจากโจทย์บอกมาเบื้องต้น

- What I want you to do is simple. Break into the server that's running the website and deface the front page
- มันหมายถึงว่าสิ่งที่ฉันต้องการให้คุณทำนั้นง่ายมากเจาะเข้าไปในเซิร์ฟเวอร์ที่กำลังรันเว็บไซต์และทำให้หน้าแรกเสียหาย


## 🛰️ 1. ข้อมูลเบื้องต้น (Target Info)

- IP เครื่องเป้าหมาย: 10.10.28.30
- พอร์ตที่เปิด: 22, 80
  
## ลองเข้าใช้งานเว็บไซต์

- เราจะลองไปที่เมนู Products เราจะเจอชื่อของสินค้าและจะเห็นว่ามีอยู่ 4 สินค้า
- ลองกด LEARN MORE ของสินค้าอันไหนก็ได้
  
![Web site](images/1.png)

- Box of Duckies
- จะเจอรูปสินค้า, คำอธิบาย, ราคา, สี
- ให้เราสังเกต Path ด้านบนจะเป็น /products/1
- ทีนี้ให้กลับไปหน้าเดิมแล้วคลิก LEARN MORE ของอีกสินค้า

![Products-Path](images/2.png)

- จะเห็นว่า Pathเหมือนเดิมแต่ตัวเลขเปลี่ยนเป็น /products/

![Products-Path](images/3.png)

- ที่นี้เราลองมาเล่นเปลี่ยนจากตัวเลข 2 เป็น 3 จาก Pathข้างบนเลย
- จะได้สินค้าที่ 3 มาอยู่ในหน้าเว็บเพจ

![Products-Path](images/4.png)

---

## 🔍 2. Enumeration

### 🔸 2.1 Nmap Scan

```bash
nmap -sC -sV -oA revenge-scan 10.10.x.x
