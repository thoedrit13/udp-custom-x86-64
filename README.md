## UDP Custom - Installer and Manager
#### * Version ⇢ 2.5-Lite
---
UDP (User Datagram Protocol) is a network communication protocol that operates on top of IP (Internet Protocol). It is a simpler protocol compared to TCP (Transmission Control Protocol), as it aims for speed rather than reliability.


---
<center><img src="https://raw.githubusercontent.com/http-custom/udp-custom/main/bin/banner.jpg" alt="banner" width="400"/></center>

# Supported OS
- ubuntu 20.04 [x86_64] ✅ _(recommended)_
- [arm] ❌

## วิธีติดตั้ง
```
sudo -i
``` 
```
git clone https://github.com/thoedrit13/udp-custom-x86-64 && cd udp-custom-x86-64 && chmod +x install.sh && ./install.sh
```
```
sudo systemctl enable udp-custom
```
หลังจากติดตั้งเสร็จ ให้ reboot เครื่อง เพื่อสร้างกฏ iptables ขึ้นมา

เช็คจาก
``` 
sudo iptables -t nat -L PREROUTING -n --line-numbers
```
จากนั้นแก้ systemd
```
nano /etc/systemd/system/udp-custom.service
```
แก้เป็น
```
[Unit]
Description=UDP Custom by ePro Dev. Team
Wants=network-online.target
After=network-online.target docker.service

[Service]
User=root
Type=simple
ExecStart=/root/udp/udp-custom server
WorkingDirectory=/root/udp/
Restart=always
RestartSec=2s

[Install]
WantedBy=multi-user.target
```


จากนั้นสามารถใช้ได้เลย จากคำสั่ง
``` 
udp
``` 
## Manually

## Note: 
 * Use optional port exclude when port udp between 1-65535 already use by other udp tunnel, like badvpn, ovpn udp and other.
 * Edit path config /root/udp/config.json, after changing it then reboot
 * Optional port exclude separated by coma, ex. 53,5300

ในเริ่นต้น จะเพิ่ม DNAT iptavles เอง ถ้าต้องการเว้น port ให้ใส่ 

เช็ค 
``` 
sudo iptables -t nat -L PREROUTING -n --line-numbers
``` 
เช่น
``` 
2    DNAT       udp  --  0.0.0.0/0            0.0.0.0/0            udp dpts:1:65535 to::36712
```


ถ้าต้องการยกเว้น port 
ให้ทำการแก้ที่ systemctl
```
nano /etc/systemd/system/udp-custom.service
```
แก้ตรง ExecStart เพิ่ม --exclude เช่น 53,68,111,546,5353,7359,12451,41641,51820,53602 ใช้จริง
```
ExecStart=/root/udp/udp-custom server --exclude 53,68,111,546,5353,7359,12451,41641,51820,53602
```
ยกเว้น 51820
```
ExecStart=/root/udp/udp-custom server --exclude 51820
```

บันทึกแล้ว
```
systemctl daemon-reload
systemctl restart udp-custom.service
```
ถ้าต้องการเช็ค หรือ ถ้าต้องการลบ iptables ตัวเดิมออก
```
iptables -t nat -L PREROUTING -n --line-numbers
```
ดูว่าคือ rule ไหน เช่น 2
```
iptables -t nat -D PREROUTING 2
sudo iptables-save > /etc/iptables/rules.v4
```



``` 

for home server

การเปิด port router
port 1- 65535
internal port 1
Protocol udp


🚀 ทางออกที่ดีที่สุด: ตั้งค่า DMZ (Demilitarized Zone)
ฟีเจอร์ DMZ คือการสั่งเราเตอร์ว่า "ถ้ามีทราฟฟิกอะไรก็ตามวิ่งเข้ามา แล้วไม่รู้จะส่งไปไหน ให้โยนมาที่ IP นี้ทั้งหมดเลย" ซึ่งมันตอบโจทย์การรับข้อมูลพอร์ตสุ่ม 1-65535 ของแอป HTTP Custom แบบ 100% แถมไม่ไปตีกับกฎ Jellyfin ของเก่าด้วยครับ

วิธีตั้งค่า (ทำตามนี้ได้เลยครับ ง่ายกว่าเดิมเยอะ):

สังเกตที่เมนูด้านซ้ายมือของเราเตอร์ (ใต้เมนู Port Forwarding) จะมีคำว่า DMZ ให้คลิกเข้าไปเลยครับ

ติ๊กเครื่องหมายถูกที่ช่อง Enable DMZ

ในช่อง DMZ Host IP Address ให้กรอก IP ของเครื่อง Home Server ลงไปครับ: 192.168.0.248

กด Apply / Save (หมายเหตุ: กฎ Port Forwarding อันไหนที่ตั้งค้างไว้แล้ว Error ให้ลบทิ้งไปได้เลยครับ ไม่ต้องใช้แล้ว)

🛡️ ใช้ DMZ แล้วปลอดภัยไหม?
หลายคนกลัวว่าเปิด DMZ แล้วจะโดนแฮก แต่สำหรับกรณีของคุณ ปลอดภัยแน่นอนครับ เพราะเซิร์ฟเวอร์ Ubuntu ของคุณมีระบบไฟร์วอลล์ UFW กางร่มป้องกันไว้อยู่แล้ว

ถึงแม้เราเตอร์จะโยนขยะหรือทราฟฟิกแปลกๆ เข้ามาทุกพอร์ต แต่ตัว UFW บน Ubuntu จะ "เตะทิ้งทั้งหมด" และจะยอมให้ผ่านเฉพาะพอร์ตที่คุณเคยพิมพ์คำสั่งเปิดไว้เท่านั้น (เช่น 22, 8096, 8080, 36712 เป็นต้น)

พอตั้งค่า DMZ เสร็จปุ๊บ ลองกด Connect ในแอป HTTP Custom อีกรอบได้เลยครับ คราวนี้ข้อมูล UDP ตั้งแต่ 1-65535 จะไหลทะลักเข้าเซิร์ฟเวอร์คุณแบบไม่มีเราเตอร์มากั้นแล้วครับ!

```


วิธีลบ

ล้างกฎ Iptables ให้หมด
```
iptables -t nat -L PREROUTING -n --line-numbers
```
ดูว่าคือ rule ไหน เช่น 2
```
iptables -t nat -D PREROUTING 2
sudo iptables-save > /etc/iptables/rules.v4
```



ปิดการทำงานและลบ Systemd Service
```
sudo systemctl stop udp-custom
sudo systemctl disable udp-custom
sudo rm /etc/systemd/system/udp-custom.service
sudo systemctl daemon-reload
```

ลบโฟลเดอร์และไฟล์โปรแกรมทิ้ง

```
sudo rm -rf /root/udp
sudo rm -rf /root/udp/udp-custom
sudo rm -rf udp-custom-x86-64
sudo rm /root/udp/config.json
```

