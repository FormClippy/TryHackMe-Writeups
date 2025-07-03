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

- IP เครื่องเป้าหมาย: 10.10.28.30  
- พอร์ตที่เปิด: 22, 80

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
- เมื่อลองใส่เครื่องหมาย ' ที่ Path พบว่าเว็บแสดงข้อผิดพลาด HTTP 500 ซึ่งเป็นสัญญาณบ่งชี้ช่องโหว่ SQL Injection  
- การใส่เครื่องหมาย ' เพื่อรบกวนการประมวลผล SQL query และดูว่ามี error message หรือข้อมูลอะไรหลุดมาหรือไม่  

![Products-Path](images/5.png)

---

## 🚪 2. Initial Access

### 🔸 2.1 ตรวจสอบช่องโหว่ SQL Injection ด้วย sqlmap

ใช้คำสั่ง:

bash
sqlmap -u "http://10.10.28.30/products/3*" --dbs


![Sqlmap](images/6.png)

📝 **อธิบายคำสั่ง:**

- `sqlmap` คือเครื่องมืออัตโนมัติสำหรับตรวจสอบช่องโหว่ SQL Injection และดึงข้อมูลฐานข้อมูล  
- ตัวเลือก `-u` ใช้ระบุ URL ที่ต้องการทดสอบ โดย "http://10.10.28.30/products/3*" ใช้ wildcard 3* เพื่อให้ sqlmap ลองค่าหลาย ๆ ค่าในพารามิเตอร์ id  
- ตัวเลือก `--dbs` สั่งให้ sqlmap ดึงชื่อฐานข้อมูลทั้งหมดที่เชื่อมต่อกับเว็บเป้าหมาย  

![Sqlmap](images/7.png)

> ผลลัพธ์: แสดงฐานข้อมูลต่าง ๆ เช่นฐานข้อมูลหลัก duckyinc และฐานข้อมูลระบบของ MySQL เช่น mysql, information_schema เป็นต้น
> เราจะมุ่งเน้นวิเคราะห์ฐานข้อมูล duckyinc ต่อไป เพราะเป็นฐานข้อมูลสำคัญของเว็บนี้

**สรุป:**  
จากการใช้ sqlmap เราพบช่องโหว่ SQL Injection ที่พารามิเตอร์ id ของ URL สามารถเข้าถึงฐานข้อมูลหลักได้ ซึ่งเป็นจุดเริ่มต้นของการเจาะระบบ

### 🔸 2.2 ดึงข้อมูลโครงสร้างฐานข้อมูล (Column Enumeration)

หลังจากพบว่าฐานข้อมูลหลักคือ duckyinc ขั้นตอนถัดไปคือการตรวจสอบว่าในฐานข้อมูลนี้มีตารางอะไรบ้าง และแต่ละตารางมีคอลัมน์อะไรที่น่าสนใจ

ใช้คำสั่ง:

```bash
sqlmap -u "http://10.10.28.30/products/3*" -D duckyinc --columns
```

![Sqlmap](images/8.png)

📝 **อธิบายคำสั่ง:**

- `-D duckyinc` ระบุชื่อฐานข้อมูลที่ต้องการให้ sqlmap ทำการดึงข้อมูล
- `--columns` ให้ sqlmap ดึงชื่อคอลัมน์ทั้งหมดจากทุกตารางภายในฐานข้อมูลที่ระบุ


เมื่อรันคำสั่งแล้ว sqlmap จะถามว่าต้องการดำเนินการต่อหรือไม่ ให้ตอบ Y

![Sqlmap](images/9.png)

ผลลัพธ์: จะเห็นรายชื่อคอลัมน์ทั้งหมดภายในฐานข้อมูล duckyinc ซึ่งเป็นข้อมูลโครงสร้างที่จำเป็นสำหรับการดึงข้อมูลเป้าหมายในขั้นตอนถัดไป

### 🔸 2.3 ดึงข้อมูลจากตาราง user (Dump Table)

หลังจากเราทราบโครงสร้างของฐานข้อมูลแล้ว ขั้นตอนนี้จะเป็นการดึงข้อมูลภายในตาราง user ซึ่งอาจมีข้อมูลสำคัญ เช่น รหัสผ่าน หรือค่า Flag ที่เราต้องการ

ใช้คำสั่ง:

```bash
sqlmap -u "http://10.10.28.30/products/3*" -D duckyinc -T user --dump
```

![Sqlmap](images/10.png)

📝 **อธิบายคำสั่ง:**

- `-D duckyinc` ระบุชื่อฐานข้อมูลที่ต้องการให้ sqlmap ทำการดึงข้อมูล
- `-T user` ระบุตารางที่ต้องการดึงข้อมูล
- `--dump ให้` sqlmap ดึงข้อมูลทั้งหมดจากตารางที่ระบุ

![Sqlmap](images/11.png)

ผลลัพธ์: แสดงข้อมูลในตาราง user ให้สังเกตคอลัมน์ credit_card ของ id = 6 ซึ่งมี Flag ซ่อนอยู่ ให้นำไปใช้ตอบใน TryHackMe

![Sqlmap](images/12.png)

---

### 🔸 2.4 ดึงข้อมูลจากตาราง `system_user`

หลังจากนั้น เราจะโฟกัสที่ตาราง `system_user` ซึ่งคาดว่าอาจมีข้อมูลผู้ใช้งานระบบ เช่น admin หรือข้อมูลที่เกี่ยวข้องกับการยกระดับสิทธิ์

ใช้คำสั่ง:

```bash
sqlmap -u "http://10.10.28.30/products/3*" -D duckyinc --dump -T system_user
```
![Sqlmap](images/13.png)

📝 **อธิบายคำสั่ง:**

- `-D duckyinc` ระบุชื่อฐานข้อมูลที่ต้องการดึงข้อมูล
- `--dump` ดึงข้อมูลทั้งหมดจากตารางนั้น
- `-T system_user` ระบุตารางที่ต้องการ

![Sqlmap](images/14.png)

ผลลัพธ์: พบ username และ password ที่เป็น hash (bcrypt)

### 🔐 2.4 Credential Cracking with John

หลังจากดึงข้อมูลจากตาราง `system_user` เราพบว่า password ของผู้ใช้งานอยู่ในรูปแบบ bcrypt ซึ่งเป็น hash ที่นิยมใช้ในระบบจริง

---

## 🛠️ ขั้นตอนการเตรียมไฟล์ hash

1. สร้างโฟลเดอร์ชื่อ `ducky` แล้วเข้าไปในโฟลเดอร์:

```bash
mkdir ducky
cd ducky
```

2. สร้างไฟล์ชื่อ hashes.txt เพื่อเก็บ hash:

```bash
nano hashes.txt
```

3. วาง bcrypt hash ที่ได้จาก sqlmap (ในที่นี้ใส่เฉพาะของ user server-admin)
> 📌 แนะนำให้ลบ hash อื่น ๆ ออก เพื่อให้ crack ได้เร็วขึ้น

4. ตรวจสอบว่า hash ถูกบันทึกเรียบร้อย:

```bash
cat hashes.txt
```

## 🧂 Crack Bcrypt Hash ด้วย John the Ripper
📦 เราจะใช้ john คู่กับ rockyou.txt ในการ Dictionary attack:

```bash
john hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

⚠️ หมายเหตุ:
> ถ้ามันขึ้นว่า No password hashes left to crack หมายความว่า john ได้ทำการถอดรหัสแฮชที่มีอยู่ในไฟล์ hashes.txt เรียบร้อยแล้วหรือไม่มีแฮชที่เหลือให้ทำการถอดรหัสจากไฟล์นั้น

- หลังจากรันเสร็จรอจนกว่าจะ crack สำเร็จ จากนั้นดูผลลัพธ์ที่ได้โดยใช้คำสั่ง:
```bash
john --show hashes.txt
```
