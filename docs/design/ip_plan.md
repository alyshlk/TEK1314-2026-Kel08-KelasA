# **IP Plan — Kelompok 8 Kelas A**

**Fase:** Design Phase (Perancangan Arsitektur & Skema IP)  
Blok IP & Subnet: 192.168.8.0/24  
Default Gateway: 192.168.8.1

# **Tabel IP Plan**

| Hostname | IP Address | OS Direncanakan | Layanan / Port Terbuka |
| :---- | :---- | :---- | :---- |
| Target Server (Korban) | 192.168.8.5 | Ubuntu Server CLI | OpenSSH (22), Next.js App (3000), Firebase Emulator (4000/8080/9000) |
| Attacker Node | 192.168.8.100 | Kali Linux | Tool serangan (Burp Suite, nmap, dll) |
| Monitoring Node | 192.168.8.200 | Security Onion | IDS/NSM (monitoring traffic) |

# **Skema Alamat IP**

| Peran | Alamat | Keterangan |
| :---- | :---- | :---- |
| Network Address | 192.168.8.0 | Awal segmen jaringan kelompok 8 |
| Default Gateway | 192.168.8.1 | Gerbang keluar jaringan (router) |
| Target Server | 192.168.8.5 | Korban / server yang akan dieksploitasi |
| Attacker Node | 192.168.8.100 | Mesin penyerang |
| Monitoring Node | 192.168.8.200 | Pemantau jaringan (Security Onion) |
| Broadcast Address | 192.168.8.255 | Akhir segmen jaringan |

# **Daftar Port (Hasil Riset Red Team)**

Port-potensi yang dibuka / dieksploitasi pada skenario **Web Server / IoT Backend**:

| Port | Service | Potensi Cacat / Serangan |
| :---- | :---- | :---- |
| 22 | OpenSSH | Brute-force SSH (hydra), user enumeration |
| 3000 | Next.js App (HTTP) | IDOR, REST API Manipulation, Web Vuln |
| 4000/8080/9000 | Firebase Emulator | Eksploitasi database lokal tanpa autentikasi |
