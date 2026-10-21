### **TEK1314-2026-Kel08-KelasA — Proyek PBL Keamanan Siber**

**Mata Kuliah:** TEK1314 \- Keamanan Siber (D4 Teknologi Rekayasa Komputer)  
**Kelompok:** 8 — Kelas A  
---

#### **Deskripsi Skenario**

Proyek ini merancang sebuah **medan perang jaringan** (network lab) yang terdiri dari tiga peran utama: penyerang, target, dan pemantau. Seluruh node berada dalam satu segmen jaringan **192.168.8.0/24** (sesuai kewajiban IP unik per kelompok, mengacu Kontrak Kuliah Poin 3a).

| Node | Peran | OS | IP |
| :---- | :---- | :---- | :---- |
| Attacker Node | Red Team (penyerang) | Kali Linux | 192.168.8.100 |
| Target Node (Korban) | Server yang dieksploitasi | Ubuntu Server CLI | 192.168.8.5 |
| Monitoring Node | Blue Team (pemantau) | Security Onion | 192.168.8.200 |

##### **Target & Layanan**

Target adalah server web yang menjalankan layanan replika dari infrastruktur IoT asli (Next.js dan Firebase) yang sengaja memiliki celah untuk didemonstrasikan:

* **Next.js App (port 3000\)** → potensi Insecure Direct Object Reference (IDOR), manipulasi REST API, dan eksploitasi API key.  
* **OpenSSH (port 22\)** → potensi brute-force & user enumeration.  
* **Firebase Emulator (port 4000/8080/9000)** → eksploitasi UI emulator dan akses database lokal tanpa autentikasi yang kuat.

Alur serangan diharapkan dari **Attacker → Target** dengan fokus pada eksploitasi HTTP REST API dan layanan lokal, sementara seluruh lalu lintas dipantau oleh **Monitoring Node (Security Onion)** untuk mendeteksi aktivitas mencurigakan (port scan, login berulang, anomali HTTP).

##### **Alasan Pemilihan OS Target**

Ubuntu Server CLI dipilih oleh tim karena:

* Ringan dan hemat resource (sesuai keterbatasan RAM per kontrak).  
* Instalasi service dan dependensi web modern (seperti Node.js untuk Next.js dan Firebase CLI) mudah dilakukan via CLI tanpa GUI.  
* Sesuai dengan lingkungan deployment backend pada umumnya (Linux environment).

---

#### **Struktur Folder GitHub & Link Deliverables**

```
docs/
├── design/
│   ├── topology.png                      # Gambar desain jaringan (Attacker, Target, Monitoring)
│   └── ip_plan.md                        # Tabel Hostname, IP Address, dan OS
├── presentation-minggu9/
│   ├── slides.pdf                        # File slide presentasi (belum tersedia)
│   └── tech_stack_declaration.md         # Tabel pemetaan teknologi asli ke replika (belum tersedia)
├── blueteam/
│   ├── hardening_checklist.md            # Checklist hardening dengan status & bukti
│   └── evidence/                         # Folder berisi screenshot/output bukti hardening (before/after)
├── redteam/
│   ├── attack_plan.md                    # Rencana 3 skenario serangan
│   └── evidence/                         # Folder berisi hasil eksekusi serangan (akan diisi Minggu 10)
└── purple-team-log.md                    # Log 5 tahap Purple Team Loop (akan diisi hingga Minggu 11)
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

