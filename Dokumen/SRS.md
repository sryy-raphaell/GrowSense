# Software Requirement Specification (SRS): GrowSense

Sistem Otonom Hidroponik NFT Berbasis YOLOv8 dan Multi-Sensor dengan Deteksi Kesehatan Daun, Dosing Nutrisi Adaptif, dan Cadangan Daya Panel Surya

| Informasi | Keterangan |
| --- | --- |
| Project name | GrowSense: Smart Hydroponic NFT Monitoring System |
| Date | 24 Oktober 2026 |
| Version | 1.3 |
| By | Kelompok 4 |
| Mata kuliah | Analisis dan Desain Perangkat Lunak (ADPL) |

## Revision history

| Version | Author | Version description | Date completed |
| --- | --- | --- | --- |
| 1.0 | Kelompok 4 | Rilis draf awal SRS GrowSense untuk peninjauan internal. | 8 Oktober 2026 |
| 1.1 | Kelompok 4 | Pembaruan kriteria penerimaan, penyesuaian nilai usulan pada Lampiran C, dan penyempurnaan alur decision layer. | 11 Oktober 2026 |
| 1.2 | Kelompok 4 | Penyeragaman satuan EC (mS/cm): konversi TDS ke EC pada FR-01, rentang valid EC pada FR-16, pembedaan sensor prototipe dan sensor target pada 3.2, penambahan butir pada Lampiran C, dan pembaruan versi acuan Use Case Specification. | 21 Oktober 2026 |
| 1.3 | Kelompok 4 | Penyelarasan pembanding intervensi manusia (bagian 1.2 dan Lampiran C) dengan BPMN AS-IS sebagai gambaran sistem hidroponik yang sudah ada. | 24 Oktober 2026 |

## Review history

| Approving party | Version approved | Signature | Date |
| --- | --- | --- | --- |
| | | | |

## Approval history

| Reviewer | Version reviewed | Signature | Date |
| --- | --- | --- | --- |
| | | | |

## Daftar isi

1. [Introduction](#1-introduction)
2. [Functional requirements](#2-functional-requirements)
3. [External interface requirements](#3-external-interface-requirements)
4. [Non-functional requirements](#4-non-functional-requirements)
5. [Definitions and acronyms](#5-definitions-and-acronyms)
6. [Referensi](#referensi)
7. [Lampiran A: Requirements Traceability Matrix](#lampiran-a-requirements-traceability-matrix)
8. [Lampiran B: Prosedur Perubahan (Change Request)](#lampiran-b-prosedur-perubahan-change-request)
9. [Lampiran C: Asumsi dan nilai usulan yang perlu divalidasi](#lampiran-c-asumsi-dan-nilai-usulan-yang-perlu-divalidasi)

---

## 1 Introduction

Dokumen ini adalah Software Requirement Specification (SRS) untuk GrowSense, sistem hidroponik Nutrient Film Technique (NFT) yang dapat berjalan mandiri dengan keterlibatan manusia sesedikit mungkin. Dokumen ini menjadi acuan bersama untuk perancangan, implementasi, dan pengujian. Setiap nilai yang diberi keterangan "usulan" adalah angka awal dari penulis yang dapat disesuaikan setelah pengujian (daftar lengkap pada Lampiran C).

### 1.1 Product scope

GrowSense memantau larutan nutrisi dan lingkungan tanaman melalui multi-sensor, menilai kesehatan daun dengan YOLOv8, lalu mengambil atau menyarankan tindakan korektif. Tujuan produk: meningkatkan keberhasilan pertumbuhan tanaman, mencegah penyakit sedini mungkin, meminimalkan intervensi manusia, dan menjaga pompa serta grow light tetap menyala saat listrik padam. Sistem NFT dipilih untuk mengatasi keterbatasan lahan horizontal.

| Lingkup | Rincian |
| --- | --- |
| In-Scope | Pemantauan multi-sensor (pH, TDS/EC, suhu air, level air, suhu udara, kelembapan, cahaya). |
| | Deteksi kesehatan daun 12 kelas dengan YOLOv8. |
| | Decision layer untuk membedakan patologi dari defisiensi dan menentukan tindakan. |
| | Dosing nutrisi adaptif dengan tiga mode operasi (Manual, Setengah AI, Penuh AI). |
| | Cadangan daya panel surya dan baterai untuk beban prioritas. |
| | Dashboard web dan aplikasi tersendiri, notifikasi, kontrol manual, riwayat data dan gambar. |
| | Persiapan dataset untuk pelatihan ulang model. |
| Out-of-Scope | Skala hidroponik komersial besar (multi-greenhouse) pada tahap ini. |
| | Integrasi dengan sistem penjualan atau distribusi hasil panen. |
| Won't have (rilis ini) | Sama dengan Out-of-Scope; lihat 2.2. |

### 1.2 Product value

Pemilik memperoleh tanaman yang lebih terjaga tanpa harus memeriksa kondisi larutan dan daun setiap saat. Nilai produk dinyatakan dengan target terukur berikut.

| Nilai | Target | Pengukuran |
| --- | --- | --- |
| Intervensi manusia minimal | Pemeriksaan fisik rutin maksimal 1 kali per minggu; pemilik bertindak terutama saat ada notifikasi (usulan; pembanding: sistem hidroponik yang sudah ada sebagaimana digambarkan pada BPMN AS-IS, yaitu deteksi daun sehat atau rusak, dosing otomatis, daya cadangan baterai atau aki, dan penanganan tanaman sakit secara manual oleh pemilik). | Catatan frekuensi pemeriksaan selama uji lapangan, dibandingkan dengan sistem pembanding |
| Deteksi dini | Penyakit atau defisiensi terdeteksi dalam satu siklus pengambilan gambar (maksimal 30 menit) dan dinotifikasikan maksimal 30 detik setelah deteksi. | Log deteksi dan notifikasi |
| Kualitas deteksi | mAP@0,5 minimal 0,90 pada 12 kelas. | Evaluasi model (REL-06) |
| Ketahanan terhadap pemadaman | Pompa dan grow light menyala minimal 8 jam (target 12 jam) tanpa listrik utama. | Uji pemadaman (REL-05) |

### 1.3 Intended audience

**Pengguna produk**

| Kelompok pengguna | Kebutuhan utama |
| --- | --- |
| Petani hidroponik | Pemantauan dan perawatan otomatis, peringatan dini penyakit. |
| Penggiat hobi | Dashboard mudah dipahami, kontrol manual, dan opsi mode otomatis. |
| Pengguna dengan waktu terbatas | Sistem berjalan mandiri dengan notifikasi hanya saat perlu tindakan. |
| Peneliti / open source data | Akses riwayat data dan gambar (atas izin pemilik) untuk analisis dan pengembangan model. |
| Dosen pengampu (opsional) | Melihat status dan data sistem untuk keperluan evaluasi. |

**Pembaca dokumen dan cara membacanya**

| Pembaca | Bagian yang dibaca |
| --- | --- |
| Pengembang web (Next.js) | Bab 2 (FR-02, FR-09 hingga FR-14, FR-17, FR-18), 3.1, 3.3, 3.4, Bab 4. |
| Pengembang embedded/IoT | Bab 2 (FR-01, FR-11, FR-15, FR-16), 3.2, 3.4, 4.4 (REL-02, REL-03, REL-05). |
| Pengembang AI | Bab 2 (FR-03 hingga FR-06, FR-18), 4.4 (REL-06, REL-08), 4.6 (MNT-06). |
| Penguji / QA | Bab 2 (Acceptance Criteria), Bab 4 (kolom Cara verifikasi), Lampiran A. |
| Dosen pengampu | Seluruh dokumen; Lampiran A untuk ketertelusuran dan Lampiran C untuk asumsi. |

### 1.4 Intended use

Pemilik memasang sensor, kamera, pompa, dan unit kontrol pada rak NFT, lalu mengatur jenis tanaman dan setpoint per fase pertumbuhan. Sistem membaca sensor secara periodik, memeriksa gambar daun, dan menyimpan seluruh data. Pemilik memantau melalui dashboard web dan aplikasi, menerima notifikasi, serta memilih mode operasi: Manual, Setengah AI (AI memberi rekomendasi, pemilik menyetujui), atau Penuh AI (AI memutuskan dan mengeksekusi). Pemilik dapat mengambil alih kontrol secara manual kapan saja.

### 1.5 General description

Sistem terdiri dari empat lapisan:

- **Lapisan perangkat (edge):** ESP32-S3 membaca sensor dan menggerakkan aktuator (pompa sirkulasi, pompa peristaltik dosing, grow light) melalui relay.
- **Lapisan AI:** model YOLOv8 12 kelas mendeteksi banyak daun dalam satu bingkai. Saat ini inferensi berjalan di laptop; rencana ke depan di Raspberry Pi 5 (lokasi jauh dari server) atau langsung di server (lokasi dekat).
- **Lapisan keputusan (decision layer):** menggabungkan hasil deteksi dan data sensor untuk menentukan dosing, penguncian, atau notifikasi.
- **Lapisan aplikasi:** server (Next.js, Prisma, PostgreSQL) menyediakan dashboard web, aplikasi tersendiri, notifikasi, dan penyimpanan riwayat.

Asumsi dan dependensi: jaringan lokal (Wi-Fi 2.4 GHz) tersedia antara ESP32-S3, broker MQTT, dan unit inferensi; broker MQTT saat ini lokal; kamera dan kapasitas panel surya/baterai ditetapkan pada tahap desain. Batasan: keterbatasan lahan dan waktu, serta biaya komponen yang membuat pengembangan perangkat keras bertahap. Model pengembangan: SDLC Iterative and Incremental.

---

## 2 Functional requirements

Prioritas memakai MoSCoW (Must, Should, Could, Won't). Setiap kebutuhan memiliki ID unik, atribut lengkap, dan kriteria penerimaan yang dapat diuji. Pemetaan ke kebutuhan pengguna, use case, komponen, dan test case ada pada Lampiran A. Penjelasan skenario tiap fitur ada pada dokumen Use Case Specification.

### 2.1 Kebutuhan fungsional FR-01 sampai FR-20

#### FR-01: Pembacaan sensor

- **Description:** Sistem membaca 7 nilai lingkungan dan larutan: pH, TDS/EC, suhu air, level air, suhu udara, kelembapan udara, dan intensitas cahaya.
- **Input / Trigger:** Pewaktu pada ESP32-S3: interval 1 menit untuk pH, TDS/EC, suhu air, dan level air; interval 10 menit untuk suhu udara, kelembapan, dan cahaya (usulan; dapat diubah, lihat MNT-07).
- **Business Logic / Process:**
  1. Baca setiap sensor dan konversikan ke satuan baku (pH, EC dalam mS/cm, °C, %, cm, lux). Pembacaan TDS (ppm) dikonversi ke EC dengan faktor 0,5 (usulan): EC (mS/cm) = TDS (ppm) x 0,5 / 1000. Telemetri dan dashboard memakai EC (mS/cm) sebagai satuan utama.
  2. Validasi rentang nilai (FR-16).
  3. Bentuk pesan JSON bercap waktu dan publikasikan ke broker MQTT.
- **Output / State Change:** Pesan telemetri terpublikasi pada topik telemetri; data diteruskan ke FR-02 dan FR-05.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Firmware ESP32-S3 (Wi-Fi 2.4 GHz). Satuan dan nama sensor tampil pada dashboard (FR-12).
- **Acceptance Criteria:** Pada uji 24 jam, minimal 95% pembacaan terkirim dan valid; selisih cap waktu perangkat dan server maksimal 5 detik.

#### FR-02: Penyimpanan data

- **Description:** Sistem menyimpan pembacaan sensor, status aktuator, hasil deteksi, dan log keputusan beserta cap waktu.
- **Input / Trigger:** Pesan MQTT (telemetri, status aktuator) dan keluaran decision layer.
- **Business Logic / Process:**
  1. Server berlangganan topik telemetri dan status.
  2. Validasi skema JSON; pesan tidak valid ditolak dan dicatat sebagai galat.
  3. Simpan ke basis data dengan cap waktu UTC.
- **Output / State Change:** Baris data tersimpan di basis data.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Server Linux; basis data PostgreSQL 15 atau lebih baru.
- **Acceptance Criteria:** Data tersimpan maksimal 5 detik setelah diterima; minimal 99% pesan valid tersimpan pada uji 24 jam.

#### FR-03: Pengambilan gambar daun

- **Description:** Sistem mengambil gambar daun secara berkala dan menyimpannya bersama metadata.
- **Input / Trigger:** Jadwal pengambilan gambar (bawaan 30 menit, dapat diatur 5 hingga 120 menit; usulan).
- **Business Logic / Process:**
  1. Kamera mengambil gambar JPEG dengan resolusi minimal 640 x 640 piksel.
  2. Kirim gambar ke unit inferensi (FR-04).
  3. Simpan berkas gambar dan metadata (waktu, id kamera, hasil deteksi).
- **Output / State Change:** Berkas gambar tersimpan dan terhubung ke hasil deteksi.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Kamera USB atau modul kamera; jumlah dan posisi ditetapkan pada tahap desain.
- **Acceptance Criteria:** Minimal 95% jadwal pengambilan menghasilkan gambar tersimpan pada uji 24 jam.

#### FR-04: Deteksi kesehatan daun 12 kelas

- **Description:** Sistem mendeteksi dan mengklasifikasikan setiap daun pada bingkai ke dalam 12 kelas: 8 penyakit/hama, 3 defisiensi (N, P, K), dan 1 sehat.
- **Input / Trigger:** Gambar daun dari FR-03.
- **Business Logic / Process:**
  1. Jalankan model YOLOv8 pada gambar.
  2. Hasilkan kotak deteksi, kelas, dan tingkat keyakinan untuk setiap daun.
  3. Deteksi dengan keyakinan di bawah 0,5 (usulan) ditandai "perlu tinjauan" dan tidak memicu tindakan otomatis.
- **Output / State Change:** Daftar deteksi per bingkai, disimpan dan diteruskan ke decision layer (FR-05).
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Unit inferensi: laptop (saat ini), Raspberry Pi 5 (rencana, lokasi jauh dari server), atau server (lokasi dekat). Linux, Python 3.10 atau lebih baru.
- **Acceptance Criteria:** mAP@0,5 minimal 0,90 pada data uji; waktu inferensi maksimal 500 ms per bingkai pada unit inferensi target (acuan awal laptop sekitar 5 ms).

#### FR-05: Distingsi patologi vs defisiensi

- **Description:** Decision layer menggabungkan hasil deteksi dan data sensor untuk membedakan penyakit/hama dari defisiensi nutrisi.
- **Input / Trigger:** Hasil deteksi (FR-04) dan pembacaan terakhir pH, EC, dan level air (FR-01).
- **Business Logic / Process:**
  1. Jika ada kelas penyakit/hama dengan keyakinan di atas ambang: status PATOLOGI (diutamakan).
  2. Jika tidak, dan ada kelas defisiensi N/P/K: bila EC di bawah batas bawah setpoint atau pH di luar rentang, status DEFISIENSI_NPK.
  3. Defisiensi terdeteksi tetapi EC dan pH normal: status DEFISIENSI_TIDAK_TERKONFIRMASI; kirim notifikasi tinjauan tanpa dosing otomatis (aturan usulan).
  4. Semua daun sehat: status SEHAT.
- **Output / State Change:** Status keputusan (PATOLOGI, DEFISIENSI_NPK, DEFISIENSI_TIDAK_TERKONFIRMASI, SEHAT) yang memicu FR-06, FR-07, atau FR-08.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Layanan server; log keputusan dapat dilihat pada riwayat (FR-14).
- **Acceptance Criteria:** Pada minimal 10 skenario uji kombinasi kelas deteksi dan kondisi sensor, 100% keputusan sesuai tabel aturan di atas.

#### FR-06: Dosing nutrisi adaptif

- **Description:** Sistem menambahkan nutrisi secara terkontrol ketika defisiensi NPK terkonfirmasi.
- **Input / Trigger:** Status DEFISIENSI_NPK, setpoint aktif (FR-09), pembacaan EC/pH/level air, mode operasi (FR-10).
- **Business Logic / Process:**
  1. Pastikan dosing tidak terkunci (FR-07), level air cukup, dan sensor valid (FR-16).
  2. Hitung dosis dari selisih nilai terhadap setpoint, dibatasi dosis maksimum per siklus (SEC-06).
  3. Sesuai mode: Penuh AI langsung eksekusi; Setengah AI menunggu persetujuan pemilik.
  4. Aktifkan pompa peristaltik selama durasi dosis; beri jeda minimum 30 menit antar-dosing (usulan).
  5. Baca ulang pH/EC; hentikan dosing berulang bila nilai tidak mendekati setpoint setelah 3 siklus (usulan) dan beri tahu pemilik.
- **Output / State Change:** Dosis diberikan; log dosing tercatat; notifikasi hasil terkirim.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Persetujuan dosing pada mode Setengah AI melalui dashboard atau aplikasi.
- **Acceptance Criteria:** Pada uji meja, EC mendekati setpoint dalam toleransi 10% setelah paling banyak 3 siklus; dosis tidak pernah melebihi batas maksimum (0 pelanggaran).

#### FR-07: Penguncian dosing saat penyakit/hama

- **Description:** Sistem mengunci dosing dan meminta pemilik mengisolasi tanaman ketika penyakit atau hama terdeteksi.
- **Input / Trigger:** Status PATOLOGI dari FR-05.
- **Business Logic / Process:**
  1. Set status dosing terkunci.
  2. Simpan gambar dan hasil deteksi sebagai bukti.
  3. Kirim notifikasi anjuran isolasi tanaman (FR-13).
  4. Buka kunci hanya setelah pemilik mengonfirmasi penanganan.
- **Output / State Change:** Status dosing terkunci tampil di dashboard; semua perintah dosing otomatis ditolak.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Indikator "Dosing terkunci" pada dashboard.
- **Acceptance Criteria:** Saat terkunci, 100% perintah dosing otomatis ditolak; notifikasi terkirim maksimal 30 detik setelah deteksi.

#### FR-08: Mempertahankan setpoint

- **Description:** Ketika tanaman sehat, sistem menjaga pH dan EC tetap dalam rentang setpoint.
- **Input / Trigger:** Status SEHAT dari FR-05 dan pembacaan pH/EC.
- **Business Logic / Process:**
  1. Bandingkan pH dan EC dengan rentang setpoint.
  2. Bila dalam rentang: tidak ada tindakan.
  3. Bila di luar rentang: lakukan koreksi sesuai mode operasi dan aturan dosis FR-06 (kecuali dosing terkunci).
- **Output / State Change:** Nilai pH/EC tetap dalam rentang; log koreksi tercatat bila ada.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Tidak ada.
- **Acceptance Criteria:** Pada uji 24 jam dengan tanaman sehat, nilai pH dan EC berada dalam rentang setpoint minimal 90% waktu.

#### FR-09: Pengaturan setpoint per fase

- **Description:** Pemilik mengatur rentang pH dan EC untuk setiap jenis tanaman dan fase pertumbuhan secara manual.
- **Input / Trigger:** Jenis tanaman, fase, pH minimum/maksimum, EC minimum/maksimum.
- **Business Logic / Process:**
  1. Validasi nilai terhadap batas aman (pH 4,5 hingga 8,0; EC 0,5 hingga 4,0 mS/cm; usulan).
  2. Simpan setpoint beserta cap waktu perubahan.
  3. Teruskan setpoint aktif ke decision layer.
- **Output / State Change:** Setpoint tersimpan dan berlaku.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Halaman setpoint pada web/aplikasi; hanya peran Owner.
- **Acceptance Criteria:** 100% input di luar batas aman ditolak dengan pesan; setpoint baru berlaku maksimal 2 detik setelah disimpan.

#### FR-10: Mode operasi

- **Description:** Pemilik memilih mode Manual, Setengah keputusan AI, atau Penuh keputusan AI.
- **Input / Trigger:** Pilihan mode dari pemilik.
- **Business Logic / Process:**
  1. Manual: sistem hanya mengirim notifikasi dan rekomendasi.
  2. Setengah AI: sistem mengirim rekomendasi dan menunggu persetujuan pemilik (batas waktu 30 menit; usulan), lewat batas dibatalkan.
  3. Penuh AI: sistem mengeksekusi keputusan otomatis dalam batas aman.
  4. Mode Penuh AI ditolak bila sensor kritis (pH, EC, level air) sedang error.
- **Output / State Change:** Mode aktif tercatat dan tampil pada dashboard.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Pemilih mode dengan penjelasan singkat tiap mode; konfirmasi saat beralih ke Penuh AI.
- **Acceptance Criteria:** Perubahan mode berlaku maksimal 2 detik; pada mode Manual tidak ada dosing otomatis (0 kejadian pada uji 24 jam).

#### FR-11: Kontrol aktuator manual

- **Description:** Pemilik menyalakan atau mematikan pompa sirkulasi dan grow light, serta memicu dosing sekali, dari dashboard.
- **Input / Trigger:** Perintah dari pemilik terautentikasi.
- **Business Logic / Process:**
  1. Periksa hak akses (FR-17) dan kondisi keselamatan (dosing terkunci, level air rendah menolak dosing).
  2. Kirim perintah melalui MQTT dengan QoS 1.
  3. ESP32-S3 menjalankan relay dan mengirim status balik.
  4. Bila tidak ada balasan dalam 5 detik, tandai gagal dan beri tahu pemilik.
  5. Catat aksi pada log.
- **Output / State Change:** Status aktuator berubah dan tampil di dashboard; log aksi tersimpan.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Halaman kontrol; maksimal 3 klik dari dashboard.
- **Acceptance Criteria:** Aksi terlihat pada perangkat maksimal 3 detik pada jaringan lokal; perintah yang melanggar kondisi keselamatan ditolak 100% dengan alasan.

#### FR-12: Dashboard monitoring

- **Description:** Dashboard menampilkan data sensor terkini, grafik historis, status aktuator, status daya, dan hasil deteksi terbaru.
- **Input / Trigger:** Pemilik atau Viewer membuka dashboard; pemilihan rentang grafik (1 jam, 24 jam, 7 hari, 30 hari).
- **Business Logic / Process:**
  1. Ambil data terbaru dari basis data.
  2. Tampilkan nilai, grafik, status aktuator, status daya (listrik utama, surya, baterai), dan deteksi terakhir.
  3. Beri penanda "data tidak terbaru" bila data lebih lama dari dua kali interval pembacaan.
- **Output / State Change:** Halaman dashboard tampil.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Web responsif (lihat 3.1); pembaruan otomatis tanpa muat ulang halaman.
- **Acceptance Criteria:** Dashboard tampil maksimal 3 detik pada jaringan lokal; 100% data lebih lama dari dua kali interval ditandai.

#### FR-13: Notifikasi

- **Description:** Sistem mengirim notifikasi ke web dan aplikasi untuk kejadian penting.
- **Input / Trigger:** Kejadian: penyakit/hama, defisiensi, sensor error, level air rendah, peralihan daya, baterai rendah, perangkat offline.
- **Business Logic / Process:**
  1. Bentuk notifikasi berisi jenis, tingkat (Info, Peringatan, Kritis), penyebab, dan saran tindakan.
  2. Kirim ke web dan aplikasi.
  3. Untuk tingkat Kritis, ulangi tiap 30 menit sampai ditanggapi (usulan).
- **Output / State Change:** Notifikasi diterima dan tercatat beserta tanggapan pemilik.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Daftar notifikasi pada web/aplikasi; tingkat tidak hanya dibedakan lewat warna.
- **Acceptance Criteria:** Notifikasi terkirim maksimal 30 detik setelah kejadian pada jaringan yang tersedia.

#### FR-14: Riwayat data dan gambar

- **Description:** Pemilik menelusuri dan mengunduh riwayat sensor, keputusan, dan gambar.
- **Input / Trigger:** Filter rentang waktu dan jenis data.
- **Business Logic / Process:**
  1. Kueri data sesuai filter.
  2. Tampilkan tabel, grafik, dan galeri gambar dengan hasil deteksi.
  3. Sediakan unduhan CSV (data) dan JPEG (gambar).
- **Output / State Change:** Data tampil atau berkas terunduh.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Halaman riwayat dengan paginasi.
- **Acceptance Criteria:** Hasil filter tampil maksimal 5 detik pada basis data 5 juta baris; berkas CSV dapat dibuka tanpa galat.

#### FR-15: Manajemen daya cadangan

- **Description:** Saat listrik utama padam, sistem beralih ke panel surya dan baterai serta memprioritaskan pompa sirkulasi dan grow light.
- **Input / Trigger:** Sinyal hilangnya listrik utama dari pengatur daya.
- **Business Logic / Process:**
  1. Beralih ke baterai/panel surya.
  2. Pertahankan beban prioritas: pompa sirkulasi, grow light, dan unit kontrol ESP32-S3 (karena mengendalikan relay).
  3. Pantau kapasitas baterai; saat di bawah 30% (usulan), tangguhkan beban non-prioritas (kamera, inferensi).
  4. Kirim notifikasi peralihan, baterai rendah, dan pemulihan listrik utama.
- **Output / State Change:** Beban prioritas tetap menyala; status daya tampil di dashboard.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Indikator sumber daya pada dashboard.
- **Acceptance Criteria:** Pada uji pemadaman terkontrol, pompa dan grow light menyala minimal 8 jam (target 12 jam); ESP32-S3 tidak restart saat peralihan.

#### FR-16: Penanganan sensor error

- **Description:** Sistem mencegah keputusan berdasarkan data sensor yang rusak atau tidak wajar.
- **Input / Trigger:** Pembacaan di luar rentang valid atau tidak ada pembacaan.
- **Business Logic / Process:**
  1. Rentang valid: pH 0 hingga 14; EC 0 hingga 4,0 mS/cm (rentang sensor target; sensor prototipe SEN-0171 hanya sampai 2,0 mS/cm, setara TDS 1000 ppm pada faktor 0,5, lihat 3.2); suhu air 5 hingga 45 °C; suhu udara 0 hingga 50 °C; kelembapan 0 hingga 100%; cahaya 0 hingga 65535 lux; level air sesuai tinggi tandon.
  2. Tandai sensor ERROR dan hentikan aksi otomatis yang bergantung padanya.
  3. Kirim notifikasi; pulihkan status saat pembacaan kembali valid.
- **Output / State Change:** Status sensor tampil di dashboard; notifikasi terkirim.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Penanda sensor error pada dashboard.
- **Acceptance Criteria:** Pembacaan tidak valid terdeteksi dalam satu siklus pembacaan; 0 dosing berlangsung saat sensor pH, EC, atau level air error.

#### FR-17: Autentikasi dan otorisasi

- **Description:** Hanya pengguna terautentikasi dengan peran yang sesuai dapat mengakses dan mengontrol sistem.
- **Input / Trigger:** Email dan kata sandi.
- **Business Logic / Process:**
  1. Verifikasi kredensial terhadap hash tersimpan.
  2. Buat sesi 60 menit; setelah 3 kali gagal berturut-turut, tunda login 5 menit (usulan).
  3. Terapkan peran: Owner (kontrol penuh) dan Viewer (lihat saja).
- **Output / State Change:** Sesi aktif atau akses ditolak.
- **Priority (MoSCoW):** Must
- **UI & OS Constraints:** Form login pada web dan aplikasi.
- **Acceptance Criteria:** Pada uji otorisasi, 100% endpoint kontrol menolak pengguna tanpa sesi atau berperan Viewer; tidak ada kata sandi tersimpan sebagai teks biasa.

#### FR-18: Persiapan dataset untuk pelatihan ulang

- **Description:** Pemilik memakai gambar riwayat untuk memperbaiki model dengan mengoreksi label dan mengekspor dataset.
- **Input / Trigger:** Gambar riwayat terpilih dan koreksi label.
- **Business Logic / Process:**
  1. Tampilkan gambar dengan hasil deteksi.
  2. Simpan label yang dikoreksi; validasi kelas termasuk 12 kelas dan kotak batas valid.
  3. Ekspor dataset berformat YOLO (berkas label dan konfigurasi) dalam satu arsip.
- **Output / State Change:** Arsip dataset terunduh.
- **Priority (MoSCoW):** Should
- **UI & OS Constraints:** Halaman galeri dengan editor label sederhana.
- **Acceptance Criteria:** Dataset hasil ekspor dapat dibaca pipeline pelatihan YOLOv8 tanpa galat.

#### FR-19: Penentuan fase pertumbuhan otomatis

- **Description:** Sistem menentukan sendiri fase pertumbuhan dan kebutuhan nutrisi berdasarkan jenis tanaman.
- **Input / Trigger:** Jenis tanaman dan tanggal tanam.
- **Business Logic / Process:**
  1. Cocokkan umur tanaman dengan tabel referensi fase per jenis tanaman.
  2. Usulkan setpoint fase; pemilik dapat menyetujui atau mengubah.
- **Output / State Change:** Fase dan setpoint usulan tampil.
- **Priority (MoSCoW):** Could (tahap lanjutan)
- **UI & OS Constraints:** Terintegrasi pada halaman setpoint (FR-09).
- **Acceptance Criteria:** Rekomendasi fase sesuai tabel referensi untuk minimal 3 jenis tanaman.

#### FR-20: Akses data terbuka untuk peneliti

- **Description:** Pemilik dapat membagikan data riwayat secara anonim kepada peneliti atau komunitas terbuka.
- **Input / Trigger:** Persetujuan eksplisit pemilik dan pilihan rentang data.
- **Business Logic / Process:**
  1. Hapus data pribadi pemilik dari ekspor.
  2. Hasilkan berkas ekspor atau tautan akses baca.
- **Output / State Change:** Berkas data anonim tersedia.
- **Priority (MoSCoW):** Could (tahap lanjutan)
- **UI & OS Constraints:** Opsi berbagi pada halaman riwayat; nonaktif secara bawaan.
- **Acceptance Criteria:** 100% ekspor tidak memuat nama atau email pemilik.

### Aturan kunci decision layer (ringkasan)

| Kondisi | Tindakan |
| --- | --- |
| PATOLOGI (penyakit/hama) | Kunci dosing, simpan bukti, kirim notifikasi isolasi tanaman (FR-07). |
| DEFISIENSI_NPK (terkonfirmasi sensor) | Dosing otomatis atau rekomendasi sesuai mode (FR-06). |
| DEFISIENSI_TIDAK_TERKONFIRMASI | Notifikasi tinjauan; tanpa dosing otomatis (FR-05). |
| SEHAT | Pertahankan setpoint (FR-08). |
| Listrik utama padam | Beralih ke daya cadangan; prioritaskan pompa dan grow light (FR-15). |
| Sensor error | Tandai sensor dan hentikan aksi yang bergantung padanya (FR-16). |

### 2.2 Won't have (rilis ini)

| ID | Item |
| --- | --- |
| W-01 | Pengelolaan skala komersial multi-greenhouse. |
| W-02 | Integrasi dengan sistem penjualan atau distribusi hasil panen. |

---

## 3 External interface requirements

### 3.1 User interface requirements

- **Platform:** dashboard web (Next.js) dan aplikasi tersendiri. Halaman: Dashboard, Kontrol, Setpoint, Riwayat, Notifikasi, Akun.
- **Responsivitas:** tata letak responsif pada resolusi minimal Desktop 1280 x 720 piksel dan Mobile 360 x 800 piksel.
- **Aksesibilitas:** WCAG 2.1 Level AA: rasio kontras teks biasa minimal 4,5:1 (3:1 untuk teks besar), navigasi keyboard penuh, label untuk pembaca layar; status penting (error, kritis) tidak hanya dibedakan lewat warna.
- **Design token:** warna, tipografi, dan jarak didefinisikan sebagai token terpusat pada tahap desain; panduan gaya dan wireframe dilampirkan pada dokumen desain.
- **Alur:** kontrol manual dapat dicapai maksimal 3 klik dari dashboard; aksi berisiko (dosing manual, Penuh AI) meminta konfirmasi.
- **Bahasa:** Indonesia.

### 3.2 Hardware interface requirements

| Komponen | Antarmuka ke ESP32-S3 | Peran |
| --- | --- | --- |
| ESP32-S3 | - | Mikrokontroler utama: membaca sensor, mengendalikan relay, klien MQTT melalui Wi-Fi 2.4 GHz. |
| PH-4502C | Analog (ADC) | Sensor pH larutan. |
| SEN-0171 (TDS), sensor prototipe | Analog (ADC) | Sensor TDS/EC larutan nutrisi pada prototipe; rentang baca sampai 2,0 mS/cm (TDS 1000 ppm). |
| Sensor EC target | Analog (ADC) atau sesuai sensor | Pengganti SEN-0171 pada tahap lanjutan; mampu membaca EC minimal 0 sampai 4,0 mS/cm agar sesuai batas aman setpoint pada FR-09. |
| DS18B20 | 1-Wire | Sensor suhu air. |
| DHT22 | Digital 1 kabel | Sensor suhu dan kelembapan udara. |
| JSN-SR04T | Digital (trigger/echo) | Sensor ultrasonik level air tandon. |
| BH1750 | I2C | Sensor intensitas cahaya. |
| Relay 4 channel | Digital (GPIO) | Saklar pompa sirkulasi (2 pompa celup), pompa peristaltik dosing, dan grow light LED. |
| Kamera USB atau modul kamera | - | Pengambil gambar daun (jumlah, posisi, dan spesifikasi ditetapkan pada tahap desain; resolusi minimal 640 x 640 piksel). |
| Unit inferensi | LAN/Wi-Fi | Laptop (saat ini), Raspberry Pi 5 (rencana), atau server. Spesifikasi minimum pada COM-04. |
| Panel surya, baterai, pengatur daya | Sinyal status daya | Cadangan daya untuk pompa sirkulasi, grow light, dan ESP32-S3 selama 8 hingga 12 jam. Kapasitas (Wp, Ah) ditetapkan pada desain kelistrikan. |

Daftar komponen di atas adalah acuan perancangan dan dapat diganti selama memenuhi rentang ukur dan antarmuka yang tertera. Penggantian sensor EC tidak mengubah kebutuhan fungsional karena nilai dikirim sebagai EC (mS/cm).

Jaringan: LAN/Wi-Fi lokal antara ESP32-S3, broker MQTT, unit inferensi, dan server; sinkronisasi waktu perangkat melalui NTP.

### 3.3 Software interface requirements

| Komponen | Versi / spesifikasi | Peran |
| --- | --- | --- |
| Next.js | Versi stabil terbaru, Node.js LTS | Dashboard web dan API server. |
| Prisma | Versi stabil terbaru | ORM akses basis data. |
| PostgreSQL | 15 atau lebih baru | Data sensor, pengguna, setpoint, log keputusan, metadata gambar. |
| YOLOv8 | Python 3.10 atau lebih baru | Model deteksi 12 kelas pada unit inferensi. |
| Broker MQTT | Lokal; MQTT 3.1.1 atau 5 | Perantara pesan perangkat, unit inferensi, dan server. |
| Firmware ESP32-S3 | Arduino framework atau ESP-IDF | Pembacaan sensor, kontrol relay, klien MQTT. |
| Penyimpanan gambar | Berkas di server | Gambar dengan referensi di basis data. |

### 3.4 Communication interface requirements

- **MQTT:** telemetri QoS 0 atau 1, perintah dan status aktuator QoS 1. Struktur topik: `growsense/{device_id}/telemetry`, `/status`, `/command`, `/ack`.
- **Format data:** JSON UTF-8 dengan kolom `schema_version`, `device_id`, `ts` (ISO 8601 UTC), dan nilai sensor. Gambar dikirim sebagai JPEG melalui HTTPS (multipart).
- **HTTP/HTTPS:** antara klien (web/aplikasi) dan server, minimal TLS 1.2 (SEC-04).
- **Notifikasi:** notifikasi web dan aplikasi tersendiri; teknologi push ditetapkan pada tahap desain. Email dan platform pesan pihak ketiga tidak diwajibkan pada rilis ini.
- **Keamanan kanal:** lihat SEC-04.

---

## 4 Non-functional requirements

Setiap persyaratan memiliki ID, nilai terukur, dan cara verifikasi (uji, inspeksi, analisis, atau demonstrasi).

### 4.1 Security

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| SEC-01 | Kata sandi disimpan sebagai hash (bcrypt atau Argon2) dan tidak pernah sebagai teks biasa. | Inspeksi basis data dan tinjauan kode |
| SEC-02 | Sesi berlaku 60 menit; setelah 3 kali gagal login berturut-turut, login ditunda 5 menit. | Uji fungsional |
| SEC-03 | Otorisasi RBAC dengan peran Owner dan Viewer. Hanya Owner dan decision layer yang dapat mengirim perintah ke pompa dan dosing. | Uji otorisasi pada seluruh endpoint kontrol |
| SEC-04 | Komunikasi klien-server memakai HTTPS (minimal TLS 1.2, TLS 1.3 diutamakan). Broker MQTT memakai autentikasi pengguna dan kata sandi, serta TLS bila lalu lintas melewati jaringan di luar LAN. | Pemindaian konfigurasi TLS dan inspeksi broker |
| SEC-05 | Data pribadi pemilik (nama, email) dienkripsi saat disimpan (AES-256) bila server berada di cloud, dan tidak dibagikan tanpa persetujuan. | Inspeksi konfigurasi |
| SEC-06 | Firmware menolak perintah dosing yang melebihi batas maksimum per siklus dan per hari, walaupun perintah berasal dari server. Nilai batas ditetapkan saat kalibrasi pompa dan dicatat sebelum uji integrasi. | Uji meja dengan perintah melebihi batas (0 pelanggaran) |
| SEC-07 | Sebelum rilis, dilakukan uji keamanan terhadap kategori OWASP Top 10 tanpa temuan tingkat High atau Critical. | Laporan uji keamanan |

### 4.2 Capacity

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| CAP-01 | Data sensor berjumlah 7 nilai per siklus. Bila semua nilai dibaca tiap 1 menit dan disimpan satu baris per nilai: 10.080 baris/hari (sekitar 3,68 juta/tahun). Dengan pembagian interval pada FR-01: 6.192 baris/hari (sekitar 2,26 juta/tahun). Basis data harus menampung minimal 5 juta baris tanpa kueri dashboard melebihi 3 detik. | Uji beban dengan 5 juta baris data sintetis |
| CAP-02 | Gambar (asumsi 1 kamera, 1 gambar per 30 menit, sekitar 300 KB per gambar): sekitar 14 MB/hari atau 5 GB/tahun. Penyimpanan server minimal 50 GB. | Analisis dan pengukuran saat uji |
| CAP-03 | Retensi: data sensor mentah 12 bulan lalu dirangkum per jam; log keputusan 12 bulan; gambar tanpa label diarsipkan setelah 90 hari; gambar berlabel dipertahankan untuk dataset. | Inspeksi kebijakan retensi |
| CAP-04 | Sistem melayani minimal 5 pengguna aktif bersamaan (1 Owner dan beberapa Viewer) tanpa error timeout. | Uji beban 5 sesi |

### 4.3 Compatibility

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| COM-01 | Browser minimal: Chrome 110, Firefox 110, Edge 110, Safari 16. | Uji lintas browser |
| COM-02 | Sistem operasi klien: Windows 10/11, macOS, Linux, Android 10 atau lebih baru, iOS 15 atau lebih baru. | Uji pada perangkat sampel |
| COM-03 | Server dan unit inferensi: Linux 64-bit (Ubuntu/Debian/Raspberry Pi OS), Python 3.10 atau lebih baru, PostgreSQL 15 atau lebih baru, Node.js versi LTS. | Inspeksi lingkungan |
| COM-04 | Perangkat keras minimum unit inferensi: laptop dengan RAM minimal 8 GB, atau Raspberry Pi 5 dengan RAM minimal 8 GB (usulan). ESP32-S3 terhubung ke Wi-Fi 2.4 GHz. | Uji pada perangkat target |
| COM-05 | Pesan telemetri berformat JSON UTF-8 dengan kolom schema_version agar firmware dan server berbeda versi tetap kompatibel. | Uji kompatibilitas dua versi skema |

### 4.4 Reliability

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| REL-01 | Ketersediaan layanan server (dashboard, API, penyimpanan) minimal 99% per bulan (maksimal sekitar 7,2 jam tidak tersedia), di luar pemeliharaan terjadwal yang diumumkan. | Log uptime |
| REL-02 | Fail-safe lokal: bila koneksi ke broker atau server terputus, ESP32-S3 mempertahankan pompa sirkulasi dan jadwal grow light terakhir, dan menghentikan dosing otomatis sampai koneksi pulih. | Uji putus jaringan 1 jam |
| REL-03 | MTBF unit kontrol minimal 720 jam (30 hari) operasi tanpa restart tidak terencana. | Log uptime perangkat selama uji 30 hari |
| REL-04 | Pemulihan layanan: RTO maksimal 4 jam dan RPO maksimal 24 jam (cadangan basis data harian otomatis). | Uji pemulihan dari cadangan |
| REL-05 | Pompa sirkulasi dan grow light tetap menyala minimal 8 jam (target 12 jam) saat listrik utama padam. | Uji pemadaman terkontrol |
| REL-06 | Akurasi model YOLOv8: mAP@0,5 minimal 0,90 pada data uji (12 kelas). | Evaluasi model pada data uji |
| REL-07 | Keberhasilan pembacaan sensor: minimal 95% pembacaan valid per 24 jam. | Rasio log pembacaan valid |
| REL-08 | Waktu inferensi maksimal 500 ms per bingkai pada unit inferensi target (acuan awal laptop sekitar 5 ms). | Pengukuran pada perangkat target |

### 4.5 Scalability

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| SCA-01 | Sistem melayani satu instalasi NFT dan dapat menambah hingga 5 node ESP32-S3 (rak) tanpa mengubah skema basis data atau struktur topik MQTT (topik memuat device_id). | Uji simulasi 5 node |
| SCA-02 | Server menerima dan menyimpan minimal 10 pesan MQTT per detik. | Uji beban |
| SCA-03 | Auto-scaling horizontal tidak diperlukan pada rilis ini (satu server). Unit inferensi dapat dipindah dari laptop ke Raspberry Pi 5 atau server tanpa mengubah antarmuka pesan. | Inspeksi desain dan uji pemindahan |

### 4.6 Maintainability

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| MNT-01 | Kode dikelola dengan Git dan setiap perubahan melalui pull request. | Inspeksi repositori |
| MNT-02 | Pipeline CI/CD (mis. GitHub Actions) menjalankan lint, build, dan uji otomatis pada setiap push, serta melakukan deployment otomatis. | Inspeksi pipeline |
| MNT-03 | Cakupan unit test minimal 80% untuk decision layer dan logika dosing, dan minimal 60% untuk keseluruhan kode server. | Laporan coverage |
| MNT-04 | Logging terpusat dengan level INFO/WARN/ERROR untuk aplikasi dan decision layer; retensi 90 hari. | Inspeksi log |
| MNT-05 | Kalibrasi sensor: pH tiap 1 minggu; TDS/EC tiap 2 minggu; suhu air, suhu udara, level air, dan cahaya tiap 1 bulan (usulan, dalam rentang seminggu hingga sebulan sesuai jenis dan kualitas sensor). Jadwal tertulis pada panduan pemeliharaan. | Inspeksi panduan pemeliharaan |
| MNT-06 | Pelatihan ulang model dilakukan bila mAP@0,5 pada data uji terbaru di bawah 0,85 atau tersedia minimal 500 gambar berlabel baru. | Tinjauan evaluasi model berkala |
| MNT-07 | Setpoint, interval pembacaan, interval gambar, dan ambang keyakinan dapat diubah tanpa mengubah firmware. | Demonstrasi |
| MNT-08 | Bug kritis (berdampak pada dosing atau pompa) diperbaiki maksimal 7 hari sejak dilaporkan (usulan). | Tinjauan catatan isu |

### 4.7 Usability

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| USA-01 | Skor System Usability Scale (SUS) minimal 75 pada evaluasi dengan minimal 5 responden. | Kuesioner SUS |
| USA-02 | Pengguna baru dapat melakukan tiga tugas utama (melihat status, mengubah mode, mengontrol pompa) dalam waktu maksimal 15 menit setelah panduan singkat. | Uji penggunaan dengan pengguna |
| USA-03 | Pencegahan kesalahan: konfirmasi sebelum dosing manual dan sebelum beralih ke Penuh AI; pesan galat memuat penyebab dan langkah perbaikan. | Demonstrasi dan inspeksi UI |
| USA-04 | Antarmuka berbahasa Indonesia; kontrol manual dapat dicapai maksimal 3 klik dari dashboard. | Inspeksi UI |
| USA-05 | Memenuhi aksesibilitas WCAG 2.1 Level AA (lihat 3.1). | Audit aksesibilitas |

### 4.8 Other non-functional requirements

| ID | Persyaratan | Cara verifikasi |
| --- | --- | --- |
| OTH-01 | Kepatuhan UU No. 27 Tahun 2022 tentang Pelindungan Data Pribadi: hanya data pribadi minimum (nama, email) yang dikumpulkan; berbagi data memerlukan persetujuan eksplisit; pemilik dapat menghapus akun dan datanya. | Inspeksi alur data dan fitur penghapusan |
| OTH-02 | Kinerja: lihat tabel waktu respons di bawah. | Pengukuran sesuai tabel |
| OTH-03 | Keselamatan listrik: proteksi arus (sekering) pada jalur pompa dan baterai; perangkat elektronik dalam enclosure minimal IP54 (usulan) dan terpisah dari jalur air. | Inspeksi instalasi |
| OTH-04 | Kondisi lingkungan operasi: suhu 10 hingga 40 °C dan kelembapan hingga 95% tanpa kondensasi (usulan). | Inspeksi spesifikasi komponen |
| OTH-05 | Kendala proyek: biaya komponen dan waktu. Pengembangan berjalan bertahap (Iterative and Incremental); fitur Could ditunda ke tahap lanjutan. | Tinjauan rencana iterasi |

**Waktu respons (OTH-02).** Diukur pada jaringan lokal dengan maksimal 5 pengguna aktif.

| Operasi | Batas waktu | Terkait |
| --- | --- | --- |
| Login hingga dashboard tampil | 3 detik | FR-17, FR-12 |
| Dashboard tampil | 3 detik | FR-12 |
| Simpan setpoint | 2 detik | FR-09 |
| Ganti mode operasi | 2 detik | FR-10 |
| Aksi manual terlihat pada perangkat | 3 detik | FR-11 |
| Data sensor tersimpan setelah pembacaan | 5 detik | FR-02 |
| Keputusan dosing setelah deteksi | 10 detik | FR-06 |
| Notifikasi terkirim setelah kejadian | 30 detik | FR-13 |
| Hasil filter riwayat tampil | 5 detik | FR-14 |

---

## 5 Definitions and acronyms

| Istilah | Definisi |
| --- | --- |
| NFT | Nutrient Film Technique, metode hidroponik dengan lapisan tipis larutan nutrisi yang mengalir di sepanjang akar. |
| SRS | Software Requirement Specification. |
| YOLOv8 | You Only Look Once versi 8, model deteksi objek waktu nyata. |
| mAP@0,5 | Mean Average Precision pada ambang IoU 0,5, ukuran kualitas deteksi objek. |
| EC / TDS | Electrical Conductivity / Total Dissolved Solids, ukuran konsentrasi nutrisi terlarut. |
| NPK | Nitrogen, Fosfor, dan Kalium, unsur hara makro utama. |
| MQTT / QoS | Message Queuing Telemetry Transport, protokol pesan ringan untuk IoT; QoS adalah tingkat jaminan pengiriman pesan. |
| ESP32-S3 | Mikrokontroler dengan Wi-Fi/Bluetooth yang dipakai sebagai unit kontrol utama. |
| Decision layer | Lapisan logika yang menggabungkan hasil deteksi dan data sensor untuk menentukan tindakan. |
| Dosing | Penambahan larutan nutrisi atau pengatur pH dalam takaran terkontrol. |
| Fail-safe | Kondisi aman bawaan: pompa sirkulasi tetap berjalan dan dosing otomatis berhenti saat koneksi terputus. |
| RBAC | Role-Based Access Control, pembatasan akses berdasarkan peran pengguna. |
| TLS | Transport Layer Security, protokol enkripsi komunikasi. |
| WCAG | Web Content Accessibility Guidelines, pedoman aksesibilitas web. |
| MTBF / RTO / RPO | Mean Time Between Failures; Recovery Time Objective (target waktu pemulihan); Recovery Point Objective (batas kehilangan data dalam satuan waktu). |
| SUS | System Usability Scale, kuesioner standar untuk mengukur kemudahan penggunaan. |
| RTM | Requirements Traceability Matrix, matriks yang menghubungkan kebutuhan pengguna, kebutuhan fungsional, komponen, dan test case. |
| MoSCoW | Skema prioritas: Must, Should, Could, Won't. |
| CR | Change Request, permintaan perubahan formal terhadap SRS. |
| SDLC | Software Development Life Cycle; model yang dipakai adalah Iterative and Incremental. |
| BPMN | Business Process Model and Notation; BPMN 2.0 Kelompok 4 dipakai sebagai pendukung konsep (AS-IS, TO-BE, user persona). |

---

## Referensi

- Sommerville, I. (2016). *Software Engineering* (10th ed.). Pearson Education.
- Pressman, R. S., & Maxim, B. R. (2020). *Software Engineering: A Practitioner's Approach* (9th ed.). McGraw-Hill Education.
- Wiegers, K., & Beatty, J. (2013). *Software Requirements* (3rd ed.). Microsoft Press.
- IEEE Computer Society. (1998). *IEEE Std 830-1998: IEEE Recommended Practice for Software Requirements Specifications*.
- ISO/IEC/IEEE. (2018). *ISO/IEC/IEEE 29148:2018 Systems and software engineering: Life cycle processes: Requirements engineering*.
- Team Asana. (2026). *How to write a software requirement document (with template)*. https://asana.com/resources/software-requirement-document-template
- Boehm, B., & Basili, V. R. (2001). Software defect reduction top 10 list. *Computer, 34*(1), 135-137.
- Republik Indonesia. (2022). *Undang-Undang No. 27 Tahun 2022 tentang Pelindungan Data Pribadi*.
- BPMN 2.0 Kelompok 4 (Lucidspark): AS-IS, TO-BE, dan User Persona.
- Use Case Specification GrowSense v1.2 (template Cockburn).

---

## Lampiran A Requirements Traceability Matrix

Matriks menghubungkan kebutuhan pengguna (User Need) ke kebutuhan fungsional, use case, komponen, dan test case. Prinsipnya: tidak ada kebutuhan tanpa pengujian (No Missing Tests) dan tidak ada fitur tanpa alasan (No Orphan Features). Kebutuhan pengguna diturunkan dari tujuan proyek yang disampaikan penulis dan dapat diselaraskan dengan BPMN Kelompok 4.

### A.1 Kebutuhan pengguna

| User Need ID | Kebutuhan |
| --- | --- |
| UN-01 | Memantau kondisi larutan dan lingkungan tanpa harus hadir di lokasi. |
| UN-02 | Mengetahui kesehatan daun dan mendeteksi penyakit/defisiensi sedini mungkin. |
| UN-03 | Nutrisi terjaga otomatis dengan intervensi manusia sesedikit mungkin. |
| UN-04 | Mencegah penyebaran penyakit/hama. |
| UN-05 | Memegang kendali penuh atas sistem (mode dan kontrol manual) dengan akses aman. |
| UN-06 | Sistem tetap hidup saat listrik padam. |
| UN-07 | Keputusan hanya berdasarkan data sensor yang tepercaya. |
| UN-08 | Menyimpan riwayat untuk evaluasi dan memperbaiki model. |

### A.2 Matriks ketertelusuran

| User Need | FR | Fitur | Use case | Komponen | Test Case ID | Status |
| --- | --- | --- | --- | --- | --- | --- |
| UN-01 | FR-01 | Pembacaan sensor | UC-01 | Firmware ESP32-S3, sensor, broker MQTT | TC-FR01-01, TC-FR01-02 | Planned |
| UN-01 | FR-02 | Penyimpanan data | UC-01 | Ingest service, PostgreSQL | TC-FR02-01, TC-FR02-02 | Planned |
| UN-02 | FR-03 | Pengambilan gambar daun | UC-02 | Kamera, unit inferensi | TC-FR03-01, TC-FR03-02 | Planned |
| UN-02 | FR-04 | Deteksi 12 kelas | UC-02 | Unit inferensi (YOLOv8) | TC-FR04-01, TC-FR04-02 | Planned |
| UN-02 | FR-05 | Distingsi patologi vs defisiensi | UC-02 | Decision layer | TC-FR05-01, TC-FR05-02 | Planned |
| UN-03 | FR-06 | Dosing nutrisi adaptif | UC-02 | Decision layer, firmware (pompa dosing) | TC-FR06-01, TC-FR06-02 | Planned |
| UN-04 | FR-07 | Penguncian dosing | UC-02 | Decision layer, notification service | TC-FR07-01, TC-FR07-02 | Planned |
| UN-03 | FR-08 | Mempertahankan setpoint | UC-02 | Decision layer | TC-FR08-01, TC-FR08-02 | Planned |
| UN-03 | FR-09 | Setpoint per fase | Pendukung | Web dashboard, PostgreSQL | TC-FR09-01, TC-FR09-02 | Planned |
| UN-03, UN-05 | FR-10 | Mode operasi | UC-02 | Web dashboard, decision layer | TC-FR10-01, TC-FR10-02 | Planned |
| UN-05 | FR-11 | Kontrol aktuator manual | Pendukung | Web dashboard, broker MQTT, firmware | TC-FR11-01, TC-FR11-02 | Planned |
| UN-01 | FR-12 | Dashboard monitoring | UC-01 | Web dashboard, PostgreSQL | TC-FR12-01, TC-FR12-02 | Planned |
| UN-02 | FR-13 | Notifikasi | UC-02, UC-03 | Notification service | TC-FR13-01, TC-FR13-02 | Planned |
| UN-08 | FR-14 | Riwayat data dan gambar | UC-01 (sebagian) | Web dashboard, PostgreSQL, penyimpanan gambar | TC-FR14-01, TC-FR14-02 | Planned |
| UN-06 | FR-15 | Manajemen daya cadangan | UC-03 | Power manager, firmware | TC-FR15-01, TC-FR15-02 | Planned |
| UN-07 | FR-16 | Penanganan sensor error | UC-01, UC-02 | Firmware, decision layer | TC-FR16-01, TC-FR16-02 | Planned |
| UN-05 | FR-17 | Autentikasi dan otorisasi | UC-01 | Modul autentikasi (Next.js) | TC-FR17-01, TC-FR17-02 | Planned |
| UN-08 | FR-18 | Dataset pelatihan ulang | Pendukung | Web dashboard, penyimpanan gambar | TC-FR18-01 | Planned |
| UN-03 | FR-19 | Fase otomatis | Pendukung | Decision layer | TC-FR19-01 | Planned |
| UN-08 | FR-20 | Akses data terbuka | Pendukung | API ekspor data | TC-FR20-01 | Planned |

Status: Planned = test case direncanakan, belum dijalankan. Test case dirinci pada dokumen pengujian berdasarkan Acceptance Criteria tiap FR dan cara verifikasi tiap NFR.

---

## Lampiran B Prosedur Perubahan (Change Request)

Perubahan pada SRS ini tidak dilakukan secara informal. Setiap usulan perubahan mengisi formulir Change Request (CR) berikut, dinilai dampaknya, dan disetujui sebelum SRS diperbarui.

| Kolom CR | Isi |
| --- | --- |
| CR ID dan tanggal | Nomor unik (CR-001, dst.) dan tanggal pengajuan. |
| Pengusul | Nama dan peran. |
| Deskripsi dan alasan | Perubahan yang diminta dan alasannya. |
| Kebutuhan terdampak | ID FR/NFR, use case, dan test case yang berubah. |
| Analisis dampak | Estimasi dampak pada biaya komponen, waktu, dan sumber daya. |
| Keputusan | Disetujui, Ditolak, atau Ditunda, beserta penyetuju (dosen pengampu/pemilik). |
| Versi SRS | Nomor versi hasil perubahan dan entri pada Revision history. |

Aturan versi: penyesuaian nilai atau redaksi menaikkan versi minor (1.1 menjadi 1.2); penambahan atau penghapusan kebutuhan berprioritas Must menaikkan versi mayor (menjadi 2.0).

---

## Lampiran C Asumsi dan nilai usulan yang perlu divalidasi

| Butir | Nilai usulan | Cara validasi |
| --- | --- | --- |
| Interval sensor | 1 menit: pH, EC, suhu air, level air; 10 menit: suhu udara, kelembapan, cahaya | Uji lapangan dan kebutuhan penulis |
| Interval gambar | 30 menit (5 hingga 120 menit) | Uji beban penyimpanan dan kecepatan perubahan kondisi daun |
| Ambang keyakinan deteksi | 0,5 | Evaluasi kurva precision-recall model |
| Aturan DEFISIENSI_TIDAK_TERKONFIRMASI (FR-05) | Tanpa dosing otomatis bila EC dan pH normal | Konfirmasi penulis dan kelompok |
| Batas aman setpoint | pH 4,5 hingga 8,0; EC 0,5 hingga 4,0 mS/cm | Referensi budidaya tanaman yang dipakai |
| Aturan dosing | Jeda 30 menit; berhenti setelah 3 siklus tanpa perbaikan | Uji meja dan kalibrasi pompa |
| Target mutu model | mAP@0,5 minimal 0,90; ambang retraining 0,85 | Evaluasi model pada data uji |
| Waktu inferensi Raspberry Pi 5 | Maksimal 500 ms per bingkai | Pengukuran saat perangkat tersedia |
| Ketersediaan, RTO, RPO | 99% per bulan; RTO 4 jam; RPO 24 jam | Tinjauan dengan dosen pengampu |
| Pengguna bersamaan, retensi data | 5 pengguna; sensor 12 bulan; gambar tanpa label 90 hari | Tinjauan kebutuhan penulis |
| SUS dan onboarding | SUS 75; 15 menit | Uji penggunaan dengan responden |
| Beban non-prioritas saat daya cadangan | Ditangguhkan saat baterai di bawah 30% | Uji pemadaman dan desain kelistrikan |
| Pembanding intervensi manusia | Sistem yang sudah ada pada BPMN AS-IS (deteksi sehat atau rusak, dosing otomatis, daya cadangan baterai atau aki, penanganan manual) | Dicocokkan dengan BPMN AS-IS dan dicatat selama uji lapangan |
| Faktor konversi TDS ke EC | 0,5 (EC mS/cm = TDS ppm x 0,5 / 1000) | Kalibrasi dengan larutan standar EC dan pencocokan dengan datasheet sensor |
| Peningkatan sensor EC | Sensor target mampu membaca 0 sampai 4,0 mS/cm; prototipe SEN-0171 sampai 2,0 mS/cm | Uji kalibrasi pada rentang penuh saat sensor target tersedia |
