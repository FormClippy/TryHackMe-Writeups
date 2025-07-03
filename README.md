# 🧠 TryHackMe - Revenge

> 🟡 หมวด: Web / Privilege Escalation  
> 🧩 ความยาก: Medium  
> 🕵️‍♂️ โหมด: CTF แบบ Capture The Flag  
> 🧩 URL: [Revenge](https://tryhackme.com/room/revenge)  
> 👨‍💻 ผู้ทำ: Thanyakorn

---

## 📌 ข้อมูลจากโจทย์

> "What I want you to do is simple. Break into the server that's running the website and deface the front page."

💬 แปล: โจทย์ต้องการให้เจาะเข้าเว็บและแก้ไขหน้าแรก (deface)

---

## 🛰️ 1. ข้อมูลเบื้องต้น (Target Info)

- IP เครื่องเป้าหมาย: `10.10.28.30`
- พอร์ตที่เปิด: `22`, `80`

---

## ทดลองเข้าใช้งานเว็บไซต์

- ไปที่เมนู **Products** จะเจอชื่อสินค้า 4 รายการ
- คลิก **LEARN MORE** ของสินค้าชิ้นใดก็ได้

![Web site](images/1.png)

- ตัวอย่างสินค้า: Box of Duckies
- จะเห็นรูปสินค้า, คำอธิบาย, ราคา, สี
- สังเกต Path ด้านบนจะเป็น `/products/1`
- กลับไปหน้าเดิม แล้วคลิก **LEARN MORE** ของสินค้าอื่น

![Products-Path](images/2.png)

- จะเห็นว่า Path เหมือนเดิมแต่ตัวเลขเปลี่ยนเป็น `/products/2`

![Products-Path](images/3.png)

- ลองเปลี่ยนตัวเลข 2 เป็น 3 ใน Path เพื่อดูสินค้าชิ้นที่ 3

![Products-Path](images/4.png)

---

## 📌 การพิจารณาเบื้องต้น

- ระบบน่าจะใช้ข้อมูลในฐานข้อมูลเป็นตัวดึงข้อมูลสินค้าโดยใช้ ID ที่ส่งผ่าน Path เป็น key หลัก
- เมื่อลองใส่เครื่องหมาย `'` ที่ Path เพื่อทดสอบ SQL Injection พบ error 500 (Internal Server Error)
- ข้อผิดพลาดนี้บ่งชี้ถึงช่องโหว่ SQL Injection ที่ backend อาจเกิดข้อผิดพลาด SQL query

![Products-Path](images/5.png)

---

## 🚪 2. Initial Access

### 🔸 2.1 ใช้ sqlmap ตรวจสอบช่องโหว่ SQL Injection

ใช้คำสั่ง:

```bash
sqlmap -u "http://10.10.28.30/products/3*" --dbs
