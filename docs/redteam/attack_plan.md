# Red Team Attack Plan - Fase 2
**Tim:** Red Team
**Target Utama:** Ubuntu Server CLI (192.168.8.5)

Berdasarkan hasil pemetaan arsitektur dan layanan yang direplikasi, Red Team merencanakan 3 skenario serangan dari 3 kategori berbeda untuk menguji efektivitas hardening Blue Team dan kemampuan deteksi Security Onion.

### Skenario 1: Reconnaissance (Pemetaan Celah & Port)
*   **Target:** `192.168.8.5` (Semua Port)
*   **Tools:** Nmap
*   **Deskripsi:** Red Team akan melakukan *aggressive port scanning* dan *OS fingerprinting* untuk memverifikasi apakah aturan firewall (UFW) yang diterapkan Blue Team benar-benar memblokir port yang tidak berkepentingan, serta mendeteksi versi layanan yang berjalan pada port 22, 3000, dan 4000.
*   **Ekspektasi Purple Team Loop:** Security Onion milik Blue Team diharapkan memunculkan *alert* terkait aktivitas *scanning* massal ini.

### Skenario 2: Exploitation / Akses (SSH Brute-Force)
*   **Target:** `192.168.8.5:22` (Layanan OpenSSH)
*   **Tools:** Hydra
*   **Deskripsi:** Menggunakan *wordlist* standar, Red Team akan melakukan serangan *brute-force* untuk menebak kredensial login SSH secara masif. Serangan ini bertujuan menguji apakah kontrol *hardening* `fail2ban` dan pembatasan otentikasi (tanpa *password* kosong) berhasil menahan intrusi.
*   **Ekspektasi Purple Team Loop:** Serangan seharusnya gagal dan IP Attacker Node (`192.168.8.100`) akan diblokir oleh target. Blue Team harus melampirkan log pemblokiran tersebut.

### Skenario 3: Web/API/Protokol IoT (Eksploitasi REST API & Firebase Emulator)
*   **Target:** `192.168.8.5:3000` (Next.js) dan `192.168.8.5:4000` (Firebase UI)
*   **Tools:** Burp Suite / OWASP ZAP
*   **Deskripsi:** Red Team akan melakukan intersepsi lalu lintas HTTP (*intercept traffic*) pada antarmuka web Next.js untuk mencari celah manipulasi parameter (IDOR). Selain itu, Red Team akan mencoba mengakses UI Firebase Emulator secara langsung untuk menguji apakah *Security Rules* baca/tulis yang diterapkan Blue Team sudah berfungsi dengan benar.
*   **Ekspektasi Purple Team Loop:** Akses ilegal ke database harus tertolak (*Permission Denied*), dan *traffic* eksploitasi HTTP yang anomali dapat terdeteksi oleh rule IDS di Security Onion.
