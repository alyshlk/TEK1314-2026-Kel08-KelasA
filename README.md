# TEK1314-2026-Kel08-KelasA — Proyek PBL Keamanan Siber

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer SV IPB)  
**Kelompok:** 8 — Kelas A  
**Konteks Sistem:** Pengamanan Infrastruktur Backend Replika **PiRoTech** (Alat Pirolisis Limbah Plastik Menjadi BBM Berbasis IoT)

---

## Deskripsi Proyek & Arsitektur Simulasi

Proyek ini merancang sebuah lingkungan simulasi keamanan jaringan (*network lab*) untuk menguji dan membuktikan siklus pertahanan siber secara utuh pada infrastruktur *backend* replika **PiRoTech**[cite: 3, 10, 13]. Seluruh pengujian dilakukan di dalam jaringan terisolasi **Host-Only VirtualBox (`192.168.8.0/24`)** menggunakan data dummy/sintetis, tanpa menyentuh sistem IoT nyata milik mitra maupun perangkat sensor fisiknya[cite: 3, 5, 10].

| Node | Peran | Sistem Operasi | Alamat IP |
| :--- | :--- | :--- | :--- |
| **Attacker Node** | Red Team (Penyerang) | Kali Linux | `192.168.8.100` |
| **Target Node** | Server Replika IoT Backend | Ubuntu Server 24.04 LTS (CLI) | `192.168.8.5` |
| **Monitoring Node** | Blue Team & Purple Team (Pemantau) | Security Onion | `192.168.8.200` |

### Target & Layanan Replika (`192.168.8.5`)
Target merupakan server *backend* replika yang menjalankan tumpukan teknologi (*tech stack*) aplikasi IoT PiRoTech beserta layanan administrasi jarak jauh[cite: 3, 4]:
* **OpenSSH Server (Port `22`)** → Jalur administrasi *remote* server (pengujian *hardening* SSH, *brute-force*, dan pemblokiran otomatis Fail2ban)[cite: 4, 6, 7].
* **Next.js App & HTTP REST API (Port `3000`)** → Replika *web dashboard* dan jalur komunikasi API sensor (pengujian celah *Insecure Direct Object Reference* / IDOR & manipulasi REST API)[cite: 4, 7].
* **Firebase Local Emulator Suite (Port `4000`, `8080`, `9000`)** → Replika antarmuka emulator (`4000`), Firestore (`8080`), dan Realtime Database (`9000`) untuk menguji *Firebase Security Rules* (`auth != null`) serta proteksi modifikasi data ilegal[cite: 4, 6, 7].

### Alasan Pemilihan OS Target
Ubuntu Server CLI dipilih oleh tim karena[cite: 3]:
* Ringan dan hemat *resource* RAM sesuai batasan lingkungan virtualisasi proyek[cite: 3].
* Mendukung instalasi dan konfigurasi layanan *backend* modern (Node.js, Next.js, Firebase CLI, UFW, Fail2ban) secara efisien melalui CLI tanpa beban GUI.
* Merepresentasikan standar industri untuk *deployment* server *backend* di lingkungan produksi[cite: 3].

---

## Rangkaian Tahapan: Blue Team, Red Team & Purple Team Loop

Pelaksanaan proyek mencakup **3 Fase** dari perancangan arsitektur, *hardening*, simulasi serangan, pemantauan SOC, hingga penyusunan *Incident Report* mengacu kerangka kerja **NIST SP 800-61r2**[cite: 3, 10, 11, 13]:

### 1. Preparation & Hardening (Blue Team)
Sebelum simulasi serangan dijalankan, Blue Team menerapkan **8 kontrol pengamanan dasar (*hardening*)** pada Target Node (`192.168.8.5`)[cite: 5, 6]:
* **A. Access & Authentication:** Menonaktifkan login *root* langsung (`PermitRootLogin no`), menonaktifkan *password* kosong (`PermitEmptyPasswords no`), serta menginstal dan mengaktifkan **Fail2ban** untuk menahan *SSH brute-force*[cite: 6].
* **B. Service & Attack Surface:** Melakukan pembaruan OS (`apt update && upgrade`), mengaktifkan *firewall* **UFW** dengan *whitelist* port spesifik (`22, 3000, 4000, 8080, 9000`), serta mengaudit port terbuka (`ss -tulpn`)[cite: 6].
* **C. Data & Application:** Memastikan file sensitif (`.env*`) masuk ke `.gitignore` dan mengamankan `firebase.rules.json` dengan mewajibkan autentikasi (`auth != null`) pada akses baca[cite: 6].

### 2. Skenario Pengujian Serangan (Red Team)
Red Team menjalankan **3 skenario serangan lintas kategori** dari Kali Linux (`192.168.8.100`) ke Target Node (`192.168.8.5`)[cite: 5, 7]:
* **Skenario 1 — Reconnaissance & Enumeration (`Nmap`):** Pemindaian port agresif (`nmap -A -p- -T4 192.168.8.5`) untuk memverifikasi efektivitas *whitelist* UFW dan memetakan *attack surface*[cite: 7].
* **Skenario 2 — Exploitation / SSH Brute-Force (`Hydra`):** Percobaan tebak *password* SSH menggunakan *wordlist* (`rockyou.txt`) untuk menguji ketahanan autentikasi dan mekanisme pemblokiran IP oleh Fail2ban[cite: 7].
* **Skenario 3 — Web & API Manipulation (`Burp Suite & Browser`):** Intersepsi lalu lintas HTTP pada Next.js (`:3000`) untuk menguji celah IDOR pada parameter API serta uji modifikasi data (*Create/Update/Delete*) langsung melalui Firebase Emulator UI (`:4000`)[cite: 7].

### 3. Siklus Kolaborasi Purple Team Loop (Tahap 1–5)
Red Team dan Blue Team berkolaborasi menjalankan evaluasi tertutup (**Purple Team Loop**) untuk setiap skenario guna membuktikan siklus pertahanan yang utuh[cite: 11, 12, 13]:
1. **Tahap 1 — Eksekusi Serangan (Red Team):** Menjalankan skenario serangan terkendali dari `192.168.8.100` ke `192.168.8.5`[cite: 5, 7, 12].
2. **Tahap 2 — Deteksi & Pemantauan (Blue/Purple Team):** Memantau lalu lintas jaringan di **Security Onion** (`192.168.8.200`) untuk memverifikasi kemunculan *alert* intrusi dan log anomali[cite: 5, 7, 11, 12].
3. **Tahap 3 — Analisis Insiden & Root Cause:** Mengklasifikasikan insiden (kategori, dampak fungsional, dampak informasi) dan menganalisis akar masalah mengacu **NIST SP 800-61r2**[cite: 11].
4. **Tahap 4 — Mitigasi (*Containment, Eradication & Recovery*):** Menerapkan dan memperkuat aturan pertahanan pada Target Node untuk menutup celah yang ditemukan[cite: 11, 12].
5. **Tahap 5 — Verifikasi Ulang:** Red Team mengeksekusi kembali skenario serangan yang sama setelah mitigasi diterapkan untuk memastikan serangan berhasil ditolak/diblokir dan tercatat aman[cite: 11, 12].

---

## Struktur Folder GitHub & Link Deliverables (Fase 1–3)

```text
docs/
├── design/
│   ├── topology.png                      # Desain topologi jaringan (Attacker, Target, Monitoring)
│   └── ip_plan.md                        # Tabel Hostname, IP Address, layanan, dan OS
├── presentation-minggu9/
│   ├── slides.pdf                        # Slide progres Fase 1–2 (Minggu 9)
│   └── tech_stack_declaration.md         # Tabel pemetaan teknologi asli ke replika
├── blueteam/
│   ├── hardening_checklist.md            # Checklist 8 kontrol hardening dengan status & bukti
│   └── evidence/                         # Bukti screenshot/output hardening (A1–C2)
├── redteam/
│   ├── attack_plan.md                    # Rencana 3 skenario serangan lintas kategori
│   └── evidence/                         # Bukti hasil eksekusi & verifikasi serangan
├── purple-team-log.md                    # Log lengkap 5 tahap Purple Team Loop
├── final/
│   ├── incident_report.pdf               # Incident Report final (struktur NIST SP 800-61r2)
│   └── video_demo.mp4                    # Video demo siklus Purple Team Loop (5–8 menit)
└── presentation-final/
    └── slides.pdf                        # Slide presentasi final (Fase 1–3)
```

**Link Deliverables:**

* [Topologi Jaringan](http://docs/design/topology.png)  
* [IP Plan & Port](http://docs/design/ip_plan.md)  
* [Hardening Checklist (Blue Team)](http://docs/blueteam/hardening_checklist.md)  
* [Attack Plan (Red Team)](http://docs/redteam/attack_plan.md)

---

#### **Anggota Tim & Peran**

| Nama | NIM | Peran |
| :---- | :---- | :---- |
| Alya Shalika | J0404241121 | Lead |
| Syah Fauza Bintang Prawira | J0404241116 | Red Team (Attacker) |
| Muhammad Alfi | J0404241024 | Blue Team (Defender / Network) |
| Faqih Muhammad Faiz | J0404241120 | Blue Team (Defender / Network) |

