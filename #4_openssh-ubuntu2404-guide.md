# 🖥️ คู่มือ OpenSSH Server บน Ubuntu 24.04 LTS + PuTTY บน Windows 10
### สำหรับมือใหม่จริงๆ — ครบทุกฟังก์ชัน เทคนิค และคำอธิบาย

---

## 📚 สารบัญ

1. [SSH คืออะไร? (อธิบายให้เข้าใจง่าย)](#1-ssh-คืออะไร)
2. [คำศัพท์สำคัญที่ต้องรู้ก่อน](#2-คำศัพท์สำคัญ)
3. [โครงสร้างระบบในบทความนี้](#3-โครงสร้างระบบ)
4. [ติดตั้ง OpenSSH Server บน Ubuntu 24.04](#4-ติดตั้ง-openssh-server)
5. [ตรวจสอบและจัดการ Service](#5-ตรวจสอบและจัดการ-service)
6. [ค้นหา IP Address ของ Ubuntu (Server)](#6-ค้นหา-ip-address-ของ-ubuntu)
7. [ติดตั้งและใช้งาน PuTTY บน Windows 10](#7-putty-บน-windows-10)
8. [การตั้งค่า SSH Config (/etc/ssh/sshd_config)](#8-การตั้งค่า-sshd_config)
9. [Key-Based Authentication (เข้าสู่ระบบด้วย Key แทน Password)](#9-key-based-authentication)
10. [Port Forwarding / Tunneling](#10-port-forwarding--tunneling)
11. [SCP และ SFTP — โอนไฟล์ผ่าน SSH](#11-scp-และ-sftp)
12. [SSH Config File (ฝั่ง Client)](#12-ssh-config-file-ฝั่ง-client)
13. [การจัดการ User และสิทธิ์](#13-การจัดการ-user-และสิทธิ์)
14. [Firewall (UFW) กับ SSH](#14-firewall-ufw-กับ-ssh)
15. [ความปลอดภัย — Best Practices](#15-ความปลอดภัย--best-practices)
16. [Troubleshooting — แก้ปัญหาที่พบบ่อย](#16-troubleshooting)
17. [ทริคและเทคนิคขั้นสูง](#17-ทริคและเทคนิคขั้นสูง)
18. [สรุปคำสั่งที่ใช้บ่อย (Cheat Sheet)](#18-cheat-sheet)

---

## 1. SSH คืออะไร?

**SSH** ย่อมาจาก **Secure Shell** — แปลตรงๆ คือ "เชลล์ที่ปลอดภัย"

### อธิบายแบบเข้าใจง่าย

> จินตนาการว่าคุณต้องการ **นั่งหน้าจอของคอมพิวเตอร์อีกเครื่องหนึ่ง** โดยไม่ต้องเดินไปนั่งหน้าเครื่องนั้นจริงๆ — SSH ทำแบบนั้นได้ผ่านเครือข่าย และที่สำคัญคือ **ข้อมูลทุกอย่างถูกเข้ารหัส** ไม่มีใครดักฟังได้

### ทำไมต้องใช้ SSH?

- ควบคุม Ubuntu Server จาก Windows โดยไม่ต้องนั่งหน้าจอ Ubuntu
- รัน command line ได้จากระยะไกล
- โอนไฟล์อย่างปลอดภัย
- ทำ automation และ script ต่างๆ
- ใช้ในงาน DevOps, System Admin, Developer

### SSH ต่างจาก Telnet อย่างไร?

| | Telnet (เก่า/ไม่ควรใช้) | SSH (ใช้งานปัจจุบัน) |
|---|---|---|
| การเข้ารหัส | ❌ ไม่มี (plain text) | ✅ เข้ารหัสทั้งหมด |
| ความปลอดภัย | ❌ ดักฟังได้ง่าย | ✅ ปลอดภัยสูง |
| Port | 23 | 22 |

---

## 2. คำศัพท์สำคัญ

> มือใหม่ **ต้องอ่านส่วนนี้** ก่อน เพราะคำศัพท์เหล่านี้จะปรากฏตลอดบทความ

| คำศัพท์ | อ่านว่า | ความหมาย |
|---|---|---|
| **Server** | เซิร์ฟเวอร์ | เครื่องที่ให้บริการ = Ubuntu ในบทความนี้ |
| **Client** | ไคลเอนต์ | เครื่องที่เชื่อมต่อเข้าไป = Windows 10 ในบทความนี้ |
| **OpenSSH** | โอเพ่น-เอสเอสเอช | โปรแกรม SSH ฟรีและโอเพ่นซอร์สที่ได้รับความนิยมมากที่สุด |
| **sshd** | เอสเอสเอช-ดี | SSH Daemon = โปรแกรม SSH ที่รันอยู่เบื้องหลังบน Server ตลอดเวลา |
| **Daemon** | ดีมอน | โปรแกรมที่ทำงานอยู่เบื้องหลังตลอดเวลา (คล้าย Windows Service) |
| **Port** | พอร์ต | "ช่องทาง" สำหรับรับส่งข้อมูล, SSH ใช้ Port 22 เป็นค่าเริ่มต้น |
| **IP Address** | ไอพี แอดเดรส | ที่อยู่ของเครื่องในเครือข่าย เช่น 192.168.1.100 |
| **DHCP** | ดี-เอช-ซี-พี | ระบบแจก IP อัตโนมัติ (Router บ้านส่วนใหญ่ทำหน้าที่นี้) |
| **LAN** | แลน | Local Area Network = เครือข่ายภายในบ้าน/องค์กร |
| **Authentication** | ออเทนทิเคชัน | การยืนยันตัวตน (login) |
| **Key Pair** | คีย์ แพร์ | คู่กุญแจ = Private Key + Public Key ใช้แทน Password |
| **Private Key** | ไพรเวท คีย์ | กุญแจส่วนตัว ห้ามให้ใคร เก็บไว้ที่ฝั่ง Client |
| **Public Key** | พับลิค คีย์ | กุญแจสาธารณะ แจกได้ เก็บไว้ที่ฝั่ง Server |
| **Firewall** | ไฟร์วอลล์ | ระบบป้องกันการเชื่อมต่อที่ไม่ได้รับอนุญาต |
| **UFW** | ยู-เอฟ-ดับบลิว | Uncomplicated Firewall = Firewall ของ Ubuntu ที่ใช้งานง่าย |
| **Terminal** | เทอร์มินัล | โปรแกรมพิมพ์คำสั่ง (command line) |
| **PuTTY** | พุตตี้ | โปรแกรม SSH Client สำหรับ Windows ที่ได้รับความนิยมสูง |
| **Passphrase** | พาสเฟรส | รหัสผ่านสำหรับล็อก Private Key (ไม่ใช่ Password ของ User) |
| **Root** | รูต | ผู้ใช้ระดับสูงสุดของ Linux มีสิทธิ์ทำทุกอย่าง |
| **sudo** | ซูโด | คำสั่งที่ใช้รันคำสั่งในฐานะ root ชั่วคราว |
| **Config File** | คอนฟิก ไฟล์ | ไฟล์ตั้งค่า |
| **Tunnel** | ทันเนล | การส่งข้อมูลผ่านช่องทางที่เข้ารหัสของ SSH |
| **SFTP** | เอส-เอฟ-ที-พี | SSH File Transfer Protocol = โอนไฟล์ผ่าน SSH |
| **SCP** | เอส-ซี-พี | Secure Copy = คำสั่งคัดลอกไฟล์ผ่าน SSH |

---

## 3. โครงสร้างระบบ

```
บ้าน / ออฟฟิศ (ระบบ LAN)
┌─────────────────────────────────────────────────────┐
│                                                     │
│   [ Router/DHCP ]                                   │
│        │                                            │
│        ├─────────────────────────────────────┐      │
│        │                                     │      │
│  [ Ubuntu 24.04 Desktop ]          [ Windows 10 ]  │
│  SSH Server (sshd)                 SSH Client       │
│  IP: 192.168.1.xxx (auto DHCP)     PuTTY           │
│  Port: 22                          IP: 192.168.1.yyy│
│                                                     │
└─────────────────────────────────────────────────────┘

การเชื่อมต่อ: Windows (PuTTY) ───SSH Port 22──→ Ubuntu (sshd)
```

> ⚠️ **หมายเหตุเรื่อง DHCP:** เพราะทั้งสองเครื่องรับ IP อัตโนมัติ IP อาจเปลี่ยนได้ทุกครั้งที่รีสตาร์ท Router ในส่วนท้ายจะมีเทคนิคแก้ปัญหานี้

---

## 4. ติดตั้ง OpenSSH Server

### ขั้นตอนที่ 1 — เปิด Terminal บน Ubuntu

กด `Ctrl + Alt + T` หรือค้นหา "Terminal" ในเมนู Applications

### ขั้นตอนที่ 2 — อัปเดตรายการ Package ก่อน

```bash
sudo apt update
```

> **อธิบาย:** `apt update` คือการดาวน์โหลดรายการโปรแกรมใหม่ล่าสุดจาก Internet (ไม่ได้ติดตั้งอะไร แค่อัปเดตรายการ) ใช้ `sudo` เพราะต้องการสิทธิ์ root

### ขั้นตอนที่ 3 — ติดตั้ง OpenSSH Server

```bash
sudo apt install openssh-server -y
```

> **อธิบาย:**
> - `apt install` = คำสั่งติดตั้งโปรแกรม
> - `openssh-server` = ชื่อโปรแกรมที่ต้องการ
> - `-y` = ตอบ "yes" อัตโนมัติ ไม่ต้องกด Enter ยืนยันอีกครั้ง

### ขั้นตอนที่ 4 — ตรวจสอบว่าติดตั้งสำเร็จ

```bash
ssh -V
```

ผลลัพธ์ที่ได้ควรเป็นประมาณนี้:

```
OpenSSH_9.6p1 Ubuntu-3ubuntu13, OpenSSL 3.0.13 30 Jan 2024
```

---

## 5. ตรวจสอบและจัดการ Service

> **Service** คือโปรแกรมที่รันอยู่เบื้องหลัง Ubuntu ใช้ระบบที่ชื่อ **systemd** ในการจัดการ

### ตรวจสอบสถานะ SSH Service

```bash
sudo systemctl status ssh
```

ผลลัพธ์ที่ดีควรเห็น **active (running)** เป็นสีเขียว:

```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; enabled)
     Active: active (running) since ...
```

### คำสั่งจัดการ Service ทั้งหมด

```bash
# เริ่ม SSH Service (ถ้ายังไม่ทำงาน)
sudo systemctl start ssh

# หยุด SSH Service
sudo systemctl stop ssh

# รีสตาร์ท SSH Service (ใช้บ่อยหลังแก้ config)
sudo systemctl restart ssh

# โหลด config ใหม่โดยไม่ตัดการเชื่อมต่อที่มีอยู่
sudo systemctl reload ssh

# ตั้งให้ SSH เริ่มอัตโนมัติตอนบูตเครื่อง
sudo systemctl enable ssh

# ปิดไม่ให้ SSH เริ่มอัตโนมัติตอนบูตเครื่อง
sudo systemctl disable ssh

# ตรวจสอบว่าเปิด auto-start ไว้หรือไม่
sudo systemctl is-enabled ssh
```

> ✅ **คำแนะนำ:** ให้รัน `sudo systemctl enable ssh` เสมอ เพื่อให้ SSH เริ่มทำงานอัตโนมัติทุกครั้งที่เปิดเครื่อง

---

## 6. ค้นหา IP Address ของ Ubuntu

เราต้องรู้ IP ของ Ubuntu เพื่อให้ PuTTY เชื่อมต่อมาได้

### วิธีที่ 1 — คำสั่ง `ip addr`

```bash
ip addr show
```

หรือสั้นกว่า:

```bash
ip a
```

ผลลัพธ์:

```
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 192.168.1.105/24 brd 192.168.1.255 scope global dynamic
```

> **อธิบาย:** มองหา `inet` ที่ขึ้นต้นด้วย `192.168.x.x` นั่นคือ IP ของคุณ
> ส่วน `/24` คือ Subnet Mask (ไม่ต้องสนใจตอนนี้)

### วิธีที่ 2 — คำสั่งสั้นๆ กรองเฉพาะ IP

```bash
hostname -I
```

ผลลัพธ์:

```
192.168.1.105
```

### วิธีที่ 3 — ดูจาก GUI

ไปที่ **Settings → Network** แล้วคลิกที่การเชื่อมต่อที่ใช้งาน จะเห็น IP Address

---

## 7. PuTTY บน Windows 10

### ดาวน์โหลด PuTTY

ดาวน์โหลดจากเว็บไซต์ทางการ: **https://www.putty.org**

เลือกไฟล์ `putty-64bit-X.XX-installer.msi` (สำหรับ Windows 64-bit)

### การเชื่อมต่อครั้งแรกด้วย PuTTY

1. เปิดโปรแกรม **PuTTY**
2. ในช่อง **Host Name (or IP address)** พิมพ์ IP ของ Ubuntu เช่น `192.168.1.105`
3. **Port:** `22` (ค่าเริ่มต้น ไม่ต้องเปลี่ยน)
4. **Connection type:** เลือก `SSH`
5. (ไม่บังคับ) ในช่อง **Saved Sessions** พิมพ์ชื่อ เช่น `Ubuntu-Home` แล้วกด **Save** เพื่อบันทึก
6. กด **Open**

### Security Alert (ครั้งแรก)

ครั้งแรกที่เชื่อมต่อ จะมี popup แจ้งเตือน:

```
The server's host key is not cached in the registry...
```

> **อธิบาย:** PuTTY กำลังบอกว่า "ฉันไม่เคยเห็นเครื่องนี้มาก่อน แน่ใจหรือว่าจะเชื่อมต่อ?"
> ถ้าอยู่ในเครือข่ายบ้านและรู้ว่าเป็นเครื่อง Ubuntu ของตัวเอง กด **Accept** ได้เลย

### Login

```
login as: your_ubuntu_username
your_ubuntu_username@192.168.1.105's password: (พิมพ์รหัสผ่าน)
```

> ⚠️ **สังเกต:** เวลาพิมพ์รหัสผ่านใน Terminal จะไม่เห็นตัวอักษรหรือ `***` เลย นี่คือพฤติกรรมปกติของ Linux

เมื่อ login สำเร็จ จะเห็น prompt แบบนี้:

```
username@ubuntu-hostname:~$
```

**ยินดีด้วย! คุณเชื่อมต่อ SSH สำเร็จแล้ว** 🎉

### การตั้งค่า PuTTY เพิ่มเติมที่ควรรู้

**เปลี่ยน Font ให้อ่านง่ายขึ้น:**
ไปที่ **Window → Appearance → Font** → เปลี่ยนเป็น `Consolas` หรือ `Courier New` ขนาด 12

**เพิ่มจำนวน Scrollback Lines:**
ไปที่ **Window → Lines of scrollback** → เปลี่ยนเป็น `5000`

**ตั้ง Keep-Alive (ป้องกัน Connection Timeout):**
ไปที่ **Connection → Seconds between keepalives** → ใส่ `60`

---

## 8. การตั้งค่า sshd_config

`/etc/ssh/sshd_config` คือไฟล์ตั้งค่าหลักของ SSH Server

> ⚠️ **สำคัญมาก:** ทุกครั้งก่อนแก้ไข ให้สำรองไฟล์ก่อนเสมอ!

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

### เปิดไฟล์ Config

```bash
sudo nano /etc/ssh/sshd_config
```

> **nano** คือ text editor ใน Terminal ที่ใช้งานง่ายที่สุด
> - บันทึก: กด `Ctrl + O` แล้ว Enter
> - ออก: กด `Ctrl + X`

### ตัวเลือกสำคัญใน sshd_config

```bash
# ─────────────────────────────────────────────
# Port — เปลี่ยน Port SSH (ค่าเริ่มต้นคือ 22)
# ─────────────────────────────────────────────
Port 22
# เทคนิค: เปลี่ยนเป็น Port อื่น เช่น 2222 ช่วยลด bot scan
# แต่ต้องอัปเดต Firewall และบอก PuTTY ด้วย

# ─────────────────────────────────────────────
# AddressFamily — รูปแบบ IP ที่ยอมรับ
# ─────────────────────────────────────────────
AddressFamily inet
# inet = IPv4 เท่านั้น
# inet6 = IPv6 เท่านั้น
# any = ทั้งสอง (ค่าเริ่มต้น)

# ─────────────────────────────────────────────
# ListenAddress — IP ที่ SSH จะ "รับฟัง"
# ─────────────────────────────────────────────
ListenAddress 0.0.0.0
# 0.0.0.0 = รับจากทุก interface (ค่าเริ่มต้น ใช้ได้เลย)

# ─────────────────────────────────────────────
# PermitRootLogin — อนุญาตให้ root login ไหม?
# ─────────────────────────────────────────────
PermitRootLogin no
# yes = อนุญาต (อันตราย! ไม่แนะนำ)
# no = ไม่อนุญาต (แนะนำ)
# prohibit-password = root login ได้แต่ใช้ Key เท่านั้น

# ─────────────────────────────────────────────
# PasswordAuthentication — อนุญาต login ด้วย password ไหม?
# ─────────────────────────────────────────────
PasswordAuthentication yes
# yes = ใช้ password ได้ (ง่ายแต่ปลอดภัยน้อยกว่า)
# no = ใช้ Key เท่านั้น (ปลอดภัยสูงสุด — แต่ต้องตั้ง Key ก่อน!)

# ─────────────────────────────────────────────
# PubkeyAuthentication — อนุญาต login ด้วย Key ไหม?
# ─────────────────────────────────────────────
PubkeyAuthentication yes
# ควรเปิดไว้เสมอ

# ─────────────────────────────────────────────
# AuthorizedKeysFile — ไฟล์ที่เก็บ Public Keys
# ─────────────────────────────────────────────
AuthorizedKeysFile .ssh/authorized_keys
# ค่าเริ่มต้น ไม่ต้องเปลี่ยน

# ─────────────────────────────────────────────
# MaxAuthTries — จำนวนครั้งสูงสุดที่ login ผิดได้
# ─────────────────────────────────────────────
MaxAuthTries 3
# ค่าเริ่มต้น 6, แนะนำลดเหลือ 3

# ─────────────────────────────────────────────
# MaxSessions — จำนวน session สูงสุดต่อ connection
# ─────────────────────────────────────────────
MaxSessions 10

# ─────────────────────────────────────────────
# LoginGraceTime — เวลาที่ให้ login (วินาที)
# ─────────────────────────────────────────────
LoginGraceTime 30
# ค่าเริ่มต้น 120 วินาที, ลดเหลือ 30 เพื่อความปลอดภัย

# ─────────────────────────────────────────────
# AllowUsers — อนุญาตเฉพาะ User ที่ระบุ
# ─────────────────────────────────────────────
AllowUsers myusername
# AllowUsers user1 user2 user3 (คั่นด้วย space)
# ถ้าไม่ใส่ = อนุญาตทุก user ที่มีในระบบ

# ─────────────────────────────────────────────
# DenyUsers — บล็อก User ที่ระบุ
# ─────────────────────────────────────────────
# DenyUsers baduser

# ─────────────────────────────────────────────
# X11Forwarding — ส่ง GUI ผ่าน SSH ได้ไหม?
# ─────────────────────────────────────────────
X11Forwarding yes
# ใช้สำหรับรัน GUI Application ผ่าน SSH

# ─────────────────────────────────────────────
# Banner — แสดงข้อความก่อน login
# ─────────────────────────────────────────────
# Banner /etc/issue.net
```

### หลังแก้ไข Config ต้อง Reload/Restart เสมอ

```bash
# ตรวจสอบ syntax ก่อน (ไม่ให้พัง!)
sudo sshd -t

# ถ้าไม่มี error ให้ reload
sudo systemctl reload ssh
```

> ✅ **เทคนิค:** ใช้ `sshd -t` ตรวจ syntax ก่อนทุกครั้ง จะได้ไม่ทำให้ SSH พังและ login ไม่ได้

---

## 9. Key-Based Authentication

### Key-Based Authentication คืออะไร?

แทนที่จะใช้รหัสผ่าน เราใช้ "คู่กุญแจ" ในการยืนยันตัวตน:

```
Private Key (กุญแจส่วนตัว)   ←→   Public Key (กุญแจสาธารณะ)
เก็บไว้ที่ Windows/Client         ติดตั้งไว้บน Ubuntu/Server
ห้ามให้ใครเด็ดขาด!               แจกได้ ไม่เป็นอะไร
```

> 🔐 **อุปมา:** Public Key เหมือน "กุญแจล็อก" ที่คุณแจกจ่ายได้ Private Key เหมือน "กุญแจไข" ที่มีแค่คุณคนเดียว ใครก็ตามที่มี Private Key จะ "เปิด" ล็อคที่ Public Key สร้างได้

### ขั้นตอนที่ 1 — สร้าง Key Pair บน Windows ด้วย PuTTYgen

1. เปิดโปรแกรม **PuTTYgen** (ติดมากับ PuTTY)
2. เลือกประเภท Key:
   - เลือก **EdDSA** (แนะนำ — ปลอดภัยและเร็ว) หรือ **RSA** (ค่าเริ่มต้น ใช้ได้)
   - ถ้าเลือก RSA ให้ตั้ง Bits เป็น **4096**
3. กด **Generate**
4. ขยับเมาส์วนๆ ในกล่องสีเทา เพื่อสร้างความสุ่ม
5. ใส่ **Key passphrase** (รหัสผ่านสำหรับ Key นี้ — แนะนำให้ใส่)
6. **บันทึก Private Key:** กด **Save private key** → บันทึกเป็น `id_rsa.ppk` หรือ `id_ed25519.ppk`
7. **คัดลอก Public Key:** ข้อความใน box "Public key for pasting into OpenSSH authorized_keys" — **คัดลอกไว้ก่อน**

> ⚠️ **ระวัง:** ไฟล์ `.ppk` คือ Private Key ห้ามส่งหรือแชร์ให้ใคร!

### ขั้นตอนที่ 2 — ติดตั้ง Public Key บน Ubuntu Server

เปิด Terminal บน Ubuntu แล้วรัน:

```bash
# สร้างโฟลเดอร์ .ssh (ถ้ายังไม่มี)
mkdir -p ~/.ssh

# ตั้งสิทธิ์โฟลเดอร์ (สำคัญมาก! SSH จะปฏิเสธถ้า permission ไม่ถูกต้อง)
chmod 700 ~/.ssh

# สร้างไฟล์ authorized_keys
touch ~/.ssh/authorized_keys

# ตั้งสิทธิ์ไฟล์
chmod 600 ~/.ssh/authorized_keys

# เพิ่ม Public Key เข้าไปในไฟล์
nano ~/.ssh/authorized_keys
```

ใน nano ให้วาง Public Key ที่คัดลอกมาจาก PuTTYgen (ลักษณะประมาณนี้):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI... (ข้อความยาวๆ)
```

บันทึกด้วย `Ctrl+O` แล้ว `Ctrl+X`

### ขั้นตอนที่ 3 — ตั้งค่า PuTTY ให้ใช้ Private Key

1. เปิด **PuTTY**
2. ใส่ IP ของ Ubuntu
3. ไปที่ **Connection → SSH → Auth → Credentials**
4. ช่อง **Private key file for authentication:** กด **Browse** แล้วเลือกไฟล์ `.ppk`
5. กลับไปที่ **Session** กด **Save** เพื่อบันทึก
6. กด **Open** เพื่อเชื่อมต่อ

### ทดสอบและปิด Password Login (ขั้นสูง)

เมื่อ Key Login ทำงานได้แล้ว สามารถปิด Password Login:

```bash
sudo nano /etc/ssh/sshd_config
```

เปลี่ยน:
```
PasswordAuthentication no
```

```bash
sudo systemctl reload ssh
```

> ⚠️ **ระวัง:** ทำขั้นตอนนี้ก็ต่อเมื่อทดสอบ Key Login สำเร็จแล้วเท่านั้น มิฉะนั้นจะ login ไม่ได้เลย!

---

## 10. Port Forwarding / Tunneling

SSH Tunneling คือการส่งข้อมูลของโปรแกรมอื่นผ่านช่องทาง SSH ที่เข้ารหัสแล้ว

### ประเภทของ Port Forwarding

```
1. Local Port Forwarding  — ส่ง port จาก Windows ไปยัง Ubuntu (และ host อื่นๆ ผ่าน Ubuntu)
2. Remote Port Forwarding — ส่ง port จาก Ubuntu ไปยัง Windows
3. Dynamic Forwarding     — ทำ SOCKS Proxy
```

### Local Port Forwarding ด้วย PuTTY

**สถานการณ์:** ต้องการเข้าถึง web server ที่รันบน Ubuntu Port 8080 จาก browser บน Windows

1. เปิด PuTTY → ไปที่ **Connection → SSH → Tunnels**
2. **Source port:** `8080` (port บน Windows ที่จะฟัง)
3. **Destination:** `localhost:8080` (port บน Ubuntu)
4. เลือก **Local**
5. กด **Add**
6. เชื่อมต่อตามปกติ

จากนั้นเปิด browser บน Windows แล้วไปที่ `http://localhost:8080`

### Remote Port Forwarding ด้วย PuTTY

**สถานการณ์:** ต้องการให้ Ubuntu เข้าถึง port บน Windows

1. เปิด PuTTY → ไปที่ **Connection → SSH → Tunnels**
2. **Source port:** `9090` (port บน Ubuntu)
3. **Destination:** `localhost:9090` (port บน Windows)
4. เลือก **Remote**
5. กด **Add**

---

## 11. SCP และ SFTP

### SCP (Secure Copy) — โอนไฟล์จาก Command Line

SCP ใช้ได้จาก Ubuntu Terminal (ส่งไฟล์หา Windows ต้องมี SSH Server บน Windows ด้วย)

**คัดลอกไฟล์จาก Ubuntu ไปยัง Windows (หรือ remote host):**

```bash
# รูปแบบ: scp [ไฟล์ต้นทาง] [user]@[ip]:[ปลายทาง]
scp /home/myuser/document.txt windowsuser@192.168.1.200:/C/Users/windowsuser/Desktop/

# คัดลอกโฟลเดอร์ทั้งหมด (ใช้ -r สำหรับ recursive)
scp -r /home/myuser/myfolder/ windowsuser@192.168.1.200:/C/Users/windowsuser/Desktop/
```

**คัดลอกไฟล์จาก Windows ไปยัง Ubuntu:**

```bash
scp windowsuser@192.168.1.200:/C/Users/windowsuser/Desktop/file.txt /home/myuser/
```

### SFTP — โอนไฟล์แบบ Interactive

**ใช้จาก Ubuntu Terminal:**

```bash
sftp myuser@192.168.1.105
```

คำสั่งใน SFTP shell:

```bash
# ดูไฟล์ฝั่ง Server (Ubuntu)
ls

# ดูไฟล์ฝั่ง Local (Windows)
lls

# เปลี่ยนโฟลเดอร์ฝั่ง Server
cd /path/on/server

# เปลี่ยนโฟลเดอร์ฝั่ง Local
lcd C:\Users\myuser\Desktop

# ดาวน์โหลดไฟล์จาก Server มายัง Local
get filename.txt

# ดาวน์โหลดทั้งโฟลเดอร์
get -r foldername/

# อัปโหลดไฟล์จาก Local ไปยัง Server
put filename.txt

# อัปโหลดทั้งโฟลเดอร์
put -r foldername/

# ออกจาก SFTP
exit
```

### WinSCP — โปรแกรม SFTP แบบ GUI สำหรับ Windows

> 📥 ดาวน์โหลดที่: **https://winscp.net**

WinSCP ใช้งานง่ายแบบ drag-and-drop โอนไฟล์ระหว่าง Windows กับ Ubuntu ได้สะดวกมาก

**ตั้งค่า WinSCP:**
1. **File protocol:** SFTP
2. **Host name:** IP ของ Ubuntu
3. **Port number:** 22
4. **User name:** username บน Ubuntu
5. กด **Login**

---

## 12. SSH Config File (ฝั่ง Client)

ถ้าใช้ SSH จาก WSL (Windows Subsystem for Linux) หรือ Git Bash บน Windows สามารถสร้างไฟล์ config เพื่อบันทึก SSH settings ได้

ไฟล์อยู่ที่: `~/.ssh/config` (บน Linux/WSL) หรือ `C:\Users\username\.ssh\config` (บน Windows)

```
# ─────────────────────────────────────────────
# ตัวอย่าง ~/.ssh/config
# ─────────────────────────────────────────────

Host ubuntu-home
    HostName 192.168.1.105
    User myusername
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host ubuntu-server2
    HostName 192.168.1.110
    User adminuser
    Port 2222
    IdentityFile ~/.ssh/id_rsa
```

จากนั้นแทนที่จะพิมพ์:
```bash
ssh myusername@192.168.1.105 -p 22 -i ~/.ssh/id_ed25519
```

พิมพ์แค่:
```bash
ssh ubuntu-home
```

---

## 13. การจัดการ User และสิทธิ์

### ดู User ที่มีในระบบ

```bash
cat /etc/passwd | grep -v nologin | grep -v false
```

### สร้าง User ใหม่สำหรับ SSH

```bash
# สร้าง user ใหม่
sudo adduser newuser

# เพิ่ม user เข้า sudo group (ถ้าต้องการให้มีสิทธิ์ admin)
sudo usermod -aG sudo newuser

# ตรวจสอบว่า user อยู่ใน group อะไรบ้าง
groups newuser
```

### เปลี่ยนรหัสผ่าน User

```bash
# เปลี่ยนรหัสผ่านตัวเอง
passwd

# เปลี่ยนรหัสผ่านของ user อื่น (ต้องใช้ sudo)
sudo passwd username
```

### ล็อค/ปลดล็อค User

```bash
# ล็อค user (ห้าม login)
sudo passwd -l username

# ปลดล็อค user
sudo passwd -u username
```

### จำกัดเฉพาะ User หรือ Group ใน sshd_config

```bash
# อนุญาตเฉพาะ user ที่ระบุ
AllowUsers user1 user2

# อนุญาตเฉพาะ group ที่ระบุ
AllowGroups sshusers

# ห้าม user ที่ระบุ
DenyUsers baduser

# ห้าม group ที่ระบุ
DenyGroups noSSH
```

---

## 14. Firewall (UFW) กับ SSH

### UFW คืออะไร?

**UFW** = Uncomplicated Firewall คือ Firewall ของ Ubuntu ที่ใช้งานง่าย เป็น front-end ของ `iptables`

### คำสั่ง UFW พื้นฐาน

```bash
# ตรวจสอบสถานะ UFW
sudo ufw status

# ตรวจสอบแบบละเอียด
sudo ufw status verbose

# เปิดใช้งาน UFW
sudo ufw enable

# ปิด UFW
sudo ufw disable
```

> ⚠️ **สำคัญมาก:** ก่อน `ufw enable` ให้อนุญาต SSH ก่อน ไม่งั้นจะ SSH ไม่ได้!

### อนุญาต SSH ผ่าน UFW

```bash
# วิธีที่ 1 — อนุญาต SSH โดยตรง (Port 22)
sudo ufw allow ssh

# วิธีที่ 2 — อนุญาตโดยระบุ port
sudo ufw allow 22/tcp

# วิธีที่ 3 — อนุญาตเฉพาะจาก IP หรือ subnet ที่ระบุ (ปลอดภัยกว่า)
sudo ufw allow from 192.168.1.0/24 to any port 22

# ถ้าเปลี่ยน SSH Port เป็น 2222
sudo ufw allow 2222/tcp
```

### ลบ Rule ออก

```bash
# ลบ rule ที่อนุญาต SSH
sudo ufw delete allow ssh

# หรือลบโดยระบุหมายเลข rule
sudo ufw status numbered
sudo ufw delete [number]
```

### ดู Log ของ UFW

```bash
sudo tail -f /var/log/ufw.log
```

---

## 15. ความปลอดภัย — Best Practices

### ✅ สิ่งที่ควรทำ

```bash
# 1. เปลี่ยน SSH Port จาก 22 (ลด bot scanning)
# ใน /etc/ssh/sshd_config:
Port 2222

# 2. ปิดการ login ของ root
PermitRootLogin no

# 3. ใช้ Key-Based Authentication
PubkeyAuthentication yes
PasswordAuthentication no  # หลังจากตั้ง Key แล้ว

# 4. จำกัดจำนวนครั้ง login ผิด
MaxAuthTries 3

# 5. จำกัด Users ที่เข้าได้
AllowUsers myuser

# 6. ตั้ง LoginGraceTime ให้สั้น
LoginGraceTime 30
```

### ❌ สิ่งที่ไม่ควรทำ

- ไม่ควรใช้ `PermitRootLogin yes`
- ไม่ควรใช้รหัสผ่านที่เดาง่าย
- ไม่ควรปิด UFW ทิ้งไว้
- ไม่ควรแชร์ Private Key ให้ใคร
- ไม่ควรใช้ SSH Version 1 (`Protocol 2` เป็นค่าเริ่มต้นของ OpenSSH รุ่นใหม่แล้ว)

### ติดตั้ง Fail2ban (ป้องกัน Brute Force)

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

**ดูสถานะ Fail2ban:**

```bash
sudo fail2ban-client status sshd
```

**ตัวอย่างผลลัพธ์:**

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 3
|  `- Total failed:    15
`- Actions
   |- Currently banned: 1
   `- Total banned:    5
```

### ติดตั้ง Logwatch (ดู Log สรุป)

```bash
sudo apt install logwatch -y
sudo logwatch --detail high --service sshd --range today
```

---

## 16. Troubleshooting

### ปัญหา: "Connection refused"

```
ssh: connect to host 192.168.1.105 port 22: Connection refused
```

**สาเหตุและวิธีแก้:**

```bash
# ตรวจสอบว่า SSH Service รันอยู่ไหม
sudo systemctl status ssh

# ถ้าไม่รัน ให้สตาร์ท
sudo systemctl start ssh

# ตรวจสอบว่า SSH ฟัง Port 22 จริงไหม
sudo ss -tlnp | grep sshd

# ตรวจสอบ UFW
sudo ufw status
```

### ปัญหา: "Connection timed out"

**สาเหตุ:** มักเป็นปัญหา Firewall หรือ IP ผิด

```bash
# ตรวจสอบ IP ของ Ubuntu อีกครั้ง
ip addr show

# ตรวจสอบ UFW ว่าเปิด SSH หรือยัง
sudo ufw status

# ทดสอบ ping จาก Windows ก่อน
ping 192.168.1.105
```

### ปัญหา: "Permission denied (publickey)"

**สาเหตุ:** Public Key บน Server หรือ Private Key บน Client มีปัญหา

```bash
# ตรวจสอบสิทธิ์ไฟล์บน Ubuntu
ls -la ~/.ssh/
# ต้องเห็น:
# drwx------ .ssh (700)
# -rw------- authorized_keys (600)

# แก้สิทธิ์ถ้าผิด
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# ตรวจสอบเนื้อหา authorized_keys
cat ~/.ssh/authorized_keys
```

```bash
# ดู SSH Log เพื่อหาสาเหตุ
sudo tail -f /var/log/auth.log | grep ssh
```

### ปัญหา: "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!"

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

**สาเหตุ:** IP ของ Ubuntu เปลี่ยนไป (เพราะ DHCP) หรือ reinstall Ubuntu

**แก้ไขบน Windows (PuTTY):** ลบ Host Key เก่าออกจาก Registry

ไปที่ Registry: `HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\SshHostKeys`
ลบ entry ของ IP นั้นออก

หรือบน Command Line:

```bash
# ถ้าใช้ OpenSSH บน Windows (ไม่ใช่ PuTTY)
ssh-keygen -R 192.168.1.105
```

### ปัญหา: IP ของ Ubuntu เปลี่ยนทุกวัน (DHCP)

**วิธีแก้ 1 — กำหนด IP แบบ Static บน Ubuntu:**

```bash
# ใช้ nmcli (NetworkManager CLI)
# ดู connection ที่มี
nmcli con show

# กำหนด static IP (เปลี่ยนชื่อ connection และค่าตามจริง)
sudo nmcli con mod "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.1.105/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 8.8.4.4"

# รีสตาร์ท connection
sudo nmcli con up "Wired connection 1"
```

**วิธีแก้ 2 — กำหนด IP จาก Router (DHCP Reservation):**

เข้า Router admin page (ปกติ 192.168.1.1 หรือ 192.168.0.1) → ค้นหาหัวข้อ "DHCP Reservation" หรือ "Static DHCP" → ผูก MAC Address ของ Ubuntu กับ IP ที่ต้องการ

### ดู SSH Log แบบ Real-time

```bash
sudo journalctl -u ssh -f
# หรือ
sudo tail -f /var/log/auth.log
```

---

## 17. ทริคและเทคนิคขั้นสูง

### ทริค 1 — SSH Escape Sequences

เมื่อ connect อยู่ใน PuTTY/SSH Session มีคำสั่งพิเศษ:

```
~.    — ตัด connection ทันที (ใช้เมื่อ session ค้าง)
~^Z   — ย้าย SSH session ไป background
~#    — แสดง forwarded connections ที่เปิดอยู่
~?    — แสดงทุก escape sequence
```

### ทริค 2 — SSH Agent (จำ Passphrase ชั่วคราว)

ถ้าใช้ Private Key ที่มี Passphrase จะต้องพิมพ์ทุกครั้ง ใช้ Pageant (PuTTY Agent) ช่วยได้:

1. เปิดโปรแกรม **Pageant** (ติดมากับ PuTTY)
2. คลิกขวาที่ icon ใน system tray → **Add Key**
3. เลือกไฟล์ `.ppk`
4. พิมพ์ Passphrase ครั้งเดียว
5. จากนั้น PuTTY จะใช้ Passphrase จาก Pageant โดยอัตโนมัติ

### ทริค 3 — Multiplexing (เปิดหลาย Terminal แชร์ Connection เดียว)

บน PuTTY ทำได้โดยการ duplicate session:
คลิกขวาที่ title bar → **Duplicate Session**

### ทริค 4 — ส่งคำสั่งโดยไม่ต้อง Login แบบ Interactive

```bash
# รันคำสั่งเดียวแล้วออกเลย
ssh myuser@192.168.1.105 "ls -la /home/myuser"

# รัน script บน Server
ssh myuser@192.168.1.105 "bash /home/myuser/myscript.sh"

# ดู memory usage
ssh myuser@192.168.1.105 "free -h"
```

### ทริค 5 — SSH Multiplexer (tmux หรือ screen)

เมื่อ SSH ขาดหาย งานที่รันค้างอยู่จะหาย ใช้ tmux แก้ปัญหานี้:

```bash
# ติดตั้ง tmux
sudo apt install tmux -y

# เริ่ม tmux session ใหม่
tmux new -s mysession

# รัน long-running command ใน tmux
python3 longscript.py

# กด Ctrl+B แล้ว D เพื่อ detach (SSH ขาดก็ไม่เป็นไร)

# กลับมา attach session เดิม
tmux attach -t mysession

# ดู sessions ทั้งหมด
tmux ls
```

### ทริค 6 — ดู Log การ Login ย้อนหลัง

```bash
# ดูการ login ล่าสุด
last

# ดูการ login ล้มเหลว (ถูก hack พยายาม)
sudo lastb

# ดูการ login ผ่าน SSH
sudo grep "Accepted" /var/log/auth.log

# ดูการ login ล้มเหลวผ่าน SSH
sudo grep "Failed" /var/log/auth.log
```

### ทริค 7 — SOCKS Proxy ผ่าน SSH

ใช้ Ubuntu เป็น proxy ให้ Windows เข้าเน็ตผ่าน:

**ใน PuTTY:**
1. ไปที่ **Connection → SSH → Tunnels**
2. **Source port:** `1080`
3. เลือก **Dynamic**
4. กด **Add**
5. เชื่อมต่อตามปกติ

**ตั้งค่า Browser:** ไปที่ Proxy Settings → SOCKS5 → localhost:1080

### ทริค 8 — ตรวจสอบว่ามีใครต่อ SSH อยู่ไหม

```bash
# ดู user ที่ login อยู่
who

# ดูแบบละเอียดมากขึ้น
w

# ดู SSH connections ที่เปิดอยู่
sudo ss -tnp | grep :22
```

### ทริค 9 — กำหนด timeout ให้ Connection ที่ Idle

ใน `/etc/ssh/sshd_config`:

```bash
# ตัด connection ถ้าไม่มีกิจกรรม 10 นาที
ClientAliveInterval 300   # ส่ง keepalive ทุก 300 วินาที
ClientAliveCountMax 2     # ถ้าไม่ตอบ 2 ครั้ง ตัด connection
```

### ทริค 10 — จำกัดการเข้าถึงตาม IP

ใน `/etc/ssh/sshd_config`:

```bash
# อนุญาตเฉพาะ user จาก subnet 192.168.1.x
AllowUsers myuser@192.168.1.*
```

---

## 18. Cheat Sheet

### คำสั่ง SSH Service

```bash
sudo systemctl status ssh         # ดูสถานะ
sudo systemctl start ssh          # เริ่ม
sudo systemctl stop ssh           # หยุด
sudo systemctl restart ssh        # รีสตาร์ท
sudo systemctl reload ssh         # โหลด config ใหม่
sudo systemctl enable ssh         # เปิด auto-start
sudo sshd -t                      # ตรวจ config syntax
```

### คำสั่ง Network

```bash
ip addr show                      # ดู IP ทั้งหมด
hostname -I                       # ดู IP สั้นๆ
ping 192.168.1.105                # ทดสอบ ping
sudo ss -tlnp | grep sshd         # ดูว่า sshd ฟัง port ไหน
```

### คำสั่ง UFW

```bash
sudo ufw status verbose           # ดูสถานะ Firewall
sudo ufw allow ssh                # อนุญาต SSH
sudo ufw allow 22/tcp             # อนุญาต Port 22
sudo ufw enable                   # เปิด Firewall
sudo ufw disable                  # ปิด Firewall
```

### คำสั่ง Key Management

```bash
mkdir -p ~/.ssh                   # สร้างโฟลเดอร์ .ssh
chmod 700 ~/.ssh                  # ตั้งสิทธิ์โฟลเดอร์
chmod 600 ~/.ssh/authorized_keys  # ตั้งสิทธิ์ไฟล์ key
```

### คำสั่ง Log

```bash
sudo tail -f /var/log/auth.log                    # ดู SSH log real-time
sudo journalctl -u ssh -f                         # ดู service log
sudo grep "Accepted" /var/log/auth.log            # login สำเร็จ
sudo grep "Failed" /var/log/auth.log              # login ล้มเหลว
last                                              # ประวัติ login
```

### คำสั่ง SCP

```bash
# คัดลอกไฟล์ไปยัง remote
scp localfile.txt user@192.168.1.105:/home/user/

# คัดลอกไฟล์จาก remote
scp user@192.168.1.105:/home/user/file.txt ./

# คัดลอกโฟลเดอร์
scp -r localfolder/ user@192.168.1.105:/home/user/
```

---

## 🎯 สรุปขั้นตอนการตั้งค่าครั้งแรก (Quick Start)

```
บน Ubuntu 24.04:
1. sudo apt update && sudo apt install openssh-server -y
2. sudo systemctl enable ssh
3. hostname -I  ← จด IP ไว้
4. sudo ufw allow ssh && sudo ufw enable

บน Windows 10:
5. ดาวน์โหลดและติดตั้ง PuTTY จาก putty.org
6. เปิด PuTTY → ใส่ IP ของ Ubuntu → Port 22 → Open
7. Login ด้วย username และ password ของ Ubuntu

เพิ่มความปลอดภัย (ทำทีหลัง):
8. สร้าง Key Pair ด้วย PuTTYgen
9. ใส่ Public Key ใน ~/.ssh/authorized_keys บน Ubuntu
10. ตั้งค่า PuTTY ให้ใช้ Private Key
11. ปิด PasswordAuthentication ใน sshd_config
```

---

> 📝 **เขียนโดย:** คู่มือนี้สร้างขึ้นสำหรับมือใหม่ที่เริ่มต้นศึกษา OpenSSH บน Ubuntu 24.04 LTS
> 🔄 **อ้างอิง:** OpenSSH 9.x Documentation, Ubuntu 24.04 LTS Official Docs
