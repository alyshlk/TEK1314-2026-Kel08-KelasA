# Rencana Serangan Red Team (Attack Plan) - Fase 2

**Mata Kuliah:** TEK1314 - Keamanan Siber
**Kelompok:** 8 — Kelas A
---

## 1. Informasi Infrastruktur Simulasi

Berdasarkan Panduan PBL, seluruh serangan HANYA akan dilakukan di dalam jaringan *Host-Only* VirtualBox terhadap sistem replika. **Tidak ada pengujian yang dilakukan ke sistem IoT.**

*   **Attacker Node (Red Team):** `192.168.8.100` (Kali Linux)
*   **Target Node (Korban):** `192.168.8.5` (Ubuntu Server CLI - Replika IoT Backend)
*   **Monitoring Node (Blue Team & Purple Team):** `192.168.8.200` (Security Onion)

---

## 2. Rincian Skenario Serangan

Red Team merencanakan 3 (tiga) skenario serangan dari kategori yang berbeda untuk menguji postur keamanan (*hardening*) yang telah diterapkan oleh Blue Team di Target Node.

### Skenario 1: Reconnaissance & Enumeration (Pemetaan Celah)
*   **Kategori Serangan:** Reconnaissance
*   **Target Layanan:** Semua port pada `192.168.8.5`
*   **Tools Utama:** Nmap
*   **Tujuan:** Memverifikasi postur *firewall* (UFW) yang diterapkan Blue Team dan memetakan *attack surface* (port yang terbuka dan versi layanannya).
*   **Langkah Eksekusi:**
    1. Buka terminal di Kali Linux (`192.168.8.100`).
    2. Jalankan pemindaian agresif secara menyeluruh: 
       `nmap -A -p- -T4 192.168.8.5`
    3. Analisis *output* untuk memastikan apakah port 22, 3000, 4000, 8080, dan 9000 terdeteksi terbuka, dan apakah ada port lain yang bocor.
*   **Ekspektasi Purple Team Loop:**
    *   **Red Team:** Mendapatkan daftar layanan yang berjalan beserta versinya.
    *   **Blue Team & Purple Team:** Security Onion mendeteksi adanya aktivitas *mass port scanning* dan menghasilkan *alert* intrusi dari IP `192.168.8.100`.

### Skenario 2: Eksploitasi Akses (SSH Brute-Force)
*   **Kategori Serangan:** Exploitation / Akses
*   **Target Layanan:** `192.168.8.5:22` (OpenSSH)
*   **Tools Utama:** Hydra
*   **Tujuan:** Menguji ketahanan autentikasi *remote server* dan memverifikasi apakah Blue Team telah menerapkan mitigasi pembatasan *login* (seperti Fail2ban) serta menonaktifkan *password* lemah/kosong.
*   **Langkah Eksekusi:**
    1. Siapkan *wordlist* berisi *password* umum (misal: `rockyou.txt`) di Kali Linux.
    2. Jalankan perintah serangan tebak *password* secara berulang:
       `hydra -l ubuntu -P /usr/share/wordlists/rockyou.txt ssh://192.168.8.5`
    3. Pantau apakah terminal Hydra terus berjalan atau mengalami pemutusan koneksi (*connection reset/timeout*).
*   **Ekspektasi Purple Team Loop:**
    *   **Red Team:** Serangan gagal menembus masuk; setelah beberapa kali percobaan, IP penyerang diblokir oleh Target Node.
    *   **Blue Team & Purple Team:** Melampirkan bukti log pemblokiran IP oleh *Fail2ban* (atau *iptables*) dan menunjukkan *alert* autentikasi gagal masif di Security Onion.

### Skenario 3: Web & API Manipulation (Eksploitasi REST API & Firebase)
*   **Kategori Serangan:** Web/API/Protokol IoT
*   **Target Layanan:** `192.168.8.5:3000` (Next.js) & `192.168.8.5:4000` (Firebase Emulator UI)
*   **Tools Utama:** Burp Suite (Community Edition) & Web Browser Kali
*   **Tujuan:** Mencari celah kerentanan lapisan aplikasi (seperti *Insecure Direct Object Reference* / IDOR) pada API Next.js dan menguji keandalan *Security Rules* pada *database* Firebase.
*   **Langkah Eksekusi:**
    1. Buka *browser* di Kali Linux, atur *proxy* agar mengarah ke Burp Suite.
    2. Akses target HTTP via *browser*: `http://192.168.8.5:3000`.
    3. Lakukan intersepsi lalu lintas HTTP (*Intercept is On*) di Burp Suite. Ubah parameter ID pengguna/sensor pada URL API untuk melihat apakah data milik entitas lain dapat diakses (Uji IDOR).
    4. Buka *tab* baru dan akses UI Firebase secara langsung: `http://192.168.8.5:4000`. Coba lakukan modifikasi data (*Create/Update/Delete*) langsung dari UI emulator.
*   **Ekspektasi Purple Team Loop:**
    *   **Red Team:** Akses manipulasi API dan manipulasi data melalui Firebase ditolak dengan pesan error `403 Forbidden` atau `Permission Denied` jika proteksi aplikasi sudah baik.
    *   **Blue Team & Purple Team:** Mengonfirmasi bahwa *Firebase Security Rules* berhasil memblokir *query* ilegal dan melampirkan log *traffic* HTTP anomali yang ditangkap oleh Security Onion.
