# Proyek PBL Keamanan Siber — Kelompok 8 (Kelas A)

* **Mata Kuliah:** TEK1314 — Keamanan Siber
* **Program Studi:** D4 Teknologi Rekayasa Komputer
* **Tahapan Proyek:** *Design Phase* (Minggu ke-3 / Pertemuan 4)

---

## 1. Ringkasan Skenario

Proyek ini berfokus pada pembangunan lingkungan uji (*isolated testbed*) untuk simulasi skenario serangan (*Red Team*) dan pemantauan lalu lintas jaringan (*Blue Team*). 

Seluruh node virtual berada dalam satu segmen jaringan lokal yang sama, yaitu **192.168.8.0/24** (alokasi subnet sesuai ketentuan Poin 3a Kontrak Kuliah).

### Pembagian Node dan Peran

| Node | Peran | Sistem Operasi | Alamat IP |
| :--- | :--- | :--- | :--- |
| **Attacker** | Red Team (Uji Penetrasi) | Kali Linux | `192.168.8.100` |
| **Target** | Server Korban / Layanan Rentan | Ubuntu Server CLI | `192.168.8.5` |
| **Monitoring** | Blue Team (NIDS / SIEM) | Security Onion | `192.168.8.200` |

---

## 2. Target Pengujian dan Permukaan Serangan (*Attack Surface*)

Node target menjalankan replika lingkungan *backend* berbasis IoT web service (Next.js dan Firebase) yang dikonfigurasikan dengan beberapa celah keamanan terencana:

* **Next.js Web Application (`Port 3000`)**  
  Menjalankan aplikasi web dengan potensi celah *Insecure Direct Object Reference* (IDOR), manipulasi *endpoint* REST API, serta kebocoran kredensial/*API key*.
* **Firebase Emulator Suite (`Port 4000 / 8080 / 9000`)**  
  Simulasi database lokal yang dapat diakses secara publik tanpa mekanisme autentikasi dan otorisasi yang ketat.
* **OpenSSH Service (`Port 22`)**  
  Layanan akses *remote* yang menjadi target untuk pengujian *user enumeration* dan serangan *brute-force login*.

### Alur Kerja Skenario
1. **Red Team (`192.168.8.100`)** melakukan *reconnaissance*, pemindaian port, dan eksploitasi celah keamanan pada aplikasi web serta *service* di node target.
2. **Node Target (`192.168.8.5`)** merespons permintaan lalu lintas uji dan mencatat aktivitas pada log sistem lokal.
3. **Blue Team (`192.168.8.200`)** menangkap seluruh lalu lintas paket di jaringan menggunakan Security Onion untuk mendeteksi anomali, serangan HTTP, dan upaya akses tidak sah.

---

## 3. Justifikasi Pemilihan OS Target

Kami memilih **Ubuntu Server CLI** sebagai basis node target dengan pertimbangan teknis berikut:

1. **Efisiensi Alokasi Sumber Daya:** Menghilangkan antarmuka grafis (GUI) meminimalkan penggunaan RAM dan CPU host, sesuai dengan batasan alokasi resource pada spesifikasi pengerjaan.
2. **Dukungan Lingkungan *Runtime* Modern:** Instalasi dan konfigurasi pustaka dependensi seperti Node.js, npm, serta Firebase CLI dapat dilakukan secara fleksibel dan terdokumentasi via baris perintah.
3. **Relevansi Industri:** Merefleksikan standar lingkungan server produksi yang umumnya diterapkan pada arsitektur *cloud* dan *backend* modern.

---

## 4. Struktur Berkas Desain

Dokumen dan artefak pendukung fase desain disimpan pada repositori dengan struktur sebagai berikut:

```text
docs/design/
├── topology.png   # Diagram topologi jaringan lab virtual
├── ip_plan.md     # Rincian alokasi IP, hostname, dan service tiap node
└── README.md      # Dokumentasi umum dan deskripsi skenario
```

**Tautan Cepat Dokumen:**
* [Diagram Topologi Jaringan](design/topology.png)
* [Tabel Alokasi IP (IP Plan)](design/ip_plan.md)

---

## 5. Anggota Tim & Pembagian Tugas

| Nama Lengkap | NIM | Peran Utama |
| :--- | :--- | :--- |
| **Alya Shalika** | J0404241121 | Project Lead & Dokumentasi |
| **Syah Fauza Bintang Prawira** | J0404241116 | Red Team (Attacker / Eksploitasi) |
| **Muhammad Alfi** | J0404241024 | Blue Team (Jaringan & Konfigurasi SIEM) |
| **Faqih Muhammad Faiz** | J0404241120 | Blue Team (Analisis Log & NIDS) |

