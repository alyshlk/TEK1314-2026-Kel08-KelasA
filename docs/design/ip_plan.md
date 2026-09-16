# IP Plan — Kelompok 8 Kelas A

**Mata Kuliah:** TEK1314 - Keamanan Siber
**Fase:** Design Phase (Perancangan Arsitektur & Skema IP)
**Blok IP & Subnet:** 192.168.8.0/24
**Default Gateway:** 192.168.8.1

---

## Tabel IP Plan

| Hostname               | IP Address    | OS Direncanakan   | Layanan / Port Terbuka                         |
| ---------------------- | ------------- | ----------------- | ---------------------------------------------- |
| Target Server (Korban) | 192.168.8.5   | Ubuntu Server CLI | OpenSSH (22), Apache2 + PHP (80), MySQL (3306) |
| Attacker Node          | 192.168.8.100 | Kali Linux        | Tool serangan (nmap, hydra, dll)               |
| Monitoring Node        | 192.168.8.200 | Security Onion    | IDS/NSM (monitoring traffic)                   |

---

## Skema Alamat IP

| Peran             | Alamat        | Keterangan                              |
| ----------------- | ------------- | --------------------------------------- |
| Network Address   | 192.168.8.0   | Awal segmen jaringan kelompok 8         |
| Default Gateway   | 192.168.8.1   | Gerbang keluar jaringan (router)        |
| Target Server     | 192.168.8.5   | Korban / server yang akan dieksploitasi |
| Attacker Node     | 192.168.8.100 | Mesin penyerang                         |
| Monitoring Node   | 192.168.8.200 | Pemantau jaringan (Security Onion)      |
| Broadcast Address | 192.168.8.255 | Akhir segmen jaringan                   |

---

## Daftar Port (Hasil Riset Red Team)

Port-potensi yang dibuka / dieksploitasi pada skenario **Web Server**:

| Port | Service       | Potensi Cacat / Serangan                   |
| ---- | ------------- | ------------------------------------------ |
| 22   | OpenSSH       | Brute-force SSH (hydra), user enumeration  |
| 80   | Apache2 + PHP | SQL Injection, XSS, file upload (web vuln) |
| 3306 | MySQL         | Brute-force cred, misconfig autentikasi    |

> Catatan: Berdasarkan koordinasi dari Lead, daftar ini dipakai sebagai dasar skenario serangan oleh Red Team dan untuk pemantauan oleh Blue Team / Security Onion.
