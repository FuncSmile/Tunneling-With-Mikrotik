MANTAP 🔥😎
Kalau sudah **BERHASIL**, berarti kamu **LAYAK** dapet tutorial **VERSI RAPAPI & TIDAK AMBIGU**.

Di bawah ini aku buat **TUTORIAL LENGKAP DARI NOL**
📌 **BISA KAMU SIMPAN / JADIKAN MODUL BELAJAR**

---

# 📘 TUTORIAL L2TP IPSEC MIKROTIK (SERVER–CLIENT)

### 🔧 Simulasi VirtualBox (LAN ↔ LAN)

---

## 🧠 TOPOLOGI

```
LAN SERVER (192.168.10.0/24)
        |
   MikroTik SERVER
        |
   L2TP + IPSEC
        |
   MikroTik CLIENT
        |
LAN CLIENT (192.168.20.0/24)
```

---

# 1️⃣ PENGATURAN VIRTUALBOX (WAJIB BENAR)

## 🔴 MikroTik SERVER

| Adapter   | Mode              | Keterangan |
| --------- | ----------------- | ---------- |
| Adapter 1 | Host-only Adapter | WAN        |
| Adapter 2 | Internal Network  | LAN SERVER |

Nama interface:

```
ether1 → WAN
ether2 → LAN
```

---

## 🔵 MikroTik CLIENT

| Adapter   | Mode              | Keterangan |
| --------- | ----------------- | ---------- |
| Adapter 1 | Host-only Adapter | WAN        |
| Adapter 2 | Internal Network  | LAN CLIENT |

---

## 📌 Host-only Network

* IP: `192.168.56.0/24`
* DHCP: ON (boleh)

---

# 2️⃣ KONFIGURASI IP DASAR

## 🔴 SERVER

```bash
/ip address
add address=192.168.56.10/24 interface=WAN
add address=192.168.10.1/24 interface=LAN
```

# 3️⃣ SETUP L2TP SERVER (SERVER)

## 3.1 Aktifkan L2TP Server

```bash
/interface l2tp-server server
set enabled=yes use-ipsec=yes ipsec-secret=L2TPSECRET
```

---

## 3.2 Buat PPP Profile

```bash
/ppp profile
add name=l2tp-profile local-address=10.10.10.1 remote-address=10.10.10.2
```

---

## 3.3 Buat User L2TP

```bash
/ppp secret
add name=mt2 password=123 service=l2tp profile=l2tp-profile
```

---

## 3.4 Firewall (WAJIB)

```bash
/ip firewall filter
add chain=input protocol=udp dst-port=500,1701,4500 action=accept
add chain=input protocol=ipsec-esp action=accept
```

---

## 🔵 CLIENT

```bash
/ip address
add address=192.168.56.11/24 interface=WAN
add address=192.168.20.1/24 interface=LAN
```

# 4️⃣ SETUP L2TP CLIENT (CLIENT)

```bash
/interface l2tp-client
add name=l2tp-to-server \
connect-to=192.168.56.10 \
user=mt2 password=123 \
use-ipsec=yes ipsec-secret=L2TPSECRET \
add-default-route=no
```

Aktifkan:

```bash
/interface l2tp-client enable l2tp-to-server
```

---

# 5️⃣ ROUTING ANTAR LAN

## 🔴 SERVER → LAN CLIENT

```bash
/ip route
add dst-address=192.168.20.0/24 gateway=10.10.10.2
```

---

## 🔵 CLIENT → LAN SERVER

```bash
/ip route
add dst-address=192.168.10.0/24 gateway=l2tp-to-server
```

---

# 6️⃣ VERIFIKASI (PENTING)

## 🔴 SERVER

```bash
/interface l2tp-server print
/ip address print
/ip route print
/ping 10.10.10.2
/ping 192.168.20.1
```

## 🔵 CLIENT

```bash
/interface l2tp-client print
/ip address print
/ip route print
/ping 10.10.10.1
/ping 192.168.10.1
```

👉 **SEMUA HARUS REPLY**

---

# 7️⃣ KENAPA INI BERHASIL?

✔ L2TP = PPP (point-to-point)
✔ IP server & client **dipasangkan benar**
✔ Routing dua arah
✔ Firewall hanya yang perlu
✔ Tidak ada NAT ganggu tunnel

---

# 🧠 CATATAN PENTING (INGAT INI)

| Item                      | Keterangan                  |
| ------------------------- | --------------------------- |
| GRE ≠ L2TP                | GRE = tunnel IP, L2TP = PPP |
| L2TP bukan bridge         | Routing wajib               |
| IP PPP harus pair         | local & remote              |
| Banyak route sama = ERROR | bersihkan                   |

---

# 🚀 NEXT LEVEL (KALAU MAU)

* Internet LAN client lewat server
* Banyak client L2TP
* Bandwidth limit PPP
* Site-to-site WireGuard
* Convert ke real ISP

Kalau mau, bilang saja:

> **“lanjut level berikutnya”** 😎
