# TEK1314-2026-Kel08-KelasA — Proyek PBL Keamanan Siber

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer)
**Kelompok:** 8 — Kelas A
**Fase:** Design Phase (Pertemuan ke-4 / Minggu ke-3 Proyek PBL)

---

## Deskripsi Skenario

Proyek ini merancang sebuah **medan perang jaringan** (network lab) yang terdiri dari tiga peran utama: penyerang, target, dan pemantau. Seluruh node berada dalam satu segmen jaringan **192.168.8.0/24** (sesuai kewajiban IP unik per kelompok, mengacu Kontrak Kuliah Poin 3a).

| Node                 | Peran                     | OS                | IP            |
| -------------------- | ------------------------- | ----------------- | ------------- |
| Attacker Node        | Red Team (penyerang)      | Kali Linux        | 192.168.8.100 |
| Target Node (Korban) | Server yang dieksploitasi | Ubuntu Server CLI | 192.168.8.5   |
| Monitoring Node      | Blue Team (pemantau)      | Security Onion    | 192.168.8.200 |

### Target & Layanan

Target adalah server web yang menjalankan layanan yang sengaja memiliki celah untuk didemonstrasikan:

- **Apache2 + PHP (port 80)** → potensi SQL Injection, XSS, file upload
- **OpenSSH (port 22)** → potensi brute-force & user enumeration
- **MySQL (port 3306)** → potensi brute-force kredensial / misconfig autentikasi

Alur serangan diharapkan dari **Attacker → Target**, sementara seluruh lalu lintas dipantau oleh **Monitoring Node (Security Onion)** untuk mendeteksi aktivitas mencurigakan (port scan, login berulang, anomali HTTP).

### Alasan Pemilihan OS Target

Ubuntu Server CLI dipilih oleh tim karena:

- Ringan dan hemat resource (sesuai keterbatasan RAM per kontrak)
- Instalasi service (Apache2, MySQL, OpenSSH) mudah via CLI tanpa GUI
- Fleksibel untuk menambah layanan pada fase implementasi nanti

---

## Struktur & Dokumen Pendukung

```
docs/design/
├── topology.png   # Gambar desain jaringan (Attacker, Target, Monitoring)
├── ip_plan.md     # Tabel Hostname, IP Address, dan OS yang direncanakan
└── README.md      # Deskripsi skenario ini
```

Link deliverables:

- [Topologi](docs/design/topology.png)
- [IP Plan](docs/design/ip_plan.md)

---

## Anggota Tim & Peran

| Nama                       | NIM         | Peran                          |
| -------------------------- | ----------- | ------------------------------ |
| Alya Shalika               | J0404241121 | Lead                           |
| Syah Fauza Bintang Prawira | J0404241116 | Red Team (Attacker)            |
| Muhammad Alfi              | J0404241024 | Blue Team (Defender / Network) |
| Faqih Muhammad Faiz        | J0404241120 | Blue Team (Defender / Network) |
