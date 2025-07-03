# 🧠 TryHackMe - Revenge

> 🟡 หมวด: Web / Privilege Escalation  
> 🧩 ความยาก: Medium  
> 🕵️‍♂️ โหมด: CTF แบบ Capture Flag
> 🧩 URL: (https://tryhackme.com/room/revenge)
> 👨‍💻 ผู้ทำ: Thanyakorn]



---
## ข้อมูลจากโจทย์บอกมาเบื้องต้น

- What I want you to do is simple. Break into the server that's running the website and deface the front page
- มันหมายถึงว่าสิ่งที่ฉันต้องการให้คุณทำนั้นง่ายมากเจาะเข้าไปในเซิร์ฟเวอร์ที่กำลังรันเว็บไซต์และทำให้หน้าแรกเสียหาย


## 🛰️ 1. ข้อมูลเบื้องต้น (Target Info)

- IP เครื่องเป้าหมาย: 10.10.28.30
- พอร์ตที่เปิด: 22, 80
- ลองเข้าใช้งานเว็บไซต์
![Nmap Scan](images/1.png)

---

## 🔍 2. Enumeration

### 🔸 2.1 Nmap Scan

```bash
nmap -sC -sV -oA revenge-scan 10.10.x.x
