# 🧠 TryHackMe - Revenge

> 🟡 หมวด: Web / Privilege Escalation  
> 🧩 ความยาก: Medium  
> 🕵️‍♂️ โหมด: CTF แบบ Capture The Flag  
> 🧩 URL: [Revenge](https://tryhackme.com/room/revenge)  
> 👨‍💻 ผู้ทำ: Thanyakorn

---

## 📌 ข้อมูลจากโจทย์

> "What I want you to do is simple. Break into the server that's running the website and deface the front page."

💬 แปล: สิ่งที่โจทย์ต้องการคือ เจาะเข้าเว็บและแก้หน้าแรก (deface)

---

## 🛰️ 1. ข้อมูลเบื้องต้น (Target Info)

- IP เครื่องเป้าหมาย: `10.10.28.30`  
- พอร์ตที่เปิด: `22`, `80`

---

## ทดลองเข้าใช้งานเว็บไซต์

- ไปที่เมนู **Products** จะเจอชื่อสินค้า 4 รายการ  
- คลิก **LEARN MORE** ของสินค้าใดก็ได้

![Web site](images/1.png)

- ตัวอย่างสินค้า: Box of Duckies  
- จะแสดงรูปสินค้า, คำอธิบาย, ราคา, สี  
- สังเกต Path ด้านบนจะเป็น `/products/1`  
- กลับไปหน้าเดิม แล้วคลิก **LEARN MORE** ของสินค้าอื่น

![Products-Path](images/2.png)

- จะเห็นว่า Path เหมือนเดิมแต่ตัวเลขเปลี่ยนเป็น `/products/2`

![Products-Path](images/3.png)

- ลองเปลี่ยนตัวเลข 2 เป็น 3 ใน Path เพื่อดูสินค้าชิ้นที่ 3

![Products-Path](images/4.png)

---

## 📌 พิจารณาเบื้องต้น

- น่าจะมีการใช้ฐานข้อมูลเพื่อดึงข้อมูลสินค้าตาม ID ที่ส่งผ่าน URL  
- เมื่อลองใส่เครื่องหมาย `'` ที่ Path พบว่าเว็บแสดงข้อผิดพลาด HTTP 500 ซึ่งเป็นสัญญาณบ่งชี้ช่องโหว่ SQL Injection  
- การใส่เครื่องหมาย `'` เพื่อรบกวนการประมวลผล SQL query และดูว่ามี error message หรือข้อมูลอะไรหลุดมาหรือไม่  

![Products-Path](images/5.png)

---

## 🚪 2. Initial Access

### 🔸 2.1 ตรวจสอบช่องโหว่ SQL Injection ด้วย sqlmap

ใช้คำสั่ง:

```bash
sqlmap -u "http://10.10.28.30/products/3*" --dbs
```
**อธิบายคำสั่ง:**

- `sqlmap` คือเครื่องมืออัตโนมัติสำหรับตรวจสอบช่องโหว่ SQL Injection และดึงข้อมูลฐานข้อมูล  
- ตัวเลือก `-u` ใช้ระบุ URL ที่ต้องการทดสอบ โดย `"http://10.10.28.30/products/3*"` ใช้ wildcard `3*` เพื่อให้ sqlmap ลองค่าหลาย ๆ ค่าในพารามิเตอร์ id  
- ตัวเลือก `--dbs` สั่งให้ sqlmap ดึงชื่อฐานข้อมูลทั้งหมดที่เชื่อมต่อกับเว็บเป้าหมาย  
- ผลลัพธ์จะแสดงฐานข้อมูลต่าง ๆ เช่นฐานข้อมูลหลัก `duckyinc` และฐานข้อมูลระบบของ MySQL เช่น `mysql`, `information_schema` เป็นต้น  
- เราจะมุ่งเน้นวิเคราะห์ฐานข้อมูล `duckyinc` ต่อไป เพราะเป็นฐานข้อมูลสำคัญของเว็บนี้

ผลลัพธ์จะแสดงฐานข้อมูลหลัก `duckyinc` และฐานข้อมูลระบบอื่น ๆ เช่น `mysql` เราจะโฟกัสที่ `duckyinc` เพื่อวิเคราะห์ต่อไป

