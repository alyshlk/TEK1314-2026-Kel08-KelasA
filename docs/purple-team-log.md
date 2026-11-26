# Purple Team Loop Log — Kelompok 8 (Kelas A)
- **Mata Kuliah**: TEK1314 - Keamanan Siber
- **Lingkungan**: Host-Only VirtualBox (`192.168.8.0/24`)
- **Attacker Node (Red Team)**: `192.168.8.100` (Kali Linux)
- **Target Node (Replika Backend PiRoTech)**: `192.168.8.5` (Ubuntu Server CLI)
- **Monitoring Node (Blue/Purple Team)**: `192.168.8.200` (Security Onion)

---

## Skenario 1: Reconnaissance & Enumeration (Nmap)
- **Target**: Semua port pada `192.168.8.5` (verifikasi whitelist UFW: `22, 3000, 4000, 8080, 9000`)
- **Tahap 1 — Serangan**: Menjalankan `nmap -A -p- -T4 192.168.8.5` dari `192.168.8.100`. Hasil awal: [isi port terbuka & versi layanan yang terdeteksi].
- **Tahap 2 — Deteksi**: [Terdeteksi / Tidak terdeteksi] di Security Onion. Alert/log dari `192.168.8.100`: [isi ringkasan alert/log].
- **Tahap 3 — Analisis**: [Klasifikasi insiden & akar masalah / evaluasi visibilitas deteksi].
- **Tahap 4 — Mitigasi**: [Penyesuaian UFW / aturan deteksi / layanan yang ditutup jika ada port tidak perlu].
- **Tahap 5 — Verifikasi**: Pengulangan pemindaian Nmap setelah mitigasi. Hasil: [konfirmasi port & status deteksi].

## Skenario 2: Exploitation / SSH Brute-Force (Hydra)
- **Target**: `192.168.8.5:22` (OpenSSH)
- **Tahap 1 — Serangan**: Menjalankan Hydra dengan wordlist (`rockyou.txt`) ke `ssh://192.168.8.5`. Hasil awal: [status percobaan login & respon koneksi].
- **Tahap 2 — Deteksi**: [Terdeteksi / Tidak terdeteksi] di Security Onion & Fail2ban. Log/alert: [isi log gagal login & blokir IP].
- **Tahap 3 — Analisis**: [Klasifikasi insiden & akar masalah].
- **Tahap 4 — Mitigasi**: [Konfigurasi Fail2ban / SSH hardening / aturan firewall tambahan].
- **Tahap 5 — Verifikasi**: Pengulangan serangan Hydra dari `192.168.8.100` setelah mitigasi. Hasil: [IP terblokir / koneksi gagal / terdeteksi].

## Skenario 3: Web & API Manipulation (Burp Suite & Firebase Emulator)
- **Target**: `192.168.8.5:3000` (Next.js / REST API) & `192.168.8.5:4000` (Firebase Emulator UI)
- **Tahap 1 — Serangan**: Intersepsi request API via Burp Suite (uji IDOR parameter ID pengguna/sensor) dan coba Create/Update/Delete di Firebase Emulator UI. Hasil awal: [respon server / status kode].
- **Tahap 2 — Deteksi**: [Terdeteksi / Tidak terdeteksi] di Security Onion. Log traffic HTTP anomali: [isi ringkasan log].
- **Tahap 3 — Analisis**: [Klasifikasi insiden & akar masalah].
- **Tahap 4 — Mitigasi**: [Perbaikan otorisasi/validasi API & penguatan `firebase.rules.json`].
- **Tahap 5 — Verifikasi**: Pengulangan percobaan akses IDOR & manipulasi data setelah mitigasi. Hasil: [ditolak 403 Forbidden / Permission Denied & tercatat].
