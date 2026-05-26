# คู่มือติดตั้งและใช้งาน FTP Server บน Ubuntu Desktop 24.04 LTS
### สำหรับผู้เริ่มต้น — ครบทุกฟังก์ชัน พร้อม FileZilla Client บน Windows 10

---

## สารบัญ

1. [FTP คืออะไร? ทำความเข้าใจก่อนเริ่ม](#1-ftp-คืออะไร-ทำความเข้าใจก่อนเริ่ม)
2. [ทำไมถึงเลือก vsftpd?](#2-ทำไมถึงเลือก-vsftpd)
3. [โครงสร้างระบบในคู่มือนี้](#3-โครงสร้างระบบในคู่มือนี้)
4. [ติดตั้ง vsftpd](#4-ติดตั้ง-vsftpd)
5. [ทำความเข้าใจไฟล์ config หลัก](#5-ทำความเข้าใจไฟล์-config-หลัก)
6. [ตั้งค่า vsftpd แบบสมบูรณ์](#6-ตั้งค่า-vsftpd-แบบสมบูรณ์)
7. [การจัดการ User และสิทธิ์ (Permission)](#7-การจัดการ-user-และสิทธิ์-permission)
8. [การตั้งค่าความปลอดภัย](#8-การตั้งค่าความปลอดภัย)
9. [เปิด Firewall สำหรับ FTP](#9-เปิด-firewall-สำหรับ-ftp)
10. [ใช้งานร่วมกับ Web Server (Apache/Nginx)](#10-ใช้งานร่วมกับ-web-server-apachenginx)
11. [การใช้งาน FileZilla Client บน Windows 10](#11-การใช้งาน-filezilla-client-บน-windows-10)
12. [คำสั่ง vsftpd ที่ใช้บ่อย](#12-คำสั่ง-vsftpd-ที่ใช้บ่อย)
13. [การดู Log และแก้ปัญหา](#13-การดู-log-และแก้ปัญหา)
14. [ทริปและเทคนิคสำหรับมือใหม่](#14-ทริปและเทคนิคสำหรับมือใหม่)
15. [ตารางสรุปคำศัพท์สำคัญ](#15-ตารางสรุปคำศัพท์สำคัญ)

---

## 1. FTP คืออะไร? ทำความเข้าใจก่อนเริ่ม

### FTP (File Transfer Protocol) คืออะไร?

**FTP** คือ "โปรโตคอล" (ภาษากลาง/กฎการสื่อสาร) สำหรับ **รับ-ส่งไฟล์** ระหว่างเครื่องคอมพิวเตอร์สองเครื่องผ่านเครือข่าย เปรียบได้กับ "บริการรับส่งพัสดุ" ในโลกดิจิทัล

```
[Windows 10 - FileZilla]  <---FTP---> [Ubuntu 24.04 - vsftpd]
      (FTP Client)                         (FTP Server)
      "ลูกค้า"                              "ร้านค้า/คลังสินค้า"
```

### คำศัพท์พื้นฐานที่ต้องรู้

| คำศัพท์ | ความหมายง่าย ๆ | ตัวอย่าง |
|---------|---------------|---------|
| **Server** | เครื่องให้บริการ คอยรอรับการเชื่อมต่อ | Ubuntu ที่ติดตั้ง vsftpd |
| **Client** | เครื่องที่ขอใช้บริการ | Windows 10 ที่ใช้ FileZilla |
| **Protocol** | ภาษา/กฎที่ทั้งสองฝ่ายตกลงใช้คุยกัน | FTP, HTTP, SSH |
| **Port** | ช่องทางสื่อสารในเครื่อง (เหมือนเลขห้อง) | FTP ใช้ Port 21 |
| **IP Address** | ที่อยู่ของเครื่องในเครือข่าย | 192.168.1.100 |
| **LAN** | เครือข่ายภายในบ้าน/สำนักงาน | เครื่องในวง 192.168.x.x |
| **DHCP** | ระบบแจก IP อัตโนมัติ (router ทำให้) | ได้ IP ใหม่ทุกครั้งที่เชื่อม |
| **Passive Mode** | โหมด FTP ที่ client เป็นฝ่าย "เปิดการเชื่อมต่อ" | เหมาะกับ firewall/NAT |
| **Active Mode** | โหมด FTP ที่ server เป็นฝ่ายเชื่อมกลับ client | มักมีปัญหากับ firewall |
| **chroot** | การกักให้ user อยู่แค่ folder ของตัวเอง | ป้องกันเข้าถึง folder อื่น |
| **Permission** | สิทธิ์การเข้าถึงไฟล์/โฟลเดอร์ | อ่านได้, เขียนได้, รันได้ |
| **Daemon** | โปรแกรมที่ทำงานเบื้องหลังตลอดเวลา | vsftpd ทำงานเป็น daemon |

### FTP มีกี่แบบ?

```
FTP  (ธรรมดา)    → ส่งข้อมูลเป็น "ข้อความเปล่า" (ไม่ปลอดภัย)
FTPS (FTP+SSL)   → เข้ารหัสด้วย SSL/TLS (ปลอดภัยขึ้น)
SFTP (SSH-based) → ใช้ SSH เป็นฐาน (ปลอดภัยสูงสุด แต่คนละโปรโตคอล)
```

> **สำหรับ LAN ภายในบ้าน/สำนักงาน:** FTP ธรรมดาก็เพียงพอ แต่ถ้าต้องการความปลอดภัยเพิ่ม แนะนำตั้งค่า FTPS (ในคู่มือนี้จะครอบคลุมทั้งคู่)

---

## 2. ทำไมถึงเลือก vsftpd?

**vsftpd** (Very Secure FTP Daemon) คือ FTP Server ที่ได้รับความนิยมสูงสุดบน Linux

### เปรียบเทียบตัวเลือกยอดนิยม

| ตัวเลือก | ข้อดี | ข้อเสีย | เหมาะกับ |
|---------|-------|---------|---------|
| **vsftpd** ✅ | เบา, เร็ว, ปลอดภัย, config ง่าย | ฟีเจอร์น้อยกว่าบางตัว | ใช้งานทั่วไป, web hosting |
| ProFTPD | ยืดหยุ่นสูง, config คล้าย Apache | ซับซ้อนกว่า | ต้องการ config ละเอียด |
| Pure-FTPd | ปลอดภัย, รองรับ virtual users | config ยากกว่า | องค์กรขนาดใหญ่ |

**เลือก vsftpd เพราะ:** มาพร้อม Ubuntu repository, เอกสารเยอะ, ชุมชนใหญ่ และเพียงพอสำหรับการใช้งานทั่วไปบน LAN

---

## 3. โครงสร้างระบบในคู่มือนี้

```
เครือข่าย LAN (192.168.1.0/24)
├── Router/DHCP Server (192.168.1.1)
│   ├── Ubuntu 24.04 LTS (ได้ IP จาก DHCP เช่น 192.168.1.100)
│   │   ├── vsftpd (FTP Server) → Port 21, 20, 40000-50000
│   │   └── Apache/Nginx (Web Server) → Port 80
│   └── Windows 10 (ได้ IP จาก DHCP เช่น 192.168.1.101)
│       └── FileZilla Client (FTP Client)
```

> **หมายเหตุ DHCP:** เนื่องจากทั้งคู่ใช้ DHCP (รับ IP อัตโนมัติ) IP อาจเปลี่ยนได้ทุกครั้งที่รีสตาร์ท จะมีวิธีจัดการในหัวข้อ Tips

---

## 4. ติดตั้ง vsftpd

### ขั้นตอนที่ 1: อัปเดตระบบก่อนเสมอ

```bash
sudo apt update && sudo apt upgrade -y
```

> **`sudo`** = ขอสิทธิ์ผู้ดูแลระบบ (super user do) ใช้กับคำสั่งที่ต้องการสิทธิ์พิเศษ
> **`apt`** = ตัวจัดการแพ็กเกจของ Ubuntu (เหมือน App Store)
> **`-y`** = ตอบ "Yes" อัตโนมัติ ไม่ต้องกด Enter

### ขั้นตอนที่ 2: ติดตั้ง vsftpd

```bash
sudo apt install vsftpd -y
```

### ขั้นตอนที่ 3: ตรวจสอบการติดตั้ง

```bash
vsftpd --version
```

ผลที่ได้จะประมาณ:
```
vsftpd: version 3.x.x
```

### ขั้นตอนที่ 4: ตรวจสอบสถานะ service

```bash
sudo systemctl status vsftpd
```

ผลที่ควรได้:
```
● vsftpd.service - vsftpd FTP server
     Loaded: loaded (/lib/systemd/system/vsftpd.service; enabled; ...)
     Active: active (running) since ...   ← ต้องเห็นคำว่า "active (running)"
```

### ขั้นตอนที่ 5: ตั้งให้เริ่มอัตโนมัติเมื่อเปิดเครื่อง

```bash
sudo systemctl enable vsftpd
```

---

## 5. ทำความเข้าใจไฟล์ config หลัก

ไฟล์ config หลักอยู่ที่: `/etc/vsftpd.conf`

### สำรอง config ต้นฉบับก่อนแก้ไขเสมอ!

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
```

> **เทคนิคมือใหม่:** สำรองไฟล์ config ทุกครั้งก่อนแก้ไข ถ้าพังก็ copy กลับมาได้

### ดู config ปัจจุบัน

```bash
cat /etc/vsftpd.conf
```

### โครงสร้าง config file

```
# นี่คือ comment (บรรทัดอธิบาย ไม่มีผลต่อการทำงาน)
option=value    ← รูปแบบการตั้งค่า (ไม่มีช่องว่างรอบ = )
YES/NO          ← ค่า boolean (เปิด/ปิด ฟีเจอร์)
```

---

## 6. ตั้งค่า vsftpd แบบสมบูรณ์

เปิดไฟล์ config ด้วย text editor:

```bash
sudo nano /etc/vsftpd.conf
```

> **nano** = text editor ง่าย ๆ ใน terminal
> ปุ่มควบคุม nano: `Ctrl+O` = บันทึก | `Ctrl+X` = ออก | `Ctrl+W` = ค้นหา

### ลบเนื้อหาเดิมและใส่ config ใหม่ทั้งหมด:

```bash
sudo nano /etc/vsftpd.conf
```

แทนที่ด้วยเนื้อหาต่อไปนี้:

```ini
# ============================================================
# vsftpd Configuration - Ubuntu 24.04 LTS
# สำหรับใช้งาน LAN ร่วมกับ Web Server
# ============================================================

# --- การทำงานพื้นฐาน ---
# ให้รับการเชื่อมต่อจาก internet/network (ถ้าปิดจะทำงานไม่ได้)
listen=YES

# ปิด IPv6 (เพราะเปิด listen=YES แล้ว ใช้ได้อย่างเดียว)
listen_ipv6=NO

# ให้ anonymous user (ไม่มีชื่อ/รหัสผ่าน) เชื่อมต่อได้ไหม
anonymous_enable=NO

# ให้ local user (user ของ Ubuntu) login ได้
local_enable=YES

# ให้ user อัปโหลดไฟล์ได้
write_enable=YES

# สิทธิ์ default ของไฟล์ที่อัปโหลด (022 = rw-r--r--)
# local_umask คือ "บิตที่จะถูกลบออก" จากสิทธิ์ 777
# 022 → ไฟล์ที่อัปโหลดจะมีสิทธิ์ 755 (rwxr-xr-x)
local_umask=022

# แสดงข้อความต้อนรับเมื่อ login
ftpd_banner=Welcome to FTP Server - Authorized Access Only

# --- Passive Mode (สำคัญมากสำหรับ LAN/Firewall) ---
# เปิด Passive Mode (แนะนำเสมอ)
pasv_enable=YES

# Range ของ port ที่ใช้สำหรับ data transfer ใน passive mode
# ต้องเปิด firewall สำหรับ port range นี้ด้วย
pasv_min_port=40000
pasv_max_port=50000

# ใส่ IP ของเซิร์ฟเวอร์ Ubuntu (ถ้าใช้ DHCP ให้ใส่ค่าจริงที่ได้)
# คอมเมนต์บรรทัดนี้ไว้ก่อน ถ้าใช้ LAN ปกติมักไม่จำเป็น
# pasv_address=192.168.1.100

# --- chroot (กักผู้ใช้ในโฟลเดอร์ตัวเอง) ---
# บังคับให้ local user อยู่แค่ใน home directory ของตัวเอง
# ป้องกันการเข้าถึงโฟลเดอร์ระบบ เช่น /etc, /var
chroot_local_user=YES

# อนุญาตให้ user ที่อยู่ในรายชื่อ chroot_list_file ข้าม chroot ได้
chroot_list_enable=NO

# allow_writeable_chroot: จำเป็นต้องเปิดถ้าใช้ chroot
# มิฉะนั้นจะ error "500 OOPS: vsftpd: refusing to run with writable root"
allow_writeable_chroot=YES

# --- ความปลอดภัย ---
# ป้องกัน symlink attack
secure_chroot_dir=/var/run/vsftpd/empty

# กำหนดให้ใช้ PAM (Pluggable Authentication Modules) สำหรับ authentication
pam_service_name=vsftpd

# ป้องกัน user เดาพอร์ตจากข้อความ error
ls_recurse_enable=NO

# จำกัดเวลา idle (นาที) ก่อน disconnect อัตโนมัติ
idle_session_timeout=300

# จำกัดเวลา transfer (วินาที) ถ้าไม่มีกิจกรรม
data_connection_timeout=120

# จำนวน connection สูงสุดทั้งหมด (0 = ไม่จำกัด)
max_clients=20

# จำนวน connection สูงสุดต่อ IP
max_per_ip=5

# --- Log ---
# บันทึก log การเชื่อมต่อและการส่งไฟล์
xferlog_enable=YES
xferlog_std_format=YES

# ไฟล์ log (บันทึกทุกการ upload/download)
xferlog_file=/var/log/vsftpd.log

# บันทึก log เพิ่มเติม (แนะนำสำหรับ debug)
dual_log_enable=YES
vsftpd_log_file=/var/log/vsftpd_full.log

# --- User List (whitelist/blacklist) ---
# เปิดใช้งาน user list
userlist_enable=YES

# ไฟล์รายชื่อ user
userlist_file=/etc/vsftpd.userlist

# userlist_deny=NO → รายชื่อใน list คือ "อนุญาต" (whitelist)
# userlist_deny=YES → รายชื่อใน list คือ "ห้าม" (blacklist)
userlist_deny=NO

# --- โฟลเดอร์ Upload ---
# ตำแหน่งที่ upload ไฟล์ได้
local_root=/home/$USER/ftp
```

บันทึกด้วย `Ctrl+O` แล้วออกด้วย `Ctrl+X`

---

## 7. การจัดการ User และสิทธิ์ (Permission)

### 7.1 สร้าง FTP User สำหรับสมาชิก

#### สร้าง user ใหม่ (ไม่มี login shell เพื่อความปลอดภัย)

```bash
# สร้าง user ชื่อ ftpuser1 (เปลี่ยนชื่อตามต้องการ)
sudo adduser ftpuser1
```

ระบบจะถามรหัสผ่านและข้อมูลอื่น ๆ กรอกตามต้องการ

#### สร้างโฟลเดอร์ FTP สำหรับ user

```bash
# สร้างโฟลเดอร์ ftp ใน home directory ของ user
sudo mkdir -p /home/ftpuser1/ftp/upload
sudo mkdir -p /home/ftpuser1/ftp/download

# กำหนดเจ้าของโฟลเดอร์
sudo chown -R ftpuser1:ftpuser1 /home/ftpuser1/ftp
sudo chmod 755 /home/ftpuser1/ftp
```

> **chown** = change owner (เปลี่ยนเจ้าของ)
> **chmod** = change mode (เปลี่ยนสิทธิ์)

### 7.2 ทำความเข้าใจ Permission (สิทธิ์)

```
chmod 755 → rwxr-xr-x
      │││
      ││└─ others (คนอื่น):  r-x = อ่านและรันได้
      │└── group (กลุ่ม):   r-x = อ่านและรันได้
      └─── owner (เจ้าของ): rwx = อ่าน, เขียน, รันได้

r = read    (4) = อ่านได้
w = write   (2) = เขียน/แก้ไขได้
x = execute (1) = รัน/เปิดโฟลเดอร์ได้
```

| สิทธิ์ | ตัวเลข | ความหมาย | ใช้กับ |
|-------|-------|---------|-------|
| rwxrwxrwx | 777 | ทุกคนทำได้ทุกอย่าง | ⚠️ อันตราย! |
| rwxr-xr-x | 755 | เจ้าของทำทุกอย่าง คนอื่นอ่าน/รันได้ | โฟลเดอร์ทั่วไป |
| rw-r--r-- | 644 | เจ้าของอ่าน/เขียน คนอื่นอ่านได้ | ไฟล์ทั่วไป |
| rw------- | 600 | เจ้าของอ่าน/เขียนเท่านั้น | ไฟล์ส่วนตัว |

### 7.3 เพิ่ม User เข้า Whitelist

```bash
# เปิดไฟล์ userlist
sudo nano /etc/vsftpd.userlist
```

เพิ่มชื่อ user แต่ละคนในบรรทัดใหม่:
```
ftpuser1
ftpuser2
webadmin
```

> **หมายเหตุ:** อย่าใส่ user `root` ลงใน whitelist เด็ดขาด!

### 7.4 สร้าง User สำหรับ Web Server (บริการคู่กัน)

```bash
# สร้าง user สำหรับ web developer
sudo adduser webdev

# เพิ่มเข้ากลุ่ม www-data (กลุ่มของ Apache/Nginx)
sudo usermod -aG www-data webdev

# สร้าง FTP root ของ webdev ชี้ไปที่ web root
sudo mkdir -p /home/webdev/ftp
sudo ln -s /var/www/html /home/webdev/ftp/www  # symlink ไปยัง web folder
```

### 7.5 ปิด Shell Access ให้ FTP User (ความปลอดภัย)

```bash
# เปลี่ยน shell ของ ftpuser1 เป็น /sbin/nologin
# ทำให้ login ผ่าน SSH ไม่ได้ แต่ FTP ยังทำงานได้
sudo usermod --shell /bin/false ftpuser1
```

ตรวจสอบ:
```bash
grep ftpuser1 /etc/passwd
# ผลที่ได้: ftpuser1:x:1001:1001::/home/ftpuser1:/bin/false
```

---

## 8. การตั้งค่าความปลอดภัย

### 8.1 เปิดใช้ TLS/SSL (FTPS) — แนะนำ

#### สร้าง SSL Certificate

```bash
# สร้าง self-signed certificate (ใช้ได้สำหรับ LAN)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/vsftpd.key \
  -out /etc/ssl/certs/vsftpd.crt
```

ระบบจะถามข้อมูล กรอกหรือกด Enter ข้ามได้:
```
Country Name: TH
State: Bangkok
Locality: Bangkok
Organization: My FTP Server
Organizational Unit: IT
Common Name: 192.168.1.100   ← ใส่ IP ของ Ubuntu server
Email: admin@example.com
```

#### เพิ่มการตั้งค่า SSL ใน vsftpd.conf

```bash
sudo nano /etc/vsftpd.conf
```

เพิ่มบรรทัดเหล่านี้ต่อท้ายไฟล์:

```ini
# --- SSL/TLS (FTPS) ---
ssl_enable=YES

# บังคับให้ใช้ TLS (ไม่ยอมรับ plain FTP)
# ตั้งเป็น NO ถ้าต้องการ compatibility กับ client เก่า
force_local_data_ssl=YES
force_local_logins_ssl=YES

# ใช้ TLS เวอร์ชันล่าสุด
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO

# ที่อยู่ไฟล์ certificate
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key

# อนุญาต cipher ที่ปลอดภัย
ssl_ciphers=HIGH
```

### 8.2 จำกัด Login ผิดพลาด (Brute Force Protection)

```bash
# ติดตั้ง fail2ban
sudo apt install fail2ban -y

# สร้าง config สำหรับ vsftpd
sudo nano /etc/fail2ban/jail.local
```

เพิ่มเนื้อหา:

```ini
[vsftpd]
enabled  = true
port     = ftp,ftp-data,ftps,ftps-data
logpath  = /var/log/vsftpd.log
maxretry = 5
bantime  = 3600
findtime = 600
```

> **fail2ban** = โปรแกรมที่คอย monitor log และ ban IP ที่ login ผิดเกินกำหนด
> **maxretry=5** = ผิด 5 ครั้งใน findtime วินาที จะถูก ban
> **bantime=3600** = โดน ban 1 ชั่วโมง (3600 วินาที)

```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

### 8.3 Config สิทธิ์โฟลเดอร์แบบละเอียด

```bash
# โฟลเดอร์อัปโหลดของสมาชิก (เขียนได้)
sudo chmod 775 /home/ftpuser1/ftp/upload

# โฟลเดอร์ดาวน์โหลด (อ่านอย่างเดียว)
sudo chmod 755 /home/ftpuser1/ftp/download

# กำหนดให้ไฟล์ใหม่ที่อัปโหลดเป็นของ ftpuser1
sudo chown ftpuser1:ftpuser1 /home/ftpuser1/ftp/upload
```

---

## 9. เปิด Firewall สำหรับ FTP

Ubuntu 24.04 ใช้ **UFW** (Uncomplicated Firewall) เป็น firewall หลัก

### ตรวจสอบ firewall ปัจจุบัน

```bash
sudo ufw status
```

### เปิด Port ที่จำเป็น

```bash
# Port 21 = FTP control port (รับ login และคำสั่ง)
sudo ufw allow 21/tcp

# Port 20 = FTP data port (active mode)
sudo ufw allow 20/tcp

# Passive port range (ต้องตรงกับ pasv_min_port/pasv_max_port ใน config)
sudo ufw allow 40000:50000/tcp

# ถ้าเปิด web server ด้วย
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
```

### เปิดใช้ UFW (ถ้ายังไม่ได้เปิด)

```bash
sudo ufw enable
sudo ufw status verbose
```

ผลที่ควรได้:
```
Status: active

To                         Action      From
--                         ------      ----
21/tcp                     ALLOW IN    Anywhere
20/tcp                     ALLOW IN    Anywhere
40000:50000/tcp            ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
```

---

## 10. ใช้งานร่วมกับ Web Server (Apache/Nginx)

### สถานการณ์: ให้ webdev upload ไฟล์เว็บไปยัง /var/www/html ผ่าน FTP

#### 10.1 ติดตั้ง Apache (ถ้ายังไม่มี)

```bash
sudo apt install apache2 -y
sudo systemctl enable apache2
```

#### 10.2 ตั้งค่าสิทธิ์ Web Root

```bash
# กำหนด group เป็น www-data สำหรับ /var/www/html
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 775 /var/www/html

# setgid bit: ไฟล์ใหม่ในโฟลเดอร์นี้จะสืบทอด group www-data
sudo chmod g+s /var/www/html
```

#### 10.3 สร้าง FTP User สำหรับ Web Developer

```bash
# สร้าง user webdev และเพิ่มเข้ากลุ่ม www-data
sudo adduser webdev
sudo usermod -aG www-data webdev

# สร้างโครงสร้างโฟลเดอร์
sudo mkdir -p /home/webdev/ftp
sudo chown webdev:webdev /home/webdev/ftp

# สร้าง symlink จาก ftp folder ไปยัง web root
sudo ln -s /var/www/html /home/webdev/ftp/html
```

> **symlink** = ทางลัด (symbolic link) เหมือน shortcut ใน Windows
> webdev จะเห็น folder ชื่อ `html` ใน FTP ซึ่งจริง ๆ คือ `/var/www/html`

#### 10.4 แก้ vsftpd config ให้รองรับ symlink

```bash
sudo nano /etc/vsftpd.conf
```

เพิ่ม:
```ini
# อนุญาตให้ follow symlink
follow_symlinks=YES
```

#### 10.5 รีสตาร์ท service

```bash
sudo systemctl restart vsftpd
sudo systemctl restart apache2
```

#### 10.6 ทดสอบ Web Server

เปิด browser บน Windows แล้วพิมพ์ IP ของ Ubuntu:
```
http://192.168.1.100
```
ถ้าเห็นหน้า Apache default = ทำงานปกติ

---

## 11. การใช้งาน FileZilla Client บน Windows 10

### 11.1 ดาวน์โหลดและติดตั้ง FileZilla

1. ไปที่ https://filezilla-project.org/download.php
2. ดาวน์โหลด **FileZilla Client** (ไม่ใช่ Server)
3. ติดตั้งตามปกติ

### 11.2 หา IP ของ Ubuntu Server

บน Ubuntu terminal ให้รัน:

```bash
ip addr show | grep "inet " | grep -v "127.0.0.1"
```

ผลที่ได้จะประมาณ:
```
inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic enp3s0
```
**IP ของ Ubuntu = 192.168.1.100**

### 11.3 เชื่อมต่อด้วย FileZilla (วิธีง่าย - Quickconnect)

```
Host:     192.168.1.100  (IP ของ Ubuntu)
Username: ftpuser1
Password: (รหัสผ่านที่ตั้งไว้)
Port:     21
```

คลิก **Quickconnect** แล้วรอ

### 11.4 เชื่อมต่อด้วย FileZilla (วิธีบันทึก - Site Manager)

1. เปิด **File → Site Manager** (หรือกด `Ctrl+S`)
2. คลิก **New Site** ตั้งชื่อเช่น "Ubuntu FTP"
3. ตั้งค่า:

```
Protocol:    FTP - File Transfer Protocol
             (ถ้าตั้ง SSL ไว้ เลือก "FTP over TLS (FTPS)" แทน)
Host:        192.168.1.100
Port:        21
Encryption:  Use explicit FTP over TLS if available
             (หรือ Only use plain FTP ถ้ายังไม่ได้ตั้ง SSL)
Logon Type:  Normal
User:        ftpuser1
Password:    (รหัสผ่าน)
```

4. คลิก **Transfer Settings** → เลือก **Passive** สำหรับ Transfer Mode
5. คลิก **Connect**

### 11.5 ทำความเข้าใจ UI ของ FileZilla

```
┌─────────────────────────────────────────────────────────┐
│ [Toolbar] Host: ___ User: ___ Pass: ___ Port: ___ [OK] │
├────────────────────────┬────────────────────────────────┤
│   Local (Windows)      │   Remote (Ubuntu FTP Server)  │
│   คอมของคุณ            │   Server                      │
│                        │                               │
│ C:\Users\YourName\     │ /home/ftpuser1/ftp/           │
│  📁 Documents          │  📁 upload                    │
│  📁 Downloads          │  📁 download                  │
│  📄 file.txt           │  📄 readme.txt                │
├────────────────────────┴────────────────────────────────┤
│ [Queue] - รายการไฟล์ที่กำลัง transfer                   │
└─────────────────────────────────────────────────────────┘
```

### 11.6 การใช้งานพื้นฐาน FileZilla

| การกระทำ | วิธีทำ |
|---------|-------|
| **Upload** (Windows → Server) | Drag ไฟล์จากซ้าย → ขวา หรือคลิกขวา → Upload |
| **Download** (Server → Windows) | Drag ไฟล์จากขวา → ซ้าย หรือคลิกขวา → Download |
| **สร้างโฟลเดอร์** | คลิกขวาในพื้นที่ว่าง → Create directory |
| **ลบไฟล์** | เลือกไฟล์แล้วกด Delete หรือคลิกขวา → Delete |
| **เปลี่ยนชื่อ** | คลิกขวา → Rename |
| **ดู Hidden Files** | Server → Force showing hidden files |

### 11.7 ถ้าเจอ Certificate Warning (กรณีใช้ FTPS)

FileZilla จะแสดงหน้าต่าง:
```
"The server's certificate is unknown. Please carefully examine the certificate..."
```

เนื่องจากใช้ self-signed certificate ให้:
1. ตรวจสอบว่า IP ตรงกับ Ubuntu server ของเรา
2. ติ๊กถูก **"Always trust certificate in future sessions"**
3. คลิก **OK**

---

## 12. คำสั่ง vsftpd ที่ใช้บ่อย

### จัดการ Service

```bash
# ดูสถานะ
sudo systemctl status vsftpd

# เริ่มต้น
sudo systemctl start vsftpd

# หยุด
sudo systemctl stop vsftpd

# รีสตาร์ท (ใช้หลังแก้ config)
sudo systemctl restart vsftpd

# Reload config (ไม่ต้องหยุด service)
sudo systemctl reload vsftpd

# ตั้งให้เริ่มอัตโนมัติเมื่อเปิดเครื่อง
sudo systemctl enable vsftpd

# ปิดการเริ่มอัตโนมัติ
sudo systemctl disable vsftpd
```

### จัดการ User

```bash
# ดูรายชื่อ user ทั้งหมด
cat /etc/passwd | grep -v nologin | grep -v false

# เปลี่ยนรหัสผ่าน user
sudo passwd ftpuser1

# ล็อก user (ห้าม login ชั่วคราว)
sudo usermod -L ftpuser1

# ปลดล็อก user
sudo usermod -U ftpuser1

# ลบ user
sudo deluser ftpuser1

# ลบ user พร้อม home directory
sudo deluser --remove-home ftpuser1
```

### ดูการเชื่อมต่อปัจจุบัน

```bash
# ดูว่ามีใครเชื่อมต่ออยู่บ้าง
sudo netstat -an | grep :21

# หรือใช้ ss (ใหม่กว่า)
sudo ss -an | grep :21
```

---

## 13. การดู Log และแก้ปัญหา

### ดู Log แบบ Real-time

```bash
# ดู log สด ๆ (กด Ctrl+C เพื่อหยุด)
sudo tail -f /var/log/vsftpd.log

# ดู log ละเอียด
sudo tail -f /var/log/vsftpd_full.log

# ดู system log ของ vsftpd
sudo journalctl -u vsftpd -f
```

### ปัญหาที่พบบ่อยและวิธีแก้

#### ❌ Error: "500 OOPS: vsftpd: refusing to run with writable root inside chroot"

```bash
# แก้โดยเพิ่มบรรทัดนี้ใน vsftpd.conf
allow_writeable_chroot=YES
sudo systemctl restart vsftpd
```

#### ❌ Error: "530 Login incorrect"

```bash
# ตรวจสอบรหัสผ่าน
sudo passwd ftpuser1

# ตรวจสอบ userlist
cat /etc/vsftpd.userlist

# ตรวจสอบว่า user ไม่ถูก lock
sudo passwd -S ftpuser1
# P = ปกติ, L = ถูก lock
```

#### ❌ ต่อได้แต่ list ไฟล์ไม่ได้ (Connection timeout)

```bash
# ปัญหา Passive Mode - ตรวจสอบ firewall
sudo ufw status | grep 40000

# เพิ่ม passive port range
sudo ufw allow 40000:50000/tcp
sudo systemctl restart vsftpd
```

#### ❌ FileZilla ขึ้น "ECONNREFUSED - Connection refused"

```bash
# ตรวจสอบว่า vsftpd กำลังรันอยู่
sudo systemctl status vsftpd

# ตรวจสอบว่า port 21 เปิดอยู่
sudo ss -tlnp | grep :21

# ตรวจสอบ firewall
sudo ufw status | grep 21
```

#### ❌ ดู IP ของ Ubuntu ไม่เจอเพราะ DHCP เปลี่ยน

```bash
# ดู IP ปัจจุบัน
hostname -I

# หรือ
ip route get 1.1.1.1 | awk '{print $7}'
```

---

## 14. ทริปและเทคนิคสำหรับมือใหม่

### 🔧 ทริปที่ 1: แก้ปัญหา DHCP IP เปลี่ยน

เนื่องจาก Ubuntu ใช้ DHCP (IP เปลี่ยนได้) ให้แก้ด้วยวิธีใดวิธีหนึ่ง:

**วิธีที่ 1: ผูก IP กับ Router (แนะนำ)**
- เข้า Router admin (ปกติที่ 192.168.1.1)
- ตั้ง "DHCP Reservation" หรือ "Static DHCP" ผูก MAC Address ของ Ubuntu กับ IP คงที่

**ดู MAC Address ของ Ubuntu:**
```bash
ip link show | grep ether
# ผล: link/ether aa:bb:cc:dd:ee:ff
```

**วิธีที่ 2: ตั้ง Static IP บน Ubuntu**
```bash
# Ubuntu 24.04 ใช้ NetworkManager
nmcli con show                     # ดูชื่อ connection
nmcli con mod "Wired connection 1" \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  ipv4.method manual
nmcli con up "Wired connection 1"
```

### 🔧 ทริปที่ 2: สร้าง Script สำรอง config อัตโนมัติ

```bash
sudo nano /usr/local/bin/backup-vsftpd.sh
```

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/root/vsftpd-backup"
mkdir -p $BACKUP_DIR
cp /etc/vsftpd.conf "$BACKUP_DIR/vsftpd.conf.$DATE"
echo "Backup saved: $BACKUP_DIR/vsftpd.conf.$DATE"
```

```bash
sudo chmod +x /usr/local/bin/backup-vsftpd.sh
sudo /usr/local/bin/backup-vsftpd.sh
```

### 🔧 ทริปที่ 3: ทดสอบ FTP จาก Terminal (Command Line FTP Client)

```bash
# ติดตั้ง ftp client
sudo apt install ftp -y

# ทดสอบ login
ftp localhost
# พิมพ์ username และ password
# พิมพ์ ls เพื่อดูไฟล์
# พิมพ์ bye เพื่อออก
```

### 🔧 ทริปที่ 4: ดูว่า Port เปิดอยู่หรือเปล่า

```bash
# จาก Ubuntu (ดูตัวเอง)
sudo ss -tlnp | grep vsftpd

# จาก Windows Command Prompt (ทดสอบการเชื่อมต่อ)
telnet 192.168.1.100 21
# ถ้าเชื่อมได้จะเห็น: 220 Welcome to FTP Server
# กด Ctrl+] แล้วพิมพ์ quit เพื่อออก
```

### 🔧 ทริปที่ 5: ตั้งข้อความต้อนรับหลาย userที่

```bash
# สร้างไฟล์ message สำหรับแต่ละโฟลเดอร์
echo "ยินดีต้อนรับสู่ FTP Upload Zone" | sudo tee /home/ftpuser1/ftp/upload/.ftpmessage

# เพิ่มใน vsftpd.conf
# dirmessage_enable=YES
```

### 🔧 ทริปที่ 6: Bandwidth Limiting (จำกัดความเร็ว)

```bash
sudo nano /etc/vsftpd.conf
```

เพิ่ม:
```ini
# จำกัดความเร็ว upload (bytes/second)
# 1048576 = 1MB/s
local_max_rate=1048576
```

### 🔧 ทริปที่ 7: Virtual Users (สำหรับผู้ใช้หลายคนโดยไม่ต้องสร้าง Linux user)

สำหรับกรณีที่ต้องการ user FTP หลายคนแต่ไม่อยากสร้าง Linux system user:

```bash
# ติดตั้ง db-util
sudo apt install db-util -y

# สร้างไฟล์ virtual users
sudo nano /etc/vsftpd/virtual_users.txt
```

```
# รูปแบบ: username บรรทัดคี่, password บรรทัดคู่
member1
password123
member2
mypassword
```

```bash
# แปลงเป็น database
sudo db_load -T -t hash -f /etc/vsftpd/virtual_users.txt /etc/vsftpd/virtual_users.db
sudo chmod 600 /etc/vsftpd/virtual_users.db
```

### 🔧 ทริปที่ 8: Monitor เรียลไทม์ด้วย watch

```bash
# ดู connections แบบ real-time ทุก 2 วินาที
watch -n 2 'sudo ss -an | grep :21'

# ดู log แบบ real-time
sudo tail -f /var/log/vsftpd.log | grep -v "^$"
```

---

## 15. ตารางสรุปคำศัพท์สำคัญ

| คำศัพท์ | ภาษาไทย | อธิบาย |
|---------|---------|-------|
| **FTP** | โปรโตคอลถ่ายโอนไฟล์ | มาตรฐานการรับ-ส่งไฟล์ในเครือข่าย |
| **vsftpd** | daemon FTP ที่ปลอดภัยมาก | FTP Server software บน Linux |
| **daemon** | โปรแกรมพื้นหลัง | ทำงานเบื้องหลังตลอดเวลาโดยไม่ต้อง login |
| **port** | พอร์ต/ช่องสื่อสาร | ช่องทางสำหรับโปรแกรมต่าง ๆ (FTP=21, Web=80) |
| **passive mode** | โหมดแบบ passive | client เป็นคนเปิด data connection เอง |
| **active mode** | โหมดแบบ active | server เป็นคนเชื่อมกลับ client |
| **chroot** | จำกัดพื้นที่ | กักให้ user อยู่แค่โฟลเดอร์ของตัวเอง |
| **chmod** | เปลี่ยนสิทธิ์ | กำหนดว่าใครอ่าน/เขียน/รันไฟล์ได้บ้าง |
| **chown** | เปลี่ยนเจ้าของ | กำหนด user/group เจ้าของไฟล์ |
| **UFW** | Firewall ของ Ubuntu | ตัวจัดการ firewall แบบใช้ง่าย |
| **SSL/TLS** | การเข้ารหัส | ทำให้ข้อมูลที่ส่งไม่สามารถดักอ่านได้ |
| **FTPS** | FTP + SSL | FTP ที่เข้ารหัสด้วย SSL |
| **DHCP** | แจก IP อัตโนมัติ | Router แจก IP ให้เครื่องในวงอัตโนมัติ |
| **LAN** | เครือข่ายท้องถิ่น | เครือข่ายภายในบ้าน/สำนักงาน |
| **symlink** | ทางลัด | ชี้ไปยังไฟล์/โฟลเดอร์อื่น (คล้าย shortcut) |
| **PAM** | ระบบ authentication | ตรวจสอบรหัสผ่านของ Linux |
| **fail2ban** | ป้องกัน brute force | ban IP ที่ login ผิดเกินกำหนด |
| **umask** | mask สิทธิ์ default | กำหนดสิทธิ์เริ่มต้นของไฟล์ที่สร้างใหม่ |
| **www-data** | กลุ่มของ web server | Apache/Nginx ใช้ group นี้ในการทำงาน |

---

## สรุปขั้นตอนทั้งหมดแบบย่อ

```bash
# 1. ติดตั้ง
sudo apt install vsftpd -y

# 2. สำรอง config
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup

# 3. แก้ไข config
sudo nano /etc/vsftpd.conf

# 4. สร้าง SSL cert (ถ้าต้องการ FTPS)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/vsftpd.key \
  -out /etc/ssl/certs/vsftpd.crt

# 5. สร้าง user
sudo adduser ftpuser1
sudo mkdir -p /home/ftpuser1/ftp/upload
sudo chown -R ftpuser1:ftpuser1 /home/ftpuser1/ftp

# 6. เพิ่มใน userlist
echo "ftpuser1" | sudo tee -a /etc/vsftpd.userlist

# 7. เปิด firewall
sudo ufw allow 21/tcp
sudo ufw allow 40000:50000/tcp

# 8. รีสตาร์ท
sudo systemctl restart vsftpd
sudo systemctl enable vsftpd

# 9. ตรวจสอบ
sudo systemctl status vsftpd
```

---

> **หมายเหตุสุดท้าย:** ทุกครั้งที่แก้ไข `/etc/vsftpd.conf` ต้องรัน `sudo systemctl restart vsftpd` เพื่อให้การเปลี่ยนแปลงมีผล และควรตรวจสอบ log ที่ `/var/log/vsftpd.log` เสมอเมื่อมีปัญหา

*คู่มือนี้ใช้ vsftpd บน Ubuntu 24.04 LTS | อัปเดต 2026*
