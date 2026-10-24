# Use Case Specification: GrowSense

| Informasi | Keterangan |
| --- | --- |
| Project name | GrowSense: Smart Hydroponic NFT Monitoring System |
| Date | 21 Oktober 2026 |
| Version | 1.2 (3 use case utama) |
| By | Kelompok 4 |
| Mata kuliah | Analisis dan Desain Perangkat Lunak (ADPL) |
| Template | Basic Use Case Template, A. Cockburn (HaT TR96.03a), format tabel |
| Dokumen terkait | APLB_Kelompok4_GrowSense_SRS.pdf (versi 1.3) |

**Riwayat revisi**

| Version | Deskripsi | Tanggal |
| --- | --- | --- |
| 1.1 | Pembatasan menjadi 3 use case utama. | 11 Oktober 2026 |
| 1.2 | Penambahan FR-13 pada UC-01 dan FR-02 pada UC-02, penyeragaman jeda antar-dosing dengan SRS FR-06 (30 menit, usulan), dan penyelarasan dengan SRS versi 1.2. | 21 Oktober 2026 |

## 1 Dasar Pemilihan Use Case

Sesuai arahan dosen, dokumen ini memuat maksimal tiga use case utama. Ketiganya dipilih karena mewakili tiga hal yang membedakan GrowSense dan mendukung tujuan sistem berjalan mandiri dengan intervensi manusia sesedikit mungkin.

| No | Use case utama | Alasan dipilih |
| --- | --- | --- |
| UC-01 | Memantau Kondisi Larutan dan Lingkungan Tanaman | Fondasi seluruh sistem: data sensor tepercaya yang menjadi masukan keputusan dan dilihat pemilik. |
| UC-02 | Mendiagnosis dan Menangani Kesehatan Tanaman | Inti inovasi: deteksi YOLOv8 12 kelas, distingsi patologi dari defisiensi, dan dosing nutrisi adaptif. |
| UC-03 | Mempertahankan Operasi Saat Listrik Padam | Pembeda versi individu: cadangan daya panel surya agar pompa NFT tidak berhenti. |

## 2 Aktor

| Aktor | Jenis | Peran |
| --- | --- | --- |
| Pemilik (Owner) | Primary | Memantau, mengatur, dan mengontrol sistem; menyetujui rekomendasi AI. |
| Viewer | Secondary (opsional) | Dosen atau peneliti dengan akses lihat saja. |
| Pewaktu dan pengatur daya | Sistem (pemicu) | Memicu pembacaan sensor, pengambilan gambar, dan peralihan daya. |
| Decision layer | Sistem | Menetapkan status kesehatan dan memutuskan dosing, penguncian, atau notifikasi. |
| ESP32-S3, sensor, aktuator | Secondary | Membaca sensor dan menjalankan perintah pompa serta grow light. |
| Unit inferensi YOLOv8 dan kamera | Secondary | Mengambil gambar dan mendeteksi kesehatan daun. |
| Basis data, broker MQTT, layanan notifikasi | Secondary | Menyimpan data, meneruskan pesan, dan mengirim notifikasi. |

## 3 Daftar Use Case Utama

| No | Use case (tujuan) | Primary actor | Level | Prioritas |
| --- | --- | --- | --- | --- |
| UC-01 | Memantau Kondisi Larutan dan Lingkungan Tanaman | Pemilik | Primary task | Top |
| UC-02 | Mendiagnosis dan Menangani Kesehatan Tanaman | Pemilik | Primary task | Top |
| UC-03 | Mempertahankan Operasi Saat Listrik Padam | Pemilik | Primary task | Top |

## 4 Spesifikasi Use Case

### UC-01 Memantau Kondisi Larutan dan Lingkungan Tanaman

| Elemen | Isi |
| --- | --- |
| Goal in Context | Pemilik mengetahui kondisi larutan nutrisi, lingkungan, aktuator, dan daya terkini beserta riwayatnya tanpa harus hadir di lokasi, dengan data yang tepercaya. |
| Scope & Level | GrowSense, Primary task |
| Preconditions | ESP32-S3 dan sensor menyala dan terhubung ke broker MQTT; server dan basis data aktif; pemilik sudah memiliki akun. |
| Success End Condition | Pembacaan valid tersimpan dan pemilik melihat nilai terkini, grafik, status aktuator, dan status daya. |
| Failed End Condition | Data tidak tersedia; dashboard menampilkan penanda "data tidak terbaru" atau status offline, dan pemilik mendapat notifikasi. |
| Primary, Secondary Actors | Primary: Pemilik (peran Owner; Viewer hanya melihat). Secondary: Pewaktu pada ESP32-S3, sensor, broker MQTT, basis data PostgreSQL. |
| Trigger | Interval pembacaan tercapai (1 menit atau 10 menit); pemilik membuka dashboard. |

**Description (Main Success Scenario)**

1. Pewaktu pada ESP32-S3 memicu pembacaan sesuai interval.
2. ESP32-S3 membaca pH, TDS/EC, suhu air, level air, suhu udara, kelembapan, dan cahaya.
3. ESP32-S3 memvalidasi rentang nilai tiap sensor.
4. ESP32-S3 mempublikasikan pesan telemetri JSON ke broker MQTT.
5. Server memvalidasi pesan dan menyimpannya ke basis data dengan cap waktu UTC.
6. Pemilik membuka dashboard web atau aplikasi.
7. Sistem menampilkan nilai terkini, status aktuator, status daya, dan hasil deteksi daun terbaru.
8. Pemilik memilih rentang waktu grafik (1 jam, 24 jam, 7 hari, atau 30 hari).
9. Sistem menampilkan grafik historis sesuai rentang.

**Extensions**

| Langkah | Percabangan |
| --- | --- |
| 3a | Nilai di luar rentang atau tidak terbaca: sistem menandai sensor ERROR, menghentikan aksi otomatis yang bergantung pada sensor itu (misalnya dosing), dan mengirim notifikasi; pembacaan lain tetap diproses. |
| 4a | Koneksi ke broker terputus: ESP32-S3 mengulang pengiriman pada siklus berikutnya. |
| 5a | Pesan tidak sesuai skema: server menolak pesan dan mencatat galat. |
| 6a | Pemilik belum login: sistem menampilkan form login; kredensial salah menampilkan pesan galat; 3 kali gagal berturut-turut menunda login 5 menit. |
| 7a | Data lebih lama dari dua kali interval pembacaan: dashboard menandai "data tidak terbaru" beserta waktu pembacaan terakhir. |
| 7b | Perangkat tidak terhubung: dashboard menampilkan status offline dan sistem mengirim notifikasi. |
| 9a | Tidak ada data pada rentang yang dipilih: sistem menampilkan pesan data kosong. |

**Sub-variations**

| Langkah | Variasi |
| --- | --- |
| 2 | Interval 1 menit untuk pH, TDS/EC, suhu air, dan level air; 10 menit untuk suhu udara, kelembapan, dan cahaya (usulan). |
| 6 | Akses melalui web atau aplikasi; peran Owner (kontrol penuh) atau Viewer (hanya melihat). |

**Related Information**

| Elemen | Isi |
| --- | --- |
| Priority | Top |
| Performance | Data tersimpan maksimal 5 detik setelah pembacaan; dashboard tampil maksimal 3 detik. |
| Frequency | Pembacaan 24/7 tiap 1 atau 10 menit; dashboard dibuka beberapa kali per hari. |
| Channels to actors | Interaktif (web/aplikasi); MQTT ke perangkat; basis data. |
| Requirement (SRS) | FR-01, FR-02, FR-12, FR-13, FR-14 (sebagian: grafik historis), FR-16, FR-17; REL-07, OTH-02 |
| Superordinates | - |
| Subordinates | Login (langkah 6a) dan penanganan sensor error (langkah 3a) dispesifikasikan sebagai bagian use case ini. |

**Open Issues**

- Sensor mana yang berinterval 1 menit dan mana yang 10 menit perlu dikonfirmasi.
- Apakah dashboard menampilkan gambar langsung atau hanya gambar terakhir?
- Apakah Viewer (dosen/peneliti) perlu dibatasi pada data tertentu?

Due Date: ditentukan per iterasi (SDLC Iterative and Incremental).

---

### UC-02 Mendiagnosis dan Menangani Kesehatan Tanaman

| Elemen | Isi |
| --- | --- |
| Goal in Context | Sistem mendeteksi kondisi daun, membedakan penyakit/hama dari defisiensi nutrisi dengan bantuan data sensor, lalu menanganinya (dosing adaptif atau penguncian dosing) sesuai mode operasi, dengan intervensi manusia sesedikit mungkin. |
| Scope & Level | GrowSense, Primary task |
| Preconditions | Kamera dan unit inferensi aktif dengan model YOLOv8 termuat; setpoint aktif tersedia (diatur pemilik per fase); mode operasi sudah dipilih (Manual, Setengah AI, atau Penuh AI); sensor pH, EC, dan level air valid. |
| Success End Condition | Kondisi daun terklasifikasi; tindakan sesuai status (pertahankan setpoint, dosing, atau penguncian) dilakukan atau direkomendasikan; gambar, hasil, dan log tersimpan; pemilik diberi tahu bila perlu. |
| Failed End Condition | Tidak ada tindakan otomatis yang diambil; kegagalan dicatat dan pemilik diberi tahu; dosing tidak pernah berjalan tanpa data yang valid. |
| Primary, Secondary Actors | Primary: Pemilik (pemangku kepentingan utama; sistem bertindak atas nama pemilik). Secondary: Pewaktu, kamera, unit inferensi YOLOv8, decision layer, ESP32-S3 dan pompa peristaltik, basis data, layanan notifikasi. |
| Trigger | Jadwal pengambilan gambar tercapai (bawaan tiap 30 menit). |

**Description (Main Success Scenario)**

1. Kamera mengambil gambar daun (JPEG, minimal 640 x 640 piksel).
2. Sistem mengirim gambar ke unit inferensi.
3. YOLOv8 mendeteksi tiap daun dan mengklasifikasikannya ke 12 kelas (8 penyakit/hama, 3 defisiensi N/P/K, 1 sehat) beserta tingkat keyakinan.
4. Sistem menyimpan gambar dan hasil deteksi.
5. Decision layer menggabungkan hasil deteksi dengan pembacaan terakhir pH, EC, dan level air.
6. Decision layer menetapkan status DEFISIENSI_NPK (kelas defisiensi terdeteksi dan EC di bawah batas bawah setpoint atau pH di luar rentang).
7. Decision layer memeriksa bahwa dosing tidak terkunci, level air cukup, dan sensor valid.
8. Decision layer menghitung dosis dari selisih terhadap setpoint, dibatasi dosis maksimum per siklus.
9. Sesuai mode, sistem melanjutkan: Penuh AI langsung eksekusi; Setengah AI mengirim rekomendasi dan menunggu persetujuan pemilik.
10. Sistem mengirim perintah dosing melalui MQTT dan ESP32-S3 mengaktifkan pompa peristaltik selama durasi dosis.
11. Setelah jeda pencampuran, sistem membaca ulang pH dan EC.
12. Sistem mencatat dosing dan mengirim notifikasi hasil kepada pemilik.

**Extensions**

| Langkah | Percabangan |
| --- | --- |
| 2a | Unit inferensi tidak tersedia: sistem memberi tahu pemilik, hanya mempertahankan setpoint, dan tidak melakukan dosing berbasis deteksi; pengambilan diulang pada jadwal berikutnya. |
| 3a | Tidak ada daun terdeteksi: sistem mencatat dan use case berakhir sampai jadwal berikutnya. |
| 3b | Keyakinan di bawah 0,5 (usulan): deteksi ditandai "perlu tinjauan", tidak ada tindakan otomatis, dan notifikasi tinjauan dikirim. |
| 6a | Status PATOLOGI (kelas penyakit/hama di atas ambang; diutamakan bila bersamaan dengan defisiensi): sistem mengunci dosing, menyimpan gambar sebagai bukti, dan mengirim notifikasi anjuran isolasi tanaman; dashboard menampilkan "Dosing terkunci". Pemilik menangani tanaman dan mengonfirmasi, lalu sistem membuka kunci dan mencatat. Bila pemilik tidak merespons, pengingat dikirim tiap 30 menit dan dosing tetap terkunci. Use case berakhir. |
| 6b | Status DEFISIENSI_TIDAK_TERKONFIRMASI (EC dan pH normal): sistem mengirim notifikasi tinjauan tanpa dosing otomatis. Use case berakhir. |
| 6c | Status SEHAT: sistem mempertahankan setpoint dan, bila pH atau EC di luar rentang, melakukan koreksi sesuai mode (kecuali dosing terkunci). Use case berakhir. |
| 7a | Sensor pH, EC, atau level air error: sistem membatalkan dosing, menandai sensor, dan mengirim notifikasi. |
| 7b | Dosing sedang terkunci atau level air rendah: dosing ditolak dan pemilik diberi tahu alasannya. |
| 9a | Mode Manual: sistem hanya mengirim notifikasi dan rekomendasi; tidak ada dosing otomatis. |
| 9b | Mode Setengah AI dan pemilik menolak atau tidak menyetujui dalam 30 menit (usulan): dosing dibatalkan dan dicatat. |
| 10a | ESP32-S3 tidak membalas dalam 5 detik: sistem menandai perintah gagal dan memberi tahu pemilik. |
| 11a | Nilai tidak mendekati setpoint setelah 3 siklus dosing (usulan): sistem menghentikan dosing berulang dan memberi tahu pemilik. |

**Sub-variations**

| Langkah | Variasi |
| --- | --- |
| 2 | Inferensi berjalan di laptop (saat ini), Raspberry Pi 5 (lokasi jauh dari server), atau server (lokasi dekat). |
| 9 | Mode operasi: Manual, Setengah keputusan AI, atau Penuh keputusan AI. Pemilik dapat mengambil alih kontrol aktuator secara manual kapan saja. Setpoint: diatur manual per jenis tanaman dan fase; penentuan fase otomatis pada tahap lanjutan. |

**Related Information**

| Elemen | Isi |
| --- | --- |
| Priority | Top |
| Performance | Inferensi maksimal 500 ms per bingkai; keputusan dosing maksimal 10 detik setelah deteksi; notifikasi maksimal 30 detik setelah kejadian; siklus gambar hingga keputusan maksimal 1 menit. |
| Frequency | Tiap 30 menit secara bawaan, 24/7. |
| Channels to actors | Jaringan lokal (kamera, unit inferensi, server); MQTT ke perangkat; web/aplikasi untuk notifikasi dan persetujuan. |
| Requirement (SRS) | FR-02, FR-03, FR-04, FR-05, FR-06, FR-07, FR-08, FR-10, FR-13, FR-16; REL-06, REL-08, SEC-06 |
| Superordinates | - |
| Subordinates | Pengaturan setpoint (FR-09), kontrol manual (FR-11), dan pembuatan dataset (FR-18) adalah fungsi pendukung yang dispesifikasikan langsung pada SRS. |

**Open Issues**

- Interval, jumlah, dan posisi kamera belum ditentukan.
- Ambang keyakinan 0,5 perlu ditetapkan setelah evaluasi model.
- Rumus dosis dan dosis maksimum per siklus belum ditetapkan. Jeda antar-dosing 30 menit (usulan, sesuai SRS FR-06) perlu divalidasi pada uji meja.
- Daftar 8 kelas penyakit/hama beserta saran tindakan tiap kelas belum disusun.
- Syarat pembukaan kunci dosing selain konfirmasi pemilik.
- Aturan status DEFISIENSI_TIDAK_TERKONFIRMASI perlu disetujui kelompok.

Due Date: ditentukan per iterasi (SDLC Iterative and Incremental).

---

### UC-03 Mempertahankan Operasi Saat Listrik Padam

| Elemen | Isi |
| --- | --- |
| Goal in Context | Pompa sirkulasi dan grow light tetap menyala saat listrik utama padam agar tanaman pada sistem NFT tidak rusak, sementara pemilik tetap diberi tahu. |
| Scope & Level | GrowSense (termasuk subsistem daya), Primary task |
| Preconditions | Panel surya, baterai, dan pengatur daya terpasang dan berfungsi; baterai terisi; ESP32-S3, pompa sirkulasi, dan grow light tersambung ke jalur daya cadangan. |
| Success End Condition | Beban prioritas tetap menyala minimal 8 jam (target 12 jam) sampai listrik utama pulih; pemilik mendapat notifikasi; sistem kembali ke daya utama. |
| Failed End Condition | Beban prioritas mati; kejadian dicatat dan pemilik diberi tahu secepatnya begitu jaringan atau daya tersedia. |
| Primary, Secondary Actors | Primary: Pemilik (pemangku kepentingan; use case dipicu oleh pengatur daya). Secondary: Pengatur daya, panel surya, baterai, ESP32-S3, layanan notifikasi. |
| Trigger | Listrik utama padam. |

**Description (Main Success Scenario)**

1. Pengatur daya mendeteksi listrik utama padam.
2. Sistem beralih ke panel surya dan baterai tanpa me-restart ESP32-S3.
3. Sistem mempertahankan beban prioritas: pompa sirkulasi, grow light, dan ESP32-S3 (karena mengendalikan relay).
4. Sistem mengirim notifikasi peralihan daya kepada pemilik.
5. Sistem memantau kapasitas baterai dan keluaran panel surya.
6. Listrik utama pulih dan sistem kembali ke daya utama.
7. Sistem mengirim notifikasi pemulihan dan mencatat kejadian (durasi dan kapasitas baterai).

**Extensions**

| Langkah | Percabangan |
| --- | --- |
| *a | Koneksi ke broker atau server terputus pada langkah mana pun: ESP32-S3 mempertahankan pompa sirkulasi dan jadwal grow light terakhir dan menghentikan dosing otomatis sampai koneksi pulih (fail-safe). |
| 4a | Jaringan ikut padam sehingga notifikasi tidak terkirim: beban prioritas tetap berjalan secara lokal; notifikasi dikirim setelah koneksi pulih. |
| 5a | Kapasitas baterai di bawah 30% (usulan): sistem menangguhkan beban non-prioritas (kamera dan inferensi) dan mengirim peringatan baterai rendah. |
| 5b | Baterai habis sebelum listrik utama pulih: beban mati; sistem mencatat kejadian dan memberi tahu pemilik begitu daya kembali. |

**Sub-variations**

| Langkah | Variasi |
| --- | --- |
| 5 | Sumber daya: baterai saja (malam atau mendung) atau panel surya ditambah baterai (siang). |
| 3 | Beban non-prioritas (kamera, unit inferensi) ditangguhkan atau dipertahankan sesuai kapasitas baterai. |

**Related Information**

| Elemen | Isi |
| --- | --- |
| Priority | Top |
| Performance | Beban prioritas menyala minimal 8 jam (target 12 jam); ESP32-S3 tidak restart saat peralihan; notifikasi peralihan maksimal 30 detik bila jaringan tersedia. |
| Frequency | Saat terjadi pemadaman (jarang, tetapi kritis). |
| Channels to actors | Perangkat keras daya; MQTT; notifikasi web/aplikasi. |
| Requirement (SRS) | FR-15, FR-13; REL-02, REL-05, OTH-03 |
| Superordinates | - |
| Subordinates | - |

**Open Issues**

- Kapasitas panel surya (Wp), baterai (Ah), pengatur pengisian, dan konverter belum ditentukan.
- Apakah grow light benar-benar dipertahankan penuh selama 8 hingga 12 jam? Konsumsi dayanya mungkin dominan.
- Router dan jaringan lokal juga butuh daya agar notifikasi tetap terkirim.
- Nasib sensor non-prioritas selama daya cadangan belum ditetapkan.

Due Date: ditentukan per iterasi (SDLC Iterative and Incremental).

## 5 Cakupan terhadap SRS

Tiga use case di atas mencakup sebagian besar kebutuhan fungsional. Fungsi lain dispesifikasikan langsung pada Bab 2 SRS (atribut dan kriteria penerimaan) dan tidak diuraikan sebagai use case tersendiri.

| Use case | Kebutuhan SRS yang dicakup |
| --- | --- |
| UC-01 | FR-01, FR-02, FR-12, FR-13, FR-14 (sebagian: grafik historis), FR-16, FR-17 |
| UC-02 | FR-02, FR-03, FR-04, FR-05, FR-06, FR-07, FR-08, FR-10, FR-13, FR-16 |
| UC-03 | FR-13, FR-15 (ditambah REL-02 dan REL-05) |
| Fungsi pendukung (langsung di SRS) | FR-09 setpoint per fase (prasyarat UC-02), FR-11 kontrol manual (pengambilalihan pada UC-02), FR-14 riwayat dan unduhan (selain grafik), FR-18 dataset pelatihan ulang, FR-19 fase otomatis, FR-20 akses data terbuka |
