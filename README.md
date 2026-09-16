# TEK1314-2026-Kel08-KelasA — Proyek PBL Keamanan Siber

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer)
**Kelompok:** 8 — Kelas A
**Fase Design (Pertemuan ke-4):** Perancangan Arsitektur & Skema IP

---

## Deskripsi Skenario

Proyek ini merancang lab jaringan berisi tiga node utama dalam segmen IP unik **192.168.8.0/24**.

| Node | Peran | OS | IP |
|---|---|---|---|
| Attacker | Red Team (penyerang) | Kali Linux | 192.168.8.100 |
| Target (Korban) | Server yang dieksploitasi | Ubuntu Server CLI | 192.168.8.5 |
| Monitoring | Blue Team (pemantau) | Security Onion | 192.168.8.200 |

Target menjalankan layanan yang memiliki potensi celah demonstrasi:
- Apache2 + PHP (port 80) — SQL Injection / XSS
- OpenSSH (port 22) — brute-force
- MySQL (port 3306) — brute-force kredensial

---

## Struktur Repository

```
├── docs/
│   ├── Laptop Cadangan/Pertemuan 2/     ← bukti lab sebelumnya
│   ├── Laptop Utama/Pertemuan2/         ← bukti lab sebelumnya
│   └── design/                          ← DESIGN PHASE (Minggu ke-3)
│       ├── topology.png                 ← Gambar topologi jaringan
│       ├── topology.drawio              ← File Draw.io (editable)
│       ├── ip_plan.md                   ← Tabel IP & port
│       └── README.md                    ← Deskripsi detail skenario
├── Panduan PBL Pertemuan4 TEK1314 2026-2027.pdf
└── README.md                            ← Halaman utama (Anda di sini)
```

---

## Dokumen Design Phase

| File | Deskripsi |
|---|---|
| [docs/design/topology.png](docs/design/topology.png) | Gambar desain topologi jaringan (Blue Team) |
| [docs/design/ip_plan.md](docs/design/ip_plan.md) | Tabel Hostname, IP Address, dan OS |
| [docs/design/README.md](docs/design/README.md) | Deskripsi skenario & alasan pemilihan OS |

---

## Anggota Tim

| Nama | Peran |
|---|---|
| *(isi sesuai pembagian peran)* | Lead |
| *(isi sesuai pembagian peran)* | Red Team |
| *(isi sesuai pembagian peran)* | Blue Team |

---

## Referensi

- Panduan PBL Pertemuan 4 — Perancangan Arsitektur & Skema IP (PDF terlampir di root repo)