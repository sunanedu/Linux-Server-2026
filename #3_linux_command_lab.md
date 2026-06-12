# แลปปฏิบัติ: คำสั่ง Linux Ubuntu 24.04 LTS

> **สภาพแวดล้อม:** Ubuntu 24.04 LTS บน Virtual Machine (VM)
> **ระดับ:** ผู้เริ่มต้น → กลาง
> **ปรับปรุงล่าสุด:** พฤษภาคม 2026

---

## ข้อมูลก่อนเริ่มแลป

เครื่องที่ใช้ในแลปนี้:
- **OS:** Ubuntu 24.04 LTS (Noble Numbat)
- **สภาพแวดล้อม:** VM (VirtualBox / VMware / Hyper-V)
- **ตั้งค่า root password ไว้แล้ว**
- **Shell เริ่มต้น:** bash

สัญลักษณ์ใน Terminal ที่ต้องรู้:

```
$   → prompt ของ user ทั่วไป (ยังไม่ได้สิทธิ์ root)
#   → prompt ของ root (มีสิทธิ์สูงสุด)
~   → ย่อจาก home directory ของผู้ใช้ปัจจุบัน
```

---

## ส่วนที่ 0: จัดการ Root — เข้า ออก เปลี่ยนรหัสผ่าน

### 0.1 เปลี่ยนรหัสผ่าน root

```bash
sudo passwd root
```

**Output ที่เห็น:**
```
[sudo] password for <ชื่อuser>:    ← พิมพ์รหัสผ่านของ user ตัวเอง
New password:                       ← พิมพ์รหัส root ใหม่
Retype new password:                ← พิมพ์ซ้ำ
passwd: password updated successfully
```

> **💡 ทริป:** รหัสผ่านที่พิมพ์จะไม่แสดงอักขระบนหน้าจอเลย (ไม่มีดอกจัน) — นี่คือพฤติกรรมปกติของ Linux เพื่อความปลอดภัย

---

### 0.2 เข้าใช้งานในฐานะ root (su)

**`su` — Switch User ไปที่ root:**

```bash
su -
```

**Output ที่เห็น:**
```
Password:     ← พิมพ์รหัสผ่าน root
root@ubuntu:~#
```

เครื่องหมาย `#` แสดงว่าอยู่ใน root shell แล้ว

> **ข้อแตกต่าง `su` กับ `su -`:**
> - `su` — เปลี่ยนเป็น root แต่ยัง environment ของ user เดิม (PATH, HOME ยังเดิม)
> - `su -` — เปลี่ยนเป็น root พร้อม environment ใหม่ของ root (Login Shell) **แนะนำให้ใช้นี้**

---

### 0.3 ออกจาก root กลับไป user เดิม

```bash
exit
```

หรือกด `Ctrl + D`

**Output ที่เห็น:**
```
logout
user@ubuntu:~$
```

---

### 0.4 รันคำสั่งเดียวในฐานะ root โดยไม่เข้า root shell

```bash
sudo <คำสั่ง>
```

ตัวอย่าง:
```bash
sudo apt update
sudo cat /etc/shadow
sudo systemctl restart networking
```

> **💡 ทริป:** `sudo` ย่อจาก "Superuser Do" — รันคำสั่งนั้นด้วยสิทธิ์ root แค่ครั้งเดียว ปลอดภัยกว่าการ `su -` แล้วทำงานทั้งวันในฐานะ root

---

### 0.5 ตรวจสอบว่าตัวเองเป็นใคร

```bash
whoami
```

**Output ตอนเป็น user:**
```
student
```

**Output ตอนเป็น root:**
```
root
```

```bash
id
```

**Output:**
```
uid=0(root) gid=0(root) groups=0(root)
```

หรือสำหรับ user ทั่วไป:
```
uid=1000(student) gid=1000(student) groups=1000(student),4(adm),24(cdrom),27(sudo)
```

> **📖 ศึกษาต่อ:** UID 0 คือ root เสมอ — เหตุใด Linux จึงระบุสิทธิ์ด้วยตัวเลข UID/GID แทนชื่อ? ลองค้นหาเรื่อง "Linux UID GID permission model"

---

## ส่วนที่ 1: คำสั่งพื้นฐาน — ระบบไฟล์และการนำทาง

### 1.1 ดูตำแหน่งปัจจุบัน

```bash
pwd
```

**Output:**
```
/home/student
```

---

### 1.2 แสดงรายชื่อไฟล์และโฟลเดอร์

```bash
ls
```

**Output:**
```
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
```

แบบละเอียด (Long format):
```bash
ls -l
```

**Output:**
```
total 32
drwxr-xr-x 2 student student 4096 May  1 10:00 Desktop
drwxr-xr-x 2 student student 4096 May  1 10:00 Documents
drwxr-xr-x 2 student student 4096 May  1 10:00 Downloads
drwxr-xr-x 2 student student 4096 May  1 10:00 Music
```

แสดงไฟล์ซ่อน (ขึ้นต้นด้วย `.`):
```bash
ls -la
```

**Output:**
```
total 64
drwxr-x--- 10 student student 4096 May  1 10:05 .
drwxr-xr-x  3 root    root    4096 Apr 30 08:00 ..
-rw-------  1 student student  220 Apr 30 08:00 .bash_logout
-rw-r--r--  1 student student 3526 Apr 30 08:00 .bashrc
-rw-r--r--  1 student student  807 Apr 30 08:00 .profile
drwxr-xr-x  2 student student 4096 May  1 10:00 Desktop
```

แสดงขนาดแบบอ่านง่าย (Human-readable):
```bash
ls -lh
```

**Output:**
```
total 32K
drwxr-xr-x 2 student student 4.0K May  1 10:00 Desktop
drwxr-xr-x 2 student student 4.0K May  1 10:00 Documents
```

> **💡 ทริป:** Options ใช้ร่วมกันได้ `ls -lah` = long + all + human-readable ในคำสั่งเดียว

---

### 1.3 เปลี่ยน Directory

```bash
cd Documents
cd /var/log
cd ~          # กลับ home
cd ..         # ขึ้นไปโฟลเดอร์แม่
cd -          # กลับไป directory ก่อนหน้า
```

**Output ของ `cd -`:**
```
/home/student
```

---

### 1.4 สร้างโฟลเดอร์

```bash
mkdir mylab
mkdir -p projects/web/css    # สร้างทุก level พร้อมกัน
```

**Output ของ `ls -l`:**
```
drwxr-xr-x 2 student student 4096 May  1 10:10 mylab
drwxr-xr-x 3 student student 4096 May  1 10:10 projects
```

---

### 1.5 สร้างไฟล์เปล่า

```bash
touch notes.txt
touch file1.txt file2.txt file3.txt    # สร้างหลายไฟล์พร้อมกัน
```

**Output ของ `ls -l notes.txt`:**
```
-rw-r--r-- 1 student student 0 May  1 10:15 notes.txt
```

ขนาด `0` เพราะไฟล์เปล่า

---

### 1.6 คัดลอก ย้าย ลบ

**คัดลอก:**
```bash
cp notes.txt notes-backup.txt
cp -r projects projects-backup    # คัดลอกทั้งโฟลเดอร์ (-r = recursive)
```

**ย้าย / เปลี่ยนชื่อ:**
```bash
mv notes.txt documents/notes.txt    # ย้ายไฟล์
mv oldname.txt newname.txt           # เปลี่ยนชื่อ
```

**ลบ:**
```bash
rm notes-backup.txt                  # ลบไฟล์
rm -r projects-backup                # ลบโฟลเดอร์และสิ่งที่อยู่ข้างใน
rm -rf /tmp/testdir                  # ลบโดยไม่ถามยืนยัน (ระวัง!)
```

> **⚠️ คำเตือน:** Linux ไม่มี Recycle Bin สำหรับ `rm` — ไฟล์หายถาวร ควรระวัง `rm -rf` เป็นพิเศษ

> **📖 ศึกษาต่อ:** เหตุใด `rm -rf /` จึงเป็นคำสั่งอันตรายที่สุด? Ubuntu มีกลไกป้องกันอย่างไร? ลองค้นหา "rm -rf / protection Linux"

---

### 1.7 อ่านและเขียนไฟล์

**แสดงเนื้อหาไฟล์:**
```bash
cat /etc/hostname
```

**Output:**
```
ubuntu
```

**แสดงพร้อมหมายเลขบรรทัด:**
```bash
cat -n /etc/os-release
```

**Output:**
```
     1	PRETTY_NAME="Ubuntu 24.04.2 LTS"
     2	NAME="Ubuntu"
     3	VERSION_ID="24.04"
     4	VERSION="24.04.2 LTS (Noble Numbat)"
     5	VERSION_CODENAME=noble
     6	ID=ubuntu
     7	ID_LIKE=debian
     8	HOME_URL="https://www.ubuntu.com/"
```

**ดูไฟล์ยาว ๆ ทีละหน้า:**
```bash
less /var/log/syslog
```

ใน `less`: กด `Space` เลื่อนหน้า, กด `q` ออก, กด `/คำ` ค้นหา

**ดูแค่ 10 บรรทัดแรก:**
```bash
head /var/log/syslog
head -n 20 /var/log/syslog    # 20 บรรทัดแรก
```

**ดูแค่ 10 บรรทัดสุดท้าย:**
```bash
tail /var/log/syslog
tail -n 20 /var/log/syslog
tail -f /var/log/syslog       # ติดตาม log แบบ real-time (กด Ctrl+C เพื่อออก)
```

**เขียนข้อความลงไฟล์:**
```bash
echo "Hello Ubuntu" > hello.txt          # เขียนทับ
echo "Line 2" >> hello.txt               # ต่อท้าย
cat hello.txt
```

**Output:**
```
Hello Ubuntu
Line 2
```

---

### 1.8 ค้นหาไฟล์

**`find` — ค้นหาไฟล์ในระบบ:**
```bash
find /home -name "*.txt"
find /etc -name "*.conf" -type f
find /var/log -name "*.log" -mtime -7    # ไฟล์ที่แก้ในช่วง 7 วัน
```

**Output:**
```
/home/student/notes.txt
/home/student/hello.txt
```

**`locate` — ค้นหาจาก database (เร็วกว่า find):**
```bash
sudo updatedb          # อัปเดต database ก่อน
locate passwd
```

**Output:**
```
/etc/passwd
/etc/pam.d/passwd
/usr/bin/passwd
/usr/share/doc/passwd
```

**`which` — หาว่าคำสั่งอยู่ที่ไหน:**
```bash
which python3
which bash
```

**Output:**
```
/usr/bin/python3
/usr/bin/bash
```

---

## ส่วนที่ 2: การจัดการ Text และ Pipe

### 2.1 grep — ค้นหาข้อความในไฟล์

```bash
grep "root" /etc/passwd
```

**Output:**
```
root:x:0:0:root:/root:/bin/bash
```

ค้นหาแบบไม่แยก upper/lowercase:
```bash
grep -i "ubuntu" /etc/os-release
```

แสดงหมายเลขบรรทัด:
```bash
grep -n "PermitRootLogin" /etc/ssh/sshd_config
```

**Output:**
```
34:#PermitRootLogin prohibit-password
```

ค้นหาแบบ recursive ในทุกไฟล์:
```bash
grep -r "127.0.0.1" /etc/
```

---

### 2.2 Pipe `|` — ส่งผลลัพธ์ต่อกัน

```bash
# นับจำนวนไฟล์ใน /etc
ls /etc | wc -l
```

**Output:**
```
187
```

```bash
# ดู process ของ sshd
ps aux | grep sshd
```

**Output:**
```
root     1234  0.0  0.1  12345  6789 ?  Ss   10:00   0:00 sshd: /usr/sbin/sshd -D
student  2345  0.0  0.0   6789  1234 pts/0 S+  10:05   0:00 grep --color=auto sshd
```

```bash
# ดู 5 process ที่ใช้ CPU สูงสุด
ps aux --sort=-%cpu | head -6
```

```bash
# ดู log error ล่าสุด 20 บรรทัด
grep "error" /var/log/syslog | tail -20
```

---

### 2.3 sort, uniq, wc

```bash
# นับจำนวนบรรทัด คำ และตัวอักษรในไฟล์
wc /etc/passwd
```

**Output:**
```
 33  66 1872 /etc/passwd
```
(33 บรรทัด, 66 คำ, 1872 ตัวอักษร)

```bash
# เรียงลำดับ
cat /etc/passwd | sort
sort -r /etc/passwd                 # เรียงกลับ
sort -t: -k3 -n /etc/passwd         # เรียงตาม UID (คอลัมน์ 3)
```

```bash
# หาบรรทัดที่ซ้ำกัน
sort testfile.txt | uniq -c         # นับความถี่
sort testfile.txt | uniq -d         # แสดงเฉพาะที่ซ้ำ
```

---

## ส่วนที่ 3: สิทธิ์ไฟล์ (File Permissions)

### 3.1 ทำความเข้าใจ Permission

```bash
ls -l hello.txt
```

**Output:**
```
-rw-r--r-- 1 student student 24 May 1 10:20 hello.txt
```

ความหมายของแต่ละตัวอักษร:

```
- rw- r-- r--
│ │   │   └── Others: อ่านได้
│ │   └────── Group: อ่านได้
│ └────────── Owner: อ่าน+เขียน
└──────────── ประเภท: - = file, d = directory, l = symlink
```

| ตัวอักษร | ความหมาย | ค่าเลข |
|----------|----------|--------|
| `r` | read (อ่าน) | 4 |
| `w` | write (เขียน) | 2 |
| `x` | execute (รัน) | 1 |
| `-` | ไม่มีสิทธิ์ | 0 |

---

### 3.2 chmod — เปลี่ยน Permission

**แบบเลข (Octal):**
```bash
chmod 755 myscript.sh     # rwxr-xr-x
chmod 644 config.txt      # rw-r--r--
chmod 600 secret.key      # rw------- (เจ้าของอ่าน/เขียนได้คนเดียว)
chmod 777 public.sh       # rwxrwxrwx (ทุกคนทำได้ทุกอย่าง — ไม่แนะนำ)
```

**แบบตัวอักษร:**
```bash
chmod +x myscript.sh      # เพิ่มสิทธิ์ execute ให้ทุกคน
chmod u+x myscript.sh     # เพิ่ม execute เฉพาะ owner
chmod g-w config.txt      # ลบ write ของ group
chmod o= secret.txt       # ลบสิทธิ์ทั้งหมดของ others
```

**ทดสอบ:**
```bash
echo "#!/bin/bash" > test.sh
echo "echo 'Hello from script!'" >> test.sh
chmod +x test.sh
./test.sh
```

**Output:**
```
Hello from script!
```

---

### 3.3 chown — เปลี่ยนเจ้าของ

```bash
sudo chown root:root hello.txt          # เปลี่ยนเจ้าของเป็น root
sudo chown student:student hello.txt    # เปลี่ยนกลับ
sudo chown -R student:student /home/student/projects    # recursive
```

**Output ของ `ls -l` หลัง chown:**
```
-rw-r--r-- 1 root root 24 May 1 10:20 hello.txt
```

> **📖 ศึกษาต่อ:** SUID, SGID และ Sticky Bit คืออะไร? ทำไม `/usr/bin/passwd` จึงมี `s` แทน `x`? ลองรัน `ls -l /usr/bin/passwd` แล้วสังเกต

---

## ส่วนที่ 4: การจัดการ Process

### 4.1 ดู Process ที่กำลังทำงาน

```bash
ps aux
```

**Output (บางส่วน):**
```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.4 167280 13456 ?        Ss   10:00   0:01 /sbin/init
root         234  0.0  0.2  12345  6789 ?        Ss   10:00   0:00 /usr/sbin/sshd
student     1234  0.0  0.1   7890  3456 pts/0    Ss   10:05   0:00 bash
student     1567  0.0  0.0   8901  1234 pts/0    R+   10:10   0:00 ps aux
```

**`top` — ดู Process แบบ Real-time:**
```bash
top
```

กด `q` เพื่อออก, กด `k` แล้วพิมพ์ PID เพื่อ Kill process

**`htop` — top ที่สวยกว่า (ต้องติดตั้งก่อน):**
```bash
sudo apt install htop -y
htop
```

---

### 4.2 หยุดและจัดการ Process

```bash
kill 1234              # ส่ง signal SIGTERM (ขอให้หยุด)
kill -9 1234           # ส่ง SIGKILL (บังคับหยุดทันที)
killall firefox        # kill ทุก process ชื่อ firefox
pkill -f "python3"     # kill process ที่มีคำว่า python3 ใน command
```

**ทดสอบ:**
```bash
sleep 9999 &           # รัน process ใน background
jobs                   # ดู background jobs
```

**Output:**
```
[1] 2345
[1]+ Running    sleep 9999 &
```

```bash
kill %1                # kill job หมายเลข 1
```

---

### 4.3 Background และ Foreground

```bash
ping google.com &            # รันใน background
fg                           # เอากลับมา foreground
bg                           # ส่งไป background
```

กด `Ctrl + Z` เพื่อ pause process แล้วค่อยรัน `bg` หรือ `fg`

---

## ส่วนที่ 5: การจัดการ Package (APT)

### 5.1 อัปเดตและติดตั้ง

```bash
# อัปเดตรายชื่อ package (ต้องรันก่อนติดตั้งเสมอ)
sudo apt update
```

**Output:**
```
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:2 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Get:3 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Fetched 126 kB in 2s (63.0 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
45 packages can be upgraded.
```

```bash
# อัปเกรด package ทั้งหมด
sudo apt upgrade -y

# ติดตั้ง package
sudo apt install curl wget git vim net-tools -y

# ลบ package
sudo apt remove vim -y
sudo apt purge vim -y         # ลบพร้อม config files
sudo apt autoremove -y        # ลบ package ที่ไม่ได้ใช้แล้ว
```

---

### 5.2 ค้นหา package

```bash
apt search nginx
apt show nginx                # ดูรายละเอียด package
apt list --installed          # รายการที่ติดตั้งแล้ว
apt list --installed | grep ssh
```

**Output ของ `apt show nginx` (บางส่วน):**
```
Package: nginx
Version: 1.24.0-2ubuntu7
Priority: optional
Section: web
Maintainer: Ubuntu Developers
Description: small, powerful, scalable web/proxy server
```

> **📖 ศึกษาต่อ:** ความแตกต่างระหว่าง `apt` กับ `apt-get`? และ `snap` กับ `flatpak` คืออะไร? ทำไม Ubuntu 24.04 จึงใช้ snap เป็นค่าเริ่มต้นสำหรับบางแอป?

---

## ส่วนที่ 6: ข้อมูลระบบ

### 6.1 ข้อมูลระบบปฏิบัติการ

```bash
uname -a
```

**Output:**
```
Linux ubuntu 6.8.0-51-generic #52-Ubuntu SMP PREEMPT_DYNAMIC Thu Dec  5 13:09:44 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

```bash
cat /etc/os-release
```

**Output:**
```
PRETTY_NAME="Ubuntu 24.04.2 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.2 LTS (Noble Numbat)"
```

```bash
lsb_release -a
```

**Output:**
```
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 24.04.2 LTS
Release:	24.04
Codename:	noble
```

---

### 6.2 ข้อมูล CPU, RAM, Disk

**CPU:**
```bash
lscpu | grep -E "Model name|CPU\(s\)|Thread"
```

**Output:**
```
CPU(s):                  2
Thread(s) per core:      1
Model name:              Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
```

**RAM:**
```bash
free -h
```

**Output:**
```
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       512Mi       2.8Gi        12Mi       512Mi       3.1Gi
Swap:          2.0Gi          0B       2.0Gi
```

**Disk:**
```bash
df -h
```

**Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        49G  8.5G   38G  19% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
/dev/sda15      105M  6.1M   99M   6% /boot/efi
```

```bash
du -sh /var/log            # ขนาดโฟลเดอร์
du -sh /home/*             # ขนาดแต่ละ home
```

**Output:**
```
24M	/var/log
512K	/home/student
```

---

### 6.3 ข้อมูล Hardware

```bash
lspci                      # อุปกรณ์ PCI ทั้งหมด
lsusb                      # อุปกรณ์ USB ทั้งหมด
lshw -short                # รายการ hardware สรุป (ต้อง sudo)
```

```bash
# uptime — เครื่องเปิดมานานแค่ไหน
uptime
```

**Output:**
```
 10:30:00 up  2:30,  1 user,  load average: 0.15, 0.10, 0.08
```

---

## ส่วนที่ 7: Network Commands — คำสั่ง Network ที่ต้องรู้ทั้งหมด

> **ส่วนนี้สำคัญมาก** — Admin Linux ต้องรู้คำสั่งเหล่านี้ทุกข้อ

---

### 7.1 ดู IP Address และ Network Interface

**วิธีใหม่ (แนะนำ) — `ip` command:**
```bash
ip addr show
```

**Output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 08:00:27:ab:cd:ef brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic enp0s3
    inet6 fe80::a00:27ff:feab:cdef/64 scope link
```

ดูแบบสั้น:
```bash
ip addr show enp0s3          # เฉพาะ interface ที่ต้องการ
ip -4 addr                   # IPv4 เท่านั้น
ip -6 addr                   # IPv6 เท่านั้น
```

**วิธีเก่า (ยังใช้ได้แต่ deprecated) — `ifconfig`:**
```bash
sudo apt install net-tools -y    # ต้องติดตั้งก่อน
ifconfig
```

**Output:**
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 08:00:27:ab:cd:ef  txqueuelen 1000  (Ethernet)
```

> **💡 ทริป:** ใน Ubuntu 24.04 ชื่อ interface จะเป็น `enp0s3`, `ens33` หรือ `eth0` ขึ้นกับ Hypervisor ที่ใช้ ไม่ใช่ `eth0` แบบเดิมเสมอไป

---

### 7.2 ดู Routing Table

```bash
ip route show
```

**Output:**
```
default via 192.168.1.1 dev enp0s3 proto dhcp src 192.168.1.100 metric 100
192.168.1.0/24 dev enp0s3 proto kernel scope link src 192.168.1.100
```

ดู default gateway:
```bash
ip route | grep default
```

**Output:**
```
default via 192.168.1.1 dev enp0s3
```

วิธีเก่า:
```bash
route -n
```

**Output:**
```
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 enp0s3
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 enp0s3
```

---

### 7.3 ping — ทดสอบการเชื่อมต่อ

```bash
ping google.com               # ping ต่อเนื่อง (กด Ctrl+C เพื่อหยุด)
ping -c 4 google.com          # ping 4 ครั้ง
ping -c 4 8.8.8.8             # ping ด้วย IP โดยตรง
ping -i 0.5 -c 10 192.168.1.1 # ping ทุก 0.5 วินาที 10 ครั้ง
```

**Output:**
```
PING google.com (142.250.185.78) 56(84) bytes of data.
64 bytes from kix06s08-in-f14.1e100.net (142.250.185.78): icmp_seq=1 ttl=118 time=15.2 ms
64 bytes from kix06s08-in-f14.1e100.net (142.250.185.78): icmp_seq=2 ttl=118 time=14.8 ms
64 bytes from kix06s08-in-f14.1e100.net (142.250.185.78): icmp_seq=3 ttl=118 time=15.1 ms
64 bytes from kix06s08-in-f14.1e100.net (142.250.185.78): icmp_seq=4 ttl=118 time=14.9 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 14.800/15.000/15.200/0.153 ms
```

---

### 7.4 traceroute / tracepath — ติดตามเส้นทาง Packet

```bash
traceroute google.com
tracepath google.com          # ไม่ต้องใช้ sudo
```

**Output:**
```
tracepath google.com
 1?: [LOCALHOST]                      pmtu 1500
 1:  192.168.1.1                      1.234ms
 2:  10.0.0.1                         5.678ms
 3:  172.16.0.1                      10.123ms
...
 8:  kix06s08-in-f14.1e100.net       15.200ms reached
```

> **💡 ทริป:** ถ้าเห็น `* * *` บางบรรทัด หมายความว่า Router นั้นไม่ตอบ ICMP ไม่ได้แปลว่า network ขาด

---

### 7.5 nslookup / dig — สอบถาม DNS

```bash
nslookup google.com
```

**Output:**
```
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	google.com
Address: 142.250.185.78
Name:	google.com
Address: 2404:6800:4005:813::200e
```

**`dig` — รายละเอียดมากกว่า:**
```bash
dig google.com
```

**Output:**
```
; <<>> DiG 9.18.28-0ubuntu0.24.04.1-Ubuntu <<>> google.com
;; ANSWER SECTION:
google.com.		300	IN	A	142.250.185.78

;; Query time: 15 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
```

ดูเฉพาะ IP:
```bash
dig +short google.com
dig +short MX gmail.com        # ดู Mail Server
dig +short NS google.com       # ดู Name Server
```

**Output ของ `dig +short google.com`:**
```
142.250.185.78
```

ดู Reverse DNS (IP → ชื่อโดเมน):
```bash
dig -x 8.8.8.8
```

**Output:**
```
;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.	21599	IN	PTR	dns.google.
```

---

### 7.6 ss / netstat — ดู Port และ Connection

**`ss` — วิธีใหม่ (แนะนำ):**
```bash
ss -tuln                       # ดู port ที่ listen อยู่ (TCP+UDP+Numeric)
```

**Output:**
```
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port
tcp    LISTEN  0       128            0.0.0.0:22           0.0.0.0:*
tcp    LISTEN  0       128               [::]:22              [::]:*
udp    UNCONN  0       0            127.0.0.53:53           0.0.0.0:*
```

```bash
ss -tulnp                      # เพิ่ม process name
ss -s                          # สรุปสถิติ
ss -tp                         # ดู TCP connection ที่ established
```

**Output ของ `ss -tulnp` (ต้อง sudo):**
```
Netid  State   Recv-Q  Send-Q  Local Address:Port   Peer Address:Port  Process
tcp    LISTEN  0       128          0.0.0.0:22          0.0.0.0:*      users:(("sshd",pid=1234,fd=3))
```

**`netstat` — วิธีเก่า (ต้องติดตั้ง net-tools):**
```bash
netstat -tuln
netstat -tulnp                 # พร้อม process
netstat -rn                    # routing table
```

---

### 7.7 curl / wget — ดาวน์โหลดและทดสอบ HTTP

**`curl` — ส่ง HTTP Request:**
```bash
curl https://example.com
curl -I https://google.com              # ดู HTTP Header เท่านั้น
curl -o output.html https://example.com # บันทึกลงไฟล์
curl -L https://example.com            # ตาม Redirect
curl -s https://api.ipify.org          # ดู Public IP ของตัวเอง
```

**Output ของ `curl -I https://google.com`:**
```
HTTP/2 301
location: https://www.google.com/
content-type: text/html; charset=UTF-8
date: Wed, 01 May 2026 10:30:00 GMT
server: gws
```

**Output ของ `curl -s https://api.ipify.org`:**
```
203.150.xxx.xxx
```

**`wget` — ดาวน์โหลดไฟล์:**
```bash
wget https://example.com/file.zip
wget -O myfile.zip https://example.com/file.zip    # ตั้งชื่อไฟล์เอง
wget -q --spider https://google.com                # ตรวจว่า URL ใช้ได้ไหม
```

---

### 7.8 ssh — เชื่อมต่อเครื่องอื่นแบบ Secure

```bash
ssh user@192.168.1.50                    # เชื่อมต่อด้วย password
ssh -p 2222 user@192.168.1.50           # ระบุ port (ถ้าไม่ใช่ 22)
ssh -i ~/.ssh/id_rsa user@server.com    # ใช้ SSH key
ssh -v user@192.168.1.50               # Verbose (debug)
```

**Output (ครั้งแรกที่เชื่อมต่อ):**
```
The authenticity of host '192.168.1.50 (192.168.1.50)' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.50' (ED25519) to the list of known hosts.
user@192.168.1.50's password:
```

**สร้าง SSH Key Pair:**
```bash
ssh-keygen -t ed25519 -C "student@ubuntu"
```

**Output:**
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/student/.ssh/id_ed25519): [Enter]
Enter passphrase (empty for no passphrase): [Enter]
Enter same passphrase again: [Enter]
Your identification has been saved in /home/student/.ssh/id_ed25519
Your public key has been saved in /home/student/.ssh/id_ed25519.pub
```

**คัดลอก Public Key ไปยัง Server:**
```bash
ssh-copy-id user@192.168.1.50
```

---

### 7.9 scp / rsync — คัดลอกไฟล์ผ่าน Network

**`scp` — คัดลอกแบบง่าย:**
```bash
# ส่งไฟล์ไปยัง server
scp hello.txt user@192.168.1.50:/home/user/

# ดาวน์โหลดไฟล์จาก server
scp user@192.168.1.50:/etc/hostname ./

# คัดลอกทั้งโฟลเดอร์
scp -r projects/ user@192.168.1.50:/home/user/
```

**`rsync` — sync ไฟล์ (ฉลาดกว่า scp):**
```bash
rsync -avz projects/ user@192.168.1.50:/home/user/projects/
rsync -avz --delete projects/ user@192.168.1.50:/home/user/projects/
```

> **💡 ทริป:** `rsync` ส่งเฉพาะไฟล์ที่เปลี่ยนแปลง ประหยัด bandwidth กว่า `scp` มาก เหมาะกับการ backup

---

### 7.10 ตั้งค่า IP แบบ Static ด้วย Netplan

Ubuntu 24.04 ใช้ **Netplan** จัดการ network configuration:

```bash
# ดูไฟล์ config ปัจจุบัน
ls /etc/netplan/
cat /etc/netplan/50-cloud-init.yaml
```

**Output:**
```
network:
    ethernets:
        enp0s3:
            dhcp4: true
    version: 2
```

**ตั้ง IP แบบ Static:**
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

แก้ไขเป็น:
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

นำไปใช้:
```bash
sudo netplan apply
```

**Output (ถ้าสำเร็จไม่มีข้อความ):**
```
(ไม่มี output = สำเร็จ)
```

ถ้ามี error:
```
Error in network definition /etc/netplan/50-cloud-init.yaml line 5 column 6: ...
```

---

### 7.11 ดูและจัดการ DNS

**ดู DNS ที่ใช้งานอยู่:**
```bash
cat /etc/resolv.conf
```

**Output:**
```
nameserver 127.0.0.53
options edns0 trust-ad
search .
```

```bash
resolvectl status               # ดู DNS แบบละเอียด
resolvectl query google.com     # ทดสอบ DNS query
```

**Output ของ `resolvectl status`:**
```
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (enp0s3)
    Current Scopes: DNS
         Protocols: +DefaultRoute +LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 8.8.8.8
       DNS Servers: 8.8.8.8 8.8.4.4
```

---

### 7.12 Firewall — UFW (Uncomplicated Firewall)

```bash
# ตรวจสอบสถานะ
sudo ufw status
```

**Output (ถ้ายังไม่เปิด):**
```
Status: inactive
```

```bash
# เปิด UFW
sudo ufw enable

# อนุญาต SSH ก่อนเปิด firewall (สำคัญมาก!)
sudo ufw allow ssh
sudo ufw allow 22/tcp

# อนุญาต port อื่น ๆ
sudo ufw allow 80/tcp          # HTTP
sudo ufw allow 443/tcp         # HTTPS
sudo ufw allow 3306/tcp        # MySQL
sudo ufw allow from 192.168.1.0/24    # อนุญาตทั้ง subnet

# ปิดกั้น port
sudo ufw deny 23/tcp           # Telnet

# ดู rules ทั้งหมด
sudo ufw status numbered
```

**Output ของ `sudo ufw status verbose`:**
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

```bash
# ลบ rule (ใช้หมายเลขจาก status numbered)
sudo ufw delete 3

# ปิด UFW
sudo ufw disable

# Reset ทุก rule
sudo ufw reset
```

> **⚠️ คำเตือน:** ถ้าใช้งานผ่าน SSH ต้อง `allow ssh` ก่อน `enable` UFW เสมอ ไม่งั้นจะ lock ตัวเองออก!

---

### 7.13 ดู Network Statistics และ Monitor

```bash
# ดู network interface statistics
ip -s link show enp0s3
```

**Output:**
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 ...
    RX:  bytes  packets  errors dropped  missed   mcast
    1234567    12345       0       0       0       0
    TX:  bytes  packets  errors dropped carrier collsns
     234567     2345       0       0       0       0
```

```bash
# ดู bandwidth แบบ real-time (ต้องติดตั้ง)
sudo apt install iftop -y
sudo iftop -i enp0s3

# หรือใช้ nload
sudo apt install nload -y
nload enp0s3
```

```bash
# ดูว่า service ไหน listen port อะไร
sudo ss -tulnp | grep LISTEN
```

---

### 7.14 ทดสอบ Port ด้วย nc (netcat)

```bash
# ทดสอบว่า port เปิดอยู่ไหม
nc -zv 192.168.1.1 22            # ทดสอบ SSH port
nc -zv google.com 443            # ทดสอบ HTTPS port
nc -zv -w 3 192.168.1.50 80     # Timeout 3 วินาที
```

**Output (port เปิด):**
```
Connection to google.com (142.250.185.78) 443 port [tcp/https] succeeded!
```

**Output (port ปิด):**
```
nc: connect to 192.168.1.50 port 80 (tcp) failed: Connection refused
```

---

## ส่วนที่ 8: Systemd — จัดการ Service

### 8.1 คำสั่งพื้นฐาน systemctl

```bash
# ดูสถานะ service
sudo systemctl status ssh
```

**Output:**
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-05-01 10:00:00 UTC; 30min ago
 Main PID: 1234 (sshd)
    Tasks: 1 (limit: 2316)
   Memory: 5.2M
      CPU: 45ms
   CGroup: /system.slice/ssh.service
           └─1234 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```

```bash
sudo systemctl start nginx        # เริ่ม service
sudo systemctl stop nginx         # หยุด service
sudo systemctl restart nginx      # restart
sudo systemctl reload nginx       # reload config (ไม่หยุด service)
sudo systemctl enable nginx       # เปิดทำงานตอน boot
sudo systemctl disable nginx      # ปิดทำงานตอน boot
sudo systemctl is-active nginx    # ตรวจว่า active ไหม
sudo systemctl is-enabled nginx   # ตรวจว่า enabled ไหม
```

---

### 8.2 ดู Log ด้วย journalctl

```bash
journalctl -u ssh                 # log ของ ssh service
journalctl -u ssh -n 50           # 50 บรรทัดสุดท้าย
journalctl -u ssh -f              # ติดตาม log แบบ real-time
journalctl --since "1 hour ago"   # log ย้อนหลัง 1 ชั่วโมง
journalctl --since "2026-05-01 10:00" --until "2026-05-01 11:00"
journalctl -p err                 # เฉพาะ error
sudo journalctl -k                # kernel log
```

---

## ส่วนที่ 9: User Management

### 9.1 สร้างและจัดการ User

```bash
# สร้าง user ใหม่
sudo adduser newstudent
```

**Output (interactive):**
```
Adding user 'newstudent' ...
Adding new group 'newstudent' (1001) ...
Adding new user 'newstudent' (1001) with group 'newstudent' ...
Creating home directory '/home/newstudent' ...
Copying files from '/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Full Name []: สมชาย ใจดี
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] Y
```

```bash
# เปลี่ยนรหัสผ่าน user
sudo passwd newstudent

# ดูข้อมูล user
id newstudent
cat /etc/passwd | grep newstudent

# เพิ่ม user เข้า group sudo
sudo usermod -aG sudo newstudent

# ลบ user
sudo deluser newstudent
sudo deluser --remove-home newstudent    # ลบพร้อม home directory
```

---

### 9.2 จัดการ Group

```bash
# สร้าง group
sudo groupadd developers

# เพิ่ม user เข้า group
sudo usermod -aG developers student

# ดู group ของ user
groups student
```

**Output:**
```
student : student sudo developers
```

```bash
# ดู group ทั้งหมด
cat /etc/group | grep developers
```

---

## ส่วนที่ 10: ทริป เทคนิค และ Shortcut

### 10.1 Keyboard Shortcut ที่ใช้บ่อย

| Shortcut | ฟังก์ชัน |
|----------|---------|
| `Ctrl + C` | หยุด process ที่กำลังรัน |
| `Ctrl + Z` | หยุด process ชั่วคราว (pause) |
| `Ctrl + D` | ออกจาก shell / logout |
| `Ctrl + L` | ล้างหน้าจอ (เหมือน `clear`) |
| `Ctrl + A` | ไปต้นบรรทัด |
| `Ctrl + E` | ไปท้ายบรรทัด |
| `Ctrl + U` | ลบทั้งบรรทัดด้านหน้า cursor |
| `Ctrl + K` | ลบทั้งบรรทัดด้านหลัง cursor |
| `Ctrl + R` | ค้นหาประวัติคำสั่ง (reverse search) |
| `Tab` | Auto-complete ชื่อไฟล์/คำสั่ง |
| `Tab Tab` | แสดงตัวเลือกทั้งหมด |
| `↑ ↓` | เลื่อนดูประวัติคำสั่ง |

---

### 10.2 ประวัติคำสั่ง (History)

```bash
history                          # แสดงประวัติทั้งหมด
history 20                       # 20 คำสั่งล่าสุด
history | grep apt               # กรองเฉพาะ apt
!45                              # รันคำสั่งหมายเลข 45 ใน history
!!                               # รันคำสั่งก่อนหน้าซ้ำ
!apt                             # รันคำสั่ง apt ล่าสุดใน history
```

---

### 10.3 Alias — ตั้งชื่อย่อคำสั่ง

```bash
# สร้าง alias ชั่วคราว (หายเมื่อปิด terminal)
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade -y'
alias myip='curl -s https://api.ipify.org && echo'

# ทดสอบ
ll
myip
```

**บันทึก Alias ถาวร:**
```bash
nano ~/.bashrc
```

เพิ่มบรรทัดเหล่านี้ที่ท้ายไฟล์:
```bash
alias ll='ls -lah'
alias la='ls -A'
alias l='ls -CF'
alias update='sudo apt update && sudo apt upgrade -y'
alias myip='curl -s https://api.ipify.org && echo'
alias ports='ss -tuln'
alias mynet='ip addr show'
```

นำไปใช้ทันที:
```bash
source ~/.bashrc
```

---

### 10.4 เทคนิค Redirect

```bash
command > file.txt          # stdout → file (เขียนทับ)
command >> file.txt         # stdout → file (ต่อท้าย)
command 2> error.txt        # stderr → file
command 2>&1 | tee log.txt  # stdout + stderr → ไฟล์ และ แสดงหน้าจอ
command > /dev/null 2>&1    # ทิ้งทั้ง stdout และ stderr
```

ตัวอย่าง:
```bash
ping -c 4 google.com > ping_result.txt 2>&1
cat ping_result.txt
```

---

### 10.5 เทคนิค tmux — ทำงานหลาย Session

```bash
sudo apt install tmux -y

tmux                          # เปิด session ใหม่
tmux new -s mysession         # เปิดพร้อมตั้งชื่อ
tmux ls                       # ดู session ทั้งหมด
tmux attach -t mysession      # กลับเข้า session
```

คีย์ใน tmux (กด `Ctrl+B` ก่อนแล้วตามด้วย):
- `c` — สร้าง window ใหม่
- `n` / `p` — window ถัดไป/ก่อนหน้า
- `%` — แบ่งหน้าจอแนวตั้ง
- `"` — แบ่งหน้าจอแนวนอน
- `d` — Detach (ออกโดย session ยังทำงานอยู่)

> **💡 ทริป:** tmux มีประโยชน์มากเมื่อทำงานผ่าน SSH — ถ้าการเชื่อมต่อหลุด process ที่รันอยู่ใน tmux ยังทำงานต่อ

---

## ส่วนที่ 11: ไฟล์ Config ที่สำคัญ

| ไฟล์ | หน้าที่ |
|------|---------|
| `/etc/passwd` | ข้อมูล user ทั้งหมด |
| `/etc/shadow` | รหัสผ่าน user (hash) — อ่านได้เฉพาะ root |
| `/etc/group` | ข้อมูล group ทั้งหมด |
| `/etc/hostname` | ชื่อเครื่อง |
| `/etc/hosts` | DNS local — map IP กับ ชื่อ |
| `/etc/resolv.conf` | DNS server ที่ใช้งาน |
| `/etc/netplan/` | Network configuration (Ubuntu 24.04) |
| `/etc/ssh/sshd_config` | ตั้งค่า SSH Server |
| `/etc/fstab` | การ mount disk ตอน boot |
| `/etc/crontab` | Cron job ของระบบ |
| `/var/log/syslog` | System log หลัก |
| `/var/log/auth.log` | Log การ login และ sudo |
| `/var/log/kern.log` | Kernel log |
| `/proc/cpuinfo` | ข้อมูล CPU (virtual file) |
| `/proc/meminfo` | ข้อมูล RAM (virtual file) |

---

## ส่วนที่ 12: คำสั่งรวมด่วน Network Troubleshooting

เมื่อ network มีปัญหา ให้รันตามลำดับนี้:

```bash
# ขั้นที่ 1 — ตรวจ interface
ip addr show

# ขั้นที่ 2 — ตรวจ routing table
ip route show

# ขั้นที่ 3 — ping loopback (ตัวเอง)
ping -c 2 127.0.0.1

# ขั้นที่ 4 — ping gateway
ping -c 2 192.168.1.1

# ขั้นที่ 5 — ping DNS public
ping -c 2 8.8.8.8

# ขั้นที่ 6 — ทดสอบ DNS resolution
nslookup google.com
dig +short google.com

# ขั้นที่ 7 — ping ด้วยชื่อโดเมน
ping -c 2 google.com

# ขั้นที่ 8 — ดู service ที่เกี่ยวข้อง
sudo systemctl status systemd-networkd
sudo systemctl status systemd-resolved
```

---

## 📋 ตาราง Quick Reference — คำสั่ง Network ทั้งหมด

| คำสั่ง | หน้าที่ | ตัวอย่าง |
|--------|---------|---------|
| `ip addr` | ดู IP address | `ip addr show enp0s3` |
| `ip route` | ดู routing table | `ip route show` |
| `ip link` | ดูสถานะ interface | `ip link set enp0s3 up` |
| `ping` | ทดสอบ connectivity | `ping -c 4 8.8.8.8` |
| `traceroute` | ติดตามเส้นทาง packet | `traceroute google.com` |
| `nslookup` | สอบถาม DNS | `nslookup google.com` |
| `dig` | สอบถาม DNS (ละเอียด) | `dig +short google.com` |
| `ss` | ดู port และ connection | `ss -tuln` |
| `netstat` | ดู port (วิธีเก่า) | `netstat -tuln` |
| `curl` | HTTP request / ดาวน์โหลด | `curl -I https://google.com` |
| `wget` | ดาวน์โหลดไฟล์ | `wget https://example.com/file` |
| `ssh` | เชื่อมต่อ remote | `ssh user@192.168.1.50` |
| `scp` | คัดลอกไฟล์ผ่าน ssh | `scp file.txt user@host:/path` |
| `rsync` | sync ไฟล์ผ่าน network | `rsync -avz src/ user@host:/dst` |
| `nc` | ทดสอบ port (netcat) | `nc -zv host 80` |
| `ufw` | Firewall management | `ufw allow 22/tcp` |
| `iftop` | monitor bandwidth real-time | `iftop -i enp0s3` |
| `resolvectl` | จัดการ DNS systemd | `resolvectl status` |
| `netplan` | ตั้งค่า network Ubuntu | `netplan apply` |
| `ifconfig` | ดู IP (วิธีเก่า) | `ifconfig enp0s3` |

---

## 🧠 คำถามทิ้งไว้ให้ศึกษาต่อ

### ระดับพื้นฐาน
1. `ls -l` แสดงตัวอักษร 10 ตัวแรกเป็น Permission — ช่วยอธิบายความหมายของแต่ละตัวอักษรให้ครบทุกตำแหน่ง
2. ทำไม Linux จึงแยกสิทธิ์เป็น Owner / Group / Others? มีประโยชน์อย่างไรในการทำงานเป็นทีม?
3. `stdin`, `stdout`, `stderr` คืออะไร? เกี่ยวข้องกับ `>`, `>>`, `2>` อย่างไร?
4. `PATH` environment variable คืออะไร? ทำไม Shell จึงรู้ว่า `ls` อยู่ที่ `/usr/bin/ls`?

### ระดับกลาง
5. Cron Job คืออะไร? ลองเขียน Cron expression สำหรับรันสคริปต์ทุกวันเวลา 02:00 น.
6. ทดลองติดตั้ง `nginx` แล้วดูว่า port อะไรถูกเปิด และ service เริ่มทำงานอัตโนมัติเมื่อ boot ไหม?
7. `iptables` คืออะไร? ต่างจาก `ufw` อย่างไร? UFW ทำอะไรกับ iptables ด้านหลัง?
8. ทำไม `/proc/cpuinfo` จึงเป็น "ไฟล์" ได้ทั้งที่ข้อมูลมาจาก kernel? ระบบ Virtual Filesystem คืออะไร?

### ระดับขั้นสูง
9. `systemd` เปลี่ยนแปลงการ boot ของ Linux อย่างไร? ต่างจาก `init` รุ่นเก่า (SysV init) อย่างไร?
10. ทดลองตั้งค่า SSH Key-based authentication และปิด Password Login ใน `/etc/ssh/sshd_config` — อธิบายว่าปลอดภัยกว่าอย่างไร
11. NAT (Network Address Translation) ทำงานอย่างไรในการเชื่อมต่อ VM กับอินเทอร์เน็ต? ต่างจาก Bridged Network อย่างไร?
12. ลองศึกษา `tcpdump` และ `wireshark` สำหรับ packet capture — สิ่งเหล่านี้มีประโยชน์อย่างไรในงาน Security?

---

## 📌 สรุปคำสั่งที่ต้องจำ (Cheat Sheet)

```
Root Management
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
sudo passwd root          เปลี่ยนรหัส root
su -                      เข้า root shell
exit / Ctrl+D             ออกจาก root
whoami                    ตรวจสอบว่าเป็นใคร

Navigation
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
pwd                       ตำแหน่งปัจจุบัน
ls -lah                   รายชื่อไฟล์ (ละเอียด)
cd /path                  เปลี่ยน directory
mkdir -p path/to/dir      สร้างโฟลเดอร์
find / -name "file"       ค้นหาไฟล์

File Operations
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
cat file                  แสดงเนื้อหาไฟล์
less file                 ดูไฟล์ยาว
tail -f /var/log/syslog   ติดตาม log
cp -r src dst             คัดลอก
mv old new                ย้าย/เปลี่ยนชื่อ
rm -rf dir                ลบโฟลเดอร์

Permissions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
chmod 755 file            rwxr-xr-x
chmod +x script.sh        เพิ่ม execute
chown user:group file     เปลี่ยนเจ้าของ

Network
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ip addr show              ดู IP address
ip route show             ดู routing
ping -c 4 8.8.8.8         ทดสอบ connectivity
ss -tuln                  ดู open ports
dig +short google.com     ทดสอบ DNS
curl -I https://site.com  ดู HTTP header
ssh user@host             เชื่อมต่อ SSH
sudo ufw status           ดู firewall

System
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ps aux                    ดู process
top / htop                monitor process
free -h                   ดู RAM
df -h                     ดู disk space
sudo systemctl status x   ดูสถานะ service
journalctl -u x -f        ติดตาม service log
sudo apt update           อัปเดต package list
sudo apt install pkg -y   ติดตั้ง package
```

---

*แลปถัดไป → Ubuntu Server: Apache, BIND9 DNS, FTP, MySQL*
