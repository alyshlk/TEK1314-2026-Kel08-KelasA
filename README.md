# TEK1314-2026-Kel08-KelasA — Proyek PBL Keamanan Siber

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer SV IPB)  
**Kelompok:** 8 - Kelas A  
**Konteks Sistem:** Pengamanan Infrastruktur Backend Replika **PiRoTech** (Alat Pirolisis Limbah Plastik Menjadi BBM Berbasis IoT)

#### **Anggota Tim & Peran**

| Nama | NIM | Peran |
| :---- | :---- | :---- |
| Alya Shalika | J0404241121 | Lead |
| Syah Fauza Bintang Prawira | J0404241116 | Red Team (Attacker) |
| Muhammad Alfi | J0404241024 | Blue Team (Defender / Network) |
| Faqih Muhammad Faiz | J0404241120 | Blue Team (Defender / Network) |
---

## Deskripsi Proyek & Arsitektur Simulasi

Proyek ini merancang sebuah lingkungan simulasi keamanan jaringan (*network lab*) untuk menguji dan membuktikan siklus pertahanan siber secara utuh pada infrastruktur *backend* replika **PiRoTech**. Seluruh pengujian dilakukan di dalam jaringan terisolasi **Host-Only VirtualBox (`192.168.8.0/24`)** menggunakan data dummy/sintetis, tanpa menyentuh sistem IoT nyata milik mitra maupun perangkat sensor fisiknya.

| Node | Peran | Sistem Operasi | Alamat IP |
| :--- | :--- | :--- | :--- |
| **Attacker Node** | Red Team (Penyerang) | Kali Linux | `192.168.8.100` |
| **Target Node** | Server Replika IoT Backend | Ubuntu Server 24.04 LTS (CLI) | `192.168.8.5` |
| **Monitoring Node** | Blue Team & Purple Team (Pemantau) | Security Onion | `192.168.8.200` |

### Target & Layanan Replika (`192.168.8.5`)
Target merupakan server *backend* replika yang menjalankan tumpukan teknologi (*tech stack*) aplikasi IoT PiRoTech beserta layanan administrasi jarak jauh:
* **OpenSSH Server (Port `22`)** → Jalur administrasi *remote* server (pengujian *hardening* SSH, *brute-force*, dan pemblokiran otomatis Fail2ban).
* **Next.js App & HTTP REST API (Port `3000`)** → Replika *web dashboard* dan jalur komunikasi API sensor (pengujian celah *Insecure Direct Object Reference* / IDOR & manipulasi REST API).
* **Firebase Local Emulator Suite (Port `4000`, `8080`, `9000`)** → Replika antarmuka emulator (`4000`), Firestore (`8080`), dan Realtime Database (`9000`) untuk menguji *Firebase Security Rules* (`auth != null`) serta proteksi modifikasi data ilegal.

### Alasan Pemilihan OS Target
Ubuntu Server CLI dipilih oleh tim karena:
* Ringan dan hemat *resource* RAM sesuai batasan lingkungan virtualisasi proyek.
* Mendukung instalasi dan konfigurasi layanan *backend* modern (Node.js, Next.js, Firebase CLI, UFW, Fail2ban) secara efisien melalui CLI tanpa beban GUI.
* Merepresentasikan standar industri untuk *deployment* server *backend* di lingkungan produksi.

---

## Rangkaian Tahapan: Blue Team, Red Team & Purple Team Loop

Pelaksanaan proyek mencakup **3 Fase** dari perancangan arsitektur, *hardening*, simulasi serangan, pemantauan SOC, hingga penyusunan *Incident Report* mengacu kerangka kerja **NIST SP 800-61r2**:

### 1. Preparation & Hardening (Blue Team)
Sebelum simulasi serangan dijalankan, Blue Team menerapkan **8 kontrol pengamanan dasar (*hardening*)** pada Target Node (`192.168.8.5`):
* **A. Access & Authentication:** Menonaktifkan login *root* langsung (`PermitRootLogin no`), menonaktifkan *password* kosong (`PermitEmptyPasswords no`), serta menginstal dan mengaktifkan **Fail2ban** untuk menahan *SSH brute-force*.
* **B. Service & Attack Surface:** Melakukan pembaruan OS (`apt update && upgrade`), mengaktifkan *firewall* **UFW** dengan *whitelist* port spesifik (`22, 3000, 4000, 8080, 9000`), serta mengaudit port terbuka (`ss -tulpn`).
* **C. Data & Application:** Memastikan file sensitif (`.env*`) masuk ke `.gitignore` dan mengamankan `firebase.rules.json` dengan mewajibkan autentikasi (`auth != null`) pada akses baca.

### 2. Skenario Pengujian Serangan (Red Team)
Red Team menjalankan **3 skenario serangan lintas kategori** dari Kali Linux (`192.168.8.100`) ke Target Node (`192.168.8.5`):
* **Skenario 1 — Reconnaissance & Enumeration (`Nmap`):** Pemindaian port agresif (`nmap -A -p- -T4 192.168.8.5`) untuk memverifikasi efektivitas *whitelist* UFW dan memetakan *attack surface*.
* **Skenario 2 — Exploitation / SSH Brute-Force (`Hydra`):** Percobaan tebak *password* SSH menggunakan *wordlist* (`rockyou.txt`) untuk menguji ketahanan autentikasi dan mekanisme pemblokiran IP oleh Fail2ban.
* **Skenario 3 — Web & API Manipulation (`Burp Suite & Browser`):** Intersepsi lalu lintas HTTP pada Next.js (`:3000`) untuk menguji celah IDOR pada parameter API serta uji modifikasi data (*Create/Update/Delete*) langsung melalui Firebase Emulator UI (`:4000`).

### 3. Siklus Kolaborasi Purple Team Loop (Tahap 1–5)
Red Team dan Blue Team berkolaborasi menjalankan evaluasi tertutup (**Purple Team Loop**) untuk setiap skenario guna membuktikan siklus pertahanan yang utuh:
1. **Tahap 1 — Eksekusi Serangan (Red Team):** Menjalankan skenario serangan terkendali dari `192.168.8.100` ke `192.168.8.5`.
2. **Tahap 2 — Deteksi & Pemantauan (Blue/Purple Team):** Memantau lalu lintas jaringan di **Security Onion** (`192.168.8.200`) untuk memverifikasi kemunculan *alert* intrusi dan log anomali.
3. **Tahap 3 — Analisis Insiden & Root Cause:** Mengklasifikasikan insiden (kategori, dampak fungsional, dampak informasi) dan menganalisis akar masalah mengacu **NIST SP 800-61r2**.
4. **Tahap 4 — Mitigasi (*Containment, Eradication & Recovery*):** Menerapkan dan memperkuat aturan pertahanan pada Target Node untuk menutup celah yang ditemukan.
5. **Tahap 5 — Verifikasi Ulang:** Red Team mengeksekusi kembali skenario serangan yang sama setelah mitigasi diterapkan untuk memastikan serangan berhasil ditolak/diblokir dan tercatat aman.

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

| Fase | Dokumen | Tautan |
| :--- | :--- | :--- |
| 1 | Topologi Jaringan | [`docs/design/topology.png`](docs/design/topology.png) |
| 1 | IP Plan & Port | [`docs/design/ip_plan.md`](docs/design/ip_plan.md) |
| 2 | Deklarasi Tech Stack | [`docs/presentation-minggu9/tech_stack_declaration.md`](docs/presentation-minggu9/tech_stack_declaration.md) |
| 2 | Hardening Checklist (Blue Team) | [`docs/blueteam/hardening_checklist.md`](docs/blueteam/hardening_checklist.md) |
| 2 | Attack Plan (Red Team) | [`docs/redteam/attack_plan.md`](docs/redteam/attack_plan.md) |
| 2 | Purple Team Loop Log | [`docs/purple-team-log.md`](docs/purple-team-log.md) |
| 3 | Incident Report Final | [`docs/final/incident_report.pdf`](docs/final/incident_report.pdf) |
| 3 | Video Demo Purple Team Loop | `docs/final/video_demo.mp4` |
| 3 | Slide Presentasi Final | `docs/presentation-final/slides.pdf` |

---

## Hasil Akhir: Rekapitulasi Tiga Siklus Purple Team Loop

Seluruh skenario telah dijalankan hingga **Tahap 5 (Verifikasi)**. Rincian lengkap per tahap ada di [`docs/purple-team-log.md`](docs/purple-team-log.md), sedangkan sintesis dan analisis akar masalah ada di [`docs/final/incident_report.pdf`](docs/final/incident_report.pdf).

| Aspek | Skenario 1 — Recon | Skenario 2 — SSH Brute-Force | Skenario 3 — Web & API |
| :--- | :--- | :--- | :--- |
| **Lapisan serangan** | Jaringan | Autentikasi | Aplikasi |
| **Perkakas** | Nmap | Hydra | Burp Suite, browser |
| **Deteksi Security Onion** | Berhasil — SID 2018489, 2018317, 2001219 | Berhasil — SID 2006546 | Tidak terdeteksi spesifik |
| **Kontrol yang menghentikan** | Tidak ada | Fail2ban + nftables | Tidak ada |
| **Mitigasi diterapkan** | Sebagian | Penuh | Belum |
| **Hasil Verifikasi** | Reconnaissance tetap dapat dilakukan | Blokir terverifikasi (`Total banned: 2`) | **Celah masih terbuka** |
| **Tingkat keparahan** | Rendah–Sedang | Sedang | Tinggi–Kritis |

### Temuan Utama

**Skenario 1 dan 2 tertahan oleh konfigurasi sistem.** UFW memperkecil attack surface, dan Fail2ban bersama nftables memblokir serangan brute-force di level kernel. Keduanya terverifikasi melalui pengulangan serangan yang sama.

**Skenario 3 mengungkap celah paling serius.** Seluruh route API pada replika tidak melakukan verifikasi token. Red Team berhasil membaca daftar pengguna beserta email dan role tanpa header `Authorization`, menaikkan hak akses dari `operator` menjadi `admin` melalui `PATCH /api/users/[uid]`, memetakan username melalui perbedaan respons `200` vs `404`, membocorkan konfigurasi environment melalui `/api/debug/env`, dan mengubah data langsung dari Firebase Emulator UI pada port `4000`.

Akar masalahnya bukan penulisan Security Rules yang buruk, melainkan **penempatan lapisan kepercayaan yang salah**. `firebase.rules.json` sudah mewajibkan `auth != null` pada hampir seluruh node, tetapi seluruh route memakai Firebase **Admin SDK** yang secara desain melewati Security Rules. Selama route API tidak memeriksa identitas pemanggilnya sendiri, rules sekuat apa pun tidak akan pernah dievaluasi.

### Pelajaran Lintas Skenario

1. **Deteksi dan pencegahan adalah dua hal berbeda**, dan keduanya dibutuhkan di lapisan yang tepat. Security Onion mendeteksi dua dari tiga skenario, tetapi tidak pernah menghentikan apa pun dengan sendirinya.
2. **Kontrol keamanan hanya berlaku pada lapisan tempat ia ditempatkan.** Rules di database tidak melindungi API yang memakai Admin SDK.
3. **Inventaris aset yang tidak lengkap menciptakan celah yang tidak terlihat.** Port `9099` dan `9199` berjalan namun tidak terdaftar dalam whitelist UFW awal — dicatat sebagai *incomplete asset inventory*.
4. **Membaca kode sumber lebih efektif daripada menebak.** Pembacaan route handler mengungkap seluruh pola kerentanan dalam hitungan menit.
5. **Verifikasi yang jujur lebih bernilai daripada klaim keberhasilan.** Verifikasi Skenario 3 menghasilkan `408 Request Timeout`, bukan `200`, dan kondisi itu dilaporkan apa adanya sebagai bukti ketiadaan lapisan otorisasi.

Efektivitas pertahanan yang sebenarnya berasal dari kombinasi tiga hal: **sensor yang melihat, kontrol yang membatasi, dan kode yang memvalidasi.** Menghilangkan salah satunya membuat pertahanan bocor tepat pada lapisan tersebut.

---

## Catatan Etika & Batasan

* Seluruh pengujian dilakukan **hanya** pada replika pada jaringan Host-Only VirtualBox `192.168.8.0/24`.
* Tidak ada pengujian, pemindaian, atau eksploitasi yang diarahkan ke sistem IoT asli milik mitra maupun perangkat sensor fisiknya.
* Seluruh data yang digunakan bersifat **dummy dan sintetis**.
* Rekomendasi hardening pada laporan ini dapat diteruskan ke sistem asli atas inisiatif kelompok bersama pembimbing proyek IoT, di luar lingkup penilaian PBL.

---

**Dosen Pengampu:** SNN
**Standar Acuan:** NIST SP 800-61r2 — Computer Security Incident Handling Guide
