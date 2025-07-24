# Cisco-HSRP-DHCP
✅ คืออะไร?
HSRP (Hot Standby Router Protocol):
โปรโตคอลของ Cisco สำหรับทำ Virtual Gateway โดยมี router ตัวหนึ่งเป็น Active อีกตัวเป็น Standby และมี IP เสมือนร่วมกัน

Track:
ใช้เพื่อติดตามสถานะของ interface หรือ route หากล้มเหลวจะ ลด Priority เพื่อให้ router อื่นเข้ามาเป็น Active แทน

DHCP Server:
ใช้แจก IP address และ Default Gateway ให้กับผู้ใช้งานแบบอัตโนมัติ

🎯 จุดประสงค์ของการตั้งค่านี้
เพื่อให้มีระบบ Gateway สำรอง: ถ้า Router แรกล่ม Router สำรองจะรับหน้าที่แทนทันที
ใช้ track ตรวจสอบสถานะ interface จริง (เช่น g0/0) ถ้าล่ม จะลด priority ของ Router นั้นลง
ให้ผู้ใช้งานใน VLAN10 รับ IP address และ default gateway (192.168.1.1) อัตโนมัติ

⚠️ เงื่อนไขสำคัญ
ทั้ง 2 router ต้องอยู่ใน network/subnet เดียวกัน (เช่น 192.168.10.0/24)
ใช้ standby 10 ip 192.168.10.1 เหมือนกัน = IP เสมือนของ Gateway
ตัวที่มี priority สูงสุดจะเป็น Active (ถ้าไม่มีการลด priority จาก track)
ถ้า interface ที่ track อยู่ down, priority จะถูกลดลง (decrement 30)

🛠️ การใช้งานจริง
ใช้ในองค์กรหรือสำนักงานที่ต้องการ ความเสถียรในการเข้าถึง Gateway แม้จะมีอุปกรณ์ตัวใดตัวหนึ่งล้ม
ใช้ร่วมกับ DHCP server บน router เพื่อให้ผู้ใช้ได้รับ Default Gateway เป็น IP เสมือน

🔗 การเชื่อมต่อ
--------------------------
       [PC Users]
            |
         [Switch]
         /      \
      [Router1] [Router2]
      192.168.10.2  192.168.10.3
          |             |
       track g0/0    track g0/0

--------------------------
ทั้งสอง Router เชื่อมต่อกับ Switch เดียวกัน
Router1 มี Priority เริ่มต้นสูงกว่า Router2
ถ้า Router1 interface g0/0 ล่ม -> Priority ถูกลดลง 30 -> Router2 กลายเป็น Active แทน

📈 ผลลัพธ์
IP เสมือนของ Gateway คือ 192.168.10.1
ถ้า interface g0/0 ของ Router1 ใช้งานได้:
Router1 เป็น Active Gateway
ถ้า interface g0/0 ของ Router1 ล่ม:
Router1 Priority = 100 - 30 = 70 < Router2 (80) -> Router2 กลายเป็น Active Gateway
DHCP แจก IP ตั้งแต่ 192.168.1.11 - 192.168.1.254
Default Gateway ของเครื่องลูกข่ายที่รับ DHCP คือ 192.168.1.1

🧠 สรุป
| รายการ              | รายละเอียด                                                      |
| ------------------- | --------------------------------------------------------------- |
| โปรโตคอล            | HSRP (กลุ่มที่ 10)                                              |
| Virtual IP          | 192.168.10.1                                                    |
| Router จริง         | 192.168.10.2, 192.168.10.3                                      |
| DHCP                | แจก IP 192.168.1.11–254, Default Gateway: 192.168.1.1           |
| Tracking            | ใช้ตรวจสถานะ interface g0/0                                     |
| Priority & Failover | Router1 Priority 100 - 30 (ถ้า down) = 70 < Router2 Priority 80 |
| ประโยชน์            | Redundancy + Failover + DHCP อัตโนมัติ                          |
