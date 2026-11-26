# Purple Team Loop Log — Kelompok 8 (Kelas A)

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer SV IPB)
**Kelompok:** 8 — Kelas A
**Tanggal Eksekusi:** 26 November 2026
**Acuan:** Panduan PBL Minggu ke-9 (Purple Team Loop Tahap 1–5)

## Lingkungan Simulasi

| Komponen | Keterangan |
| :--- | :--- |
| Segmen Jaringan | Host-Only VirtualBox `192.168.8.0/24`, gateway `192.168.8.1` |
| Attacker Node (Red Team) | `192.168.8.100` — Kali Linux |
| Target Node (Replika PiRoTech) | `192.168.8.5` — Ubuntu Server CLI |
| Monitoring Node (Blue/Purple Team) | `192.168.8.200` — Security Onion |
| Batasan | Seluruh pengujian hanya pada replika. Tidak ada serangan ke sistem IoT asli |

> **Catatan pembacaan.** Dokumen ini adalah catatan kronologis mentah per tahap. Sintesis, klasifikasi insiden, dan analisis akar masalah yang lebih dalam berada di `docs/final/incident_report.md`. Kedua dokumen saling melengkapi dan tidak boleh saling menyalin.

---

# SKENARIO 1 — Reconnaissance & Enumeration

**Kategori:** Reconnaissance
**Target:** Seluruh port pada `192.168.8.5`
**Perkakas:** Nmap 7.94

## Tahap 1 — Attack (Red Team)

Perintah yang dijalankan dari `192.168.8.100`:

```bash
nmap -A -p 22,3000,4000,9000,9099 -T4 192.168.8.5 -oN ~/evidence/01_recon_nmap.txt
```

Pemindaian difokuskan pada port yang tercantum dalam IP Plan, bukan `-p-` seluruh 65535 port. Alasan: efisiensi waktu rekaman. Pemindaian penuh membutuhkan 3–10 menit, sementara pemindaian terarah selesai jauh lebih cepat dengan cakupan uji yang tetap relevan terhadap arsitektur.

**Hasil aktual:** lima port terbuka.

| Port | State | Layanan | Versi / Keterangan |
| --- | --- | --- | --- |
| 22/tcp | open | ssh | OpenSSH 10.2p1 Ubuntu |
| 3000/tcp | open | http | Next.js, `X-Powered-By: Next.js` |
| 4000/tcp | open | http | Firebase Emulator Suite |
| 9000/tcp | open | cslistener | Realtime Database Emulator |
| 9099/tcp | open | unknown | Firebase Auth Emulator |

Informasi tambahan yang berhasil dikumpulkan: OS fingerprint, MAC address VirtualBox (`08:00:27:C8:2A:73`), banner versi layanan, dan banner HTTP aplikasi.

Terdapat peringatan `mass_dns: Unable to open /etc/resolv.conf` pada sisi Kali yang menyebabkan reverse DNS dinonaktifkan. Ini masalah konfigurasi node penyerang, tidak memengaruhi hasil pemindaian port.

## Tahap 2 — Detection (Blue/Purple Team)

Security Onion mendeteksi aktivitas pemindaian melalui tiga signature:

| Signature | SID | Jumlah Event | Klasifikasi |
| --- | --- | --- | --- |
| ET SCAN NMAP OS Detection Probe | 2018489 | 24 | Attempted Recon |
| ET SCAN NMAP SIP Version Detect | 2018317 | 3 | Attempted Recon |
| ET SCAN Potential SSH Scan | 2001219 | 6 | Attempted Recon |

Pada salah satu take, signature yang tampil adalah `ET SCAN Potential SSH Scan Outbound` dengan SID `2003066`. Perbedaan ini wajar karena rule set Suricata memilih signature berdasarkan karakteristik trafik yang sedikit berbeda antar take, dan seluruhnya tetap masuk kategori `attempted-recon`.

**Status:** Terdeteksi.

**Keterbatasan dokumentasi:** sebagian tangkapan layar dashboard SOC belum tersimpan ke repositori. Event terverifikasi secara visual saat simulasi, namun bukti gambar baru tersedia untuk sebagian sesi.

## Tahap 3 — Analysis

**Klasifikasi:** Reconnaissance / Attempted Information Gathering
**Dampak fungsional:** Rendah — tidak ada layanan terganggu, tidak ada perubahan data.
**Dampak informasi:** Sedang — daftar port, versi layanan, dan jenis OS terbaca.
**Tingkat keparahan:** Rendah sampai Sedang.

**Akar masalah:** eksposur attack surface melebihi kebutuhan fungsional, ditambah tidak adanya pembatasan laju koneksi dari sumber tunggal. Reconnaissance tidak dapat dicegah sepenuhnya; yang dapat dikendalikan adalah seberapa banyak informasi yang diberikan sebagai balasan.

**Sikap terhadap deteksi:** deteksi berhasil namun tidak menghentikan apa pun. Pemindai tetap memperoleh seluruh informasi yang dibutuhkan sebelum Blue Team bereaksi. Ini menunjukkan karakteristik IDS sebagai alat observasi, bukan alat pencegahan.

## Tahap 4 — Mitigasi (Containment, Eradication & Recovery)

| Tindakan | Lapisan | Efek | Bukti |
| --- | --- | --- | --- |
| UFW aktif, default deny incoming | Jaringan | Hanya port yang dibutuhkan dapat dijangkau | `B2_ufw_status.png` |
| Whitelist port `22, 3000, 4000, 8080, 9000` | Jaringan | Permukaan serangan dipersempit | `B2_ufw_status.png` |
| Audit port dengan `ss -tulpn` | Sistem | Memastikan tidak ada layanan tak terduga | `B3_port_audit.png` |
| Pembaruan paket OS | Sistem | Menutup kerentanan yang sudah diketahui | `B1_os_update.png` |

**Temuan tambahan saat mitigasi:** audit port mengungkap bahwa port `9099` dan `9199` berjalan namun tidak terdaftar dalam whitelist UFW. Keduanya ditambahkan. Tindakan ini adalah pelengkapan inventaris aset, bukan penghapusan hardening, dan dicatat sebagai *incomplete asset inventory*.

## Tahap 5 — Verifikasi

Perintah yang dijalankan:

```bash
nmap -sT -p 22,3000,9099 192.168.8.5 -oN ~/evidence/01b_recon_verify.txt
```

**Hasil:** port 22, 3000, dan 9099 tetap terbuka. Pemindaian tanpa flag `-A` tidak memicu deteksi versi, sehingga version disclosure sebagian tertahan.

**Kesimpulan:** Reconnaissance **tetap dapat dilakukan**. Ini hasil yang diharapkan, bukan kegagalan. Yang berhasil dikurangi adalah kedalaman informasi yang diperoleh, bukan kemampuan memindai itu sendiri.

Dokumentasi: `docs/purple-team-log.md` bagian Verifikasi, bukti `01b_recon_verify.txt`.

---

# SKENARIO 2 — Exploitation / SSH Brute-Force

**Kategori:** Exploitation / Initial Access
**Target:** `192.168.8.5:22` (OpenSSH)
**Perkakas:** Hydra 9.7, wordlist rockyou

## Tahap 1 — Attack (Red Team)

**Persiapan wordlist:**

```bash
head -n 200 /usr/share/wordlists/rockyou.txt > ~/wl.txt
wc -l ~/wl.txt
```

Wordlist dipotong menjadi 200 baris. `rockyou.txt` penuh berukuran 134 MB dan akan memakan waktu terlalu lama untuk demonstrasi. Jumlah dan kecepatan percobaan sudah cukup untuk memicu kontrol pertahanan.

**Prasyarat — penyimpanan host key.** Satu koneksi SSH manual dilakukan terlebih dahulu agar host key tersimpan. Tanpa langkah ini, Hydra gagal pada tahap verifikasi host key dan hasilnya tidak mencerminkan pengujian autentikasi yang sebenarnya.

```bash
ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 kelompok8@192.168.8.5 exit
```

**Perintah serangan:**

```bash
hydra -l kelompok8 -P ~/wl.txt -t 4 ssh://192.168.8.5 -V 2>&1 | tee ~/evidence/02_hydra.txt
```

**Penyimpangan dari attack plan:** dokumen awal menyebut user `ubuntu`, sedangkan eksekusi memakai user `kelompok8`. Ini koreksi terhadap ketidaksesuaian antara rencana dan konfigurasi aktual target, karena user `ubuntu` tidak ada pada sistem replika. Penyimpangan dicatat untuk transparansi.

**Hasil aktual:** Hydra membuka empat koneksi paralel dan memulai percobaan password. Pada take yang direkam, percobaan dihentikan manual pada percobaan ke-12 karena koneksi telah diputus target. Pada take lain, percobaan berlanjut lebih jauh sebelum terputus. Tidak ada password yang berhasil ditemukan; Hydra tidak pernah mencetak baris keberhasilan `[22][ssh] host: ... login: ... password: ...`.

**Catatan:** pada satu take, Hydra menampilkan `[WARNING] Restorefile from a previous session found`. Ini menandakan Hydra menyimpan state dari sesi sebelumnya. Untuk rekaman final, file restore sebaiknya dihapus terlebih dahulu agar percobaan terlihat bersih.

## Tahap 2 — Detection (Blue/Purple Team)

**Deteksi host — fail2ban:**

```bash
sudo fail2ban-client status sshd
```

Hasil:

| Field | Nilai |
| --- | --- |
| Currently failed | 1 |
| Total failed | **18** |
| Currently banned | **1** |
| Banned IP list | **192.168.8.100** |

**Bukti di level kernel — nftables:**

```bash
sudo nft list ruleset | grep -A5 f2b
```

```text
table inet f2b-table
  set addr-set-sshd { 192.168.8.100 }
  chain f2b-chain
    tcp dport 22 ip saddr @addr-set-sshd reject with icmp port-unreachable
```

Ini membuktikan blokir tidak hanya dicatat dalam log, tetapi benar-benar diterapkan sebagai aturan firewall di level kernel.

**Deteksi jaringan — Security Onion:**

Total 63 event dengan 7 signature berbeda. Dua event berkategori high (3,2%), 61 event medium (96,8%).

| Signature | SID | Event | Waktu |
| --- | --- | --- | --- |
| ET SCAN LibSSH Based Frequent SSH Connections Likely BruteForce Attack | 2006546 | 2 | 11:21:36 UTC |

**Status:** Terdeteksi oleh fail2ban dan Security Onion.

**Keterbatasan dokumentasi:** pada sebagian take, panel Security Onion belum sempat dibuka sebelum rekaman berhenti. Event tercatat di sensor, hanya tidak terdokumentasi visual pada take tersebut.

## Tahap 3 — Analysis

**Klasifikasi:** Credential Attack / Brute Force
**Dampak fungsional:** Rendah — layanan SSH tetap tersedia bagi pengguna sah, hanya sumber penyerang diblokir.
**Dampak informasi:** Rendah — tidak ada kredensial yang berhasil diperoleh.
**Tingkat keparahan:** Sedang, karena berpotensi tinggi apabila kontrol tidak ada.

**Akar masalah:** ketergantungan pada autentikasi berbasis password sebagai satu-satunya faktor pembatas, tanpa mekanisme kriptografi yang lebih kuat. Fail2ban tidak menutup celah ini; ia hanya memperlambat dan membatasi percobaan.

**Mengapa jumlah kegagalan lebih kecil dari jumlah percobaan Hydra.** Bukan bug. Setelah alamat diblokir, paket dari penyerang di-drop di level firewall dan tidak pernah mencapai layanan SSH. Karena tidak sampai, tidak ada kegagalan autentikasi yang tercatat. Ini justru bukti bahwa blokir bekerja tepat pada lapisannya.

**Sikap terhadap deteksi:** dua lapisan bekerja bersama. Security Onion melihat pola dan mencatatnya; fail2ban menghentikannya. Satu lapisan untuk visibilitas, satu lapisan untuk penegakan. Tanpa fail2ban, Security Onion akan tetap mencatat seluruh percobaan sampai selesai tanpa mengubah hasil.

## Tahap 4 — Mitigasi (Containment, Eradication & Recovery)

| Tindakan | Lapisan | Efek | Bukti |
| --- | --- | --- | --- |
| Fail2ban aktif pada jail `sshd` | Host | Blokir otomatis setelah ambang terlampaui | `A2_fail2ban.png` |
| `maxretry` 5–6, `findtime` 600, `bantime` 600 | Host | Lima sampai enam kegagalan dalam sepuluh menit memicu blokir sepuluh menit | Konfigurasi fail2ban |
| Aturan nftables `f2b-chain` | Kernel | Paket dari IP terblokir ditolak dengan ICMP port-unreachable | `S2_T4_T5_fail2ban_nftables_sshd_config.jpg` |
| `PermitRootLogin no` | Layanan | Menutup jalur eskalasi langsung ke root | `A1_ssh_root.png` |
| `PermitEmptyPasswords no` | Layanan | Menutup autentikasi tanpa password | `A3_ssh_empty_pass.png` |
| `ignoreip` kosong | Host | Memastikan penyerang tidak dikecualikan dari kontrol | Konfigurasi fail2ban |

Konfigurasi yang terverifikasi:

```text
maxretry: 6
findtime: 600
bantime:  600
ignoreip: No IP address/network is ignored
```

**Efektivitas:** tinggi. Serangan gagal total, blokir terpasang di level kernel, dan penyerang tidak dikecualikan dari kontrol.

## Tahap 5 — Verifikasi

**Prosedur:** blokir dilepas manual agar siklus dapat diamati penuh dalam satu rekaman, kemudian serangan dijalankan ulang dengan parameter identik.

```bash
sudo fail2ban-client set sshd unbanip 192.168.8.100
sudo fail2ban-client status sshd
```

Status setelah unban: `Currently banned: 0`, daftar IP terblokir kosong.

**Hasil:** setelah serangan ulang, IP `192.168.8.100` kembali masuk daftar blokir. `Total banned` meningkat menjadi **2**.

**Kesimpulan:** mitigasi **berhasil diverifikasi**. Blokir bukan kebetulan, melainkan hasil kontrol yang bekerja secara deterministik. Prosedur unban sengaja dilakukan agar penonton menyaksikan prosesnya, bukan hanya hasil akhir.

Bukti: `S2_T5_fail2ban_banned_ip.jpg`.

---

# SKENARIO 3 — Web & API Manipulation

**Kategori:** Web / API / Protokol Aplikasi
**Target:** `192.168.8.5:3000` (Next.js & REST API) dan `192.168.8.5:4000` (Firebase Emulator UI)
**Perkakas:** Burp Suite Community Edition, browser

## Tahap 1 — Attack (Red Team)

Sesuai attack plan, fokus pengujian adalah IDOR pada API Next.js dan ketahanan Security Rules pada database Firebase.

**Langkah tambahan di luar rencana:** sebelum serangan dijalankan, dilakukan pembacaan kode sumber terhadap seluruh route handler di `src/app/api/`. Langkah ini adalah inisiatif Red Team dan justru menghasilkan temuan terpenting dalam proyek.

**Hasil pembacaan kode — tujuh route handler, tidak satu pun memverifikasi token:**

| Endpoint | Metode | Verifikasi Token | Otorisasi | Dampak |
| --- | --- | --- | --- | --- |
| `/api/users` | GET | Tidak ada | Tidak ada | Enumerasi pengguna + email + role |
| `/api/users` | POST | Tidak ada | Tidak ada | Pembuatan pengguna dengan role dari body |
| `/api/users/[uid]` | PATCH | Tidak ada | Tidak ada | Perubahan role dan claims pengguna mana pun |
| `/api/users/[uid]` | DELETE | Tidak ada | Tidak ada | Penghapusan akun mana pun |
| `/api/sessions/[logId]/data` | GET | Tidak ada | Tidak ada | Pembacaan data sesi pengguna lain |
| `/api/debug/env` | GET | Tidak ada | Tidak ada | Pembocoran konfigurasi environment |
| `/api/auth/lookup` | GET | Tidak ada | Tidak ada | User enumeration via perbedaan respons |

Perintah verifikasi:

```bash
grep -rn "verifyIdToken\|Authorization" src/app/api/
```

**Hasil: tidak ada keluaran sama sekali.**

Keseluruhan route memakai Firebase Admin SDK, yang secara desain **melewati Firebase Security Rules**. Admin SDK ditujukan untuk operasi istimewa dari sisi server tepercaya dan karenanya tidak tunduk pada aturan yang mengikat klien.

**Bukti eksploitasi:**

**Uji 1 — Enumerasi pengguna tanpa autentikasi.** Melalui Burp Repeater tanpa header Authorization:

```http
GET /api/users HTTP/1.1
Host: 192.168.8.5:3000
Accept: application/json
```

Respons `200 OK`, mengembalikan data pengguna beserta `uid`, `email`, `role`, `creationTime`, dan `lastSignInTime`. Bukti: `E1_user_enum.jpg`.

**Uji 2 — Eskalasi hak akses.**

```http
PATCH /api/users/<uid> HTTP/1.1
Host: 192.168.8.5:3000
Content-Type: application/json

{"role":"admin"}
```

Respons `200 OK` dengan body `{"message":"Profil berhasil diperbarui","claims":{"role":"admin"}}`. Pembacaan ulang melalui `GET /api/users` mengonfirmasi role berubah dari `operator` menjadi `admin`. Bukti: `E2_privesc_request.jpg`.

Catatan akurasi: request ini membawa cookie sesi `next-auth.session-token`, tetapi endpoint tidak memverifikasi maupun memanfaatkannya. Yang tidak ada adalah **header Authorization**. Perbedaan ini penting dan tidak boleh dikaburkan.

**Uji 3 — User enumeration melalui endpoint lookup.**

```http
GET /api/auth/lookup?username=admin  → 200, {"email":"admin@pirotech.local"}
GET /api/auth/lookup?username=zzz    → 404, {"error":"Username tidak ditemukan"}
```

Perbedaan kode status membentuk oracle yang memungkinkan penebakan username tanpa autentikasi. Bukti: `E3_user_enum.jpg`, `E3b_user_enum.jpg`.

**Uji 4 — Pembocoran konfigurasi environment.**

```http
GET /api/debug/env
```

Respons `200 OK` tanpa autentikasi, membocorkan status variabel environment dan hasil uji inisialisasi Admin SDK. Bukti: `E4_debug_env.jpg`.

**Uji 5 — Manipulasi database melalui Firebase Emulator UI.** Antarmuka pada `http://192.168.8.5:4000` dapat diakses sepenuhnya tanpa autentikasi. Node `batches`, `config`, `control`, `log_activity`, `notifications`, `pirotech`, dan `sensor_data` dapat dibaca dan diubah langsung dari browser. Nilai `config.overheat_limit` berhasil diubah dari `250` menjadi `9999`, yang pada sistem nyata berarti mematikan pengaman suhu alat. Bukti: `E5_firebase_ui.jpg`.

## Tahap 2 — Detection (Blue/Purple Team)

Security Onion mencatat lalu lintas HTTP ke port `3000`, namun **tidak menghasilkan alert spesifik** untuk pola manipulasi API.

Alasannya bersifat struktural. Signature IDS dirancang mengenali pola serangan yang memiliki ciri khas pada level protokol, seperti SQL injection, path traversal, atau payload XSS. Manipulasi API yang memanfaatkan endpoint sah dengan parameter sah, hanya tanpa token, tidak memiliki ciri khas semacam itu. Dari sudut pandang jaringan, request tersebut terlihat identik dengan request normal.

**Status:** Tidak terdeteksi secara spesifik.

Bukti: `D3_soc_http.png`.

## Tahap 3 — Analysis

**Klasifikasi:** Broken Access Control (OWASP A01:2021) dan Security Misconfiguration (OWASP A05:2021)
**CWE:** CWE-306 (Missing Authentication for Critical Function), CWE-639 (Authorization Bypass Through User-Controlled Key), CWE-862 (Missing Authorization), CWE-200 (Exposure of Sensitive Information)
**Dampak fungsional:** Tinggi — role pengguna dapat dinaikkan menjadi administrator, ambang batas keselamatan alat dapat diubah tanpa otorisasi.
**Dampak informasi:** Tinggi — enumerasi pengguna, email, role, konfigurasi environment, potensi kredensial.
**Tingkat keparahan:** Tinggi / Kritis.

**Akar masalah:** kesalahan penempatan lapisan kepercayaan, bukan satu bug tunggal.

Tim sudah melakukan hal benar di dua tempat. `firebase.rules.json` mewajibkan `auth != null` pada hampir seluruh node. File `.env` sudah masuk `.gitignore` sehingga kredensial tidak bocor ke repositori. Namun keduanya tidak bermakna karena seluruh route API memakai Admin SDK.

Security Rules mengikat **klien**. Admin SDK berjalan dengan kredensial istimewa dari sisi server dan melewati seluruh evaluasi rules. Konsekuensinya, selama route API tidak memeriksa identitas pemanggilnya sendiri, rules tidak akan pernah dievaluasi untuk request yang datang melalui route tersebut.

**Empat faktor yang memperburuk:**

1. `role` diterima mentah dari body request, tanpa allowlist nilai sah — bentuk mass assignment
2. Identitas diambil dari path (`[uid]`), bukan dari token — inti CWE-639
3. Pesan kesalahan dikembalikan mentah ke klien — perbedaan `200` vs `404` pada lookup memberi informasi struktural
4. Endpoint debug ikut ter-build ke lingkungan bersama, padahal hanya alat bantu pengembangan

**Sikap terhadap deteksi:** selama tiga skenario, Security Onion mendeteksi dua dari tiga. Yang gagal dideteksi adalah yang paling berbahaya, dan kegagalan itu bukan kesalahan konfigurasi sensor. IDS tidak dapat mendeteksi apa yang secara protokol terlihat sah. Ini batas fundamental pendekatan berbasis jaringan.

## Tahap 4 — Mitigasi (Containment, Eradication & Recovery)

| Kontrol | Status | Efektivitas terhadap API |
| --- | --- | --- |
| Firebase Security Rules `auth != null` | Ada di file | **Tidak berlaku** — Admin SDK melewati rules |
| `.env*` pada `.gitignore` | Ada | Sebagian — melindungi repositori, tidak melindungi endpoint |
| Verifikasi token di route API | **Tidak ada** | **Tidak ada** |
| Pengecekan kepemilikan objek | **Tidak ada** | **Tidak ada** |
| Allowlist field `role` | **Tidak ada** | **Tidak ada** |
| Penghapusan endpoint debug | **Tidak ada** | **Tidak ada** |
| Pembatasan akses Emulator UI | **Tidak ada** | **Tidak ada** |

**Status: mitigasi pada lapisan aplikasi belum diterapkan pada saat verifikasi. Celah masih terbuka.**

**Temuan tambahan pada `firebase.rules.json`.** Sebagian node justru terbuka lebar:

```json
"monitoring": { "read": "auth != null", "write": true }
"contact": { "messages": { "write": true } }
"sensor_data": { "write": true }
```

`write: true` berarti siapa pun dapat menulis tanpa autentikasi. Ini temuan baru di luar attack plan awal dan harus diperbaiki.

**Rekomendasi perbaikan, berurutan berdasarkan prioritas:**

1. Tambahkan `verifyIdToken()` pada setiap route handler, ambil identitas dari token terverifikasi, bukan dari path atau body
2. Bandingkan `decoded.uid` dengan objek yang diakses, tolak jika berbeda kecuali pemanggil berperan admin
3. Terapkan allowlist nilai `role` (`operator` atau `admin`), jangan terima nilai mentah dari body
4. Hapus `/api/debug/env` dari build produksi atau lindungi dengan pemeriksaan peran
5. Jangan kembalikan `error.message` mentah ke klien; gunakan pesan generik dan catat detail di sisi server
6. Batasi Firebase Emulator UI hanya pada `127.0.0.1`, atau matikan pada lingkungan bersama
7. Perbaiki node dengan `write: true`, khususnya `monitoring`, `contact.messages`, dan `sensor_data`
8. Tambahkan audit log otorisasi — satu-satunya lapisan yang mampu mendeteksi manipulasi API yang secara protokol sah

Bukti: `D3_no_token_verification.png`, `D3_rules_gitignore.png`.

## Tahap 5 — Verifikasi

**Prosedur:** `GET /api/users` tanpa header Authorization dikirim ulang melalui Burp Repeater.

**Hasil:** respons berupa `HTTP/1.1 408 Request Timeout`, bukan `401 Unauthorized` maupun `403 Forbidden`.

**Kesimpulan: serangan tidak berhasil digagalkan.** Endpoint tidak menolak permintaan; ia menggantung. Tidak adanya `401` atau `403` adalah bukti bahwa lapisan otorisasi memang tidak ada, bukan bukti bahwa lapisan tersebut menolak.

Kegagalan memperoleh `200` pada verifikasi ini tidak boleh ditafsirkan sebagai perbaikan. Timeout menandakan masalah pada lapisan di belakang endpoint, bukan adanya kontrol keamanan. Kondisi dilaporkan apa adanya, karena menafsirkannya sebagai keberhasilan mitigasi akan menyesatkan.

**Status akhir Skenario 3: CELAH MASIH TERBUKA.**

---

# REKAPITULASI TIGA SIKLUS

| Aspek | Skenario 1 | Skenario 2 | Skenario 3 |
| --- | --- | --- | --- |
| Lapisan serangan | Jaringan | Autentikasi | Aplikasi |
| Perkakas | Nmap | Hydra | Burp Suite, browser |
| Deteksi Security Onion | Berhasil — 3 signature | Berhasil — SID 2006546 | Tidak terdeteksi spesifik |
| Kontrol yang menghentikan | Tidak ada | fail2ban + nftables | Tidak ada |
| Mitigasi diterapkan | Sebagian | Penuh | Belum |
| Hasil verifikasi | Reconnaissance tetap jalan | Blokir terverifikasi, `Total banned: 2` | Celah masih terbuka |
| Tingkat keparahan | Rendah–Sedang | Sedang | Tinggi–Kritis |

## Ringkasan Siklus

| Skenario | Tahap 1 | Tahap 2 | Tahap 3 | Tahap 4 | Tahap 5 |
| --- | --- | --- | --- | --- | --- |
| 1 | Selesai | Selesai | Selesai | Selesai | Selesai |
| 2 | Selesai | Selesai | Selesai | Selesai | Selesai |
| 3 | Selesai | Selesai | Selesai | Selesai (analisis, implementasi belum) | Selesai — hasil: celah terbuka |

## Pelajaran Lintas Skenario

1. Deteksi dan pencegahan adalah dua hal berbeda dan keduanya dibutuhkan di lapisan yang tepat
2. Kontrol keamanan hanya berlaku pada lapisan tempat ia ditempatkan
3. Inventaris aset yang tidak lengkap menciptakan celah yang tidak terlihat
4. Membaca kode sumber lebih efektif daripada mencoba menebak
5. Verifikasi yang jujur lebih bernilai daripada klaim keberhasilan

---

# INDEKS BUKTI

## Hardening (Blue Team)

| Kode | Bukti | Berkas |
| --- | --- | --- |
| A1 | SSH root login dinonaktifkan | `docs/blueteam/evidence/A1_ssh_root.png` |
| A2 | Fail2ban aktif | `docs/blueteam/evidence/A2_fail2ban.png` |
| A3 | Empty password dinonaktifkan | `docs/blueteam/evidence/A3_ssh_empty_pass.png` |
| B1 | Pembaruan OS | `docs/blueteam/evidence/B1_os_update.png` |
| B2 | Status UFW | `docs/blueteam/evidence/B2_ufw_status.png` |
| B3 | Audit port | `docs/blueteam/evidence/B3_port_audit.png` |
| C1 | Gitignore env | `docs/blueteam/evidence/C1_gitignore.png` |
| C2 | Firebase rules | `docs/blueteam/evidence/C2_firebase_rules.png` |

## Serangan (Red Team)

| Kode | Bukti | Berkas |
| --- | --- | --- |
| E1 | Enumerasi pengguna tanpa autentikasi | `docs/redteam/evidence/E1_user_enum.jpg` |
| E2 | Eskalasi hak akses (PATCH role) | `docs/redteam/evidence/E2_privesc_request.jpg` |
| E3 | User enumeration pada lookup | `docs/redteam/evidence/E3_user_enum.jpg` |
| E3b | Perbandingan lookup 200 vs 404 | `docs/redteam/evidence/E3b_user_enum.jpg` |
| E4 | Pembocoran environment | `docs/redteam/evidence/E4_debug_env.jpg` |
| E5 | Manipulasi Firebase UI | `docs/redteam/evidence/E5_firebase_ui.jpg` |

## Deteksi (Blue/Purple Team)

| Kode | Bukti | Berkas |
| --- | --- | --- |
| S1_T2 | Deteksi Nmap di Kibana | `docs/blueteam/evidence/S1_T2_kibana_nmap_detection.jpg` |
| S1_T2 | Deteksi Nmap di Squert | `docs/blueteam/evidence/S1_T2_squert_nmap_detection.jpg` |
| S2_T2 | Deteksi SSH brute-force di Kibana | `docs/blueteam/evidence/S2_T2_kibana_ssh_bruteforce.jpg` |
| S2_T2 | Deteksi SSH brute-force di Squert | `docs/blueteam/evidence/S2_T2_squert_ssh_bruteforce.jpg` |
| S2_T4_T5 | Fail2ban, nftables, sshd config | `docs/blueteam/evidence/S2_T4_T5_fail2ban_nftables_sshd_config.jpg` |
| S2_T5 | IP terblokir setelah verifikasi ulang | `docs/blueteam/evidence/S2_T5_fail2ban_banned_ip.jpg` |
| D3 | Lalu lintas HTTP di SOC | `docs/blueteam/evidence/D3_soc_http.png` |
| D3 | Tidak ada verifikasi token | `docs/blueteam/evidence/D3_no_token_verification.png` |
| D3 | Rules dan gitignore | `docs/blueteam/evidence/D3_rules_gitignore.png` |

---

**Dokumen terkait:** `docs/final/incident_report.pdf` (sintesis dan analisis), `docs/redteam/attack_plan.md` (rencana), `docs/blueteam/hardening_checklist.md` (baseline).
