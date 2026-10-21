# Hardening Checklist - Target Node Replika
**Tim:** Blue Team
**Target:** Ubuntu Server CLI (192.168.8.5)

| No  | Kategori                    | Kontrol Keamanan (Tindakan)                                                                         | Status  | Bukti (Evidence)                 |
| --- | --------------------------- | --------------------------------------------------------------------------------------------------- | ------- | -------------------------------- |
| 1   | A. Access & Authentication  | Menonaktifkan login root langsung pada SSH (`PermitRootLogin no`).                                  | Selesai | `evidence/A1_ssh_root.png`       |
| 2   | A. Access & Authentication  | Menginstal dan mengaktifkan `fail2ban` untuk memblokir brute-force SSH.                             | Selesai | `evidence/A2_fail2ban.png`       |
| 3   | A. Access & Authentication  | Menonaktifkan autentikasi password kosong pada SSH (`PermitEmptyPasswords no`).                     | Selesai | `evidence/A3_ssh_empty_pass.png` |
| 4   | B. Service & Attack Surface | Memperbarui (_update & upgrade_) paket OS Ubuntu ke versi terbaru.                                  | Selesai | `evidence/B1_os_update.png`      |
| 5   | B. Service & Attack Surface | Mengaktifkan Firewall (UFW) dan melakukan _whitelist_ port spesifik (22, 3000, 4000, 8080, 9000).   | Selesai | `evidence/B2_ufw_status.png`     |
| 6   | B. Service & Attack Surface | Melakukan audit port untuk memastikan hanya _service_ esensial yang terbuka (`ss -tulpn`).          | Selesai | `evidence/B3_port_audit.png`     |
| 7   | C. Data & Application       | Mengamankan _environment variables_ dengan memastikan file `.env*` masuk ke `.gitignore`.           | Selesai | `evidence/C1_gitignore.png`      |
| 8   | C. Data & Application       | Mengamankan `firebase.rules.json` dengan mewajibkan autentikasi (`"auth != null"`) pada akses baca. | Selesai | `evidence/C2_firebase_rules.png` |
