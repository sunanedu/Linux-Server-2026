# คู่มือติดตั้งและใช้งาน DNS Server บน Ubuntu Desktop 24.04 LTS
### สำหรับผู้เริ่มต้น — BIND9 ครบทุกฟังก์ชัน + บริการคู่กับ LAMP Web Server

---

## สารบัญ

1. [DNS คืออะไร? ทำความเข้าใจก่อนเริ่ม](#1-dns-คืออะไร-ทำความเข้าใจก่อนเริ่ม)
2. [ทำไมถึงเลือก BIND9?](#2-ทำไมถึงเลือก-bind9)
3. [โครงสร้างระบบและ Network ในคู่มือนี้](#3-โครงสร้างระบบและ-network-ในคู่มือนี้)
4. [ตั้งค่า Network การ์ด LAN ทั้งสองใบ](#4-ตั้งค่า-network-การ์ด-lan-ทั้งสองใบ)
5. [ติดตั้ง BIND9](#5-ติดตั้ง-bind9)
6. [ทำความเข้าใจโครงสร้างไฟล์ BIND9](#6-ทำความเข้าใจโครงสร้างไฟล์-bind9)
7. [ตั้งค่าไฟล์หลัก named.conf.options](#7-ตั้งค่าไฟล์หลัก-namedconfoptions)
8. [ตั้งค่า Zone (named.conf.local)](#8-ตั้งค่า-zone-namedconflocal)
9. [สร้าง Zone File (Forward Zone)](#9-สร้าง-zone-file-forward-zone)
10. [สร้าง Reverse Zone File](#10-สร้าง-reverse-zone-file)
11. [เพิ่ม Subdomain ต่าง ๆ](#11-เพิ่ม-subdomain-ต่าง-)
12. [ตรวจสอบและเริ่มใช้งาน BIND9](#12-ตรวจสอบและเริ่มใช้งาน-bind9)
13. [ตั้งค่า Firewall สำหรับ DNS](#13-ตั้งค่า-firewall-สำหรับ-dns)
14. [ตั้งค่า Windows 10 Client ให้ใช้ DNS นี้](#14-ตั้งค่า-windows-10-client-ให้ใช้-dns-นี้)
15. [บูรณาการกับ LAMP Web Server](#15-บูรณาการกับ-lamp-web-server)
16. [การดูแลรักษาและ Log](#16-การดูแลรักษาและ-log)
17. [ทริปและเทคนิคสำหรับมือใหม่](#17-ทริปและเทคนิคสำหรับมือใหม่)
18. [การแก้ปัญหาที่พบบ่อย](#18-การแก้ปัญหาที่พบบ่อย)
19. [ตารางสรุปคำศัพท์สำคัญ](#19-ตารางสรุปคำศัพท์สำคัญ)

---

## 1. DNS คืออะไร? ทำความเข้าใจก่อนเริ่ม

### DNS (Domain Name System) คืออะไร?

**DNS** คือ "สมุดโทรศัพท์ของอินเทอร์เน็ต" — แปลงชื่อที่มนุษย์อ่านได้ เช่น `www.it-sisat-dns.com` ให้กลายเป็น IP Address ที่เครื่องคอมพิวเตอร์เข้าใจ เช่น `172.16.10.1`

```
ผู้ใช้พิมพ์:  www.it-sisat-dns.com
                      │
                      ▼
              [DNS Server BIND9]
              "www.it-sisat-dns.com = 172.16.10.1"
                      │
                      ▼
              Browser เชื่อมต่อ 172.16.10.1
                      │
                      ▼
              [Apache Web Server บน Ubuntu]
                      │
                      ▼
              แสดงหน้าเว็บ ✅
```

### ทำไมต้องมี DNS?

- **ไม่ต้องจำ IP** — จำแค่ชื่อ เช่น `wp.it-sisat-dns.com` แทน `172.16.10.1`
- **ยืดหยุ่น** — เปลี่ยน IP ได้โดยไม่ต้องบอก client ทุกคน แก้แค่ DNS
- **รองรับหลาย subdomain** — `www`, `wp`, `bot`, `ftp` ชี้ไป IP เดียวกันได้
- **ใช้งาน LAN ได้** — ตั้งชื่อโดเมนสมมุติใช้ภายในได้โดยไม่ต้องจดทะเบียน

### การทำงานของ DNS แบบ Step-by-Step

```
1. User พิมพ์ www.it-sisat-dns.com ใน Browser
   │
2. Windows 10 ถาม DNS Server (172.16.10.1) ว่า IP คือเลขอะไร?
   │
3. BIND9 ตรวจสอบ Zone File ของ it-sisat-dns.com
   │
4. BIND9 ตอบกลับ: "172.16.10.1"
   │
5. Browser เชื่อมต่อ 172.16.10.1 → Apache ตอบสนอง
```

### คำศัพท์สำคัญที่ต้องรู้

| คำศัพท์ | อธิบายแบบมือใหม่ | ตัวอย่าง |
|---------|----------------|---------|
| **DNS** | สมุดโทรศัพท์แปลชื่อ→IP | google.com → 142.250.x.x |
| **Domain** | ชื่อที่ตั้งให้เครือข่าย | it-sisat-dns.com |
| **Subdomain** | ชื่อย่อยของ domain | www, wp, bot |
| **FQDN** | ชื่อเต็มสมบูรณ์ (ลงท้าย.) | www.it-sisat-dns.com. |
| **Zone** | พื้นที่ความรับผิดชอบของ DNS | it-sisat-dns.com zone |
| **Zone File** | ไฟล์ที่เก็บข้อมูล DNS record | db.it-sisat-dns.com |
| **Record** | บันทึกข้อมูลใน Zone File | A, CNAME, MX, NS |
| **A Record** | แปลงชื่อ → IPv4 | www → 172.16.10.1 |
| **CNAME** | ชื่อเล่น/alias ชี้ไปชื่ออื่น | wp → www.it-sisat-dns.com |
| **NS Record** | บอกว่า DNS Server อยู่ที่ไหน | @ → ns1.it-sisat-dns.com |
| **SOA Record** | ข้อมูลหลักของ Zone | ผู้ดูแล, เวลา refresh |
| **PTR Record** | แปลง IP → ชื่อ (Reverse DNS) | 172.16.10.1 → www... |
| **Forward Zone** | แปลงชื่อ → IP (ปกติ) | www.it-sisat-dns.com → IP |
| **Reverse Zone** | แปลง IP → ชื่อ (ย้อนกลับ) | IP → www.it-sisat-dns.com |
| **Forwarder** | DNS ที่ BIND จะถามต่อถ้าไม่รู้จัก | 8.8.8.8 (Google DNS) |
| **Recursive** | DNS ที่ค้นหาคำตอบให้จนได้ | DNS ของ ISP |
| **TTL** | เวลาที่ client cache คำตอบ (วินาที) | 3600 = 1 ชั่วโมง |
| **BIND** | ซอฟต์แวร์ DNS ชื่อดังที่สุด | Berkeley Internet Name Domain |
| **named** | ชื่อ process ของ BIND | มาจาก "name daemon" |
| **Serial** | หมายเลขเวอร์ชัน Zone File | ต้องเพิ่มทุกครั้งที่แก้ไข |

---

## 2. ทำไมถึงเลือก BIND9?

**BIND9** (Berkeley Internet Name Domain version 9) คือ DNS Server ที่ใช้งานมากที่สุดในโลก

### เปรียบเทียบตัวเลือก

| ตัวเลือก | ข้อดี | ข้อเสีย | เหมาะกับ |
|---------|-------|---------|---------|
| **BIND9** ✅ | มาตรฐานอุตสาหกรรม, ฟีเจอร์ครบ, เอกสารมาก | config ซับซ้อนกว่า | ทุกกรณี, เรียนรู้ได้ดี |
| dnsmasq | เบา, config ง่าย | ฟีเจอร์น้อย | Home, Raspberry Pi |
| PowerDNS | GUI ดี, API | ติดตั้งซับซ้อน | องค์กรใหญ่ |
| Unbound | เน้น security, เร็ว | ไม่รองรับ Authoritative | Resolver เท่านั้น |

**เลือก BIND9 เพราะ:** เป็นมาตรฐาน, เรียนรู้แล้วใช้ได้ทุกที่, มี package พร้อมใน Ubuntu, รองรับทั้ง authoritative และ recursive DNS

---

## 3. โครงสร้างระบบและ Network ในคู่มือนี้

### แผนผังระบบทั้งหมด

```
                    [อินเทอร์เน็ต]
                          │
                     [Router/Modem]
                     192.168.1.1 (DHCP)
                          │
            ┌─────────────┴────────────────┐
            │                              │
    [Ubuntu 24.04 LTS]                     │
    ┌──────────────────┐                   │
    │  NIC #1 (enp2s0) │──────────────────►│ (รับ IP จาก DHCP)
    │  192.168.1.x/24  │   ออกอินเทอร์เน็ต  │
    │  (DHCP Auto)     │                   │
    ├──────────────────┤                   │
    │  NIC #2 (enp3s0) │                   │
    │  172.16.10.1/24  │                   │
    │  (Static IP)     │                   │
    │  DNS Server      │                   │
    │  Web Server      │                   │
    └────────┬─────────┘                   │
             │                             │
         [Switch]                          │
             │                             │
    ┌────────┴───────────────┐             │
    │                        │             │
[Windows 10 #1]      [Windows 10 #2]       │
172.16.10.10/24      172.16.10.11/24       │
DNS: 172.16.10.1     DNS: 172.16.10.1      │
```

### ข้อมูล IP ที่ใช้ในคู่มือนี้

| อุปกรณ์ | Interface | IP Address | หน้าที่ |
|---------|-----------|-----------|---------|
| Ubuntu NIC #1 | enp2s0 | DHCP (192.168.1.x) | ออกอินเทอร์เน็ต |
| Ubuntu NIC #2 | enp3s0 | **172.16.10.1/24** | DNS + Web Server ให้ LAN |
| Switch | — | — | เชื่อม Ubuntu กับ Windows |
| Windows 10 Client | — | 172.16.10.10–20/24 | ใช้บริการ |

### โดเมนและ Subdomain ที่จะตั้งค่า

| FQDN | IP | บริการ |
|------|----|-------|
| `it-sisat-dns.com` | 172.16.10.1 | Root Domain |
| `www.it-sisat-dns.com` | 172.16.10.1 | Apache Web (LAMP) |
| `wp.it-sisat-dns.com` | 172.16.10.1 | WordPress |
| `bot.it-sisat-dns.com` | 172.16.10.1 | Bot Service |
| `ftp.it-sisat-dns.com` | 172.16.10.1 | FTP Server |
| `db.it-sisat-dns.com` | 172.16.10.1 | phpMyAdmin |
| `ns1.it-sisat-dns.com` | 172.16.10.1 | Name Server |

---

## 4. ตั้งค่า Network การ์ด LAN ทั้งสองใบ

### 4.1 ตรวจสอบชื่อ Network Interface

```bash
ip link show
```

ผลที่ได้จะประมาณ:
```
1: lo: <LOOPBACK> ...          ← loopback (ไม่ใช้)
2: enp2s0: <BROADCAST,...>     ← NIC #1 (ต่อ Router/Internet)
3: enp3s0: <BROADCAST,...>     ← NIC #2 (ต่อ Switch/LAN)
```

> **หมายเหตุ:** ชื่อ interface อาจต่างกัน เช่น `eth0`, `eth1`, `ens33`, `ens34` ขึ้นกับเครื่อง
> ให้ใช้คำสั่ง `ip link show` เพื่อดูชื่อที่แท้จริงในเครื่องของคุณ

### 4.2 ตรวจสอบ IP ปัจจุบัน

```bash
ip addr show
```

ดูว่า NIC ใดได้ IP จาก DHCP อยู่แล้ว (NIC #1) และ NIC ใดยังไม่มี IP (NIC #2)

### 4.3 ตั้งค่า NIC #2 เป็น Static IP ด้วย NetworkManager

Ubuntu 24.04 ใช้ **NetworkManager** และเครื่องมือ `nmcli` สำหรับตั้งค่าเครือข่าย

```bash
# ดูรายชื่อ connection ทั้งหมด
nmcli connection show
```

ผลตัวอย่าง:
```
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  ethernet  enp2s0
Wired connection 2  yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy  ethernet  enp3s0
```

### 4.4 ตั้งค่า NIC #1 ให้รับ IP จาก DHCP (ตรวจสอบ)

```bash
# ตรวจสอบว่า NIC #1 ใช้ DHCP อยู่หรือไม่
nmcli connection show "Wired connection 1" | grep ipv4.method
```

ถ้าผล = `auto` = ใช้ DHCP อยู่แล้ว ✅

ถ้ายังไม่ได้ตั้ง:
```bash
nmcli connection modify "Wired connection 1" \
  ipv4.method auto
nmcli connection up "Wired connection 1"
```

### 4.5 ตั้งค่า NIC #2 เป็น Static IP (172.16.10.1)

```bash
# ตั้ง Static IP ให้ NIC #2
nmcli connection modify "Wired connection 2" \
  ipv4.addresses "172.16.10.1/24" \
  ipv4.method manual \
  ipv4.gateway "" \
  ipv4.dns ""

# เปิดใช้งาน connection
nmcli connection up "Wired connection 2"
```

> **ทำไมไม่ใส่ gateway และ dns สำหรับ NIC #2?**
> เพราะ NIC #2 ทำหน้าที่ให้บริการ LAN เท่านั้น การออกอินเทอร์เน็ตใช้ NIC #1
> การมี default gateway สองเส้นจะทำให้เกิดปัญหา routing

### 4.6 ตรวจสอบการตั้งค่า

```bash
ip addr show enp3s0
```

ผลที่ควรได้:
```
3: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 172.16.10.1/24 brd 172.16.10.255 scope global noprefixroute enp3s0
```

```bash
# ทดสอบ ping ระหว่าง NIC
ping -c 3 172.16.10.1
```

### 4.7 ตั้งค่า Netplan (วิธีสำรอง — ถ้า nmcli ไม่ได้ผล)

บางกรณีอาจต้องแก้ผ่าน netplan โดยตรง:

```bash
ls /etc/netplan/
# จะเห็นไฟล์ เช่น 01-network-manager-all.yaml หรือ 00-installer-config.yaml
sudo nano /etc/netplan/01-network-manager-all.yaml
```

```yaml
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp2s0:
      dhcp4: true          # NIC #1: รับ IP อัตโนมัติ
    enp3s0:
      dhcp4: false         # NIC #2: Static IP
      addresses:
        - 172.16.10.1/24
      nameservers:
        addresses: [127.0.0.1]   # ใช้ตัวเองเป็น DNS
```

```bash
sudo netplan apply
```

---

## 5. ติดตั้ง BIND9

### 5.1 อัปเดตระบบ

```bash
sudo apt update && sudo apt upgrade -y
```

### 5.2 ติดตั้ง BIND9 และเครื่องมือ

```bash
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```

**อธิบายแต่ละ package:**

| Package | หน้าที่ |
|---------|---------|
| `bind9` | ตัว DNS Server หลัก (named daemon) |
| `bind9utils` | เครื่องมือ เช่น `named-checkconf`, `named-checkzone` |
| `bind9-doc` | เอกสารประกอบ |
| `dnsutils` | เครื่องมือทดสอบ เช่น `dig`, `nslookup` |

### 5.3 ตรวจสอบการติดตั้ง

```bash
named -version
```

ผลที่ได้จะประมาณ:
```
BIND 9.18.x (Extended Support Version) ...
```

### 5.4 ตรวจสอบสถานะ

```bash
sudo systemctl status bind9
```

ควรเห็น `active (running)` ✅

### 5.5 ตั้งให้เริ่มอัตโนมัติ

```bash
sudo systemctl enable bind9
```

---

## 6. ทำความเข้าใจโครงสร้างไฟล์ BIND9

```
/etc/bind/                          ← โฟลเดอร์ config หลัก
├── named.conf                      ← ไฟล์ config หลัก (include ไฟล์อื่น)
├── named.conf.options              ← ตั้งค่าทั่วไป (forwarders, listen, etc.)
├── named.conf.local                ← ประกาศ Zone ของเรา
├── named.conf.default-zones        ← Zone มาตรฐาน (localhost, etc.)
├── db.local                        ← Zone file ของ localhost
├── db.127                          ← Reverse zone ของ 127.x.x.x
└── zones/                          ← โฟลเดอร์เก็บ zone file ของเรา (สร้างเอง)
    ├── db.it-sisat-dns.com         ← Forward zone file
    └── db.172.16.10                ← Reverse zone file

/var/log/named/                     ← Log ของ BIND9
/var/cache/bind/                    ← Cache ของ BIND9
```

### ความสัมพันธ์ระหว่างไฟล์

```
named.conf
├── include "named.conf.options"    ← ตั้งค่า server
├── include "named.conf.local"      ← บอกว่ามี zone อะไรบ้าง
└── include "named.conf.default-zones"

named.conf.local
└── zone "it-sisat-dns.com" {
        file "/etc/bind/zones/db.it-sisat-dns.com";  ← ชี้ไปไฟล์นี้
    }
```

---

## 7. ตั้งค่าไฟล์หลัก named.conf.options

ไฟล์นี้กำหนดพฤติกรรมหลักของ BIND9

### 7.1 สำรองไฟล์เดิมก่อน

```bash
sudo cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
```

### 7.2 แก้ไข named.conf.options

```bash
sudo nano /etc/bind/named.conf.options
```

แทนที่เนื้อหาทั้งหมดด้วย:

```nginx
// ============================================================
// named.conf.options
// BIND9 สำหรับ it-sisat-dns.com
// Ubuntu 24.04 LTS
// ============================================================

// กำหนด ACL (Access Control List) สำหรับ LAN ของเรา
// ACL คือการจัดกลุ่ม IP เพื่อใช้ในกฎต่าง ๆ
acl "trusted-lan" {
    172.16.10.0/24;    // วง LAN ที่ต่อ Switch
    127.0.0.1;         // ตัวเอง (loopback)
    localhost;         // localhost
};

options {
    // โฟลเดอร์สำหรับเก็บ cache
    directory "/var/cache/bind";

    // ============================================================
    // กำหนด Interface ที่ BIND9 รับฟัง
    // ============================================================
    // รับฟังเฉพาะ NIC #2 (172.16.10.1) และ loopback
    // ไม่รับฟัง NIC #1 (ออกอินเทอร์เน็ต) เพื่อความปลอดภัย
    listen-on {
        127.0.0.1;
        172.16.10.1;
    };

    // ปิด IPv6 (ถ้าไม่ใช้)
    listen-on-v6 { none; };

    // ============================================================
    // Forwarders — DNS ที่ BIND จะถามต่อเมื่อไม่รู้จัก domain
    // ============================================================
    // ถ้า user ถาม google.com ซึ่งเราไม่มีข้อมูล
    // BIND จะส่งคำถามต่อให้ DNS เหล่านี้
    forwarders {
        8.8.8.8;        // Google DNS (Primary)
        8.8.4.4;        // Google DNS (Secondary)
        1.1.1.1;        // Cloudflare DNS (Backup)
    };

    // ใช้ forwarders แทนการค้นหาเอง (เร็วกว่าสำหรับ internal DNS)
    forward only;

    // ============================================================
    // ความปลอดภัย — ใครสามารถถาม DNS นี้ได้บ้าง
    // ============================================================
    // allow-query: ใครถามได้บ้าง
    allow-query { trusted-lan; };

    // allow-recursion: ใครขอให้ค้นหาแบบ recursive ได้
    allow-recursion { trusted-lan; };

    // ป้องกันไม่ให้คนนอกถาม DNS เรา
    allow-query-cache { trusted-lan; };

    // ปิด zone transfer (ป้องกันการคัดลอก zone ไปที่อื่น)
    allow-transfer { none; };

    // ============================================================
    // การตั้งค่าเพิ่มเติม
    // ============================================================
    // ซ่อนเวอร์ชัน BIND (ความปลอดภัย)
    version "Not disclosed";

    // เปิด DNSSEC validation (ตรวจสอบความถูกต้องของ DNS)
    dnssec-validation auto;

    // ปิด Response Rate Limiting สำหรับ LAN (ไม่จำเป็น)
    // rate-limit { responses-per-second 10; };

    // ขนาด cache สูงสุด (MB)
    max-cache-size 128m;

    // เวลา cache สูงสุด (วินาที) = 1 ชั่วโมง
    max-cache-ttl 3600;

    // Recursive timeout (วินาที)
    recursive-clients 1000;
};
```

### 7.3 ตรวจสอบ syntax

```bash
sudo named-checkconf
```

ถ้าไม่มีข้อความ error = ถูกต้อง ✅

---

## 8. ตั้งค่า Zone (named.conf.local)

ไฟล์นี้ประกาศว่าเรามีสิทธิ์ดูแล zone อะไรบ้าง

### 8.1 สำรองไฟล์เดิม

```bash
sudo cp /etc/bind/named.conf.local /etc/bind/named.conf.local.backup
```

### 8.2 แก้ไข named.conf.local

```bash
sudo nano /etc/bind/named.conf.local
```

```nginx
// ============================================================
// named.conf.local
// Zone declarations สำหรับ it-sisat-dns.com
// ============================================================

// ============================================================
// FORWARD ZONE — แปลงชื่อ → IP
// ============================================================
zone "it-sisat-dns.com" {
    // type master = เราเป็นเจ้าของ zone นี้ (primary DNS)
    type master;

    // ไฟล์ที่เก็บข้อมูล DNS record
    file "/etc/bind/zones/db.it-sisat-dns.com";

    // ใครอนุญาตให้ query zone นี้ได้
    allow-query { trusted-lan; };

    // ปิด zone transfer (ไม่มี slave DNS)
    allow-transfer { none; };

    // ปิด dynamic update
    allow-update { none; };
};

// ============================================================
// REVERSE ZONE — แปลง IP → ชื่อ (PTR Record)
// ชื่อ reverse zone คือเลข network ย้อนกลับ + .in-addr.arpa
// 172.16.10.x → 10.16.172.in-addr.arpa
// ============================================================
zone "10.16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.172.16.10";
    allow-query { trusted-lan; };
    allow-transfer { none; };
    allow-update { none; };
};
```

> **ทำไม reverse zone ชื่อ `10.16.172.in-addr.arpa`?**
> เพราะ IP `172.16.10.x` เมื่อย้อนกลับ = `10.16.172` แล้วต่อด้วย `.in-addr.arpa`
> เหมือนอ่านเลข IP จากขวาไปซ้าย นี่คือมาตรฐาน DNS สากล

---

## 9. สร้าง Zone File (Forward Zone)

Zone File คือ "สมุดโทรศัพท์" จริง ๆ ที่เก็บข้อมูลว่าชื่ออะไรเท่ากับ IP อะไร

### 9.1 สร้างโฟลเดอร์สำหรับ zone files

```bash
sudo mkdir -p /etc/bind/zones
```

### 9.2 สร้าง Forward Zone File

```bash
sudo nano /etc/bind/zones/db.it-sisat-dns.com
```

```dns
; ============================================================
; Forward Zone File สำหรับ it-sisat-dns.com
; อัปเดตล่าสุด: 2026-05-26
; ============================================================
; หมายเหตุ: บรรทัดที่เริ่มด้วย ; คือ comment
; FQDN (Fully Qualified Domain Name) ต้องลงท้ายด้วย จุด (.)
; ถ้าไม่ใส่ จุด → จะเติม zone name ต่อท้ายอัตโนมัติ
; ============================================================

; ============================================================
; SOA Record (Start of Authority)
; บอกข้อมูลพื้นฐานของ zone นี้
; ============================================================
$TTL    3600    ; Default TTL = 3600 วินาที (1 ชั่วโมง)
                ; TTL คือเวลาที่ client จะ cache คำตอบ

@   IN  SOA     ns1.it-sisat-dns.com.  admin.it-sisat-dns.com. (
; @         = ชื่อ zone (it-sisat-dns.com)
; IN        = Internet class (ใช้ทุกครั้ง)
; SOA       = Start of Authority record
; ns1...    = Name Server หลัก (Primary DNS)
; admin...  = อีเมลผู้ดูแล (@ แทนด้วย .)
;             admin@it-sisat-dns.com → admin.it-sisat-dns.com.

                2026052601 ; Serial — หมายเลขเวอร์ชัน (YYYYMMDDXX)
                           ; ต้องเพิ่มทุกครั้งที่แก้ไข zone file!
                           ; format: ปีเดือนวัน + ลำดับ (01, 02, ...)
                3600       ; Refresh — slave DNS ตรวจสอบ update ทุกกี่วินาที
                900        ; Retry — ถ้า refresh ล้มเหลว ลองใหม่ทุกกี่วินาที
                604800     ; Expire — ถ้าไม่ได้ update นานเท่านี้ slave หยุดตอบ
                300        ; Negative Cache TTL — cache ผลลัพธ์ "ไม่พบ" กี่วินาที
                )

; ============================================================
; NS Record (Name Server)
; บอกว่า DNS Server ของ zone นี้คือใคร
; ============================================================
@           IN  NS      ns1.it-sisat-dns.com.
; @ คือ it-sisat-dns.com เอง
; บอกว่า: "DNS Server ของ it-sisat-dns.com คือ ns1.it-sisat-dns.com"

; ============================================================
; A Records (Address Records)
; แปลงชื่อ → IPv4 Address
; ============================================================

; Name Server เอง
ns1         IN  A       172.16.10.1
; ns1.it-sisat-dns.com → 172.16.10.1

; Root domain (it-sisat-dns.com)
@           IN  A       172.16.10.1
; it-sisat-dns.com → 172.16.10.1

; www — เว็บไซต์หลัก (Apache LAMP)
www         IN  A       172.16.10.1
; www.it-sisat-dns.com → 172.16.10.1

; WordPress
wp          IN  A       172.16.10.1
; wp.it-sisat-dns.com → 172.16.10.1

; Bot Service
bot         IN  A       172.16.10.1
; bot.it-sisat-dns.com → 172.16.10.1

; FTP Server
ftp         IN  A       172.16.10.1
; ftp.it-sisat-dns.com → 172.16.10.1

; phpMyAdmin / Database Admin
db          IN  A       172.16.10.1
; db.it-sisat-dns.com → 172.16.10.1

; Mail Server (ถ้ามีในอนาคต)
mail        IN  A       172.16.10.1

; API Endpoint (ถ้ามีในอนาคต)
api         IN  A       172.16.10.1

; ============================================================
; CNAME Records (Canonical Name / Alias)
; ชี้ชื่อหนึ่งไปยังชื่ออื่น (ไม่ชี้ตรง IP)
; ใช้เมื่อต้องการหลายชื่อสำหรับ server เดียวกัน
; ข้อดี: แก้ IP ที่ A record เดียว ทุก CNAME อัปเดตอัตโนมัติ
; ============================================================

; ตัวอย่าง CNAME ชี้ไป www
; shop        IN  CNAME   www.it-sisat-dns.com.
; dev         IN  CNAME   www.it-sisat-dns.com.

; ============================================================
; MX Record (Mail Exchanger)
; บอกว่า Email ของ domain นี้ส่งไปที่ server ไหน
; หมายเลขหน้า = priority (น้อยกว่า = ใช้ก่อน)
; ============================================================
@           IN  MX  10  mail.it-sisat-dns.com.
; เมื่อส่ง email ถึง @it-sisat-dns.com จะไปที่ mail.it-sisat-dns.com

; ============================================================
; TXT Records
; เก็บข้อความทั่วไป (ใช้สำหรับ SPF, DKIM, verification)
; ============================================================
@           IN  TXT     "v=spf1 mx -all"
; SPF record — ป้องกัน email spoofing
```

### 9.3 ตรวจสอบ Zone File

```bash
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com
```

ผลที่ถูกต้อง:
```
zone it-sisat-dns.com/IN: loaded serial 2026052601
OK
```

---

## 10. สร้าง Reverse Zone File

Reverse Zone แปลง IP กลับเป็นชื่อ — ใช้สำหรับ verify ว่า IP นั้นตรงกับชื่ออะไร

### 10.1 สร้าง Reverse Zone File

```bash
sudo nano /etc/bind/zones/db.172.16.10
```

```dns
; ============================================================
; Reverse Zone File สำหรับ 172.16.10.0/24
; แปลง IP → ชื่อ (PTR Records)
; ============================================================

$TTL    3600
@   IN  SOA     ns1.it-sisat-dns.com.  admin.it-sisat-dns.com. (
                2026052601  ; Serial (ต้องตรงกับ Forward Zone)
                3600        ; Refresh
                900         ; Retry
                604800      ; Expire
                300         ; Negative Cache TTL
                )

; NS Record
@           IN  NS      ns1.it-sisat-dns.com.

; ============================================================
; PTR Records (Pointer Records)
; รูปแบบ: [เลข octet สุดท้าย]  IN  PTR  [FQDN ลงท้าย .]
; 172.16.10.1 → เขียนแค่ "1" (ส่วนที่เหลือมาจากชื่อ zone)
; ============================================================

1           IN  PTR     ns1.it-sisat-dns.com.
; 172.16.10.1 → ns1.it-sisat-dns.com

1           IN  PTR     www.it-sisat-dns.com.
; 172.16.10.1 → www.it-sisat-dns.com

; ถ้า client แต่ละเครื่องมี IP ต่างกัน ใส่ได้เช่น:
; 10          IN  PTR     client1.it-sisat-dns.com.
; 11          IN  PTR     client2.it-sisat-dns.com.
```

### 10.2 ตรวจสอบ Reverse Zone File

```bash
sudo named-checkzone 10.16.172.in-addr.arpa /etc/bind/zones/db.172.16.10
```

ผลที่ถูกต้อง:
```
zone 10.16.172.in-addr.arpa/IN: loaded serial 2026052601
OK
```

---

## 11. เพิ่ม Subdomain ต่าง ๆ

### 11.1 วิธีเพิ่ม Subdomain ใหม่

เปิดไฟล์ `/etc/bind/zones/db.it-sisat-dns.com` แล้วเพิ่ม A Record หรือ CNAME:

```bash
sudo nano /etc/bind/zones/db.it-sisat-dns.com
```

#### ตัวอย่าง: เพิ่ม subdomain `app.it-sisat-dns.com` ชี้ไป IP อื่น

```dns
app         IN  A       172.16.10.2
```

#### ตัวอย่าง: เพิ่ม CNAME (alias) ชี้ไปอีกชื่อ

```dns
portal      IN  CNAME   www.it-sisat-dns.com.
shop        IN  CNAME   www.it-sisat-dns.com.
```

### 11.2 ต้องอัปเดต Serial ทุกครั้งที่แก้ไข!

```bash
sudo nano /etc/bind/zones/db.it-sisat-dns.com
```

เปลี่ยน Serial จาก `2026052601` เป็น `2026052602` (เพิ่มทีละ 1):

```dns
                2026052602 ; Serial — เปลี่ยนจาก 01 เป็น 02
```

> **ทำไมต้องอัปเดต Serial?**
> BIND และ client ใช้ Serial เป็น "เลขเวอร์ชัน" ถ้า Serial ไม่เปลี่ยน
> DNS Server อาจยังใช้ข้อมูลเดิมจาก cache แทนที่จะโหลดใหม่

### 11.3 Reload หลังจากแก้ไข Zone File

```bash
# ตรวจสอบ syntax ก่อน
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com

# Reload ถ้าไม่มี error
sudo systemctl reload bind9
```

### 11.4 ตารางสรุป Subdomain ทั้งหมด

```dns
; รายการ A Records ทั้งหมดในระบบนี้
ns1         IN  A       172.16.10.1   ; DNS Server
@           IN  A       172.16.10.1   ; it-sisat-dns.com
www         IN  A       172.16.10.1   ; Web หลัก (LAMP)
wp          IN  A       172.16.10.1   ; WordPress
bot         IN  A       172.16.10.1   ; Bot Service
ftp         IN  A       172.16.10.1   ; FTP Server
db          IN  A       172.16.10.1   ; phpMyAdmin
mail        IN  A       172.16.10.1   ; Mail Server
api         IN  A       172.16.10.1   ; API
```

---

## 12. ตรวจสอบและเริ่มใช้งาน BIND9

### 12.1 ตรวจสอบ config ทั้งหมด

```bash
# ตรวจสอบ named.conf ทั้งหมด
sudo named-checkconf

# ตรวจสอบ Forward Zone
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com

# ตรวจสอบ Reverse Zone
sudo named-checkzone 10.16.172.in-addr.arpa /etc/bind/zones/db.172.16.10
```

ทั้งหมดต้องได้ผล `OK` ก่อนดำเนินการต่อ

### 12.2 แก้ไข Permission ไฟล์

```bash
# กำหนดสิทธิ์ไฟล์ zone
sudo chown -R bind:bind /etc/bind/zones/
sudo chmod -R 644 /etc/bind/zones/
sudo chmod 755 /etc/bind/zones/
```

### 12.3 รีสตาร์ท BIND9

```bash
sudo systemctl restart bind9
sudo systemctl status bind9
```

ควรเห็น `active (running)` ✅

### 12.4 ทดสอบ DNS จากตัวเอง (Ubuntu)

```bash
# ทดสอบ Forward Lookup (ชื่อ → IP)
dig @127.0.0.1 www.it-sisat-dns.com

# ทดสอบ Root Domain
dig @127.0.0.1 it-sisat-dns.com

# ทดสอบ Subdomain ต่าง ๆ
dig @127.0.0.1 wp.it-sisat-dns.com
dig @127.0.0.1 bot.it-sisat-dns.com
dig @127.0.0.1 ftp.it-sisat-dns.com

# ทดสอบ Reverse Lookup (IP → ชื่อ)
dig @127.0.0.1 -x 172.16.10.1

# ทดสอบ Forward ออกอินเทอร์เน็ต (ผ่าน forwarder)
dig @127.0.0.1 google.com
```

**ผลที่ถูกต้องของ `dig @127.0.0.1 www.it-sisat-dns.com`:**

```
;; ANSWER SECTION:
www.it-sisat-dns.com.   3600    IN      A       172.16.10.1
;; Query time: 1 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
```

### 12.5 ทดสอบด้วย nslookup

```bash
nslookup www.it-sisat-dns.com 127.0.0.1
```

ผลที่ถูกต้อง:
```
Server:         127.0.0.1
Address:        127.0.0.1#53

Name:   www.it-sisat-dns.com
Address: 172.16.10.1
```

### 12.6 ตั้งค่า Ubuntu ให้ใช้ตัวเองเป็น DNS

```bash
# ตั้งให้ NIC #2 ใช้ตัวเองเป็น DNS
nmcli connection modify "Wired connection 2" \
  ipv4.dns "127.0.0.1"
nmcli connection up "Wired connection 2"
```

---

## 13. ตั้งค่า Firewall สำหรับ DNS

### 13.1 เปิด Port DNS (Port 53)

```bash
# Port 53 TCP — สำหรับ DNS query ขนาดใหญ่
sudo ufw allow in on enp3s0 to any port 53 proto tcp comment 'DNS TCP from LAN'

# Port 53 UDP — สำหรับ DNS query ปกติ
sudo ufw allow in on enp3s0 to any port 53 proto udp comment 'DNS UDP from LAN'
```

> **ทำไมต้องระบุ `on enp3s0`?**
> เพื่อให้รับ DNS query เฉพาะจาก NIC #2 (LAN) เท่านั้น
> ป้องกันไม่ให้คนภายนอก (NIC #1 ที่ออกอินเทอร์เน็ต) ถาม DNS เรา

### 13.2 เปิด Port อื่น ๆ ที่จำเป็น

```bash
# Web Server (Apache)
sudo ufw allow in on enp3s0 to any port 80 proto tcp comment 'HTTP from LAN'
sudo ufw allow in on enp3s0 to any port 443 proto tcp comment 'HTTPS from LAN'

# SSH (เฉพาะจาก LAN)
sudo ufw allow in on enp3s0 to any port 22 proto tcp comment 'SSH from LAN'
```

### 13.3 ตรวจสอบ Firewall

```bash
sudo ufw status verbose
```

ผลที่ควรได้:
```
Status: active

To                         Action      From
--                         ------      ----
53/tcp on enp3s0           ALLOW IN    Anywhere
53/udp on enp3s0           ALLOW IN    Anywhere
80/tcp on enp3s0           ALLOW IN    Anywhere
443/tcp on enp3s0          ALLOW IN    Anywhere
22/tcp on enp3s0           ALLOW IN    Anywhere
```

---

## 14. ตั้งค่า Windows 10 Client ให้ใช้ DNS นี้

### 14.1 ตั้งค่า IP และ DNS บน Windows 10

1. คลิกขวาที่ Start → **Network Connections**
2. คลิก **Change adapter options**
3. คลิกขวาที่ adapter ที่ต่อกับ Switch → **Properties**
4. เลือก **Internet Protocol Version 4 (TCP/IPv4)** → **Properties**
5. ตั้งค่า:

```
○ Use the following IP address:
  IP address:     172.16.10.10      ← กำหนด IP ให้ client
  Subnet mask:    255.255.255.0     ← /24
  Default gateway: 172.16.10.1     ← Ubuntu NIC #2 เป็น gateway

○ Use the following DNS server addresses:
  Preferred DNS server:   172.16.10.1    ← DNS Server (Ubuntu)
  Alternate DNS server:   8.8.8.8        ← สำรอง (Google)
```

6. คลิก **OK** → **Close**

> **ทำไม Default Gateway คือ 172.16.10.1?**
> Ubuntu จะทำหน้าที่ NAT/routing ให้ traffic จาก client ออกอินเทอร์เน็ต
> ผ่าน NIC #1 ได้ (ต้องเปิด IP forwarding ด้วย — ดูในหัวข้อ Tips)

### 14.2 ทดสอบจาก Windows 10

เปิด **Command Prompt** (cmd):

```cmd
:: ทดสอบ ping ไปที่ Ubuntu
ping 172.16.10.1

:: ทดสอบ DNS resolution
nslookup www.it-sisat-dns.com
nslookup wp.it-sisat-dns.com
nslookup bot.it-sisat-dns.com

:: ทดสอบออกอินเทอร์เน็ตผ่าน DNS
nslookup google.com
```

ผล nslookup ที่ถูกต้อง:
```
Server:  it-sisat-dns.com
Address: 172.16.10.1

Name:    www.it-sisat-dns.com
Address: 172.16.10.1
```

### 14.3 ทดสอบจาก Browser

เปิด Chrome หรือ Edge บน Windows แล้วพิมพ์:

```
http://www.it-sisat-dns.com
http://wp.it-sisat-dns.com
http://bot.it-sisat-dns.com
http://db.it-sisat-dns.com/phpmyadmin
```

ควรเข้าเว็บได้โดยไม่ต้องพิมพ์ IP ✅

---

## 15. บูรณาการกับ LAMP Web Server

### 15.1 ตั้งค่า Apache Virtual Host สำหรับแต่ละ Subdomain

เมื่อ DNS ชี้ทุก subdomain มาที่ IP เดียว (172.16.10.1) Apache ต้องรู้ว่าชื่อไหนให้ไปโฟลเดอร์ไหน ใช้ **Virtual Host** ในการแยก

#### Virtual Host สำหรับ www.it-sisat-dns.com (เว็บหลัก)

```bash
sudo nano /etc/apache2/sites-available/www.it-sisat-dns.com.conf
```

```apache
<VirtualHost *:80>
    ServerName   www.it-sisat-dns.com
    ServerAlias  it-sisat-dns.com
    DocumentRoot /var/www/html/www

    <Directory /var/www/html/www>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/www.it-sisat-dns.com_error.log
    CustomLog ${APACHE_LOG_DIR}/www.it-sisat-dns.com_access.log combined
</VirtualHost>
```

#### Virtual Host สำหรับ wp.it-sisat-dns.com (WordPress)

```bash
sudo nano /etc/apache2/sites-available/wp.it-sisat-dns.com.conf
```

```apache
<VirtualHost *:80>
    ServerName   wp.it-sisat-dns.com
    DocumentRoot /var/www/html/wordpress

    <Directory /var/www/html/wordpress>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/wp.it-sisat-dns.com_error.log
    CustomLog ${APACHE_LOG_DIR}/wp.it-sisat-dns.com_access.log combined
</VirtualHost>
```

#### Virtual Host สำหรับ bot.it-sisat-dns.com

```bash
sudo nano /etc/apache2/sites-available/bot.it-sisat-dns.com.conf
```

```apache
<VirtualHost *:80>
    ServerName   bot.it-sisat-dns.com
    DocumentRoot /var/www/html/bot

    <Directory /var/www/html/bot>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/bot.it-sisat-dns.com_error.log
    CustomLog ${APACHE_LOG_DIR}/bot.it-sisat-dns.com_access.log combined
</VirtualHost>
```

#### Virtual Host สำหรับ ftp.it-sisat-dns.com

```bash
sudo nano /etc/apache2/sites-available/ftp.it-sisat-dns.com.conf
```

```apache
<VirtualHost *:80>
    ServerName   ftp.it-sisat-dns.com
    DocumentRoot /var/www/html/ftp-portal

    <Directory /var/www/html/ftp-portal>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### 15.2 สร้างโฟลเดอร์และเปิดใช้ Virtual Host

```bash
# สร้างโฟลเดอร์
sudo mkdir -p /var/www/html/www
sudo mkdir -p /var/www/html/bot
sudo mkdir -p /var/www/html/ftp-portal

# สร้างหน้า index ทดสอบ
echo "<h1>Welcome to www.it-sisat-dns.com</h1>" | sudo tee /var/www/html/www/index.html
echo "<h1>Bot Service - bot.it-sisat-dns.com</h1>" | sudo tee /var/www/html/bot/index.html
echo "<h1>FTP Portal - ftp.it-sisat-dns.com</h1>" | sudo tee /var/www/html/ftp-portal/index.html

# กำหนดสิทธิ์
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/

# เปิดใช้งาน Virtual Host
sudo a2ensite www.it-sisat-dns.com.conf
sudo a2ensite wp.it-sisat-dns.com.conf
sudo a2ensite bot.it-sisat-dns.com.conf
sudo a2ensite ftp.it-sisat-dns.com.conf

# เปิด mod_rewrite (สำหรับ WordPress)
sudo a2enmod rewrite

# ตรวจสอบ config
sudo apache2ctl configtest

# Reload Apache
sudo systemctl reload apache2
```

### 15.3 แผนผัง DNS → Apache Virtual Host

```
Client พิมพ์ URL
       │
       ▼
DNS ค้นหา IP
       │
       ▼ (ทุก subdomain → 172.16.10.1)
       │
       ▼
Apache รับ Request
       │ (ดูที่ Host header ว่าชื่ออะไร)
       ├─── www.it-sisat-dns.com    → /var/www/html/www/
       ├─── wp.it-sisat-dns.com     → /var/www/html/wordpress/
       ├─── bot.it-sisat-dns.com    → /var/www/html/bot/
       ├─── ftp.it-sisat-dns.com    → /var/www/html/ftp-portal/
       └─── db.it-sisat-dns.com     → phpMyAdmin
```

### 15.4 ตั้งค่า phpMyAdmin สำหรับ db.it-sisat-dns.com

```bash
sudo nano /etc/apache2/sites-available/db.it-sisat-dns.com.conf
```

```apache
<VirtualHost *:80>
    ServerName   db.it-sisat-dns.com

    # Alias ชี้ไปที่ phpmyadmin
    Alias /phpmyadmin /usr/share/phpmyadmin
    Alias / /usr/share/phpmyadmin/

    <Directory /usr/share/phpmyadmin>
        Options SymLinksIfOwnerMatch
        DirectoryIndex index.php
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog  ${APACHE_LOG_DIR}/db.it-sisat-dns.com_error.log
    CustomLog ${APACHE_LOG_DIR}/db.it-sisat-dns.com_access.log combined
</VirtualHost>
```

```bash
sudo a2ensite db.it-sisat-dns.com.conf
sudo systemctl reload apache2
```

ทดสอบจาก Windows: `http://db.it-sisat-dns.com` → เข้า phpMyAdmin ได้เลย ✅

---

## 16. การดูแลรักษาและ Log

### 16.1 ดู Log แบบ Real-time

```bash
# Log หลักของ BIND9
sudo journalctl -u named -f

# Log ของ BIND9 (ถ้าตั้ง logging ใน config)
sudo tail -f /var/log/syslog | grep named

# ดู query log (เปิดได้เพื่อ debug)
sudo tail -f /var/log/named/query.log
```

### 16.2 เปิด Query Logging (สำหรับ Debug)

```bash
sudo nano /etc/bind/named.conf.options
```

เพิ่มท้าย options block:
```nginx
logging {
    channel query_log {
        file "/var/log/named/query.log" versions 3 size 10m;
        severity dynamic;
        print-time yes;
        print-severity yes;
    };
    category queries { query_log; };
};
```

```bash
sudo mkdir -p /var/log/named
sudo chown bind:bind /var/log/named
sudo systemctl restart bind9
```

### 16.3 คำสั่ง BIND9 ที่ใช้บ่อย

```bash
# ดูสถานะ
sudo systemctl status bind9

# รีสตาร์ท (หลังแก้ config หลัก)
sudo systemctl restart bind9

# Reload (หลังแก้ zone file — เร็วกว่า restart)
sudo systemctl reload bind9

# หรือ reload เฉพาะ zone ที่แก้
sudo rndc reload it-sisat-dns.com

# ดูสถิติ
sudo rndc stats

# Flush cache (ล้าง cache ทั้งหมด)
sudo rndc flush

# Flush domain เฉพาะ
sudo rndc flushname google.com
```

### 16.4 Script อัปเดต Zone อัตโนมัติ

```bash
sudo nano /usr/local/bin/update-dns-zone.sh
```

```bash
#!/bin/bash
# Script อัปเดต Serial และ Reload Zone อัตโนมัติ

ZONE_FILE="/etc/bind/zones/db.it-sisat-dns.com"
ZONE_NAME="it-sisat-dns.com"

# อ่าน Serial ปัจจุบัน
CURRENT_SERIAL=$(grep -oP '\d{10}' "$ZONE_FILE" | head -1)
DATE_TODAY=$(date +%Y%m%d)

# สร้าง Serial ใหม่
CURRENT_DATE_PART="${CURRENT_SERIAL:0:8}"
CURRENT_SEQ="${CURRENT_SERIAL:8:2}"

if [ "$CURRENT_DATE_PART" == "$DATE_TODAY" ]; then
    NEW_SEQ=$(printf "%02d" $((10#$CURRENT_SEQ + 1)))
else
    NEW_SEQ="01"
fi

NEW_SERIAL="${DATE_TODAY}${NEW_SEQ}"

# แทนที่ Serial
sed -i "s/$CURRENT_SERIAL/$NEW_SERIAL/" "$ZONE_FILE"
echo "Serial อัปเดต: $CURRENT_SERIAL → $NEW_SERIAL"

# ตรวจสอบ Zone
if sudo named-checkzone "$ZONE_NAME" "$ZONE_FILE" > /dev/null 2>&1; then
    sudo rndc reload "$ZONE_NAME"
    echo "Zone '$ZONE_NAME' reload สำเร็จ ✅"
else
    echo "❌ Zone file มีข้อผิดพลาด กรุณาตรวจสอบ"
    # คืน Serial เดิม
    sed -i "s/$NEW_SERIAL/$CURRENT_SERIAL/" "$ZONE_FILE"
fi
```

```bash
sudo chmod +x /usr/local/bin/update-dns-zone.sh
```

---

## 17. ทริปและเทคนิคสำหรับมือใหม่

### 🔧 ทริปที่ 1: เปิด IP Forwarding (ให้ Client ออกอินเทอร์เน็ตผ่าน Ubuntu)

ถ้าต้องการให้ Windows 10 ออกอินเทอร์เน็ตได้โดยใช้ Ubuntu เป็น gateway:

```bash
# เปิด IP Forwarding ชั่วคราว
sudo sysctl -w net.ipv4.ip_forward=1

# เปิดถาวร (ยังคงอยู่หลัง reboot)
sudo nano /etc/sysctl.conf
```

ค้นหาและเปิดบรรทัด:
```ini
net.ipv4.ip_forward=1
```

```bash
sudo sysctl -p

# ตั้งค่า NAT (Network Address Translation)
# NIC #1 (enp2s0) คือ interface ที่ออกอินเทอร์เน็ต
sudo iptables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE
sudo iptables -A FORWARD -i enp3s0 -o enp2s0 -j ACCEPT
sudo iptables -A FORWARD -i enp2s0 -o enp3s0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# บันทึกถาวร
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

### 🔧 ทริปที่ 2: ทดสอบ DNS ด้วย dig อย่างละเอียด

```bash
# ดูข้อมูลละเอียดทั้งหมด
dig @172.16.10.1 www.it-sisat-dns.com ANY

# ดูเฉพาะ answer (ไม่แสดง header)
dig @172.16.10.1 www.it-sisat-dns.com +short

# วัดเวลาตอบสนอง
dig @172.16.10.1 www.it-sisat-dns.com | grep "Query time"

# ทดสอบทุก record type
dig @172.16.10.1 it-sisat-dns.com A
dig @172.16.10.1 it-sisat-dns.com NS
dig @172.16.10.1 it-sisat-dns.com SOA
dig @172.16.10.1 it-sisat-dns.com MX
dig @172.16.10.1 it-sisat-dns.com TXT

# Reverse lookup
dig @172.16.10.1 -x 172.16.10.1
```

### 🔧 ทริปที่ 3: เพิ่ม subdomain ใหม่อย่างถูกต้อง (Checklist)

ทำตามลำดับนี้ทุกครั้ง:

```bash
# 1. เพิ่ม A record ใน zone file
sudo nano /etc/bind/zones/db.it-sisat-dns.com
# เพิ่ม: newsite   IN  A  172.16.10.1

# 2. อัปเดต Serial (เพิ่มตัวเลขท้ายขึ้น 1)
# เปลี่ยน 2026052601 → 2026052602

# 3. ตรวจสอบ syntax
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com

# 4. Reload BIND9
sudo rndc reload it-sisat-dns.com

# 5. สร้าง Apache Virtual Host
sudo nano /etc/apache2/sites-available/newsite.it-sisat-dns.com.conf
sudo a2ensite newsite.it-sisat-dns.com.conf
sudo apache2ctl configtest
sudo systemctl reload apache2

# 6. ทดสอบ
dig @172.16.10.1 newsite.it-sisat-dns.com
```

### 🔧 ทริปที่ 4: Wildcard DNS Record

ให้ทุก subdomain ที่ไม่ได้ตั้งค่าไว้ชี้ไป IP เดียวกันอัตโนมัติ:

```bash
sudo nano /etc/bind/zones/db.it-sisat-dns.com
```

เพิ่มบรรทัด:
```dns
*           IN  A       172.16.10.1
; *.it-sisat-dns.com → 172.16.10.1 (ทุก subdomain ที่ไม่ได้ตั้งค่า)
```

> **ระวัง:** Wildcard จะ match ทุกชื่อที่ไม่มี record เฉพาะ
> ดีสำหรับ dev/test แต่ควรระวังใน production

### 🔧 ทริปที่ 5: ตั้งค่า Split-Brain DNS (ใช้ชื่อเดียวกันในและนอก LAN)

ถ้าต้องการให้ชื่อ domain เดียวกัน resolve ต่างกันในและนอก LAN:

```bash
# สร้าง view ใน named.conf.local
sudo nano /etc/bind/named.conf.local
```

```nginx
// Internal View — สำหรับ LAN
view "internal" {
    match-clients { 172.16.10.0/24; 127.0.0.1; };

    zone "it-sisat-dns.com" {
        type master;
        file "/etc/bind/zones/db.it-sisat-dns.com";  // Internal IP
    };
};

// External View — สำหรับคนนอก (ถ้ามี public IP)
view "external" {
    match-clients { any; };

    zone "it-sisat-dns.com" {
        type master;
        file "/etc/bind/zones/db.it-sisat-dns.com.external";  // Public IP
    };
};
```

### 🔧 ทริปที่ 6: ตรวจสอบ BIND9 กินทรัพยากรเท่าไหร่

```bash
# ดูการใช้ RAM และ CPU
sudo ps aux | grep named

# ดูสถิติ BIND9
sudo rndc stats
cat /var/cache/bind/named_stats.txt
```

### 🔧 ทริปที่ 7: สำรอง Zone Files อัตโนมัติ

```bash
sudo nano /usr/local/bin/backup-dns.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/root/dns-backup/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"
cp -r /etc/bind/ "$BACKUP_DIR/"
echo "DNS backup saved to $BACKUP_DIR"
```

```bash
sudo chmod +x /usr/local/bin/backup-dns.sh

# ตั้ง cron backup ทุกวัน 2 AM
(sudo crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/backup-dns.sh") | sudo crontab -
```

---

## 18. การแก้ปัญหาที่พบบ่อย

### ❌ BIND9 ไม่เริ่มทำงาน

```bash
# ดู error log
sudo journalctl -u named -n 50

# ตรวจสอบ syntax ทั้งหมด
sudo named-checkconf
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com
```

**สาเหตุบ่อย:**
- ลืม จุด (.) ท้าย FQDN ใน zone file
- Serial ไม่ถูก format
- Syntax ผิดใน named.conf

### ❌ dig ไม่ตอบสนอง (connection refused)

```bash
# ตรวจสอบว่า BIND รับฟัง port 53 หรือไม่
sudo ss -tlnup | grep :53

# ตรวจสอบ listen-on ใน named.conf.options
grep -A5 "listen-on" /etc/bind/named.conf.options

# ตรวจสอบ firewall
sudo ufw status | grep 53
```

### ❌ DNS resolve ได้แต่ browser เข้าไม่ได้

```bash
# ตรวจสอบ Apache ทำงานอยู่หรือไม่
sudo systemctl status apache2

# ตรวจสอบ Virtual Host ถูกต้องหรือไม่
sudo apache2ctl -S | grep it-sisat-dns

# ตรวจสอบ firewall สำหรับ port 80
sudo ufw status | grep 80
```

### ❌ Windows 10 ถาม DNS ไม่ได้

```bash
# จาก Windows — flush DNS cache ก่อน
ipconfig /flushdns

# ทดสอบ ping ไปที่ Ubuntu NIC #2 ก่อน
ping 172.16.10.1

# ถ้า ping ไม่ได้ = ปัญหา network (สาย, switch, IP)
# ถ้า ping ได้แต่ DNS ไม่ทำงาน = ปัญหา BIND หรือ firewall
```

### ❌ "SERVFAIL" ใน dig output

```bash
# มักเกิดจาก DNSSEC validation ล้มเหลว
# แก้โดยปิด dnssec สำหรับ internal zone
sudo nano /etc/bind/named.conf.options
```

เปลี่ยน:
```nginx
dnssec-validation auto;
```
เป็น:
```nginx
dnssec-validation no;
```

```bash
sudo systemctl restart bind9
```

### ❌ Serial ไม่เพิ่มขึ้นทำให้ cache เก่า

```bash
# ตรวจสอบ Serial ปัจจุบัน
dig @127.0.0.1 it-sisat-dns.com SOA

# ถ้า Serial ไม่เปลี่ยน ให้ flush cache BIND9
sudo rndc flush

# Reload zone
sudo rndc reload it-sisat-dns.com
```

### ❌ Windows ยังใช้ DNS เก่าอยู่

```cmd
:: Flush DNS cache Windows 10
ipconfig /flushdns

:: ดู DNS ที่ใช้อยู่
ipconfig /all | findstr "DNS"

:: ทดสอบ DNS
nslookup www.it-sisat-dns.com 172.16.10.1
```

---

## 19. ตารางสรุปคำศัพท์สำคัญ

| คำศัพท์ | ภาษาไทย | ความหมาย |
|---------|---------|---------|
| **DNS** | ระบบชื่อโดเมน | แปลงชื่อ → IP |
| **BIND9** | ซอฟต์แวร์ DNS | Berkeley Internet Name Domain v9 |
| **named** | กระบวนการ DNS | ชื่อ process ของ BIND (name daemon) |
| **Zone** | โซน | พื้นที่ domain ที่ DNS รับผิดชอบ |
| **Zone File** | ไฟล์โซน | ไฟล์เก็บข้อมูล DNS record |
| **A Record** | Address Record | ชื่อ → IPv4 |
| **AAAA Record** | Address Record | ชื่อ → IPv6 |
| **CNAME** | Canonical Name | ชื่อเล่น/alias ชี้ไปชื่ออื่น |
| **NS Record** | Name Server | บอกว่า DNS Server คือใคร |
| **SOA Record** | Start of Authority | ข้อมูลหลักของ zone |
| **PTR Record** | Pointer Record | IP → ชื่อ (Reverse DNS) |
| **MX Record** | Mail Exchanger | บอก email server ของ domain |
| **TXT Record** | Text Record | ข้อความทั่วไป, SPF, DKIM |
| **TTL** | Time to Live | เวลา cache คำตอบ (วินาที) |
| **Serial** | หมายเลขเวอร์ชัน | ต้องเพิ่มทุกครั้งที่แก้ zone |
| **Forwarder** | ผู้ส่งต่อ | DNS ที่ถามต่อเมื่อไม่รู้จัก |
| **Recursive** | แบบเรียกซ้ำ | ค้นหาจนได้คำตอบ |
| **FQDN** | ชื่อเต็ม | Fully Qualified Domain Name (ลงท้าย .) |
| **ACL** | รายการควบคุมการเข้าถึง | กำหนดว่าใคร query ได้บ้าง |
| **Forward Zone** | โซนปกติ | ชื่อ → IP |
| **Reverse Zone** | โซนย้อนกลับ | IP → ชื่อ |
| **in-addr.arpa** | โดเมน Reverse DNS | มาตรฐาน reverse lookup |
| **rndc** | Remote Name Daemon Control | tool ควบคุม BIND9 |
| **dig** | Domain Information Groper | tool ทดสอบ DNS |
| **nslookup** | Name Server Lookup | tool ทดสอบ DNS (Windows/Linux) |
| **NIC** | การ์ดเครือข่าย | Network Interface Card |
| **Static IP** | IP คงที่ | IP ที่ตั้งเองไม่เปลี่ยน |
| **Virtual Host** | โฮสต์เสมือน | Apache แยก site ด้วยชื่อ domain |
| **Split-Brain DNS** | DNS สองหน้า | resolve ต่างกันในและนอก LAN |
| **Wildcard** | ตัวจับทุกชื่อ | * ใน DNS จับ subdomain ทั้งหมด |

---

## สรุปขั้นตอนทั้งหมดแบบย่อ

```bash
# === 1. ตั้งค่า Network ===
# NIC #1 (enp2s0) = DHCP (ออกอินเทอร์เน็ต)
# NIC #2 (enp3s0) = Static 172.16.10.1/24 (ให้บริการ LAN)
nmcli connection modify "Wired connection 2" \
  ipv4.addresses "172.16.10.1/24" ipv4.method manual
nmcli connection up "Wired connection 2"

# === 2. ติดตั้ง BIND9 ===
sudo apt install bind9 bind9utils dnsutils -y

# === 3. ตั้งค่า named.conf.options ===
sudo nano /etc/bind/named.conf.options
# (ใส่ ACL, listen-on, forwarders, allow-query)

# === 4. ตั้งค่า named.conf.local ===
sudo nano /etc/bind/named.conf.local
# (ประกาศ zone it-sisat-dns.com และ reverse zone)

# === 5. สร้าง Zone Files ===
sudo mkdir -p /etc/bind/zones
sudo nano /etc/bind/zones/db.it-sisat-dns.com
sudo nano /etc/bind/zones/db.172.16.10

# === 6. แก้ Permission ===
sudo chown -R bind:bind /etc/bind/zones/

# === 7. ตรวจสอบและ Start ===
sudo named-checkconf
sudo named-checkzone it-sisat-dns.com /etc/bind/zones/db.it-sisat-dns.com
sudo systemctl restart bind9
sudo systemctl enable bind9

# === 8. Firewall ===
sudo ufw allow in on enp3s0 to any port 53 proto tcp
sudo ufw allow in on enp3s0 to any port 53 proto udp

# === 9. ทดสอบ ===
dig @127.0.0.1 www.it-sisat-dns.com
dig @127.0.0.1 wp.it-sisat-dns.com
dig @127.0.0.1 -x 172.16.10.1

# === 10. ตั้งค่า Apache Virtual Host ===
sudo a2ensite www.it-sisat-dns.com.conf
sudo a2ensite wp.it-sisat-dns.com.conf
sudo systemctl reload apache2
```

---

## แผนภาพสรุประบบทั้งหมด

```
[อินเทอร์เน็ต] ←→ [Router] ←→ [NIC #1: enp2s0 / DHCP]
                                         │
                              [Ubuntu 24.04 LTS]
                              ┌────────────────────────┐
                              │ BIND9 (Port 53)        │
                              │ Apache LAMP (Port 80)  │
                              │ MySQL (Port 3306)      │
                              │ vsftpd (Port 21)       │
                              └────────┬───────────────┘
                                       │
                              [NIC #2: enp3s0]
                              172.16.10.1/24
                                       │
                                  [Switch]
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                   │
             [Win10 #1]          [Win10 #2]         [Win10 #N]
             172.16.10.10         172.16.10.11       172.16.10.x
             DNS: 172.16.10.1    DNS: 172.16.10.1

Client พิมพ์: www.it-sisat-dns.com
              → DNS resolve: 172.16.10.1
              → Apache Virtual Host: /var/www/html/www/
              → หน้าเว็บแสดงผล ✅
```

---

*คู่มือนี้ใช้ BIND9 บน Ubuntu 24.04 LTS*
*Domain: it-sisat-dns.com (สมมุติ — ใช้ได้เฉพาะ LAN)*
*Network: LAN 172.16.10.0/24 ผ่าน NIC #2 ต่อกับ Switch*
*บูรณาการกับ LAMP Stack: Apache + MySQL + PHP + WordPress*
*อัปเดต: 26 พฤษภาคม 2569*
