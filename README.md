# GrowSense

**Smart Hydroponic NFT Monitoring System**

Sistem hidroponik Nutrient Film Technique (NFT) yang dirancang berjalan mandiri: memantau larutan nutrisi dan lingkungan, menilai kesehatan daun dengan YOLOv8, mengambil atau menyarankan tindakan korektif, dan tetap menyala saat listrik padam.

Repositori ini memuat hasil analisis dan perancangan untuk Tugas Kelompok Evaluasi Tengah Semester, mata kuliah INF.62.0025 Analisis dan Desain Perangkat Lunak.

---

## Identitas Kelompok

| | |
| --- | --- |
| Kelompok | 4 |
| Mata kuliah | INF.62.0025 Analisis dan Desain Perangkat Lunak |
| Dosen pengampu | Randi Proska Sandra, S.Pd, M.Sc |
| Program studi | Informatika, Departemen Teknik Elektronika, Fakultas Teknik, Universitas Negeri Padang |

| NIM | Nama |
| --- | --- |
| 24343073 | Nayaka Qinthar Fasya |
| 24343087 | Facri Raihan |
| 25343077 | Rafael Tirtha Ramadhan |

---

## Ringkasan

**Masalah.** Hidroponik NFT hemat lahan, tetapi menuntut larutan nutrisi yang stabil dan sirkulasi yang tidak boleh berhenti. Pemantauan manual mudah terlewat, penyakit sering baru terlihat saat sudah parah, dan pemadaman listrik menghentikan aliran nutrisi ke akar.

**Solusi.** GrowSense menyatukan tiga hal yang pada kajian awal umumnya dibahas terpisah:

1. **Pemantauan multi-sensor** untuk pH, TDS/EC, suhu air, level air, suhu udara, kelembapan, dan cahaya.
2. **Keputusan berbasis deteksi daun.** YOLOv8 12 kelas (8 penyakit atau hama, 3 defisiensi N/P/K, 1 sehat) digabung dengan data sensor. Bila penyakit terdeteksi, dosing dikunci; bila defisiensi terkonfirmasi sensor, nutrisi ditambahkan sesuai mode operasi.
3. **Ketahanan terhadap pemadaman** melalui cadangan panel surya dan baterai, dengan mode aman lokal bila koneksi terputus.

**Inspirasi.** Arah pengembangan terinspirasi vertical farming, controlled environment agriculture, dan smart agriculture di Jepang.

## Fitur Utama

- Dashboard web dan aplikasi: data terkini, grafik historis, status aktuator, status daya, hasil deteksi terbaru
- Tiga mode operasi: Manual, Setengah AI (pemilik menyetujui rekomendasi), Penuh AI
- Penguncian dosing saat penyakit atau hama terdeteksi, disertai notifikasi anjuran isolasi
- Kontrol manual pompa sirkulasi, grow light, dan dosing sekali
- Notifikasi bertingkat (Info, Peringatan, Kritis) dan penanganan sensor error
- Riwayat data dan gambar, serta persiapan dataset untuk pelatihan ulang model

## Arsitektur Singkat

```text
Sensor + Kamera
      |
  ESP32-S3 --MQTT--> Broker MQTT --> Server (Next.js, Prisma, PostgreSQL)
      |                                  |            |
   Relay: pompa, dosing, grow light      |        Dashboard web dan aplikasi
                                         |
                     Unit inferensi YOLOv8 --> Decision layer
                     (laptop, Raspberry Pi 5, atau server)

Panel surya + baterai + pengatur daya --> beban prioritas (pompa, grow light, ESP32-S3)
```

## Teknologi

| Lapisan | Teknologi |
| --- | --- |
| Perangkat | ESP32-S3, sensor pH, TDS/EC, suhu air, level air, suhu dan kelembapan udara, cahaya; relay; kamera |
| AI | YOLOv8 (Python 3.10 atau lebih baru) |
| Komunikasi | MQTT, JSON, HTTPS |
| Aplikasi | Next.js, Prisma, PostgreSQL |
| Daya | Panel surya, baterai, pengatur daya |

## Struktur Repositori

```text
GrowSense/
|-- Diagram/
|   |-- BPMN_AsIs.png
|   |-- BPMN_ToBe.png
|   |-- UserPersona.png
|   |-- UML_UseCase.png
|   |-- UML_Activity.png
|   `-- UML_Class.png
|-- Dokumen/
|   |-- SRS.md
|   |-- Use_Case_Specification.md
|   `-- RTM.md
`-- README.md
```

## Dokumen

| Dokumen | Isi |
| --- | --- |
| [SRS](Dokumen/SRS.md) | 20 kebutuhan fungsional, kebutuhan nonfungsional terukur, antarmuka, dan Lampiran A sampai C |
| [Use Case Specification](Dokumen/Use_Case_Specification.md) | Tiga use case utama dengan template Cockburn |
| [RTM](Dokumen/RTM.md) | Matriks ketertelusuran kebutuhan ke use case dan konsep domain |

## Pendekatan Pengembangan

SDLC **Iterative and Incremental** dalam empat iterasi: (1) fondasi pemantauan, (2) deteksi dan keputusan, (3) ketahanan daya, dan (4) fungsi pendukung. Kebutuhan berprioritas Must dikerjakan lebih dulu; kebutuhan Could ditunda ke tahap lanjutan.

## Status

Tahap analisis dan perancangan (Evaluasi Tengah Semester). SRS dan Use Case Specification versi 1.2.
