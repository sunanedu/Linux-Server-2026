# คู่มือติดตั้ง LAMP + WordPress บน Ubuntu Server 24.04 สำหรับมือใหม่

คู่มือนี้อธิบายตั้งแต่การเชื่อมต่อเข้าเซิร์ฟเวอร์ ติดตั้ง Apache, PHP, MariaDB, phpMyAdmin และ WordPress ไปจนถึงการเปิดหน้าเว็บไซต์ครั้งแรก

> **คำแนะนำ:** ให้คัดลอกคำสั่งไปรันทีละชุด อย่าวางทุกคำสั่งพร้อมกัน เพราะเมื่อเกิดข้อผิดพลาดจะหาสาเหตุได้ยาก

---

## สิ่งที่ต้องมีก่อนเริ่ม

1. เซิร์ฟเวอร์ที่ติดตั้ง **Ubuntu Server 24.04** แล้ว
2. ชื่อผู้ใช้ที่สามารถใช้คำสั่ง `sudo ip addr` ได้
3. IP Address ของเซิร์ฟเวอร์ เช่น `192.168.1.50` หรือ Public IP ที่ผู้ให้บริการ Cloud กำหนดให้
4. โปรแกรมสำหรับเชื่อมต่อ SSH
   - Windows: ใช้ MobaXterm , PowerShell, Windows Terminal หรือ PuTTY
   - macOS/Linux: ใช้ Terminal
5. เครื่องของคุณต้องเชื่อมต่อเครือข่ายที่เข้าถึงเซิร์ฟเวอร์ได้

ในคู่มือนี้จะใช้ข้อความต่อไปนี้เป็น **ตัวแทนค่า**:

- `SERVER-IP` หมายถึง IP Address ของเซิร์ฟเวอร์
- `YOUR-USERNAME` หมายถึงชื่อผู้ใช้บน Ubuntu เช่น it รหัสผ่าน 1234
- `YOUR_DB_PASSWORD` หมายถึงรหัสผ่านฐานข้อมูลที่คุณตั้งเอง

เมื่อพบข้อความเหล่านี้ ให้เปลี่ยนเป็นค่าจริงของคุณ โดยไม่ต้องใส่เครื่องหมาย `<` หรือ `>` เพิ่ม

---

## 0. เชื่อมต่อเข้า Ubuntu Server ด้วย MobaXterm

ใส่ไอพี ใส่ user จากนั้นใส่รหัสผ่านของผู้ใช้ it

> ขณะพิมพ์รหัสผ่าน หน้าจอจะไม่แสดงตัวอักษรหรือเครื่องหมายดอกจัน ซึ่งเป็นพฤติกรรมปกติ ให้พิมพ์รหัสผ่านแล้วกด `Enter`

เมื่อเชื่อมต่อสำเร็จ จะเห็นข้อความรับคำสั่งคล้ายแบบนี้:

```text
it@server:~$
```

คำสั่งตั้งแต่หัวข้อถัดไปให้รันบน **Ubuntu Server ผ่านหน้าต่าง SSH นี้**

### ตรวจสอบ IP Address ของเซิร์ฟเวอร์

รัน:

```bash
hostname -I
```

ระบบอาจแสดงหลาย IP ให้เลือก IP ที่เครื่องของคุณเข้าถึงได้ หากเป็น Cloud Server ให้ตรวจสอบ Public IP จากหน้า Control Panel ของผู้ให้บริการด้วย

---

## 1. อัปเดตรายการแพ็กเกจ

รัน:

```bash
sudo apt update 
```

ใช้ในกรณีต้องการอัพทุกแพ็กเกจให้เป็นปัจจุบัน ไม่แนะนำให้รันคำสั่งนี้เพราะใช้เวลานานมาก 
```bash
sudo apt upgrade -y
```

หากระบบแจ้งว่ามีบริการที่ต้องรีสตาร์ต ให้ยอมรับค่าที่ระบบเลือกไว้ตามปกติ

---

## 2. ติดตั้ง Apache, MariaDB, PHP และเครื่องมือที่จำเป็น

เพื่อให้ตรวจสอบข้อผิดพลาดได้ง่าย คู่มือนี้จะแยกคำสั่งเดิมออกเป็นหลายขั้นตอน

ติดตั้ง Apache และ MariaDB:

```bash
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-cli php-common php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl unzip curl wget
```

สั่งให้ Apache และ MariaDB เริ่มทำงานทันที และเริ่มอัตโนมัติเมื่อเปิดเครื่อง:

```bash
sudo systemctl enable --now apache2 mariadb
```

### ตรวจสอบว่า Apache ทำงานหรือไม่

```bash
sudo systemctl status apache2 --no-pager
```

ควรเห็นคำว่า:

```text
active (running)
```

กด `q` เพื่อออก หากหน้าจอสถานะยังเปิดค้างอยู่

ตรวจสอบ MariaDB:

```bash
sudo systemctl status mariadb --no-pager
```

ควรเห็นคำว่า:

```text
active (running)
```

### ทดสอบหน้า Apache

เปิดเว็บเบราว์เซอร์บนเครื่องของคุณ แล้วเข้า:

```text
http://SERVER-IP
```

ตัวอย่าง:

```text
http://192.168.1.50
```

หากติดตั้งสำเร็จ จะเห็นหน้า **Apache2 Ubuntu Default Page**

### กรณีเปิดหน้าเว็บไม่ได้

ตรวจสอบสถานะ Firewall:

```bash
sudo ufw status
```

หากแสดงว่า `Status: active` ให้เปิดพอร์ต SSH และเว็บก่อน:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Apache Full'
```

ตรวจสอบอีกครั้ง:

```bash
sudo ufw status
```

> หากใช้ Cloud Server ต้องเปิดพอร์ต `80` สำหรับ HTTP และพอร์ต `443` สำหรับ HTTPS ใน Firewall หรือ Security Group ของผู้ให้บริการด้วย การเปิดเฉพาะ UFW ภายใน Ubuntu อาจยังไม่เพียงพอ

---

## 3. ตั้งค่าความปลอดภัยเบื้องต้นให้ MariaDB

รัน:

```bash
sudo mysql_secure_installation
```

คำถามที่แสดงอาจแตกต่างกันเล็กน้อยตามรุ่นของ MariaDB โดยคำตอบที่เหมาะสำหรับการติดตั้งทั่วไปมีดังนี้:

| คำถามโดยประมาณ | คำตอบแนะนำ | ความหมาย |
|---|---:|---|
| Enter current password for root | กด `Enter` | การติดตั้งใหม่มักยังไม่มีรหัสผ่าน root แบบฐานข้อมูล |
| Switch to unix_socket authentication | `Y` | ให้ผู้ดูแลระบบเข้า MariaDB ผ่าน `sudo` |
| Change the root password | `N` | ไม่จำเป็นเมื่อใช้ unix socket; หากระบบของคุณถามต่างออกไปให้ตั้งตามนโยบายที่ต้องการ |
| Remove anonymous users | `Y` | ลบผู้ใช้ที่ไม่ระบุตัวตน |
| Disallow root login remotely | `Y` | ไม่อนุญาตให้ root เข้า MariaDB จากภายนอก |
| Remove test database | `Y` | ลบฐานข้อมูลทดสอบ |
| Reload privilege tables now | `Y` | โหลดสิทธิ์ใหม่ทันที |

ให้กด `Enter` หลังพิมพ์คำตอบแต่ละข้อ

---

## 4. สร้างฐานข้อมูลและผู้ใช้สำหรับ WordPress

### 4.1 สร้างรหัสผ่านฐานข้อมูล

คุณสามารถสร้างรหัสผ่านแบบสุ่มด้วยคำสั่งนี้:

```bash
openssl rand -hex 24
```

ระบบจะแสดงข้อความยาวหนึ่งบรรทัด ให้คัดลอกเก็บไว้ชั่วคราว เพราะต้องใช้รหัสผ่านเดียวกันในไฟล์ `wp-config.php` ภายหลัง

ในคู่มือนี้จะเรียกรหัสผ่านนั้นว่า:

```text
YOUR_DB_PASSWORD
```

> ห้ามใช้คำว่า `YOUR_DB_PASSWORD` เป็นรหัสผ่านจริง และไม่ควรใช้รหัสผ่านตัวอย่าง `StrongPassword123!` บนระบบที่เปิดใช้งานจริง

### 4.2 เข้า MariaDB ในฐานะผู้ดูแลระบบ

```bash
sudo mariadb
```

เมื่อเข้าได้แล้ว ข้อความรับคำสั่งจะเปลี่ยนเป็นลักษณะนี้:

```text
MariaDB [(none)]>
```

### 4.3 สร้างฐานข้อมูล

คัดลอกคำสั่งต่อไปนี้ไปรันทีละบรรทัด โดยทุกบรรทัดต้องลงท้ายด้วยเครื่องหมาย `;`

```sql
CREATE DATABASE wordpress CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

หากสำเร็จจะเห็นข้อความคล้าย:

```text
Query OK, 1 row affected
```

### 4.4 สร้างผู้ใช้ฐานข้อมูล

เปลี่ยน `YOUR_DB_PASSWORD` เป็นรหัสผ่านที่สร้างไว้ โดยยังคงเครื่องหมายอัญประกาศเดี่ยว `' '` รอบรหัสผ่าน

```sql
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'YOUR_DB_PASSWORD';
```

ตัวอย่างรูปแบบเท่านั้น:

```sql
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'ใส่รหัสผ่านจริงของคุณตรงนี้';
หรือกำเองแบบง่ายๆ
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'Password123!';
```

### 4.5 ให้สิทธิ์ผู้ใช้กับฐานข้อมูล WordPress

```sql
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
```

โหลดสิทธิ์ใหม่:

```sql
FLUSH PRIVILEGES;
```

ออกจาก MariaDB:

```sql
EXIT;
```

เมื่อออกสำเร็จ จะกลับมาเห็นข้อความรับคำสั่ง Ubuntu คล้าย:

```text
it@server:~$
```

## 5. ติดตั้ง phpMyAdmin

phpMyAdmin เป็นหน้าเว็บสำหรับจัดการฐานข้อมูล ขั้นตอนนี้ไม่จำเป็นต่อการทำงานของ WordPress แต่ช่วยให้มือใหม่ตรวจสอบฐานข้อมูลได้สะดวกขึ้น

รัน:

```bash
sudo apt install phpmyadmin
```

ระหว่างติดตั้งอาจมีหน้าจอแบบข้อความปรากฏขึ้น

### 5.1 เลือก Web Server

เมื่อเห็นหัวข้อประมาณว่า:

```text
Web server to reconfigure automatically
```

ให้ทำดังนี้:

1. ใช้ปุ่มลูกศรเลื่อนไปที่ `apache2`
2. กดปุ่ม `Space` เพื่อให้มีเครื่องหมาย `*` หน้า `apache2`
3. กด `Tab` เพื่อเลื่อนไปที่ `<Ok>`
4. กด `Enter`

> การเลื่อนไปที่ `apache2` แล้วกด `Enter` ทันทีอาจไม่ได้เลือก Apache ต้องกด `Space` ให้เกิดเครื่องหมาย `*` ก่อน

### 5.2 ตั้งค่าฐานข้อมูลสำหรับ phpMyAdmin

หากระบบถาม:

```text
Configure database for phpmyadmin with dbconfig-common?
```

เลือก `<Yes>` แล้วกด `Enter`

จากนั้นระบบอาจขอให้ตั้ง **phpMyAdmin application password** รหัสผ่านนี้เป็นของฐานข้อมูลภายในที่ phpMyAdmin ใช้เอง ไม่จำเป็นต้องเป็นรหัสเดียวกับ `wpuser`

ตั้งรหัสผ่านใหม่ แล้วพิมพ์ซ้ำเพื่อยืนยัน หรือปล่อยว่างในกรณีที่หน้าติดตั้งระบุว่าสามารถให้ระบบสร้างให้อัตโนมัติ

### 5.3 เปิดการตั้งค่า phpMyAdmin ใน Apache

หลังติดตั้งเสร็จ ลองเปิดการตั้งค่าด้วยคำสั่ง:

```bash
sudo a2enconf phpmyadmin
```

จากนั้นตรวจสอบไฟล์ตั้งค่า Apache:

```bash
sudo apache2ctl configtest
```

หากถูกต้องควรแสดง:

```text
Syntax OK
```

รีโหลด Apache:

```bash
sudo systemctl reload apache2
```

### 5.4 เปิด phpMyAdmin

เปิดเว็บเบราว์เซอร์แล้วเข้า:

```text
http://SERVER-IP/phpmyadmin
```

เข้าสู่ระบบด้วย:

```text
Username: wpuser
Password: YOUR_DB_PASSWORD		หรือ Password123!
```

ผู้ใช้ `wpuser` จะมองเห็นและจัดการเฉพาะฐานข้อมูล `wordpress` ตามสิทธิ์ที่ตั้งไว้

### หากเข้า phpMyAdmin แล้วขึ้น 404 Not Found

ตรวจสอบว่ามีไฟล์ตั้งค่าหรือไม่:

```bash
ls -l /etc/apache2/conf-available/phpmyadmin.conf
```

หากไฟล์มีอยู่ ให้รันอีกครั้ง:

```bash
sudo a2enconf phpmyadmin
sudo systemctl reload apache2
```

หากไม่มีไฟล์ดังกล่าว แต่มีโฟลเดอร์ `/usr/share/phpmyadmin` ให้ใช้วิธีสำรองนี้:

```bash
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
sudo systemctl restart apache2
```

หากคำสั่ง `ln` แจ้งว่าไฟล์มีอยู่แล้ว ไม่ต้องสร้างซ้ำ

---

## 6. ดาวน์โหลดและติดตั้งไฟล์ WordPress

ไปยังโฟลเดอร์ชั่วคราว:

```bash
cd /tmp
```

ลบไฟล์ดาวน์โหลดเก่าที่อาจค้างจากการทดลองครั้งก่อน:

```bash
rm -f latest.zip
rm -rf wordpress
```

ดาวน์โหลด WordPress รุ่นล่าสุด:

```bash
wget https://wordpress.org/latest.zip
```

แตกไฟล์:

```bash
unzip latest.zip
```

สร้างโฟลเดอร์เว็บไซต์:

```bash
sudo mkdir -p /var/www/html/wp
```

คัดลอกไฟล์ WordPress เข้าไป:

```bash
sudo cp -a wordpress/. /var/www/html/wp/
```

ตั้งเจ้าของไฟล์ให้ Apache สามารถใช้งานได้:

```bash
sudo chown -R www-data:www-data /var/www/html/wp
```

ตั้งสิทธิ์ของโฟลเดอร์เป็น `755`:

```bash
sudo find /var/www/html/wp -type d -exec chmod 755 {} \;
```

ตั้งสิทธิ์ของไฟล์เป็น `644`:

```bash
sudo find /var/www/html/wp -type f -exec chmod 644 {} \;
```

ตรวจสอบว่าไฟล์ถูกคัดลอกแล้ว:

```bash
ls -la /var/www/html/wp
```

ควรเห็นไฟล์และโฟลเดอร์ เช่น:

```text
index.php
wp-admin
wp-content
wp-includes
wp-config-sample.php
```

---

## 7. สร้างและแก้ไขไฟล์ wp-config.php

ไปยังโฟลเดอร์ WordPress:

```bash
cd /var/www/html/wp
```

คัดลอกไฟล์ตัวอย่างเป็นไฟล์ตั้งค่าจริง:

```bash
sudo cp wp-config-sample.php wp-config.php
```

เปิดไฟล์ด้วยโปรแกรม `nano`:

```bash
sudo nano wp-config.php
```

### 7.1 ค้นหาส่วนตั้งค่าฐานข้อมูล

ใน `nano` กด:

```text
Ctrl + W
```

พิมพ์:

```text
DB_NAME
```

แล้วกด `Enter`

แก้บรรทัดฐานข้อมูลให้เป็นรูปแบบต่อไปนี้:

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'YOUR_DB_PASSWORD' );
define( 'DB_HOST', 'localhost' );
```

เปลี่ยนเฉพาะ `YOUR_DB_PASSWORD` เป็นรหัสผ่านจริงที่สร้างไว้

> รูปแบบที่ถูกต้องต้องมี `define(...)`, เครื่องหมายอัญประกาศ และเครื่องหมาย `;` ท้ายบรรทัด ไม่ควรเขียนเพียง `DB_NAME=wordpress`

ค่าอื่น เช่น `DB_CHARSET` และ `DB_COLLATE` ให้ใช้ค่าเดิมในไฟล์

### 7.2 บันทึกและออกจาก nano

1. กด `Ctrl + O` เพื่อบันทึก
2. กด `Enter` เพื่อยืนยันชื่อไฟล์
3. กด `Ctrl + X` เพื่อออก

### 7.3 สร้าง Security Keys และ Salts

Security Keys ช่วยให้ Cookie และข้อมูลการเข้าสู่ระบบของ WordPress คาดเดาได้ยากขึ้น

แสดงชุดคีย์ใหม่ด้วยคำสั่ง:

```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

ระบบจะแสดงคำสั่ง `define(...)` จำนวน 8 บรรทัด เริ่มตั้งแต่ `AUTH_KEY` ไปจนถึง `NONCE_SALT` ให้คัดลอกผลลัพธ์ทั้ง 8 บรรทัด

เปิดไฟล์ตั้งค่าอีกครั้ง:

```bash
sudo nano /var/www/html/wp/wp-config.php
```

กด `Ctrl + W` ค้นหาคำว่า:

```text
AUTH_KEY
```

ลบบรรทัดคีย์ตัวอย่างเดิมทั้ง 8 บรรทัด แล้ววางชุดคีย์ที่คัดลอกมาแทน จากนั้นบันทึกด้วย `Ctrl + O`, กด `Enter` และออกด้วย `Ctrl + X`

> อย่านำชุด Security Keys จากตัวอย่างของผู้อื่นมาใช้ และไม่ต้องจดจำคีย์เหล่านี้ WordPress จะอ่านจากไฟล์โดยอัตโนมัติ

### 7.4 ตรวจสอบว่า PHP อ่านไฟล์ได้

```bash
sudo php -l /var/www/html/wp/wp-config.php
```

หากถูกต้องควรแสดงข้อความคล้าย:

```text
No syntax errors detected in /var/www/html/wp/wp-config.php
```

### 7.5 จำกัดสิทธิ์ของไฟล์ตั้งค่า

ไฟล์นี้มีรหัสผ่านฐานข้อมูล จึงควรจำกัดสิทธิ์มากกว่าไฟล์ทั่วไป:

```bash
sudo chown root:www-data /var/www/html/wp/wp-config.php
sudo chmod 640 /var/www/html/wp/wp-config.php
```

---

## 8. เปิดใช้งานลิงก์ถาวรของ WordPress

เปิดโมดูล rewrite ของ Apache:

```bash
sudo a2enmod rewrite
```

สร้างไฟล์ตั้งค่า Apache สำหรับโฟลเดอร์ WordPress:

```bash
sudo nano /etc/apache2/conf-available/wordpress.conf
```

วางข้อความต่อไปนี้ลงในไฟล์:

```apache
<Directory /var/www/html/wp/>
    Options FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

บันทึกด้วย `Ctrl + O`, กด `Enter` แล้วออกด้วย `Ctrl + X`

เปิดใช้งานไฟล์ตั้งค่า:

```bash
sudo a2enconf wordpress
```

ตรวจสอบไวยากรณ์ Apache:

```bash
sudo apache2ctl configtest
```

ต้องเห็น:

```text
Syntax OK
```

รีโหลด Apache:

```bash
sudo systemctl reload apache2
```

---

## 9. เปิดหน้าติดตั้ง WordPress ครั้งแรก

เปิดเว็บเบราว์เซอร์บนเครื่องของคุณ แล้วเข้า:

```text
http://SERVER-IP/wp
```

ตัวอย่าง:

```text
http://192.168.1.50/wp
```

หากทุกอย่างถูกต้อง จะเห็นหน้าสำหรับเลือกภาษาและตั้งค่าเว็บไซต์

กรอกข้อมูลดังนี้:

- **Site Title:** ชื่อเว็บไซต์
- **Username:** ชื่อผู้ดูแลระบบ WordPress
- **Password:** รหัสผ่านผู้ดูแลระบบ WordPress
- **Your Email:** อีเมลของผู้ดูแลระบบ
- **Search engine visibility:** ใช้เฉพาะกรณียังไม่ต้องการให้ Search Engine เก็บเว็บไซต์

> รหัสผ่านผู้ดูแล WordPress เป็นคนละรหัสกับรหัสผ่าน `wpuser` ของ MariaDB ควรใช้รหัสที่แตกต่างกัน

ไม่แนะนำให้ใช้ชื่อผู้ดูแลว่า `admin` เพราะคาดเดาได้ง่าย

กด **Install WordPress** แล้วรอจนระบบติดตั้งเสร็จ

---

## 10. URL สำคัญหลังติดตั้ง

หน้าเว็บไซต์:

```text
http://SERVER-IP/wp/
```

หน้าเข้าสู่ระบบผู้ดูแล:

```text
http://SERVER-IP/wp/wp-admin/
```

หน้า phpMyAdmin:

```text
http://SERVER-IP/phpmyadmin/
```

---

## 11. ตรวจสอบระบบหลังติดตั้ง

ตรวจสอบเวอร์ชัน PHP:

```bash
php -v
```

ตรวจสอบโมดูล PHP ที่เปิดใช้งาน:

```bash
php -m
```

ตรวจสอบบริการทั้งหมด:

```bash
sudo systemctl is-active apache2
sudo systemctl is-active mariadb
```

แต่ละคำสั่งควรตอบว่า:

```text
active
```

---

## 12. วิธีแก้ปัญหาเบื้องต้น

### 12.1 เปิด `http://SERVER-IP` ไม่ได้

ตรวจสอบ Apache:

```bash
sudo systemctl status apache2 --no-pager
```

ตรวจสอบพอร์ตที่ Apache กำลังฟัง:

```bash
sudo ss -tulpn | grep -E ':80|:443'
```

ตรวจสอบ UFW:

```bash
sudo ufw status
```

หากเป็น Cloud Server ให้ตรวจสอบ Firewall หรือ Security Group ของผู้ให้บริการด้วย

### 12.2 WordPress แสดง Error establishing a database connection

ทดสอบการเข้าสู่ฐานข้อมูล:

```bash
mariadb -u wpuser -p wordpress
```

หากเข้าได้ แสดงว่าฐานข้อมูลและรหัสผ่านถูกต้อง ให้กลับไปตรวจสอบค่าในไฟล์:

```bash
sudo nano /var/www/html/wp/wp-config.php
```

ตรวจสอบสี่ค่านี้:

```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'รหัสผ่านจริงของคุณ' );
define( 'DB_HOST', 'localhost' );
```

### 12.3 หน้าเว็บขึ้น 403 Forbidden

ตั้งเจ้าของและสิทธิ์ใหม่:

```bash
sudo chown -R www-data:www-data /var/www/html/wp
sudo find /var/www/html/wp -type d -exec chmod 755 {} \;
sudo find /var/www/html/wp -type f -exec chmod 644 {} \;
sudo chown root:www-data /var/www/html/wp/wp-config.php
sudo chmod 640 /var/www/html/wp/wp-config.php
sudo systemctl reload apache2
```

### 12.4 หน้าเว็บว่างหรือเกิด Internal Server Error

ดูบันทึกข้อผิดพลาดล่าสุดของ Apache:

```bash
sudo tail -n 50 /var/log/apache2/error.log
```

ดูบันทึกแบบต่อเนื่อง:

```bash
sudo tail -f /var/log/apache2/error.log
```

กด `Ctrl + C` เพื่อหยุดการดูแบบต่อเนื่อง

ตรวจสอบไวยากรณ์ Apache:

```bash
sudo apache2ctl configtest
```

### 12.5 แก้ไฟล์ใน nano แล้วไม่รู้จะออกอย่างไร

- บันทึก: `Ctrl + O` แล้วกด `Enter`
- ออก: `Ctrl + X`
- ออกโดยไม่บันทึก: `Ctrl + X` แล้วกด `N`
- ค้นหาข้อความ: `Ctrl + W`

เครื่องหมาย `^` ที่แสดงด้านล่างของ nano หมายถึงปุ่ม `Ctrl` เช่น `^X` หมายถึง `Ctrl + X`

---

## 13. คำแนะนำด้านความปลอดภัยก่อนใช้งานจริง

1. อย่าใช้รหัสผ่านตัวอย่างจากคู่มือ
2. ใช้รหัสผ่านคนละชุดสำหรับ Ubuntu, MariaDB และผู้ดูแล WordPress
3. อัปเดต Ubuntu เป็นประจำ:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

4. อัปเดต WordPress, Theme และ Plugin ผ่านหน้า Dashboard
5. สำรองทั้งไฟล์เว็บไซต์และฐานข้อมูลก่อนอัปเดตครั้งใหญ่
6. หากเปิดเว็บไซต์ผ่านอินเทอร์เน็ต ควรมี Domain Name และติดตั้ง HTTPS
7. ไม่ควรเปิดพอร์ต MariaDB `3306` ออกสู่อินเทอร์เน็ต หาก WordPress และ MariaDB อยู่บนเซิร์ฟเวอร์เครื่องเดียวกัน
8. จำกัดการเข้าถึง phpMyAdmin หรือปิดใช้งานเมื่อไม่จำเป็น

---

## 14. เช็กลิสต์สรุป

- [ ] เชื่อมต่อ SSH เข้าเซิร์ฟเวอร์ได้
- [ ] Apache แสดงสถานะ `active (running)`
- [ ] MariaDB แสดงสถานะ `active (running)`
- [ ] เปิด `http://SERVER-IP` แล้วเห็นหน้า Apache
- [ ] สร้างฐานข้อมูล `wordpress` แล้ว
- [ ] สร้างผู้ใช้ `wpuser` แล้ว
- [ ] ทดสอบ `mariadb -u wpuser -p` ผ่าน
- [ ] เปิด `http://SERVER-IP/phpmyadmin` ได้
- [ ] คัดลอก WordPress ไปที่ `/var/www/html/wp` แล้ว
- [ ] แก้ `wp-config.php` ด้วยรหัสผ่านจริงแล้ว
- [ ] คำสั่ง `php -l wp-config.php` ไม่พบ Syntax Error
- [ ] เปิด `http://SERVER-IP/wp` และติดตั้ง WordPress สำเร็จ
- [ ] เข้า `http://SERVER-IP/wp/wp-admin` ได้

---

## คำสั่งรวมสำหรับผู้ที่เข้าใจขั้นตอนแล้ว

ส่วนนี้เป็นคำสั่งแบบย่อสำหรับใช้อ้างอิงภายหลัง ไม่แนะนำให้มือใหม่เริ่มจากส่วนนี้

```bash
sudo apt update
sudo apt install -y apache2 mariadb-server php libapache2-mod-php php-cli php-common php-mysql php-curl php-gd php-mbstring php-xml php-zip php-intl unzip curl wget
sudo systemctl enable --now apache2 mariadb
```

จากนั้นดำเนินการตามหัวข้อ MariaDB, phpMyAdmin, WordPress และ `wp-config.php` ด้านบน เนื่องจากขั้นตอนเหล่านั้นต้องใช้รหัสผ่านและมีหน้าจอโต้ตอบ จึงไม่ควรรวมเป็นคำสั่งอัตโนมัติบรรทัดเดียว
