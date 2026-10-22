# Deklarasi Tech Stack & Replikasi Target Node

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer)  
**Kelompok:** 8 - Kelas A  

Berikut adalah pemetaan teknologi dari proyek IoT asli kami ke sistem replika:

| Komponen Teknologi pada Proyek IoT Asli | Direplikasi sebagai (Target Node VM - `192.168.8.5`) |
| :--- | :--- |
| **Sistem Operasi backend** | Ubuntu Server CLI |
| **Database/penyimpanan data** | Firebase Emulator (Port 4000, 8080, 9000) |
| **Protokol komunikasi sensor** | HTTP REST API |
| **Dashboard/monitoring** | Next.js App (Port 3000) |

### Catatan Tambahan (Kesesuaian Replikasi)
* **Sistem Operasi:** Kami memilih **Ubuntu Server CLI** karena ringan, menghemat resource RAM (sesuai batasan proyek), dan merupakan standar industri untuk *deployment server backend*.
* **Database & Dashboard:** Penggunaan **Next.js** dan **Firebase Emulator** di dalam VM target benar-benar meniru tumpukan teknologi (tech stack) aplikasi IoT kami yang sesungguhnya. Hal ini memastikan bahwa kerentanan yang kami temukan (seperti *Insecure Direct Object Reference/IDOR* atau manipulasi REST API) dan teknik mitigasi yang kami terapkan di lab akan relevan 100% dengan keamanan sistem IoT asli kami di dunia nyata.
* **Layanan Ekstra:** Target Node juga menjalankan **OpenSSH (Port 22)** untuk meniru akses administrasi *remote* yang umum digunakan pada pengelolaan *server backend*.
