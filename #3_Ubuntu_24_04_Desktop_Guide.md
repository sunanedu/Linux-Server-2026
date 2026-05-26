# คู่มือการใช้งาน Linux Ubuntu Desktop 24.04 LTS ฉบับสมบูรณ์

> ครอบคลุมทุกฟังก์ชันการใช้งานทั่วไปสำหรับผู้ใช้ระดับเริ่มต้นถึงกลาง

---

## สารบัญ

1. [ส่วนติดต่อผู้ใช้ (GNOME Desktop)](#1-ส่วนติดต่อผู้ใช้-gnome-desktop)
2. [การจัดการไฟล์และโฟลเดอร์](#2-การจัดการไฟล์และโฟลเดอร์)
3. [การติดตั้งและจัดการซอฟต์แวร์](#3-การติดตั้งและจัดการซอฟต์แวร์)
4. [การตั้งค่าระบบ (Settings)](#4-การตั้งค่าระบบ-settings)
5. [การใช้งาน Terminal](#5-การใช้งาน-terminal)
6. [การจัดการผู้ใช้และสิทธิ์](#6-การจัดการผู้ใช้และสิทธิ์)
7. [การจัดการเครือข่าย](#7-การจัดการเครือข่าย)
8. [การจัดการดิสก์และพาร์ติชัน](#8-การจัดการดิสก์และพาร์ติชัน)
9. [การจัดการกระบวนการ (Process)](#9-การจัดการกระบวนการ-process)
10. [การตั้งค่าเสียงและจอภาพ](#10-การตั้งค่าเสียงและจอภาพ)
11. [การสำรองข้อมูล (Backup)](#11-การสำรองข้อมูล-backup)
12. [ความปลอดภัยของระบบ](#12-ความปลอดภัยของระบบ)
13. [คีย์บอร์ดลัดที่ควรรู้](#13-คีย์บอร์ดลัดที่ควรรู้)
14. [การแก้ปัญหาเบื้องต้น](#14-การแก้ปัญหาเบื้องต้น)

---

## 1. ส่วนติดต่อผู้ใช้ (GNOME Desktop)

Ubuntu 24.04 ใช้ GNOME 46 เป็น Desktop Environment หลัก

### 1.1 ส่วนประกอบของหน้าจอหลัก

| ส่วนประกอบ | คำอธิบาย |
|---|---|
| **Top Bar** | แถบด้านบน แสดงเวลา วันที่ การแจ้งเตือน และเมนูระบบ |
| **Activities Overview** | กดปุ่ม `Super` (Windows key) เพื่อดูหน้าต่างที่เปิดอยู่ทั้งหมด |
| **Dash (Dock)** | แถบแอปพลิเคชันด้านซ้าย ปักหมุดแอปที่ใช้บ่อย |
| **App Grid** | กดปุ่มตารางจุดที่ Dash เพื่อดูแอปทั้งหมด |
| **Workspace** | พื้นที่ทำงานเสมือน สามารถสร้างได้หลายพื้นที่ |

### 1.2 Activities Overview

- กด `Super` หรือคลิก **Activities** มุมบนซ้ายเพื่อเข้าสู่โหมด Overview
- พิมพ์ชื่อแอปทันทีเพื่อค้นหา (ไม่ต้องคลิกช่องค้นหา)
- ลากหน้าต่างไปวางบน Workspace ต่าง ๆ ทางขวาของหน้าจอ

### 1.3 การจัดการ Workspace

```
Super + Page Up/Down    → สลับ Workspace
Super + Shift + Page Up/Down → ย้ายหน้าต่างไป Workspace อื่น
Super + Home/End        → ไป Workspace แรก/สุดท้าย
```

### 1.4 การปรับแต่ง Dash และ Desktop

- **คลิกขวา** บน Dash เพื่อเพิ่ม/ลบแอป
- ติดตั้ง **GNOME Tweaks** เพื่อปรับแต่งเพิ่มเติม:

```bash
sudo apt install gnome-tweaks
```

- ติดตั้ง **GNOME Extensions** เพื่อเพิ่มฟังก์ชัน:

```bash
sudo apt install gnome-shell-extensions
```

---

## 2. การจัดการไฟล์และโฟลเดอร์

### 2.1 Nautilus (Files) — โปรแกรมจัดการไฟล์

เปิดได้จาก Dash หรือกด `Super` แล้วพิมพ์ "Files"

**มุมมองการแสดงผล:**
- `Ctrl + 1` → มุมมองรายการ (List View)
- `Ctrl + 2` → มุมมองตาราง (Grid View)
- `Ctrl + +` / `Ctrl + -` → ซูมเข้า/ออก

**การนำทาง:**
- `Alt + ←` / `Alt + →` → ย้อนกลับ/ไปข้างหน้า
- `Alt + ↑` → ขึ้นไปโฟลเดอร์แม่
- `Ctrl + L` → พิมพ์ path ตรง ๆ
- `Ctrl + H` → แสดง/ซ่อนไฟล์ที่ซ่อนอยู่ (ไฟล์ที่ขึ้นต้นด้วย `.`)

**การจัดการไฟล์:**

| การกระทำ | วิธี |
|---|---|
| สร้างโฟลเดอร์ใหม่ | `Ctrl + Shift + N` หรือคลิกขวา → New Folder |
| เปลี่ยนชื่อ | `F2` หรือคลิกขวา → Rename |
| ลบไฟล์ (ไปถังขยะ) | `Delete` |
| ลบถาวร | `Shift + Delete` |
| คัดลอก | `Ctrl + C` |
| ตัด | `Ctrl + X` |
| วาง | `Ctrl + V` |
| เลือกทั้งหมด | `Ctrl + A` |
| ยกเลิก | `Ctrl + Z` |

### 2.2 โครงสร้างไดเรกทอรีสำคัญ

```
/               → Root ของระบบทั้งหมด
├── home/       → โฟลเดอร์ผู้ใช้ (เช่น /home/username)
│   └── username/
│       ├── Desktop/    → เดสก์ท็อป
│       ├── Documents/  → เอกสาร
│       ├── Downloads/  → ดาวน์โหลด
│       ├── Music/      → เพลง
│       ├── Pictures/   → รูปภาพ
│       └── Videos/     → วิดีโอ
├── etc/        → ไฟล์ config ของระบบ
├── var/        → ข้อมูลที่เปลี่ยนแปลงบ่อย (log, cache)
├── usr/        → โปรแกรมที่ติดตั้ง
├── tmp/        → ไฟล์ชั่วคราว
├── media/      → USB, CD ที่เชื่อมต่อ
└── mnt/        → จุด mount ไดรฟ์เพิ่มเติม
```

### 2.3 การบีบอัดและแตกไฟล์

**ผ่าน GUI:** คลิกขวาที่ไฟล์/โฟลเดอร์ → **Compress** หรือ **Extract Here**

**ผ่าน Terminal:**

```bash
# บีบอัดเป็น .tar.gz
tar -czvf ชื่อไฟล์.tar.gz โฟลเดอร์/

# แตกไฟล์ .tar.gz
tar -xzvf ชื่อไฟล์.tar.gz

# บีบอัดเป็น .zip
zip -r ชื่อไฟล์.zip โฟลเดอร์/

# แตกไฟล์ .zip
unzip ชื่อไฟล์.zip
```

### 2.4 การค้นหาไฟล์

**ผ่าน GUI:** กด `Ctrl + F` ใน Files

**ผ่าน Terminal:**

```bash
# ค้นหาตามชื่อ
find /home -name "*.pdf"

# ค้นหาตามเนื้อหา
grep -r "คำที่ต้องการ" /home/username/Documents/

# ค้นหาด้วย locate (เร็วกว่า find)
sudo updatedb
locate ชื่อไฟล์
```

---

## 3. การติดตั้งและจัดการซอฟต์แวร์

### 3.1 Ubuntu Software Center (GUI)

- เปิดจาก Dash → **Ubuntu Software**
- ค้นหาแอปตามชื่อ → คลิก **Install**
- ดูแอปที่ติดตั้งแล้ว → แท็บ **Installed**
- อัปเดตแอป → แท็บ **Updates**

### 3.2 APT — ตัวจัดการแพ็กเกจหลัก

```bash
# อัปเดตรายการแพ็กเกจ (ควรทำก่อนติดตั้งเสมอ)
sudo apt update

# อัปเกรดแพ็กเกจทั้งหมด
sudo apt upgrade

# อัปเดตและอัปเกรดพร้อมกัน
sudo apt update && sudo apt upgrade -y

# ติดตั้งแพ็กเกจ
sudo apt install ชื่อแพ็กเกจ

# ติดตั้งหลายแพ็กเกจพร้อมกัน
sudo apt install vlc gimp inkscape

# ลบแพ็กเกจ
sudo apt remove ชื่อแพ็กเกจ

# ลบแพ็กเกจพร้อม config
sudo apt purge ชื่อแพ็กเกจ

# ลบแพ็กเกจที่ไม่ใช้แล้ว
sudo apt autoremove

# ค้นหาแพ็กเกจ
apt search ชื่อแพ็กเกจ

# ดูข้อมูลแพ็กเกจ
apt show ชื่อแพ็กเกจ

# รายการแพ็กเกจที่ติดตั้งแล้ว
dpkg -l
```

### 3.3 Snap — แพ็กเกจรูปแบบใหม่

```bash
# ค้นหา Snap
snap find ชื่อแอป

# ติดตั้ง Snap
sudo snap install ชื่อแอป

# รายการ Snap ที่ติดตั้ง
snap list

# อัปเดต Snap
sudo snap refresh

# ลบ Snap
sudo snap remove ชื่อแอป
```

### 3.4 Flatpak — อีกทางเลือกสำหรับแพ็กเกจ

```bash
# ติดตั้ง Flatpak
sudo apt install flatpak

# เพิ่ม Flathub repository
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

# ติดตั้งแอปจาก Flathub
flatpak install flathub ชื่อแอป

# รันแอป Flatpak
flatpak run ชื่อแอป

# อัปเดต
flatpak update

# ลบแอป
flatpak uninstall ชื่อแอป
```

### 3.5 การติดตั้งจากไฟล์ .deb

```bash
# ติดตั้งไฟล์ .deb
sudo dpkg -i ชื่อไฟล์.deb

# แก้ dependency หากเกิดปัญหา
sudo apt -f install
```

### 3.6 แอปพลิเคชันที่แนะนำ

| หมวดหมู่ | ชื่อแอป | คำสั่งติดตั้ง |
|---|---|---|
| Office | LibreOffice | ติดตั้งมาแล้ว |
| เบราว์เซอร์ | Firefox | ติดตั้งมาแล้ว |
| เบราว์เซอร์ | Chromium | `sudo apt install chromium-browser` |
| วิดีโอ | VLC | `sudo apt install vlc` |
| รูปภาพ | GIMP | `sudo apt install gimp` |
| โค้ด | VS Code | ติดตั้งผ่าน Snap |
| สื่อสาร | Telegram | `sudo snap install telegram-desktop` |
| ขจัดไวรัส | ClamAV | `sudo apt install clamav` |

---

## 4. การตั้งค่าระบบ (Settings)

เปิด Settings ได้จาก: **เมนูบนขวา → Settings** หรือ `Super` แล้วพิมพ์ "Settings"

### 4.1 การแสดงผล (Displays)

- ปรับความละเอียดหน้าจอ (Resolution)
- ตั้งค่าจอหลายจอ (Multiple Displays): Mirror / Extend / Single
- ปรับอัตราการรีเฟรช (Refresh Rate)
- เปิด Night Light (ลดแสงสีฟ้า)
- ปรับ Display Scale (ขนาดองค์ประกอบบนหน้าจอ)

### 4.2 เสียง (Sound)

- เลือกอุปกรณ์เล่นเสียง Output
- เลือกไมโครโฟน Input
- ปรับระดับเสียงแต่ละแอป (Per-app volume)
- ทดสอบเสียง (Test Speakers)

### 4.3 เมาส์และทัชแพด (Mouse & Touchpad)

- เปิด/ปิด Tap to Click (ทัชแพด)
- ปรับความเร็วเคอร์เซอร์
- เปลี่ยนปุ่มหลักซ้าย/ขวา (Left-handed)
- เปิด Natural Scrolling

### 4.4 แป้นพิมพ์ (Keyboard)

- เพิ่มภาษา Input (เช่น ภาษาไทย)
- ตั้ง Keyboard Shortcuts เอง
- ปรับ Repeat Keys delay/speed

**การเพิ่มภาษาไทย:**
Settings → Keyboard → Input Sources → กดปุ่ม `+` → เลือก Thai

สลับภาษา: `Super + Space` หรือ `Shift + Alt` (แล้วแต่การตั้งค่า)

### 4.5 ภูมิภาคและภาษา (Region & Language)

- ตั้งภาษาหลักของระบบ
- ตั้งรูปแบบวันที่ เวลา และสกุลเงิน
- ตั้ง Timezone

### 4.6 การจัดการพลังงาน (Power)

- กำหนดเวลาดับหน้าจออัตโนมัติ
- ตั้งการทำงานเมื่อปิดฝาแล็ปท็อป
- โหมดประหยัดพลังงาน (Power Saver)
- ดูข้อมูลแบตเตอรี่

### 4.7 การแจ้งเตือน (Notifications)

- เปิด/ปิดการแจ้งเตือนของแต่ละแอป
- ตั้ง Do Not Disturb
- กำหนดการแสดงบน Lock Screen

### 4.8 ความเป็นส่วนตัว (Privacy)

- ตั้งเวลาล็อกหน้าจออัตโนมัติ
- จัดการ Location Services
- ตั้งค่า File History (ประวัติไฟล์ที่เปิด)
- ควบคุมสิทธิ์การเข้าถึงกล้องและไมโครโฟน

---

## 5. การใช้งาน Terminal

เปิด Terminal: `Ctrl + Alt + T` หรือ `Super` → พิมพ์ "Terminal"

### 5.1 คำสั่งพื้นฐาน

```bash
# แสดงที่อยู่ปัจจุบัน
pwd

# แสดงรายการไฟล์
ls
ls -la        # แสดงรายละเอียดและไฟล์ซ่อน
ls -lh        # แสดงขนาดอ่านง่าย

# เปลี่ยนไดเรกทอรี
cd /home/username
cd ..         # ขึ้นหนึ่งระดับ
cd ~          # กลับ Home
cd -          # กลับไดเรกทอรีก่อนหน้า

# สร้างโฟลเดอร์
mkdir ชื่อโฟลเดอร์
mkdir -p parent/child/grandchild    # สร้างซ้อนกัน

# สร้างไฟล์เปล่า
touch ชื่อไฟล์.txt

# คัดลอก
cp ต้นฉบับ ปลายทาง
cp -r โฟลเดอร์ ปลายทาง    # คัดลอกโฟลเดอร์

# ย้าย / เปลี่ยนชื่อ
mv ต้นฉบับ ปลายทาง

# ลบไฟล์
rm ชื่อไฟล์
rm -rf ชื่อโฟลเดอร์   # ลบโฟลเดอร์และทุกอย่างข้างใน (ระวัง!)

# ดูเนื้อหาไฟล์
cat ไฟล์.txt
less ไฟล์.txt         # ดูแบบเลื่อนได้ (q เพื่อออก)
head -n 20 ไฟล์.txt   # ดู 20 บรรทัดแรก
tail -n 20 ไฟล์.txt   # ดู 20 บรรทัดสุดท้าย
tail -f log.txt       # ดู log แบบ real-time
```

### 5.2 การจัดการระบบผ่าน Terminal

```bash
# ดูข้อมูลระบบ
uname -a              # ข้อมูล Kernel
lsb_release -a        # เวอร์ชัน Ubuntu
hostname              # ชื่อเครื่อง
whoami                # ผู้ใช้ปัจจุบัน
uptime                # เวลาที่ระบบทำงานมา
date                  # วันเวลาปัจจุบัน

# ดูการใช้งานทรัพยากร
free -h               # RAM
df -h                 # พื้นที่ดิสก์
du -sh โฟลเดอร์/       # ขนาดโฟลเดอร์
top                   # Monitor แบบ real-time
htop                  # Monitor ที่ดูง่ายกว่า (sudo apt install htop)
```

### 5.3 การใช้ Pipe และ Redirect

```bash
# Pipe: ส่งผลลัพธ์คำสั่งหนึ่งให้อีกคำสั่ง
ls -la | grep ".pdf"
cat ไฟล์.txt | wc -l      # นับจำนวนบรรทัด

# Redirect: บันทึกผลลัพธ์ลงไฟล์
ls -la > รายการ.txt        # เขียนทับ
ls -la >> รายการ.txt       # ต่อท้าย
command 2> error.log       # บันทึก error
command &> output.log      # บันทึกทั้ง output และ error
```

### 5.4 การเขียน Shell Script เบื้องต้น

```bash
#!/bin/bash
# สร้างไฟล์ myscript.sh

echo "สวัสดีครับ"
echo "วันนี้คือ: $(date)"
echo "ผู้ใช้: $USER"

# ทำให้ไฟล์รันได้
chmod +x myscript.sh

# รันสคริปต์
./myscript.sh
```

### 5.5 Alias — ชื่อย่อคำสั่ง

```bash
# สร้าง alias ชั่วคราว (เฉพาะ session นี้)
alias update='sudo apt update && sudo apt upgrade -y'
alias ll='ls -la'

# สร้าง alias ถาวร (เพิ่มใน ~/.bashrc)
echo "alias update='sudo apt update && sudo apt upgrade -y'" >> ~/.bashrc
source ~/.bashrc      # รีโหลด bashrc
```

### 5.6 SSH — เชื่อมต่อเครื่องอื่นจากระยะไกล

```bash
# เชื่อมต่อ
ssh username@ip_address

# คัดลอกไฟล์ผ่าน SSH
scp ไฟล์.txt username@ip:/path/to/destination/

# สร้าง SSH Key (ปลอดภัยกว่า password)
ssh-keygen -t rsa -b 4096
ssh-copy-id username@ip_address
```

---

## 6. การจัดการผู้ใช้และสิทธิ์

### 6.1 การจัดการผู้ใช้

```bash
# เพิ่มผู้ใช้ใหม่
sudo adduser ชื่อผู้ใช้

# ลบผู้ใช้
sudo deluser ชื่อผู้ใช้
sudo deluser --remove-home ชื่อผู้ใช้    # ลบพร้อม home folder

# เปลี่ยนรหัสผ่าน
passwd                     # เปลี่ยนของตัวเอง
sudo passwd ชื่อผู้ใช้       # เปลี่ยนของคนอื่น (ต้องมีสิทธิ์ root)

# ดูรายชื่อผู้ใช้
cat /etc/passwd

# สลับผู้ใช้
su - ชื่อผู้ใช้

# เพิ่ม/ลบผู้ใช้จาก group
sudo usermod -aG groupname username
sudo gpasswd -d username groupname
```

### 6.2 การจัดการสิทธิ์ไฟล์ (Permissions)

```bash
# ดูสิทธิ์ไฟล์
ls -l

# ผลลัพธ์ตัวอย่าง: -rwxr-xr-- 1 owner group size date filename
# -    = ชนิดไฟล์ (- = ไฟล์, d = directory, l = link)
# rwx  = สิทธิ์ owner (read, write, execute)
# r-x  = สิทธิ์ group
# r--  = สิทธิ์ others

# เปลี่ยนสิทธิ์ (chmod)
chmod 755 ไฟล์         # owner:rwx, group:r-x, others:r-x
chmod +x ไฟล์          # เพิ่มสิทธิ์ execute
chmod -w ไฟล์          # ลบสิทธิ์ write
chmod -R 755 โฟลเดอร์  # เปลี่ยนทั้งหมดในโฟลเดอร์

# เปลี่ยนเจ้าของ (chown)
sudo chown username ไฟล์
sudo chown username:groupname ไฟล์
sudo chown -R username โฟลเดอร์
```

### 6.3 sudo — สิทธิ์ Administrator

```bash
# รันคำสั่งในฐานะ root
sudo คำสั่ง

# เปิด shell ในฐานะ root (ระวังการใช้งาน)
sudo -i

# ออกจาก root shell
exit

# แก้ไข sudoers (กำหนดสิทธิ์ sudo)
sudo visudo
```

---

## 7. การจัดการเครือข่าย

### 7.1 การตั้งค่าผ่าน GUI

Settings → **Network**:
- เชื่อมต่อ Wi-Fi
- ตั้งค่า VPN
- ดู IP Address ปัจจุบัน
- ตั้ง Proxy

### 7.2 คำสั่งเครือข่าย

```bash
# ดู IP Address ทั้งหมด
ip addr show
ip a               # ย่อ

# ดู IP เฉพาะ interface
ip addr show eth0
ip addr show wlan0

# ดูตาราง routing
ip route show

# ทดสอบการเชื่อมต่อ
ping google.com
ping -c 4 google.com    # ping 4 ครั้งแล้วหยุด

# ดู DNS
cat /etc/resolv.conf
nslookup google.com
dig google.com

# ดู port ที่เปิดอยู่
ss -tulpn
netstat -tulpn          # (sudo apt install net-tools)

# ดาวน์โหลดไฟล์
wget https://example.com/file.zip
curl -O https://example.com/file.zip

# ดูสถิติเครือข่าย
ifstat                  # (sudo apt install ifstat)
```

### 7.3 Firewall (UFW)

```bash
# สถานะ Firewall
sudo ufw status

# เปิด/ปิด Firewall
sudo ufw enable
sudo ufw disable

# อนุญาต/บล็อก Port
sudo ufw allow 22/tcp        # SSH
sudo ufw allow 80/tcp        # HTTP
sudo ufw deny 8080/tcp       # บล็อก port 8080

# อนุญาต Service ตามชื่อ
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# ลบกฎ
sudo ufw delete allow 8080/tcp

# รีเซ็ต Firewall
sudo ufw reset
```

### 7.4 การตั้งค่า IP แบบ Static

```bash
# แก้ไขไฟล์ Netplan
sudo nano /etc/netplan/01-netcfg.yaml
```

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
# ใช้งานการตั้งค่า
sudo netplan apply
```

---

## 8. การจัดการดิสก์และพาร์ติชัน

### 8.1 GNOME Disks (GUI)

เปิดจาก: `Super` → พิมพ์ "Disks"

- ดูข้อมูลดิสก์และพาร์ติชัน
- ฟอร์แมตไดรฟ์
- Mount / Unmount พาร์ติชัน
- ตรวจสอบสุขภาพดิสก์ (SMART Data)
- สร้าง Disk Image

### 8.2 คำสั่งจัดการดิสก์

```bash
# ดูดิสก์ทั้งหมด
lsblk
fdisk -l

# ดูการใช้พื้นที่
df -h

# ดูขนาดโฟลเดอร์
du -sh *
du -sh /home/*

# Mount ไดรฟ์
sudo mount /dev/sdb1 /mnt/mydrive

# Unmount ไดรฟ์
sudo umount /mnt/mydrive

# ตรวจสอบและซ่อมแซม filesystem
sudo fsck /dev/sdb1    # ทำเมื่อไม่ได้ mount เท่านั้น
```

### 8.3 การจัดการ USB และสื่อภายนอก

- Ubuntu จะ Mount อัตโนมัติเมื่อเสียบ USB
- ไอคอนจะปรากฏบน Dash และ Files
- คลิกขวาใน Files → **Eject** หรือ **Unmount** ก่อนถอด
- หรือกดปุ่ม ⏏ ที่ไอคอนใน Dash

---

## 9. การจัดการกระบวนการ (Process)

### 9.1 System Monitor (GUI)

เปิดจาก: `Super` → พิมพ์ "System Monitor"

- แท็บ **Processes** — ดูและปิดโปรแกรม
- แท็บ **Resources** — กราฟ CPU, RAM, เครือข่าย
- แท็บ **File Systems** — การใช้พื้นที่ดิสก์

### 9.2 คำสั่งจัดการ Process

```bash
# ดู process ทั้งหมด
ps aux
ps aux | grep firefox     # กรองเฉพาะ firefox

# Monitor แบบ real-time
top
htop                      # ใช้งานง่ายกว่า

# หยุด process
kill PID
kill -9 PID               # บังคับหยุด
killall firefox           # หยุดตามชื่อโปรแกรม
pkill firefox             # หยุดตามชื่อ

# รัน process ใน background
command &

# ดู background jobs
jobs

# นำ background job มาทำงานข้างหน้า
fg %1

# ส่ง foreground job ไป background
Ctrl + Z   # หยุดชั่วคราว
bg %1      # รันต่อใน background

# ดู process ที่ใช้ CPU/RAM มากสุด
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
```

### 9.3 Systemd — การจัดการ Service

```bash
# ดูสถานะ service
sudo systemctl status ชื่อservice

# เริ่ม/หยุด/รีสตาร์ท service
sudo systemctl start ชื่อservice
sudo systemctl stop ชื่อservice
sudo systemctl restart ชื่อservice

# เปิด/ปิด service อัตโนมัติเมื่อบูต
sudo systemctl enable ชื่อservice
sudo systemctl disable ชื่อservice

# ดู service ทั้งหมด
systemctl list-units --type=service

# ดู log ของ service
sudo journalctl -u ชื่อservice
sudo journalctl -u ชื่อservice -f    # ดูแบบ real-time
```

---

## 10. การตั้งค่าเสียงและจอภาพ

### 10.1 PipeWire / PulseAudio (เสียง)

Ubuntu 24.04 ใช้ PipeWire เป็นระบบเสียงหลัก

```bash
# ดูสถานะเสียง
pactl info
pactl list sinks short      # อุปกรณ์เล่น
pactl list sources short    # อุปกรณ์บันทึก

# ปรับระดับเสียง
pactl set-sink-volume @DEFAULT_SINK@ 50%
pactl set-sink-mute @DEFAULT_SINK@ toggle

# ติดตั้ง PavuControl (GUI เพิ่มเติม)
sudo apt install pavucontrol
```

### 10.2 การปรับแต่งจอภาพ

```bash
# ดูข้อมูล GPU
lspci | grep -i vga
glxinfo | grep "OpenGL renderer"

# ดูความละเอียดปัจจุบัน
xrandr

# เปลี่ยนความละเอียด
xrandr --output HDMI-1 --mode 1920x1080

# ตั้งค่าจอสองจอ (extend)
xrandr --output HDMI-1 --auto --right-of eDP-1
```

### 10.3 Night Light

Settings → Displays → **Night Light**
- กำหนดเวลาเปิด/ปิดอัตโนมัติ (Sunset to Sunrise)
- ปรับโทนสีอุ่น (Color Temperature)

---

## 11. การสำรองข้อมูล (Backup)

### 11.1 Déjà Dup (GUI)

เปิดจาก: `Super` → พิมพ์ "Backups"

- สำรองข้อมูลไปยัง Local folder, Network, หรือ Cloud
- ตั้งเวลาสำรองอัตโนมัติ
- กู้คืนไฟล์ที่ต้องการ
- เข้ารหัสข้อมูลสำรอง

### 11.2 Rsync — สำรองข้อมูลผ่าน Terminal

```bash
# สำรองไปยังโฟลเดอร์อื่น
rsync -av /home/username/ /backup/home/

# สำรองพร้อม progress bar
rsync -av --progress /home/username/ /backup/

# สำรองไปยังเครื่องอื่น (ผ่าน SSH)
rsync -av -e ssh /home/username/ user@192.168.1.2:/backup/

# สำรองแบบ incremental (เฉพาะที่เปลี่ยนแปลง)
rsync -av --delete /home/username/ /backup/home/
```

### 11.3 Timeshift — สำรอง System Snapshot

```bash
# ติดตั้ง Timeshift
sudo apt install timeshift

# เปิด GUI
sudo timeshift-gtk
```

Timeshift สำรองระบบทั้งหมด (คล้าย System Restore ของ Windows) เหมาะสำหรับก่อนอัปเกรดหรือทดลองการตั้งค่าใหม่

---

## 12. ความปลอดภัยของระบบ

### 12.1 การอัปเดตความปลอดภัย

```bash
# ตรวจสอบการอัปเดตความปลอดภัย
sudo apt update
sudo unattended-upgrades --dry-run

# เปิดการอัปเดตอัตโนมัติ
sudo dpkg-reconfigure unattended-upgrades
```

### 12.2 การตรวจสอบและสแกนไวรัส

```bash
# ติดตั้ง ClamAV
sudo apt install clamav clamav-daemon

# อัปเดต database
sudo freshclam

# สแกนโฟลเดอร์
sudo clamscan -r /home/username/

# สแกนและย้ายไฟล์ติดเชื้อ
sudo clamscan -r --move=/quarantine /home/
```

### 12.3 การล็อกหน้าจอ

- ล็อกทันที: `Super + L`
- ตั้งเวลาล็อกอัตโนมัติ: Settings → Privacy → Screen Lock

### 12.4 AppArmor — ระบบความปลอดภัยเพิ่มเติม

```bash
# ตรวจสอบสถานะ AppArmor
sudo aa-status

# โหลด profile ใหม่
sudo apparmor_parser -r /etc/apparmor.d/

# ดู log ของ AppArmor
sudo journalctl -k | grep apparmor
```

### 12.5 การตรวจสอบ Log ระบบ

```bash
# Log ทั้งหมด
sudo journalctl

# Log ล่าสุด 50 รายการ
sudo journalctl -n 50

# Log ของ boot ปัจจุบัน
sudo journalctl -b

# Log เฉพาะวันนี้
sudo journalctl --since today

# Auth log (การเข้าสู่ระบบ)
sudo cat /var/log/auth.log | tail -50
```

---

## 13. คีย์บอร์ดลัดที่ควรรู้

### 13.1 ระบบ (System)

| คีย์ลัด | การกระทำ |
|---|---|
| `Super` | เปิด Activities Overview |
| `Super + A` | เปิด App Grid |
| `Super + L` | ล็อกหน้าจอ |
| `Super + D` | แสดง/ซ่อน Desktop |
| `Super + M` | เปิด Notification Tray |
| `Alt + F2` | เปิด Run Dialog |
| `Ctrl + Alt + T` | เปิด Terminal |
| `Ctrl + Alt + Delete` | เปิด Power Off Dialog |
| `PrtSc` | จับภาพหน้าจอ (Screenshot) |
| `Shift + PrtSc` | จับภาพเฉพาะส่วน |

### 13.2 การจัดการหน้าต่าง (Window)

| คีย์ลัด | การกระทำ |
|---|---|
| `Super + ↑` | ขยายหน้าต่างเต็มจอ |
| `Super + ↓` | ย่อหน้าต่าง |
| `Super + ←` | จัดหน้าต่างชิดซ้าย (Half screen) |
| `Super + →` | จัดหน้าต่างชิดขวา (Half screen) |
| `Alt + F4` | ปิดหน้าต่าง |
| `Alt + F10` | Toggle เต็มจอ |
| `Super + H` | ซ่อนหน้าต่าง (Minimize) |
| `Alt + Tab` | สลับหน้าต่าง |
| `Super + Tab` | สลับแอป |
| `Alt + ` ` ` ` | สลับหน้าต่างของแอปเดียวกัน |

### 13.3 ภายใน แอปพลิเคชัน

| คีย์ลัด | การกระทำ |
|---|---|
| `Ctrl + C` | คัดลอก |
| `Ctrl + X` | ตัด |
| `Ctrl + V` | วาง |
| `Ctrl + Z` | ยกเลิก |
| `Ctrl + Y` | ทำซ้ำ |
| `Ctrl + A` | เลือกทั้งหมด |
| `Ctrl + S` | บันทึก |
| `Ctrl + F` | ค้นหา |
| `Ctrl + W` | ปิด Tab/หน้าต่าง |
| `Ctrl + Q` | ออกจากแอป |
| `F11` | Toggle เต็มจอ (ในหลายแอป) |

### 13.4 Terminal

| คีย์ลัด | การกระทำ |
|---|---|
| `Ctrl + C` | ยกเลิกคำสั่งที่รันอยู่ |
| `Ctrl + D` | ออกจาก Terminal |
| `Ctrl + L` | ล้างหน้าจอ (clear) |
| `Ctrl + R` | ค้นหาคำสั่งที่เคยพิมพ์ |
| `Tab` | Auto-complete |
| `↑` / `↓` | คำสั่งก่อนหน้า/ถัดไป |
| `Ctrl + A` | ไปต้นบรรทัด |
| `Ctrl + E` | ไปท้ายบรรทัด |
| `Ctrl + U` | ลบทั้งบรรทัด |
| `Ctrl + Shift + C` | คัดลอกใน Terminal |
| `Ctrl + Shift + V` | วางใน Terminal |
| `Ctrl + Shift + T` | เปิด Tab ใหม่ |

---

## 14. การแก้ปัญหาเบื้องต้น

### 14.1 ระบบค้าง / ไม่ตอบสนอง

```bash
# บังคับปิดโปรแกรมที่ค้าง
xkill       # คลิกหน้าต่างที่ต้องการปิด

# หรือกด Alt + F2 แล้วพิมพ์ xkill

# หาและปิด process ที่ใช้ทรัพยากรมาก
top
# กด 'k' แล้วพิมพ์ PID

# รีสตาร์ท GNOME Shell (ไม่ต้อง logout)
Alt + F2 → พิมพ์ r → Enter   # สำหรับ X11 session
```

### 14.2 แก้ปัญหาแพ็กเกจ

```bash
# แก้ปัญหา package broken
sudo apt --fix-broken install
sudo dpkg --configure -a

# ล้าง cache
sudo apt clean
sudo apt autoclean

# รีเซ็ต package database
sudo rm /var/lib/dpkg/lock-frontend
sudo dpkg --configure -a
```

### 14.3 แก้ปัญหาการบูต

หากบูตไม่ได้ ให้กด `Shift` ค้างระหว่างบูตเพื่อเข้า GRUB Menu:
- เลือก **Advanced options** → **recovery mode**
- เลือก **root** เพื่อเข้า shell
- ใช้ `fsck` ตรวจ filesystem
- ใช้ `dpkg --configure -a` แก้แพ็กเกจ

### 14.4 แก้ปัญหาไดรเวอร์

```bash
# ดูอุปกรณ์ทั้งหมด
lspci
lsusb

# ดูไดรเวอร์ที่ติดตั้ง
ubuntu-drivers list

# ติดตั้งไดรเวอร์ที่แนะนำอัตโนมัติ
sudo ubuntu-drivers autoinstall

# ติดตั้งไดรเวอร์ NVIDIA
sudo apt install nvidia-driver-535
```

### 14.5 แก้ปัญหาเสียงไม่ออก

```bash
# รีสตาร์ท PipeWire
systemctl --user restart pipewire
systemctl --user restart pipewire-pulse

# ตรวจสอบ Mute
pactl list sinks | grep -E "Name|Mute|Volume"

# รีสตาร์ท PulseAudio (สำรอง)
pulseaudio --kill
pulseaudio --start
```

### 14.6 แก้ปัญหา Wi-Fi ไม่เชื่อมต่อ

```bash
# รีสตาร์ท NetworkManager
sudo systemctl restart NetworkManager

# ดูสถานะ Wi-Fi
nmcli device status
nmcli radio wifi

# เปิด Wi-Fi
nmcli radio wifi on

# ดูเครือข่ายที่มี
nmcli device wifi list

# เชื่อมต่อ
nmcli device wifi connect "ชื่อ_WiFi" password "รหัสผ่าน"
```

---

## ข้อมูลอ้างอิงเพิ่มเติม

- **Ubuntu Official Documentation:** https://help.ubuntu.com
- **Ubuntu Community Help:** https://help.ubuntu.com/community
- **Ask Ubuntu (Q&A):** https://askubuntu.com
- **Ubuntu Forums:** https://ubuntuforums.org
- **Man Pages (คู่มือคำสั่ง):** พิมพ์ `man ชื่อคำสั่ง` ใน Terminal

---

*คู่มือนี้จัดทำสำหรับ Ubuntu Desktop 24.04 LTS (Noble Numbat)*  
*เหมาะสำหรับผู้ใช้ระดับเริ่มต้นถึงกลาง*
