## ✨ บทนำ
ห้องนี้เป็นห้องแนะนำเครื่องมือ ffuf (Fuzz Faster U Fool) ซึ่งเป็นเครื่องมือ fuzzing สำหรับเว็บแอปพลิเคชันที่เร็วและยืดหยุ่นมาก ใช้สำหรับ brute-force URL paths, parameters, virtual hosts, และอื่น ๆ ผ่านคำสั่งที่สามารถปรับแต่งได้หลากหลาย

## 🎯 เป้าหมายของโจทย์
- เรียนรู้การใช้งานเครื่องมือ ffuf เพื่อทำ Directory และ File fuzzing บน Web Server
- วิเคราะห์และตีความค่า HTTP status code ที่ตอบกลับจากการ fuzz
- ฝึกการใช้ wordlists กับ ffuf เพื่อค้นหา paths ที่ซ่อนอยู่ เช่น /admin, /backup, /login ฯลฯ
- ใช้ฟีเจอร์เพิ่มเติมของ ffuf เช่นการตั้ง header, การเจาะพารามิเตอร์, และการปรับ output format
- เข้าใจการ fuzz แบบมีโครงสร้าง เช่น fuzz ใน URL, ในพารามิเตอร์, หรือในค่าต่าง ๆ ของฟอร์ม

# 🧠 TryHackMe - FFUF 🔍💥

🟡 **หมวด:** Web / Fuzzing / Content Discovery  
🧩 **ความยาก:** Easy
🕵️‍♂️ **โหมด:** CTF แบบ Walkthrough + Hands-on Lab  
🔗 **URL:** [FFUF](https://tryhackme.com/room/ffuf)  
👨‍💻 **ผู้ทำ:** Thanyakorn

---

# 🛠️ ขั้นตอนการทำ

## ขั้นตอนที่ 1. **เข้าถึงเว็บเป้าหมาย**
   - เปิดเว็บเบราว์เซอร์ แล้วใส่ IP Address ที่โจทย์กำหนด (เช่น `http://10.201.120.42`)
   - จะพบหน้าเว็บหลักตามภาพด้านล่าง

![Web](images/1-1.png)

## ขั้นตอนที่ 2: **ใช้คำสั่ง ffuf เพื่อค้นหาไฟล์และโฟลเดอร์** 🔍

- ใช้คำสั่งนี้เพื่อฟัซซ์ (fuzz) ชื่อไฟล์หรือโฟลเดอร์บนเว็บเซิร์ฟเวอร์

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/big.txt
```

![ffuf](images/2.png)

📌 คำอธิบายคำสั่ง:  
- `-u` คือ URL ที่ต้องการทดสอบ โดยตำแหน่ง `FUZZ` จะถูกแทนที่ด้วยคำจาก wordlist  
- `-w` คือไฟล์ wordlist ที่ใช้ค้นหาคำที่เป็นไปได้ในตำแหน่ง `FUZZ`  

📊 ผลลัพธ์ที่ได้คือ รายการไฟล์หรือโฟลเดอร์ที่พบในเซิร์ฟเวอร์ซึ่งตอบกลับด้วย HTTP status code ที่น่าสนใจ เช่น 200, 301 เป็นต้น

✅ จากการใช้คำสั่ง ffuf พบว่าไฟล์แรกที่มีสถานะ HTTP 200 คือ favicon.ico

## ขั้นตอนที่ 3: 🔍 Fuzz หาชื่อไฟล์ด้วย Wordlist แบบละเอียด

🛠️ คำสั่งที่ใช้

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt
```

![ffuf](images/3.png)

📌 คำอธิบายคำสั่ง:
- `-w` ชี้ไปยังไฟล์ wordlist ที่ใช้ ซึ่งในที่นี้คือ wordlist สำหรับชื่อ ไฟล์ ขนาดกลางและเป็น lowercase ทั้งหมด
(เหมาะสำหรับการเจาะลึกหาไฟล์สำคัญที่ไม่ได้อยู่ใน root directory แบบพื้นฐาน)

✅ จากการใช้คำสั่ง ffuf พบว่า ไฟล์แรกที่มีสถานะ HTTP 200 คือ favicon.ico
แต่ถ้าดูเฉพาะ ไฟล์ที่เป็น .txt จะพบว่า:

➡️ robots.txt คือไฟล์ที่ค้นพบและตรงกับคำถาม

## ขั้นตอนที่ 4: ตรวจสอบนามสกุลไฟล์ของ index

- 📥 เป้าหมายคือค้นหาว่าไฟล์ index มีนามสกุลอะไรบ้างที่มีอยู่จริงในระบบ โดยใช้คำสั่ง ffuf กับ wordlist สำหรับนามสกุลของไฟล์เว็บ

```bash
ffuf -u http://10.201.120.42/indexFUZZ -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt
```

![ffuf](images/4.png)

📌 คำอธิบายคำสั่ง:

- `indexFUZZ` จะถูกแทนที่ด้วยนามสกุลจาก wordlist เช่น `.php`, `.html`, `.bak` ฯลฯ
- Wordlist ที่ใช้: `web-extensions.txt` เป็นลิสต์ของนามสกุลไฟล์ยอดนิยมที่มักพบในเว็บแอป

✅ ผลลัพธ์ที่ค้นพบ:
- 🔒 `index.phps` → [Status: 403] (ถูกห้ามเข้าถึง)
- 🔁 `index.php` → [Status: 302] (ถูก redirect ไปที่หน้าอื่น)

## ขั้นตอนที่ 5: เพิ่มการค้นหาด้วยนามสกุลไฟล์ .php และ .txt

- ในขั้นตอนนี้ เราจะใช้คำสั่ง ffuf เพื่อค้นหาไฟล์หรือหน้าเว็บที่มีนามสกุล .php และ .txt

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -e .php,.txt
```

![ffuf](images/5.png)

📌 คำอธิบายคำสั่งเพิ่มเติม:
- `-u` กำหนด URL ที่ต้องการทดสอบ โดยตำแหน่ง FUZZ จะถูกแทนที่ด้วยคำจาก wordlist
- `-w` ระบุไฟล์ wordlist ที่ใช้ค้นหาคำที่เป็นไปได้
- `-e .php,.txt` บอก `ffuf` ให้ทดสอบด้วยนามสกุลไฟล์ที่ระบุ คือ `.php` และ `.txt` (extension) เพื่อค้นหาไฟล์ที่มีนามสกุลเหล่านี้

✅ จากการใช้คำสั่ง ffuf ค้นหาไฟล์โดยเพิ่มนามสกุล .php และ .txt พบว่าไฟล์ about.php มีขนาด 4840 ไบต์

## ขั้นตอนที่ 6: ค้นหา Directory ที่มีอยู่ทั้งหมด

🛠️ คำสั่งที่ใช้:

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt
```

![ffuf](images/6.png)

📊 ผลลัพธ์ที่ได้
- เจอ directory ทั้งหมดได้แก่:
  - `docs`
  - `config`
  - `external`
  - `server-status` 

✅ จำนวน directory ที่พบและเข้าถึงได้ คือ docs, config, external และ server-status

## ขั้นตอนที่ 7: ใช้ Filter เพื่อตัดสถานะ HTTP 403 ออก

🛠️ คำสั่งที่ใช้:

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -fc 403
```

![ffuf](images/7.png)

📌 คำอธิบายคำสั่ง:

- `-fc 403` คือการกรอง (filter) ผลลัพธ์ที่มีสถานะ HTTP 403 Forbidden ออกไป
(หมายความว่า ffuf จะไม่แสดงผลลัพธ์ที่ตอบกลับด้วย 403)

🎯 วัตถุประสงค์:
เพื่อให้เห็นเฉพาะไฟล์หรือโฟลเดอร์ที่ไม่ถูกบล็อกด้วยสิทธิ์ (ไม่ใช่ 403) ทำให้ค้นหาได้สะดวกขึ้น

📊 ผลลัพธ์:
จะเห็นรายการที่ตอบกลับสถานะอื่น ๆ ที่ไม่ใช่ 403 เช่น 200, 302, 301 เป็นต้น

✅ หลังจากใช้ filter ตัดสถานะ 403 แล้ว จำนวนผลลัพธ์ที่ได้คือ 11 รายการ

## ขั้นตอนที่ 8: ใช้ Filter เพื่อแสดงเฉพาะหน้าเว็บที่มีสถานะ HTTP 200 เท่านั้น

🛠️ คำสั่งที่ใช้:

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -mc 200
```

![ffuf](images/8.png)

📌 คำอธิบายคำสั่ง:
- `-mc 200` ย่อมาจาก `match code` หมายถึง แสดงผลเฉพาะรายการที่มีสถานะ HTTP 200 OK เท่านั้น
- การกรองแบบนี้ช่วยให้เรามุ่งเน้นเฉพาะหน้าเว็บที่โหลดสำเร็จ ไม่รวมพวก 302 redirect, 403 forbidden หรือ 404 not found

🎯 วัตถุประสงค์ของขั้นตอนนี้:
เพื่อหาว่ามีหน้าเว็บไหนที่สามารถเข้าถึงได้สำเร็จจริง (status 200) จากการ brute-force ด้วย wordlist

✅📊 ผลลัพธ์ที่ได้มีทั้งหมด 6 รายการ ที่แสดงผลลัพธ์ status 200

 ## ขั้นตอนที่ 9: เปรียบเทียบผลระหว่าง -fc และ -fr เพื่อหาว่าไฟล์สำคัญไหนจะถูกซ่อนไป

 
🔍 คำถามจาก TryHackMe:
> "Which valuable file would have been hidden if you used -fc 403 instead of -fr?"
> (ไฟล์สำคัญอะไรที่เราจะมองไม่เห็นถ้าใช้ -fc 403 แทน -fr?)

🛠️ คำสั่งที่ใช้:

```bash
ffuf -u http://10.201.120.42/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files-lowercase.txt -fr '/\..*'
```

![ffuf](images/9.png)

📌 คำอธิบายคำสั่ง:
- `-fr '/\..*'` = Filter Regex ใช้กรองรายการผลลัพธ์ที่ขึ้นต้นด้วยจุด (`.`) เช่น `.git` หรือ `.env` ออกจากผลลัพธ์
- การใช้ `-fr` ไม่ได้กรองตาม Status Code เหมือน `-fc` แต่กรองตามแพทเทิร์นใน path ที่เรากำหนด
- ในข้อนี้เราไม่กรอง status 403 ออกไป ทำให้ยังเห็นไฟล์ที่ถูก block อยู่ (เช่น 403 Forbidden)

หากใช้คำสั่งแบบนี้แทน:

```bash
ffuf ... -fc 403
```

มันจะ กรองผลลัพธ์ทั้งหมดที่มีสถานะ 403 ออกไปทันที ซึ่งหมายความว่าไฟล์ wp-forum.phps จะ ไม่ถูกแสดงให้เห็นเลย

✅ จากผลลัพธ์ มีไฟล์หนึ่งที่ได้สถานะ 403 คือ: wp-forum.phps

🧠 Note เพิ่มเติม (เชิงเทคนิคจริงจัง):
ไฟล์ `.phps` มักจะถูกเซิร์ฟเวอร์ config ไว้ให้แสดงซอร์สโค้ดของ PHP แทนที่จะ execute — ดังนั้นหากหลุดออกมา เป็นประตูให้เราเห็น logic ภายในโดยไม่ต้อง execute จริง นี่ถือว่า "ของดี" ในการเจาะระบบ และควรถูกจับตา

## 🧪 Task 5 – Fuzzing Parameters
> 🔍 Base URL ที่ใช้: `http://10.201.92.170/sqli-labs/Less-1/`

🔧 เป้าหมายของ Task นี้:
1. ทดสอบว่า endpoint `/sqli-labs/Less-1/` รับ parameter อะไรบ้าง
2. ใช้ ffuf เพื่อค้นหาพารามิเตอร์ที่ใช้งานอยู่ เช่น `id`, `user`, `page`, etc.
3. เตรียมไว้สำหรับการทดสอบ SQLi หรือ input validation ถัดไป

### ขั้นตอนที่ 1: ลองเข้าหน้า URL ก่อน

เปิดใน browser:

```bash
http://10.201.92.170/sqli-labs/Less-1/
```

![ffuf](images/10.png)

### ขั้นตอนที่ 2: Fuzz หาชื่อพารามิเตอร์ที่ใช้งานได้

🎯 วัตถุประสงค์
เราจะทำการ fuzz เพื่อค้นหาว่า server ยอมรับ พารามิเตอร์ชื่ออะไร ผ่าน URL นี้:

```bash
http://10.201.92.170/sqli-labs/Less-1/
```

🛠️ คำสั่งที่ใช้

```bash
ffuf -u 'http://10.201.92.170/sqli-labs/Less-1/?FUZZ=1' -c -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fw 39
```

![ffuf](images/11.png)

📌 อธิบายคำสั่ง
- `-u 'http://.../?FUZZ=1'`: จุดที่จะทำ Fuzz คือชื่อของพารามิเตอร์ เช่น `?id=1`, `?user=1`
- `-w /.../burp-parameter-names.txt`: Wordlist ที่ใช้สำหรับชื่อพารามิเตอร์ โดยอิงจาก Burp Suite ซึ่งรวบรวมชื่อพารามิเตอร์ที่พบบ่อย
- `-fw 39`:	เป็นการกรอง (filter) ไม่แสดงผลลัพธ์ที่จำนวนคำ (Words) เท่ากับ 39 — ซึ่งมักเป็น response เดิม ๆ ที่ไม่ตอบสนองต่อพารามิเตอร์
- `-c`: แสดงผลลัพธ์เป็นสี ช่วยให้มองเห็นได้ชัดเจนขึ้น

✅ ผลลัพธ์

```bash
id        [Status: 200, Size: 721, Words: 37, Lines: 29, Duration: 360ms]
```





