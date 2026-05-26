# คู่มือ LAMP Web Server บน Ubuntu Desktop 24.04 LTS ฉบับสมบูรณ์
### สำหรับมือใหม่ ณ วันที่ 26/5/2026

> **LAMP** ย่อมาจาก **L**inux + **A**pache + **M**ySQL + **P**HP
> คู่มือนี้ครอบคลุมการติดตั้ง ตัวอย่างโค้ด Frontend/Backend และ WordPress พร้อมการใช้งานผ่าน LAN

---

## สารบัญ

1. [ทำความเข้าใจ LAMP คืออะไร](#1-ทำความเข้าใจ-lamp-คืออะไร)
2. [เตรียมระบบก่อนติดตั้ง](#2-เตรียมระบบก่อนติดตั้ง)
3. [ติดตั้ง Apache Web Server](#3-ติดตั้ง-apache-web-server)
4. [ติดตั้ง MySQL Database Server](#4-ติดตั้ง-mysql-database-server)
5. [ติดตั้ง PHP](#5-ติดตั้ง-php)
6. [ทดสอบ LAMP ด้วยโค้ดจริง](#6-ทดสอบ-lamp-ด้วยโค้ดจริง)
7. [ตัวอย่าง Frontend: หน้า Sign In (HTML CSS JS)](#7-ตัวอย่าง-frontend-หน้า-sign-in-html-css-js)
8. [ตัวอย่าง Backend: PHP PDO MySQL REST API](#8-ตัวอย่าง-backend-php-pdo-mysql-rest-api)
9. [การเข้าถึงผ่าน LAN (Windows 10 + Chrome)](#9-การเข้าถึงผ่าน-lan-windows-10--chrome)
10. [ติดตั้ง WordPress](#10-ติดตั้ง-wordpress)
11. [การจัดการ Virtual Host (หลายเว็บไซต์)](#11-การจัดการ-virtual-host-หลายเว็บไซต์)
12. [ทริปและเทคนิคที่ควรรู้](#12-ทริปและเทคนิคที่ควรรู้)
13. [การแก้ปัญหาที่พบบ่อย](#13-การแก้ปัญหาที่พบบ่อย)

---

## 1. ทำความเข้าใจ LAMP คืออะไร

### 1.1 อธิบายแต่ละส่วนแบบมือใหม่

ลองนึกภาพร้านอาหาร:

```
🏠 Linux (Ubuntu)  = ตัวอาคารร้านอาหาร — รองรับทุกอย่าง
🚪 Apache          = พนักงานต้อนรับ — รับ-ส่งคำขอของลูกค้า (Browser)
🗄️  MySQL          = คลังเก็บวัตถุดิบ — เก็บข้อมูลทั้งหมด
👨‍🍳 PHP            = พ่อครัว — ทำอาหาร (ประมวลผล) แล้วส่งให้ลูกค้า
```

**การทำงานเมื่อเปิดเว็บ:**

```
Browser (Chrome) ──► Apache รับคำขอ
                         │
                         ▼
                    PHP ประมวลผล ──► MySQL อ่าน/เขียนข้อมูล
                         │
                         ▼
                    Apache ส่ง HTML กลับ
                         │
                         ▼
                    Browser แสดงผล
```

### 1.2 คำศัพท์สำคัญที่มือใหม่ต้องรู้

| คำศัพท์ | ความหมาย |
|---|---|
| **Web Server** | โปรแกรมที่รับคำขอจาก Browser แล้วส่งหน้าเว็บกลับ |
| **Port 80** | "ประตู" มาตรฐานของเว็บ HTTP เหมือนเลขที่บ้าน |
| **Port 443** | ประตูของ HTTPS (เข้ารหัส) |
| **localhost** | ชื่อพิเศษหมายถึงเครื่องตัวเอง เท่ากับ IP 127.0.0.1 |
| **Document Root** | โฟลเดอร์หลักที่ Apache มองหาไฟล์เว็บ (ค่าเริ่ม `/var/www/html`) |
| **Virtual Host** | การตั้งค่าให้ Apache รองรับหลายเว็บไซต์ในเครื่องเดียว |
| **PDO** | PHP Data Objects — วิธีเชื่อมต่อฐานข้อมูลที่ปลอดภัยใน PHP |
| **REST API** | รูปแบบการรับ-ส่งข้อมูลระหว่าง Frontend กับ Backend ผ่าน HTTP |
| **LAN** | Local Area Network — เครือข่ายภายในบ้าน/ออฟฟิศ |
| **DHCP** | ระบบแจก IP อัตโนมัติในเครือข่าย |
| **IP Address** | เลขที่อยู่ของอุปกรณ์ในเครือข่าย เช่น 192.168.1.10 |

---

## 2. เตรียมระบบก่อนติดตั้ง

### 2.1 ตรวจสอบเวอร์ชัน Ubuntu

```bash
lsb_release -a
```

ผลลัพธ์ที่ถูกต้อง:
```
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.x LTS
Release:        24.04
Codename:       noble
```

### 2.2 อัปเดตระบบให้ล่าสุด

> **ทำไมต้องอัปเดตก่อน?** เพราะ package list อาจเก่า ทำให้ติดตั้งไม่ได้เวอร์ชันล่าสุด

```bash
sudo apt update && sudo apt upgrade -y
```

- `sudo` = ขอสิทธิ์ Administrator
- `apt update` = อัปเดต**รายการ**แพ็กเกจ (ไม่ได้ติดตั้งอะไร)
- `apt upgrade -y` = อัปเกรดแพ็กเกจที่มีเวอร์ชันใหม่ (`-y` ตอบ yes อัตโนมัติ)

### 2.3 ติดตั้งเครื่องมือพื้นฐาน

```bash
sudo apt install -y curl wget nano git unzip
```

> `nano` คือโปรแกรมแก้ไขข้อความใน Terminal ใช้ง่ายสำหรับมือใหม่

### 2.4 ตรวจสอบ IP ของเครื่อง Ubuntu (สำหรับใช้ใน LAN)

```bash
ip addr show
```

หรือย่อๆ:
```bash
hostname -I
```

จดจำ IP นี้ไว้ เช่น `192.168.1.50` — จะใช้เข้าถึงจาก Windows ภายหลัง

---

## 3. ติดตั้ง Apache Web Server

### 3.1 ติดตั้ง Apache

```bash
sudo apt install apache2 -y
```

### 3.2 เริ่มและตั้งให้รันอัตโนมัติเมื่อบูต

```bash
# เริ่มการทำงาน Apache
sudo systemctl start apache2

# ตั้งให้เริ่มอัตโนมัติเมื่อเปิดเครื่อง
sudo systemctl enable apache2

# ตรวจสอบสถานะ
sudo systemctl status apache2
```

ผลลัพธ์ที่ดี จะเห็น:
```
Active: active (running) ...
```

### 3.3 ทดสอบ Apache ครั้งแรก

เปิด Browser แล้วพิมพ์:
```
http://localhost
```

จะเห็นหน้า **"Apache2 Ubuntu Default Page"** — Apache ทำงานแล้ว! 🎉

### 3.4 เปิด Firewall สำหรับ LAN

> **Firewall** คือกำแพงป้องกัน ต้องเปิดประตู (Port) ให้ Apache ด้วย

```bash
# เปิดพอร์ต HTTP (80)
sudo ufw allow 'Apache'

# ตรวจสอบสถานะ Firewall
sudo ufw status
```

ถ้า UFW ยังไม่เปิด:
```bash
sudo ufw enable
sudo ufw allow 'Apache'
sudo ufw allow 'OpenSSH'
```

### 3.5 โครงสร้างโฟลเดอร์ Apache ที่ควรรู้

```
/etc/apache2/              → โฟลเดอร์ config หลัก
├── apache2.conf           → config หลักของ Apache
├── sites-available/       → ไฟล์ Virtual Host ที่สร้างไว้ (ยังไม่เปิดใช้)
├── sites-enabled/         → Virtual Host ที่เปิดใช้งานแล้ว (symlink)
├── mods-available/        → module ทั้งหมด
└── mods-enabled/          → module ที่เปิดใช้งาน

/var/www/html/             → Document Root — วางไฟล์เว็บที่นี่
/var/log/apache2/          → Log ไฟล์
├── access.log             → บันทึกคำขอทั้งหมด
└── error.log              → บันทึก error
```

### 3.6 คำสั่ง Apache ที่ใช้บ่อย

```bash
sudo systemctl start apache2     # เริ่ม
sudo systemctl stop apache2      # หยุด
sudo systemctl restart apache2   # รีสตาร์ท (ปิดแล้วเปิดใหม่)
sudo systemctl reload apache2    # โหลด config ใหม่ (ไม่หยุด)
sudo apache2ctl configtest       # ตรวจสอบ config ก่อน restart
```

> **เทคนิค:** ใช้ `reload` แทน `restart` เมื่อแก้ config เพราะไม่ทำให้ผู้ใช้ที่เชื่อมต่ออยู่หลุด

---

## 4. ติดตั้ง MySQL Database Server

### 4.1 ติดตั้ง MySQL

```bash
sudo apt install mysql-server -y
```

### 4.2 เริ่มและตั้งให้รันอัตโนมัติ

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
sudo systemctl status mysql
```

### 4.3 ตั้งค่าความปลอดภัย MySQL (สำคัญมาก!)

```bash
sudo mysql_secure_installation
```

ระบบจะถามหลายข้อ แนะนำให้ตอบดังนี้:

```
VALIDATE PASSWORD component? → Y (เปิดการตรวจสอบความแข็งแกร่งรหัสผ่าน)
Password strength level       → 1 (MEDIUM) หรือ 0 (LOW) สำหรับ dev
New password:                 → ใส่รหัสผ่านที่แข็งแกร่ง เช่น MyP@ss2026!
Remove anonymous users?       → Y (ลบผู้ใช้ไม่ระบุชื่อ)
Disallow root login remotely? → Y (ป้องกัน root เข้าจากนอก)
Remove test database?         → Y (ลบฐานข้อมูลทดสอบ)
Reload privilege tables?      → Y (บันทึกการเปลี่ยนแปลง)
```

### 4.4 เข้า MySQL Shell

```bash
sudo mysql
```

> เหตุใดต้องใช้ `sudo`? เพราะ Ubuntu 24.04 ตั้งค่า MySQL root ใช้ auth_socket (ไม่ต้องรหัสผ่าน แต่ต้องเป็น root ของระบบ)

จะเห็น prompt:
```
mysql>
```

### 4.5 คำสั่ง MySQL พื้นฐานที่ควรรู้

```sql
-- ดูฐานข้อมูลทั้งหมด
SHOW DATABASES;

-- สร้างฐานข้อมูลใหม่ (รองรับภาษาไทยด้วย utf8mb4)
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- เลือกฐานข้อมูล
USE mydb;

-- ดูตารางทั้งหมด
SHOW TABLES;

-- สร้าง User ใหม่
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'MyP@ss2026!';

-- ให้สิทธิ์ทั้งหมดบน Database
GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'localhost';

-- บังคับใช้การเปลี่ยนแปลงสิทธิ์
FLUSH PRIVILEGES;

-- ออกจาก MySQL
EXIT;
```

> **คำอธิบาย `'myuser'@'localhost'`:**
> - `myuser` = ชื่อผู้ใช้
> - `@'localhost'` = เข้าได้เฉพาะจากเครื่องเดียวกัน (ปลอดภัยกว่า `@'%'` ซึ่งเข้าจากทุกที่)

---

## 5. ติดตั้ง PHP

### 5.1 PHP เวอร์ชันในปัจจุบัน (พ.ค. 2026)

| เวอร์ชัน | สถานะ | แนะนำ |
|---|---|---|
| PHP 8.3 | ค่าเริ่มต้นของ Ubuntu 24.04 | ✅ เสถียร แนะนำ |
| PHP 8.4 | ล่าสุด (ต้องเพิ่ม repo) | ✅ ใช้งาน production ได้ |
| PHP 8.2 | Security support ถึง ธ.ค. 2026 | ⚠️ ใกล้หมดอายุ |

### 5.2 ติดตั้ง PHP 8.3 (ค่าเริ่มต้น Ubuntu 24.04)

```bash
sudo apt install php libapache2-mod-php php-mysql \
  php-curl php-gd php-mbstring php-xml php-zip \
  php-bcmath php-json php-opcache php-intl -y
```

**อธิบายแต่ละ package:**

| Package | ใช้ทำอะไร |
|---|---|
| `php` | ตัวหลักของ PHP |
| `libapache2-mod-php` | เชื่อม PHP กับ Apache |
| `php-mysql` | ให้ PHP คุยกับ MySQL ได้ |
| `php-curl` | ดึงข้อมูลจาก URL ภายนอก (API calls) |
| `php-gd` | ประมวลผลรูปภาพ |
| `php-mbstring` | รองรับตัวอักษรหลายภาษา (ภาษาไทย!) |
| `php-xml` | อ่าน/เขียน XML |
| `php-zip` | จัดการไฟล์ .zip |
| `php-bcmath` | คำนวณตัวเลขขนาดใหญ่ (บัญชี/การเงิน) |
| `php-opcache` | Cache โค้ด PHP → เว็บเร็วขึ้น |

### 5.3 ตรวจสอบเวอร์ชัน PHP

```bash
php -v
```

ผลลัพธ์:
```
PHP 8.3.x (cli) ...
```

### 5.4 ปรับแต่ง PHP สำหรับการพัฒนา

```bash
sudo nano /etc/php/8.3/apache2/php.ini
```

ค้นหาและแก้ไขค่าเหล่านี้ (กด `Ctrl+W` เพื่อค้นหาใน nano):

```ini
; แสดง error บนหน้าจอ (เปิดเฉพาะตอน development)
display_errors = On
error_reporting = E_ALL

; ขนาดไฟล์อัปโหลดสูงสุด
upload_max_filesize = 64M
post_max_size = 64M

; เวลาประมวลผลสูงสุด (วินาที)
max_execution_time = 300

; หน่วยความจำสูงสุด
memory_limit = 256M

; ตั้ง Timezone เป็นไทย
date.timezone = Asia/Bangkok
```

บันทึกด้วย `Ctrl+O` → `Enter` → ออกด้วย `Ctrl+X`

```bash
# รีสตาร์ท Apache เพื่อโหลด PHP ใหม่
sudo systemctl restart apache2
```

### 5.5 ทดสอบ PHP กับ Apache

```bash
sudo nano /var/www/html/info.php
```

ใส่โค้ด:
```php
<?php
phpinfo();
?>
```

บันทึกแล้วเปิด Browser:
```
http://localhost/info.php
```

จะเห็นหน้าสีม่วง แสดงข้อมูล PHP ทั้งหมด — PHP ทำงานกับ Apache แล้ว! 🎉

> **⚠️ ความปลอดภัย:** ลบไฟล์นี้ทิ้งเมื่อทดสอบเสร็จแล้ว!
> ```bash
> sudo rm /var/www/html/info.php
> ```

---

## 6. ทดสอบ LAMP ด้วยโค้ดจริง

### 6.1 ทดสอบการเชื่อมต่อ PHP กับ MySQL

```bash
sudo nano /var/www/html/test_db.php
```

```php
<?php
// ==============================
// ทดสอบการเชื่อมต่อ PHP → MySQL
// ==============================

$host     = 'localhost';
$dbname   = 'test_lamp';
$username = 'lampuser';
$password = 'LampP@ss2026!';

// สร้างการเชื่อมต่อแบบ PDO (วิธีมาตรฐานและปลอดภัย)
try {
    $pdo = new PDO(
        "mysql:host=$host;dbname=$dbname;charset=utf8mb4",
        $username,
        $password,
        [
            PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]
    );

    echo "<h2 style='color:green'>✅ เชื่อมต่อ MySQL สำเร็จ!</h2>";
    echo "<p>PHP Version: " . phpversion() . "</p>";
    echo "<p>MySQL เวอร์ชัน: " . $pdo->getAttribute(PDO::ATTR_SERVER_VERSION) . "</p>";

} catch (PDOException $e) {
    echo "<h2 style='color:red'>❌ เชื่อมต่อล้มเหลว</h2>";
    echo "<p>ข้อผิดพลาด: " . $e->getMessage() . "</p>";
}
?>
```

**เตรียม MySQL ก่อนทดสอบ:**

```bash
sudo mysql
```

```sql
CREATE DATABASE test_lamp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'lampuser'@'localhost' IDENTIFIED BY 'LampP@ss2026!';
GRANT ALL PRIVILEGES ON test_lamp.* TO 'lampuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

เปิด Browser: `http://localhost/test_db.php`

---

## 7. ตัวอย่าง Frontend: หน้า Sign In (HTML CSS JS)

### 7.1 สร้างโครงสร้างโปรเจกต์

```bash
sudo mkdir -p /var/www/html/myapp
sudo chown -R $USER:$USER /var/www/html/myapp
```

> `chown -R $USER:$USER` = เปลี่ยนเจ้าของโฟลเดอร์เป็นผู้ใช้ปัจจุบัน เพื่อแก้ไขไฟล์ได้โดยไม่ต้องใช้ sudo

### 7.2 สร้างหน้า Sign In

```bash
nano /var/www/html/myapp/index.html
```

```html
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>เข้าสู่ระบบ - MyApp</title>

    <!-- ============================================================
         CSS: ตกแต่งหน้าตา
         ============================================================ -->
    <style>
        /* Reset: ล้างค่าเริ่มต้นของ Browser ให้หน้าตาเหมือนกันทุก Browser */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box; /* ขนาด element รวม padding/border แล้ว */
        }

        body {
            font-family: 'Segoe UI', Tahoma, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh; /* vh = viewport height, 100vh = สูงเท่าหน้าจอ */
            display: flex;
            align-items: center;     /* จัดกลางแนวตั้ง */
            justify-content: center; /* จัดกลางแนวนอน */
        }

        /* Card: กล่องฟอร์ม */
        .card {
            background: white;
            border-radius: 16px;
            padding: 40px;
            width: 100%;
            max-width: 400px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }

        .card-logo {
            text-align: center;
            font-size: 48px;
            margin-bottom: 8px;
        }

        .card-title {
            text-align: center;
            font-size: 24px;
            font-weight: 700;
            color: #333;
            margin-bottom: 4px;
        }

        .card-subtitle {
            text-align: center;
            font-size: 14px;
            color: #999;
            margin-bottom: 32px;
        }

        /* Form Group: ชุดของ Label + Input */
        .form-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            font-size: 13px;
            font-weight: 600;
            color: #555;
            margin-bottom: 6px;
        }

        input {
            width: 100%;
            padding: 12px 16px;
            border: 2px solid #e1e5e9;
            border-radius: 8px;
            font-size: 15px;
            color: #333;
            outline: none; /* ลบเส้นขอบสีฟ้าเริ่มต้น */
            transition: border-color 0.3s; /* animation เปลี่ยนสีขอบ */
        }

        /* :focus = เมื่อ element ถูกคลิก/กำลังพิมพ์ */
        input:focus {
            border-color: #667eea;
        }

        /* ปุ่ม Submit */
        .btn-submit {
            width: 100%;
            padding: 14px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer; /* เปลี่ยน cursor เป็นมือ */
            transition: opacity 0.3s;
            margin-top: 8px;
        }

        .btn-submit:hover { opacity: 0.9; }
        .btn-submit:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }

        /* Alert: แสดงข้อความ error/success */
        .alert {
            padding: 12px 16px;
            border-radius: 8px;
            font-size: 14px;
            margin-bottom: 16px;
            display: none; /* ซ่อนไว้ก่อน */
        }

        .alert-error {
            background: #fff5f5;
            border: 1px solid #feb2b2;
            color: #c53030;
        }

        .alert-success {
            background: #f0fff4;
            border: 1px solid #9ae6b4;
            color: #276749;
        }

        /* Loading Spinner */
        .spinner {
            display: inline-block;
            width: 18px;
            height: 18px;
            border: 2px solid rgba(255,255,255,0.3);
            border-top-color: white;
            border-radius: 50%;
            animation: spin 0.8s linear infinite;
            margin-right: 8px;
            vertical-align: middle;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        .register-link {
            text-align: center;
            margin-top: 20px;
            font-size: 14px;
            color: #999;
        }

        .register-link a {
            color: #667eea;
            text-decoration: none;
            font-weight: 600;
        }
    </style>
</head>

<body>
    <div class="card">
        <div class="card-logo">🔐</div>
        <h1 class="card-title">เข้าสู่ระบบ</h1>
        <p class="card-subtitle">กรุณากรอกข้อมูลของคุณ</p>

        <!-- Alert Box: แสดงผล error/success -->
        <div class="alert alert-error" id="alertError"></div>
        <div class="alert alert-success" id="alertSuccess"></div>

        <!-- ฟอร์ม Sign In -->
        <!-- onsubmit="return false" ป้องกัน Browser refresh หน้าเอง -->
        <form id="loginForm" onsubmit="return false">
            <div class="form-group">
                <label for="email">อีเมล</label>
                <input
                    type="email"
                    id="email"
                    name="email"
                    placeholder="example@email.com"
                    required
                    autocomplete="email"
                >
            </div>

            <div class="form-group">
                <label for="password">รหัสผ่าน</label>
                <input
                    type="password"
                    id="password"
                    name="password"
                    placeholder="••••••••"
                    required
                    minlength="6"
                >
            </div>

            <button type="button" class="btn-submit" id="btnLogin" onclick="doLogin()">
                เข้าสู่ระบบ
            </button>
        </form>

        <p class="register-link">
            ยังไม่มีบัญชี? <a href="register.html">สมัครสมาชิก</a>
        </p>
    </div>

    <!-- ============================================================
         JavaScript: ส่งข้อมูลไปยัง Backend API
         ============================================================ -->
    <script>
        // URL ของ Backend API
        // ถ้า Backend อยู่เครื่องเดียวกัน ใช้ /myapp/api/...
        // ถ้าเข้าจาก LAN ใช้ http://192.168.1.50/myapp/api/...
        const API_BASE = '/myapp/api';

        async function doLogin() {
            // 1. ดึงค่าจากฟอร์ม
            const email    = document.getElementById('email').value.trim();
            const password = document.getElementById('password').value;
            const btn      = document.getElementById('btnLogin');

            // 2. Validate ฝั่ง Frontend (ตรวจสอบเบื้องต้น ก่อนส่ง)
            if (!email || !password) {
                showAlert('error', 'กรุณากรอกข้อมูลให้ครบ');
                return;
            }

            // 3. แสดง Loading state
            btn.disabled   = true;
            btn.innerHTML  = '<span class="spinner"></span>กำลังตรวจสอบ...';
            hideAlerts();

            try {
                // 4. ส่งข้อมูลไปยัง Backend ด้วย fetch() (REST API)
                //    fetch() คือ API ของ Browser สำหรับยิง HTTP Request
                const response = await fetch(`${API_BASE}/auth.php`, {
                    method: 'POST',           // HTTP Method POST = ส่งข้อมูล
                    headers: {
                        'Content-Type': 'application/json', // บอก Backend ว่าส่ง JSON มา
                    },
                    body: JSON.stringify({    // JSON.stringify = แปลง Object เป็น String JSON
                        action:   'login',
                        email:    email,
                        password: password
                    })
                });

                // 5. รับและแปลงผลลัพธ์จาก JSON
                const data = await response.json();

                // 6. ตรวจสอบผลลัพธ์
                if (data.success) {
                    // บันทึก Token ใน sessionStorage (หายเมื่อปิด Browser)
                    sessionStorage.setItem('authToken', data.token);
                    sessionStorage.setItem('userName', data.user.name);

                    showAlert('success', `ยินดีต้อนรับ ${data.user.name}! กำลังเข้าสู่ระบบ...`);

                    // รอ 1.5 วินาที แล้วเปลี่ยนหน้า
                    setTimeout(() => {
                        window.location.href = 'dashboard.html';
                    }, 1500);

                } else {
                    // แสดง error จาก Backend
                    showAlert('error', data.message || 'เกิดข้อผิดพลาด กรุณาลองใหม่');
                    btn.disabled  = false;
                    btn.innerHTML = 'เข้าสู่ระบบ';
                }

            } catch (err) {
                // Network error (Backend ไม่ตอบสนอง)
                showAlert('error', 'ไม่สามารถเชื่อมต่อ Server ได้ กรุณาตรวจสอบ Network');
                btn.disabled  = false;
                btn.innerHTML = 'เข้าสู่ระบบ';
                console.error('Fetch error:', err); // ดู error ใน Browser DevTools
            }
        }

        // ฟังก์ชันแสดง Alert
        function showAlert(type, message) {
            hideAlerts();
            const alertEl = document.getElementById(
                type === 'error' ? 'alertError' : 'alertSuccess'
            );
            alertEl.textContent    = message;
            alertEl.style.display  = 'block';
        }

        // ฟังก์ชันซ่อน Alert ทั้งหมด
        function hideAlerts() {
            document.getElementById('alertError').style.display   = 'none';
            document.getElementById('alertSuccess').style.display = 'none';
        }

        // กด Enter เพื่อ login
        document.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') doLogin();
        });
    </script>
</body>
</html>
```

---

## 8. ตัวอย่าง Backend: PHP PDO MySQL REST API

### 8.1 สร้างโครงสร้าง API

```bash
mkdir -p /var/www/html/myapp/api
```

### 8.2 สร้างฐานข้อมูล

```bash
sudo mysql
```

```sql
-- สร้างฐานข้อมูล
CREATE DATABASE myapp_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- สร้างผู้ใช้ฐานข้อมูล
CREATE USER 'myapp_user'@'localhost' IDENTIFIED BY 'AppP@ss2026!';
GRANT ALL PRIVILEGES ON myapp_db.* TO 'myapp_user'@'localhost';
FLUSH PRIVILEGES;

-- ใช้ฐานข้อมูล
USE myapp_db;

-- สร้างตารางผู้ใช้
CREATE TABLE users (
    id         INT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(100)        NOT NULL,
    email      VARCHAR(150) UNIQUE NOT NULL,
    password   VARCHAR(255)        NOT NULL,  -- เก็บ hash เท่านั้น ไม่เก็บ plain text
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ใส่ข้อมูลทดสอบ (password = "test1234")
-- password_hash() จาก PHP — ใช้ bcrypt
INSERT INTO users (name, email, password) VALUES
('สมชาย ใจดี', 'somchai@test.com',
 '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi');

EXIT;
```

> **หมายเหตุ:** ค่า hash ข้างบนตรงกับ password `"password"` (ค่าตัวอย่าง) ในการใช้งานจริงใช้ฟังก์ชัน `password_hash()` ของ PHP เสมอ

### 8.3 สร้างไฟล์ Config ฐานข้อมูล

```bash
nano /var/www/html/myapp/api/config.php
```

```php
<?php
// ============================================================
// config.php — การตั้งค่าการเชื่อมต่อฐานข้อมูล
// ไฟล์นี้ไม่ควรถูกเข้าถึงโดยตรงจาก Browser
// ============================================================

define('DB_HOST', 'localhost');
define('DB_NAME', 'myapp_db');
define('DB_USER', 'myapp_user');
define('DB_PASS', 'AppP@ss2026!');
define('DB_CHARSET', 'utf8mb4');

// Secret key สำหรับสร้าง Token (เปลี่ยนเป็นค่าสุ่มในการใช้จริง)
define('SECRET_KEY', 'myapp_secret_key_2026_change_this!');

// ฟังก์ชันสร้างการเชื่อมต่อ PDO
// PDO = PHP Data Objects — มาตรฐานการเชื่อมต่อ DB ที่ปลอดภัย
function getDB(): PDO {
    static $pdo = null; // static = ใช้ค่าเดิมถ้าเรียกซ้ำ (Singleton pattern)

    if ($pdo === null) {
        $dsn = "mysql:host=" . DB_HOST
             . ";dbname=" . DB_NAME
             . ";charset=" . DB_CHARSET;

        $options = [
            PDO::ATTR_ERRMODE            => PDO::ERRMODE_EXCEPTION,   // throw Exception เมื่อ error
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,         // return array แบบ key=>value
            PDO::ATTR_EMULATE_PREPARES   => false,                     // ปิด emulation (ปลอดภัยกว่า)
        ];

        $pdo = new PDO($dsn, DB_USER, DB_PASS, $options);
    }

    return $pdo;
}
```

### 8.4 สร้าง Helper สำหรับ Response และ Token

```bash
nano /var/www/html/myapp/api/helpers.php
```

```php
<?php
// ============================================================
// helpers.php — ฟังก์ชันช่วยเหลือทั่วไป
// ============================================================

// ส่ง JSON Response กลับไปให้ Frontend
// $data    = ข้อมูลที่จะส่ง (array)
// $status  = HTTP Status Code (200=OK, 400=Bad Request, 401=Unauthorized, 500=Server Error)
function sendResponse(array $data, int $status = 200): void {
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    // CORS Headers: อนุญาตให้ Frontend ต่าง Origin เรียกได้ (สำหรับ LAN)
    header('Access-Control-Allow-Origin: *');
    header('Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS');
    header('Access-Control-Allow-Headers: Content-Type, Authorization');
    echo json_encode($data, JSON_UNESCAPED_UNICODE); // JSON_UNESCAPED_UNICODE = แสดงภาษาไทยได้
    exit;
}

// รับ JSON Body จาก Request
// Frontend ส่ง JSON มาใน Request Body เราต้องอ่านด้วย php://input
function getRequestBody(): array {
    $raw  = file_get_contents('php://input');
    $data = json_decode($raw, true); // true = แปลงเป็น array (ไม่ใช่ object)
    return is_array($data) ? $data : [];
}

// สร้าง Token อย่างง่าย (สำหรับ Demo)
// ในโปรเจกต์จริง ควรใช้ JWT library
function generateToken(int $userId): string {
    $payload = base64_encode(json_encode([
        'user_id' => $userId,
        'exp'     => time() + 86400, // หมดอายุใน 24 ชั่วโมง
    ]));
    $signature = hash_hmac('sha256', $payload, SECRET_KEY);
    return $payload . '.' . $signature;
}

// ตรวจสอบ Token
function verifyToken(string $token): ?array {
    $parts = explode('.', $token);
    if (count($parts) !== 2) return null;

    [$payload, $signature] = $parts;

    // ตรวจสอบ signature
    $expectedSig = hash_hmac('sha256', $payload, SECRET_KEY);
    if (!hash_equals($expectedSig, $signature)) return null;

    // แปลง payload
    $data = json_decode(base64_decode($payload), true);

    // ตรวจสอบวันหมดอายุ
    if (!$data || $data['exp'] < time()) return null;

    return $data;
}

// ดึง Token จาก Authorization Header
// Frontend ส่งมาใน Header: "Authorization: Bearer <token>"
function getBearerToken(): ?string {
    $headers = getallheaders();
    $auth    = $headers['Authorization'] ?? '';
    if (preg_match('/Bearer\s+(.+)/', $auth, $matches)) {
        return $matches[1];
    }
    return null;
}
```

### 8.5 สร้าง Authentication API

```bash
nano /var/www/html/myapp/api/auth.php
```

```php
<?php
// ============================================================
// auth.php — REST API สำหรับ Login / Register
// URL: http://[server]/myapp/api/auth.php
//
// รองรับ actions:
//   POST { action: "login",    email, password }
//   POST { action: "register", name, email, password }
// ============================================================

require_once 'config.php';
require_once 'helpers.php';

// รองรับ Preflight Request (OPTIONS) — Browser ส่งมาตรวจสอบ CORS ก่อน
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    sendResponse(['ok' => true]);
}

// รับข้อมูล JSON จาก Request Body
$body   = getRequestBody();
$action = $body['action'] ?? '';

// ===============================
// Router: เลือกการทำงานตาม action
// ===============================
switch ($action) {

    case 'login':
        handleLogin($body);
        break;

    case 'register':
        handleRegister($body);
        break;

    default:
        sendResponse([
            'success' => false,
            'message' => 'Action ไม่ถูกต้อง'
        ], 400);
}

// ===============================
// ฟังก์ชัน Login
// ===============================
function handleLogin(array $body): void {
    $email    = trim($body['email']    ?? '');
    $password = trim($body['password'] ?? '');

    // Validate: ตรวจสอบข้อมูลเบื้องต้น
    if (empty($email) || empty($password)) {
        sendResponse([
            'success' => false,
            'message' => 'กรุณากรอกอีเมลและรหัสผ่าน'
        ], 400);
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        sendResponse([
            'success' => false,
            'message' => 'รูปแบบอีเมลไม่ถูกต้อง'
        ], 400);
    }

    try {
        $db = getDB();

        // ค้นหาผู้ใช้ด้วยอีเมล
        // Prepared Statement: ป้องกัน SQL Injection
        // ? คือ placeholder ที่จะถูกแทนที่ด้วยค่าจริงอย่างปลอดภัย
        $stmt = $db->prepare(
            "SELECT id, name, email, password FROM users WHERE email = ? LIMIT 1"
        );
        $stmt->execute([$email]);
        $user = $stmt->fetch(); // ดึงแถวเดียว

        // ตรวจสอบว่ามีผู้ใช้และรหัสผ่านถูกต้อง
        // password_verify() เปรียบเทียบ plain text กับ bcrypt hash
        if (!$user || !password_verify($password, $user['password'])) {
            sendResponse([
                'success' => false,
                'message' => 'อีเมลหรือรหัสผ่านไม่ถูกต้อง'
            ], 401);
        }

        // สร้าง Token
        $token = generateToken($user['id']);

        sendResponse([
            'success' => true,
            'message' => 'เข้าสู่ระบบสำเร็จ',
            'token'   => $token,
            'user'    => [
                'id'    => $user['id'],
                'name'  => $user['name'],
                'email' => $user['email'],
            ]
        ]);

    } catch (PDOException $e) {
        // อย่า expose ข้อผิดพลาดจริงให้ผู้ใช้เห็น (เก็บ log แทน)
        error_log("Login error: " . $e->getMessage());
        sendResponse([
            'success' => false,
            'message' => 'เกิดข้อผิดพลาด กรุณาลองใหม่'
        ], 500);
    }
}

// ===============================
// ฟังก์ชัน Register
// ===============================
function handleRegister(array $body): void {
    $name     = trim($body['name']     ?? '');
    $email    = trim($body['email']    ?? '');
    $password = trim($body['password'] ?? '');

    // Validate
    if (empty($name) || empty($email) || empty($password)) {
        sendResponse([
            'success' => false,
            'message' => 'กรุณากรอกข้อมูลให้ครบ'
        ], 400);
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        sendResponse([
            'success' => false,
            'message' => 'รูปแบบอีเมลไม่ถูกต้อง'
        ], 400);
    }

    if (strlen($password) < 8) {
        sendResponse([
            'success' => false,
            'message' => 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร'
        ], 400);
    }

    try {
        $db = getDB();

        // ตรวจสอบอีเมลซ้ำ
        $stmt = $db->prepare("SELECT id FROM users WHERE email = ?");
        $stmt->execute([$email]);
        if ($stmt->fetch()) {
            sendResponse([
                'success' => false,
                'message' => 'อีเมลนี้ถูกใช้งานแล้ว'
            ], 409);
        }

        // Hash รหัสผ่าน (ห้ามเก็บ plain text เด็ดขาด!)
        // PASSWORD_BCRYPT = อัลกอริทึมที่ปลอดภัย ช้าพอที่จะป้องกัน brute force
        $hashedPassword = password_hash($password, PASSWORD_BCRYPT);

        // บันทึกลงฐานข้อมูล
        $stmt = $db->prepare(
            "INSERT INTO users (name, email, password) VALUES (?, ?, ?)"
        );
        $stmt->execute([$name, $email, $hashedPassword]);
        $newId = $db->lastInsertId(); // ดึง ID ที่เพิ่งสร้าง

        $token = generateToken($newId);

        sendResponse([
            'success' => true,
            'message' => 'สมัครสมาชิกสำเร็จ',
            'token'   => $token,
            'user'    => [
                'id'    => $newId,
                'name'  => $name,
                'email' => $email,
            ]
        ], 201); // 201 Created

    } catch (PDOException $e) {
        error_log("Register error: " . $e->getMessage());
        sendResponse([
            'success' => false,
            'message' => 'เกิดข้อผิดพลาด กรุณาลองใหม่'
        ], 500);
    }
}
```

### 8.6 สร้าง Users API (CRUD)

```bash
nano /var/www/html/myapp/api/users.php
```

```php
<?php
// ============================================================
// users.php — REST API สำหรับจัดการข้อมูลผู้ใช้
// ต้องมี Token ที่ถูกต้องทุก request
//
// GET    /api/users.php        → ดึงรายชื่อผู้ใช้ทั้งหมด
// GET    /api/users.php?id=1   → ดึงข้อมูลผู้ใช้ตาม ID
// POST   /api/users.php        → สร้างผู้ใช้ใหม่
// PUT    /api/users.php?id=1   → อัปเดตผู้ใช้
// DELETE /api/users.php?id=1   → ลบผู้ใช้
// ============================================================

require_once 'config.php';
require_once 'helpers.php';

// OPTIONS Preflight
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    sendResponse(['ok' => true]);
}

// ตรวจสอบ Token ก่อนทุก request
$token    = getBearerToken();
$tokenData = $token ? verifyToken($token) : null;

if (!$tokenData) {
    sendResponse([
        'success' => false,
        'message' => 'กรุณาเข้าสู่ระบบก่อน'
    ], 401);
}

$method = $_SERVER['REQUEST_METHOD'];
$id     = isset($_GET['id']) ? (int)$_GET['id'] : null;

// Router ตาม HTTP Method
switch ($method) {
    case 'GET':
        $id ? getUserById($id) : getAllUsers();
        break;
    case 'POST':
        createUser();
        break;
    case 'PUT':
        $id ? updateUser($id) : sendResponse(['success'=>false,'message'=>'ต้องระบุ ID'], 400);
        break;
    case 'DELETE':
        $id ? deleteUser($id) : sendResponse(['success'=>false,'message'=>'ต้องระบุ ID'], 400);
        break;
    default:
        sendResponse(['success'=>false,'message'=>'Method ไม่รองรับ'], 405);
}

// GET: ดึงผู้ใช้ทั้งหมด
function getAllUsers(): void {
    $db   = getDB();
    $stmt = $db->query("SELECT id, name, email, created_at FROM users ORDER BY id DESC");
    $users = $stmt->fetchAll();
    sendResponse(['success' => true, 'data' => $users, 'total' => count($users)]);
}

// GET: ดึงผู้ใช้ตาม ID
function getUserById(int $id): void {
    $db   = getDB();
    $stmt = $db->prepare("SELECT id, name, email, created_at FROM users WHERE id = ?");
    $stmt->execute([$id]);
    $user = $stmt->fetch();
    if (!$user) {
        sendResponse(['success' => false, 'message' => 'ไม่พบผู้ใช้'], 404);
    }
    sendResponse(['success' => true, 'data' => $user]);
}

// POST: สร้างผู้ใช้ใหม่
function createUser(): void {
    $body     = getRequestBody();
    $name     = trim($body['name']     ?? '');
    $email    = trim($body['email']    ?? '');
    $password = trim($body['password'] ?? '');

    if (!$name || !$email || !$password) {
        sendResponse(['success'=>false,'message'=>'ข้อมูลไม่ครบ'], 400);
    }

    $db       = getDB();
    $stmt     = $db->prepare("INSERT INTO users (name,email,password) VALUES (?,?,?)");
    $stmt->execute([$name, $email, password_hash($password, PASSWORD_BCRYPT)]);
    sendResponse(['success'=>true,'message'=>'สร้างผู้ใช้สำเร็จ','id'=>$db->lastInsertId()], 201);
}

// PUT: อัปเดตผู้ใช้
function updateUser(int $id): void {
    $body  = getRequestBody();
    $name  = trim($body['name']  ?? '');
    $email = trim($body['email'] ?? '');

    if (!$name || !$email) {
        sendResponse(['success'=>false,'message'=>'ข้อมูลไม่ครบ'], 400);
    }

    $db   = getDB();
    $stmt = $db->prepare("UPDATE users SET name=?, email=? WHERE id=?");
    $stmt->execute([$name, $email, $id]);

    if ($stmt->rowCount() === 0) {
        sendResponse(['success'=>false,'message'=>'ไม่พบผู้ใช้หรือข้อมูลไม่เปลี่ยนแปลง'], 404);
    }
    sendResponse(['success'=>true,'message'=>'อัปเดตสำเร็จ']);
}

// DELETE: ลบผู้ใช้
function deleteUser(int $id): void {
    $db   = getDB();
    $stmt = $db->prepare("DELETE FROM users WHERE id=?");
    $stmt->execute([$id]);

    if ($stmt->rowCount() === 0) {
        sendResponse(['success'=>false,'message'=>'ไม่พบผู้ใช้'], 404);
    }
    sendResponse(['success'=>true,'message'=>'ลบผู้ใช้สำเร็จ']);
}
```

### 8.7 หน้าทดสอบ API (Dashboard)

```bash
nano /var/www/html/myapp/dashboard.html
```

```html
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title>Dashboard - MyApp</title>
    <style>
        body { font-family: sans-serif; padding: 20px; background: #f5f5f5; }
        .container { max-width: 900px; margin: 0 auto; }
        h1 { color: #333; margin-bottom: 20px; }
        .card { background: white; border-radius: 8px; padding: 20px; margin-bottom: 16px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); }
        table { width: 100%; border-collapse: collapse; }
        th, td { padding: 10px 12px; border-bottom: 1px solid #eee; text-align: left; }
        th { background: #f8f9fa; font-weight: 600; color: #555; }
        .btn { padding: 8px 16px; border: none; border-radius: 6px; cursor: pointer; font-size: 14px; }
        .btn-primary { background: #667eea; color: white; }
        .btn-danger  { background: #e53e3e; color: white; }
        #status { padding: 10px; border-radius: 6px; margin-bottom: 16px; display: none; }
        .success { background: #c6f6d5; color: #276749; }
        .error   { background: #fed7d7; color: #c53030; }
    </style>
</head>
<body>
<div class="container">
    <h1>🏠 Dashboard — ระบบจัดการผู้ใช้</h1>
    <div id="status"></div>

    <div class="card">
        <h2>รายชื่อผู้ใช้ในระบบ
            <button class="btn btn-primary" onclick="loadUsers()" style="float:right; font-size:13px">
                🔄 โหลดใหม่
            </button>
        </h2>
        <br>
        <table>
            <thead>
                <tr><th>ID</th><th>ชื่อ</th><th>อีเมล</th><th>วันที่สมัคร</th><th></th></tr>
            </thead>
            <tbody id="userTable"><tr><td colspan="5">กำลังโหลด...</td></tr></tbody>
        </table>
    </div>

    <button class="btn btn-danger" onclick="logout()">ออกจากระบบ</button>
</div>

<script>
    const API_BASE = '/myapp/api';
    const token    = sessionStorage.getItem('authToken');

    // ตรวจสอบ Token ก่อน
    if (!token) window.location.href = 'index.html';

    // โหลดรายชื่อผู้ใช้
    async function loadUsers() {
        try {
            const res = await fetch(`${API_BASE}/users.php`, {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            const data = await res.json();

            if (data.success) {
                const tbody = document.getElementById('userTable');
                tbody.innerHTML = data.data.map(u => `
                    <tr>
                        <td>${u.id}</td>
                        <td>${u.name}</td>
                        <td>${u.email}</td>
                        <td>${new Date(u.created_at).toLocaleDateString('th-TH')}</td>
                        <td><button class="btn btn-danger" onclick="deleteUser(${u.id})">ลบ</button></td>
                    </tr>
                `).join('');
            }
        } catch (e) {
            showStatus('error', 'โหลดข้อมูลล้มเหลว');
        }
    }

    // ลบผู้ใช้
    async function deleteUser(id) {
        if (!confirm(`ยืนยันลบ ID: ${id}?`)) return;
        const res  = await fetch(`${API_BASE}/users.php?id=${id}`, {
            method: 'DELETE',
            headers: { 'Authorization': `Bearer ${token}` }
        });
        const data = await res.json();
        showStatus(data.success ? 'success' : 'error', data.message);
        if (data.success) loadUsers();
    }

    function logout() {
        sessionStorage.clear();
        window.location.href = 'index.html';
    }

    function showStatus(type, msg) {
        const el = document.getElementById('status');
        el.className = type;
        el.textContent = msg;
        el.style.display = 'block';
        setTimeout(() => el.style.display = 'none', 3000);
    }

    // โหลดข้อมูลเมื่อเปิดหน้า
    loadUsers();
</script>
</body>
</html>
```

### 8.8 ทดสอบ API ผ่าน Terminal (curl)

```bash
# ทดสอบ Login
curl -X POST http://localhost/myapp/api/auth.php \
  -H "Content-Type: application/json" \
  -d '{"action":"login","email":"somchai@test.com","password":"password"}' \
  | python3 -m json.tool

# ทดสอบ Register
curl -X POST http://localhost/myapp/api/auth.php \
  -H "Content-Type: application/json" \
  -d '{"action":"register","name":"ทดสอบ","email":"test@new.com","password":"Test1234!"}' \
  | python3 -m json.tool
```

> `python3 -m json.tool` = แสดง JSON แบบจัดรูปแบบสวยงาม

---

## 9. การเข้าถึงผ่าน LAN (Windows 10 + Chrome)

### 9.1 หาค่า IP ของเครื่อง Ubuntu

```bash
hostname -I
# หรือ
ip addr show | grep "inet " | grep -v "127.0.0.1"
```

ตัวอย่างผลลัพธ์: `192.168.1.50`

### 9.2 ตั้งค่า Firewall ให้ LAN เข้าถึงได้

```bash
# เปิดพอร์ต 80 (HTTP)
sudo ufw allow 80/tcp

# ตรวจสอบ
sudo ufw status verbose
```

### 9.3 ทดสอบจากเครื่อง Windows 10

เปิด Chrome แล้วพิมพ์:
```
http://192.168.1.50
```
→ จะเห็นหน้า Apache Default Page

```
http://192.168.1.50/myapp/
```
→ จะเห็นหน้า Sign In ที่สร้างไว้

### 9.4 แก้ปัญหา: เครื่อง Windows เข้าไม่ได้

**ตรวจสอบทีละขั้น:**

```bash
# 1. ดู IP Ubuntu
hostname -I

# 2. ตรวจสอบ Apache ทำงานอยู่
sudo systemctl status apache2

# 3. ตรวจสอบ Firewall
sudo ufw status

# 4. ทดสอบ port 80 เปิดอยู่ไหม
sudo ss -tlnp | grep :80
```

จาก Windows ใช้ CMD:
```cmd
ping 192.168.1.50
```

ถ้า ping ได้ แต่เว็บไม่ขึ้น → ปัญหา Firewall Ubuntu
ถ้า ping ไม่ได้ → ปัญหา Network/Router

### 9.5 ทริปเพิ่มเติม: ตั้ง IP แบบ Static บน Ubuntu

เพื่อให้ IP ของเครื่อง Ubuntu ไม่เปลี่ยนแปลง (เหมาะกับ Server):

```bash
# ดูชื่อ Interface
ip link show

# แก้ไข Netplan
sudo nano /etc/netplan/01-network-manager-all.yaml
```

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp3s0:          # เปลี่ยนตามชื่อ interface จริง
      dhcp4: no
      addresses:
        - 192.168.1.50/24
      routes:
        - to: default
          via: 192.168.1.1   # IP ของ Router
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
sudo netplan apply
```

---

## 10. ติดตั้ง WordPress

### 10.1 สร้างฐานข้อมูลสำหรับ WordPress

```bash
sudo mysql
```

```sql
CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'WpP@ss2026!';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 10.2 ดาวน์โหลดและตั้งค่า WordPress

```bash
# ดาวน์โหลด WordPress ล่าสุด
cd /tmp
wget https://wordpress.org/latest.tar.gz

# แตกไฟล์
tar -xzf latest.tar.gz

# ย้ายไปยัง Document Root
sudo cp -r wordpress /var/www/html/wordpress

# ตั้งสิทธิ์ให้ Apache เข้าถึงได้
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 755 /var/www/html/wordpress

# แก้ค่าเฉพาะโฟลเดอร์ที่ WordPress ต้องเขียน
sudo chmod -R 775 /var/www/html/wordpress/wp-content
```

> **www-data** คือชื่อ user ของ Apache — เราต้องให้เป็นเจ้าของไฟล์เพื่อให้ WordPress อัปโหลดได้

### 10.3 สร้างไฟล์ Config WordPress

```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```

แก้ไขส่วนนี้:
```php
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wpuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'WpP@ss2026!' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8mb4' );
```

**เพิ่ม Secret Keys** — ไปที่ https://api.wordpress.org/secret-key/1.1/salt/ แล้วคัดลอกค่ามาแทนที่ block เดิม (สำคัญสำหรับความปลอดภัย!)

### 10.4 ตั้งค่า Apache Virtual Host สำหรับ WordPress

```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```

```apache
<VirtualHost *:80>
    # ServerName = ชื่อโดเมน ถ้าใช้ LAN ไม่มีโดเมนจริง ใช้ IP แทน
    # ServerName yourdomain.com

    DocumentRoot /var/www/html/wordpress

    <Directory /var/www/html/wordpress>
        # AllowOverride All = ให้ .htaccess ของ WordPress ทำงานได้
        # (สำคัญมาก! ถ้าปิดอยู่ WordPress Permalinks จะพัง)
        AllowOverride All
        Require all granted
    </Directory>

    # Log ไฟล์ของ WordPress
    ErrorLog  ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```

### 10.5 เปิดใช้งาน WordPress Site และ Module

```bash
# เปิดใช้งาน config ที่เพิ่งสร้าง
sudo a2ensite wordpress.conf

# เปิด mod_rewrite (สำหรับ WordPress Permalinks)
sudo a2enmod rewrite

# ตรวจสอบ config ก่อน restart
sudo apache2ctl configtest

# รีสตาร์ท Apache
sudo systemctl restart apache2
```

> **mod_rewrite** คือ module ของ Apache ที่ช่วยแปลง URL สวยๆ เช่น `/blog/hello-world` แทนที่จะเป็น `/?p=1`

### 10.6 ติดตั้ง WordPress ผ่าน Browser

เปิด Chrome แล้วไปที่:
```
http://localhost/wordpress/
```

หรือจาก Windows LAN:
```
http://192.168.1.50/wordpress/
```

ทำตามขั้นตอน Installation Wizard:
1. เลือกภาษา (ภาษาไทยมีให้เลือก)
2. กรอก Site Title, Username, Password, Email
3. คลิก **Install WordPress**
4. เข้า Admin Dashboard ที่: `http://192.168.1.50/wordpress/wp-admin/`

### 10.7 ทริปและเทคนิค WordPress ที่ควรรู้

#### Permalinks (URL สวยๆ)
Settings → Permalinks → เลือก **Post name** → Save
```
ก่อน: http://192.168.1.50/wordpress/?p=1
หลัง: http://192.168.1.50/wordpress/hello-world/
```

#### เพิ่มขนาดไฟล์อัปโหลด
แก้ไข `/etc/php/8.3/apache2/php.ini`:
```ini
upload_max_filesize = 64M
post_max_size       = 64M
```

หรือสร้างไฟล์ `.htaccess` ใน `/var/www/html/wordpress/`:
```apache
php_value upload_max_filesize 64M
php_value post_max_size 64M
```

#### Plugin ที่แนะนำ
| Plugin | ใช้ทำอะไร |
|---|---|
| **Yoast SEO** | ปรับแต่ง SEO |
| **WP Super Cache** | Cache เพิ่มความเร็ว |
| **Wordfence** | ความปลอดภัย |
| **UpdraftPlus** | สำรองข้อมูล |
| **Contact Form 7** | ฟอร์มติดต่อ |
| **WooCommerce** | ร้านค้าออนไลน์ |

#### ย้าย WordPress ไปยัง Root (ไม่มี /wordpress/)

แก้ไข Settings → General:
- WordPress Address: `http://192.168.1.50`
- Site Address: `http://192.168.1.50`

แล้วย้ายไฟล์:
```bash
sudo cp /var/www/html/wordpress/index.php /var/www/html/
sudo cp /var/www/html/wordpress/.htaccess /var/www/html/
sudo nano /var/www/html/index.php
```
แก้บรรทัดสุดท้าย:
```php
require( dirname( __FILE__ ) . '/wordpress/wp-blog-header.php' );
```

---

## 11. การจัดการ Virtual Host (หลายเว็บไซต์)

Virtual Host ช่วยให้เครื่องเดียวรองรับหลายเว็บไซต์

```bash
# ตัวอย่าง: สร้างเว็บไซต์ชื่อ mysite
sudo mkdir -p /var/www/html/mysite
sudo chown -R www-data:www-data /var/www/html/mysite
sudo nano /var/www/html/mysite/index.html
```

```html
<h1>ยินดีต้อนรับสู่ My Site!</h1>
```

```bash
sudo nano /etc/apache2/sites-available/mysite.conf
```

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/mysite
    <Directory /var/www/html/mysite>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog  ${APACHE_LOG_DIR}/mysite_error.log
    CustomLog ${APACHE_LOG_DIR}/mysite_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite mysite.conf
sudo systemctl reload apache2
```

---

## 12. ทริปและเทคนิคที่ควรรู้

### 12.1 ความปลอดภัยพื้นฐาน

```bash
# 1. ซ่อนเวอร์ชัน Apache ไม่ให้แสดงใน Error Page
sudo nano /etc/apache2/conf-available/security.conf
```
แก้:
```apache
ServerTokens Prod
ServerSignature Off
```

```bash
# 2. ปิด Directory Listing (ป้องกันไม่ให้เห็นรายการไฟล์)
sudo nano /etc/apache2/apache2.conf
```
ค้นหา `Options Indexes` แก้เป็น:
```apache
Options -Indexes +FollowSymLinks
```

```bash
sudo systemctl reload apache2
```

### 12.2 phpMyAdmin — จัดการ MySQL แบบ GUI

```bash
sudo apt install phpmyadmin -y
```

ระหว่างติดตั้งจะถาม:
- Web server: เลือก **apache2** (กด Space เพื่อเลือก แล้ว Tab → OK)
- Configure database: เลือก **Yes**
- Password: ตั้งรหัสผ่าน phpMyAdmin

```bash
# เปิดใช้งาน phpMyAdmin
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-enabled/phpmyadmin.conf
sudo systemctl reload apache2
```

เข้าใช้งาน: `http://192.168.1.50/phpmyadmin/`

> **ทริปความปลอดภัย:** เปลี่ยน URL ของ phpMyAdmin ไม่ให้ใช้ชื่อ phpmyadmin ที่ทุกคนรู้จัก

### 12.3 ดู Log แบบ Real-Time

```bash
# ดู Access Log (คำขอทั้งหมด)
sudo tail -f /var/log/apache2/access.log

# ดู Error Log (ข้อผิดพลาด)
sudo tail -f /var/log/apache2/error.log

# ดู MySQL Log
sudo tail -f /var/log/mysql/error.log

# ดูพร้อมกัน
sudo tail -f /var/log/apache2/error.log /var/log/mysql/error.log
```

### 12.4 Permissions ที่ถูกต้องสำหรับ Web

```bash
# โฟลเดอร์: 755 (owner rwx, group r-x, others r-x)
# ไฟล์:     644 (owner rw-, group r--, others r--)
# เจ้าของ:  www-data (Apache user)

sudo find /var/www/html/myapp -type d -exec chmod 755 {} \;
sudo find /var/www/html/myapp -type f -exec chmod 644 {} \;
sudo chown -R www-data:www-data /var/www/html/myapp
```

### 12.5 .htaccess — ไฟล์ config ของ Apache ระดับ Directory

```bash
nano /var/www/html/myapp/.htaccess
```

```apache
# ป้องกันการเข้าถึงไฟล์ config
<Files "config.php">
    Require all denied
</Files>

# Redirect /api ไปยัง /api/index.php
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^api/(.*)$ api/index.php?path=$1 [QSA,L]

# เปิด CORS Header
Header always set Access-Control-Allow-Origin "*"
```

### 12.6 MySQL Backup และ Restore

```bash
# Backup ฐานข้อมูล
mysqldump -u root -p myapp_db > backup_$(date +%Y%m%d).sql

# Restore ฐานข้อมูล
mysql -u root -p myapp_db < backup_20260526.sql

# Backup พร้อมบีบอัด
mysqldump -u root -p myapp_db | gzip > backup_$(date +%Y%m%d).sql.gz

# Restore จากไฟล์บีบอัด
zcat backup_20260526.sql.gz | mysql -u root -p myapp_db
```

### 12.7 ตรวจสอบ Performance

```bash
# ดูการใช้งาน RAM ของ Apache
sudo ps aux | grep apache2 | awk '{sum += $6} END {print sum/1024 " MB"}'

# ดูจำนวน Request พร้อมกัน
sudo apache2ctl status   # ต้องเปิด mod_status ก่อน

# ดูการใช้งาน MySQL
sudo mysqladmin -u root -p status
```

---

## 13. การแก้ปัญหาที่พบบ่อย

### 13.1 Apache ไม่เริ่มทำงาน

```bash
# ดู log error
sudo journalctl -u apache2 -n 50

# ตรวจสอบ config
sudo apache2ctl configtest

# ดู port ที่ใช้งาน (อาจมีโปรแกรมอื่นใช้ port 80 อยู่)
sudo ss -tlnp | grep :80
```

### 13.2 PHP Error: "Call to undefined function"

```bash
# แสดง extension ที่ติดตั้ง
php -m

# ติดตั้ง extension ที่ขาด เช่น gd
sudo apt install php-gd
sudo systemctl restart apache2
```

### 13.3 MySQL Access Denied

```bash
# รีเซ็ตสิทธิ์
sudo mysql
```
```sql
DROP USER IF EXISTS 'myuser'@'localhost';
CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'NewPass2026!';
GRANT ALL PRIVILEGES ON mydb.* TO 'myuser'@'localhost';
FLUSH PRIVILEGES;
```

### 13.4 WordPress แสดง "Error establishing a database connection"

ตรวจสอบ:
1. MySQL ทำงานอยู่: `sudo systemctl status mysql`
2. ข้อมูล DB ใน `wp-config.php` ถูกต้อง
3. User มีสิทธิ์เข้า DB:
```bash
mysql -u wpuser -p wordpress
```

### 13.5 WordPress Permalinks ไม่ทำงาน (404)

```bash
# ตรวจสอบว่า mod_rewrite เปิดอยู่
sudo a2enmod rewrite
sudo systemctl restart apache2

# ตรวจสอบ AllowOverride ใน config
grep -r "AllowOverride" /etc/apache2/
# ต้องเป็น AllowOverride All ไม่ใช่ AllowOverride None
```

### 13.6 ไฟล์ CSS/JS ไม่โหลด (403 Forbidden)

```bash
# แก้ permission
sudo chmod -R 755 /var/www/html/myapp
sudo chown -R www-data:www-data /var/www/html/myapp
```

### 13.7 Upload ไฟล์ไม่ได้ (File too large)

```bash
sudo nano /etc/php/8.3/apache2/php.ini
```
แก้:
```ini
upload_max_filesize = 64M
post_max_size       = 64M
```
```bash
sudo systemctl restart apache2
```

---

## สรุปคำสั่งที่ใช้บ่อย (Quick Reference)

```bash
# ===== Apache =====
sudo systemctl start|stop|restart|reload apache2
sudo apache2ctl configtest        # ตรวจ config
sudo a2ensite|a2dissite xxx.conf  # เปิด/ปิด site
sudo a2enmod|a2dismod rewrite     # เปิด/ปิด module
sudo tail -f /var/log/apache2/error.log

# ===== MySQL =====
sudo systemctl start|stop|restart mysql
sudo mysql                        # เข้า shell
mysql -u username -p              # เข้าด้วย user
mysqldump -u root -p db > bak.sql # backup
sudo tail -f /var/log/mysql/error.log

# ===== PHP =====
php -v                            # ดูเวอร์ชัน
php -m                            # ดู extension
sudo nano /etc/php/8.3/apache2/php.ini

# ===== Permissions =====
sudo chown -R www-data:www-data /var/www/html/myapp
sudo chmod -R 755 /var/www/html/myapp

# ===== Network =====
hostname -I                       # IP เครื่อง
sudo ufw allow 80/tcp             # เปิด port 80
sudo ufw status                   # ดู firewall
```

---

## ข้อมูลอ้างอิง

- Apache Docs: https://httpd.apache.org/docs/2.4/
- MySQL Docs: https://dev.mysql.com/doc/
- PHP Docs: https://www.php.net/manual/
- WordPress Codex: https://codex.wordpress.org/
- Ubuntu Docs: https://help.ubuntu.com/

---

*คู่มือนี้จัดทำ ณ วันที่ 26 พฤษภาคม 2566*
*Ubuntu Desktop 24.04 LTS | Apache 2.4 | MySQL 8.x | PHP 8.3*
*สำหรับการใช้งานใน LAN ระหว่าง Ubuntu Server กับ Windows 10 Client*
