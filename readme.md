# Database and Big Data Pipeline Part 1
## Hands-on: Database & Data Pipeline Fundamentals for AI

**Applied AI Engineer Bootcamp**  
**Topik:** OLTP dan OLAP, desain skema, serta layering data  
**Peserta:** IT dan non-IT dengan basic coding  
**Durasi:** kelas 180 menit, termasuk challenge hands-on 60 menit  
**Tools:** PostgreSQL dan pgAdmin

Selamat datang di praktikum Database & Data Pipeline Fundamentals for AI.

Dalam praktikum ini, Anda akan menyiapkan data untuk sebuah aplikasi AI yang memperkirakan keterlambatan pengiriman. Anda akan membaca relasi tabel, menemukan masalah kualitas data, membentuk layer raw–cleansed–curated, dan menghasilkan dataset yang memisahkan fitur dari label.

Kita menggunakan kasus toko fiktif **KirimKita**. Tim AI ingin membuat prediksi **saat checkout selesai**, ketika nilai pesanan, kota tujuan, dan janji pengiriman sudah diketahui. Hasil pengiriman aktual baru tersedia kemudian.

Dataset berukuran kecil agar setiap perubahan dapat ditelusuri. Seluruh data bersifat sintetis. Praktikum ini membangun fondasi untuk pipeline data berskala lebih besar; distributed processing dan pelatihan model berada di sesi lanjutan.

## Tujuan Pembelajaran

Setelah menyelesaikan praktikum ini, Anda mampu:

1. **Membedakan kebutuhan OLTP dan OLAP:** menjelaskan perbedaan antara menyimpan transaksi pesanan dan menyiapkan data untuk analisis atau AI.
2. **Membaca dan melengkapi desain skema:** menentukan grain, primary key, foreign key, dan cardinality pada tabel pelanggan, pesanan, dan pengiriman.
3. **Menerapkan layering data:** menjaga raw, membersihkan data melalui cleansed, dan menyusun dataset sesuai kebutuhan pada curated.
4. **Memeriksa kualitas data:** menemukan duplikasi, referensi tidak valid, nilai kosong, nilai negatif, dan tanggal yang tidak logis.
5. **Menyiapkan dataset AI tanpa kebocoran outcome:** memilih fitur yang tersedia saat prediksi, membentuk label, dan memisahkan baris yang belum memiliki label valid.

## Pengaturan Environment & Tools

Praktikum menggunakan dua tools:

| Tool | Fungsi |
|---|---|
| **PostgreSQL 17** | Menyimpan tabel sumber dan menjalankan transformasi serta pemeriksaan SQL. |
| **pgAdmin 4** | Menulis, menjalankan, dan menyimpan query melalui Query Tool. |

Jika kelas sudah menggunakan editor SQL lain, misalnya DBeaver, Anda dapat memakainya sebagai pengganti pgAdmin. Cukup gunakan satu editor.

**Informasi penting mengenai berkas praktikum**

- Setup, data awal, demo, dan pemeriksaan SQL sudah tersedia.
- Anda mengerjakan query di `workspace/queries.sql` dan penjelasan di `workspace/jawaban.md`.
- Semua tugas pemrograman menggunakan SQL. Tidak ada dependensi Python atau package manager.
- Desain relasi cukup dibaca dari tabel di panduan ini. Tidak diperlukan aplikasi diagram tambahan.
- Instalasi dan pemeriksaan koneksi dilakukan **sebelum kelas**, dengan perkiraan waktu 30–45 menit. Challenge kelas berlangsung 60 menit.

### Alokasi sesi tiga jam

| Bagian | Durasi |
|---|---:|
| Konteks AI dan OLTP vs OLAP | 35 menit |
| Desain skema dan relasi | 25 menit |
| Layering dan kualitas data | 15 menit |
| Istirahat | 10 menit |
| Demo SQL terpandu | 15 menit |
| Challenge hands-on | 60 menit |
| Review dan kuis | 20 menit |
| **Total** | **180 menit** |

## Hands-On-Lab

### Langkah 1: Verifikasi Prasyarat Instalasi Tools

#### 1.1 PostgreSQL

Panitia dapat menyediakan database kelas atau menggunakan PostgreSQL lokal yang sudah terpasang. **Pilih salah satu jalur**, sesuai persiapan kelas.

- **Database kelas:** gunakan host, port, database, username, dan password dari panitia. Setiap peserta atau pasangan memperoleh database latihan sendiri.
- **Database lokal:** instal PostgreSQL sebelum kelas melalui [distribusi resmi PostgreSQL](https://www.postgresql.org/download/). Simpan username, password, dan port yang dipilih saat instalasi.

Praktikum menggunakan sintaks PostgreSQL dan telah diuji pada PostgreSQL 17. Pastikan layanan database berjalan sebelum membuka koneksi.

#### 1.2 pgAdmin

Buka pgAdmin yang telah terpasang. Jika belum tersedia, gunakan [installer resmi pgAdmin](https://www.pgadmin.org/download/). pgAdmin merupakan aplikasi klien; membuka pgAdmin saja tidak otomatis menyediakan server PostgreSQL.

Di panel **Servers**, pilih server yang sudah tersedia. Jika belum ada, gunakan **Register → Server**, lalu isi:

| Parameter | PostgreSQL lokal | Database kelas |
|---|---|---|
| Name | `AI Engineer Lab` | Nama bebas untuk mengenali koneksi |
| Host | `localhost` | Dari panitia |
| Port | `5432`, atau port instalasi Anda | Dari panitia |
| Maintenance database | `postgres` | Dari panitia |
| Username | Akun yang dibuat saat instalasi | Dari panitia |
| Password | Password akun PostgreSQL Anda | Dari panitia |

Klik **Save**, lalu pastikan daftar database dapat dibuka. Jangan menyalin password orang lain atau mengasumsikan password tertentu.

### Langkah 2: Siapkan Database Latihan

Jika panitia sudah menyediakan database, pilih database tersebut dan lanjutkan ke Langkah 3.

Untuk PostgreSQL lokal, klik kanan **Databases → Create → Database**, lalu beri nama:

```text
aien_pipeline_lab
```

Alternatif bagi pengguna yang sudah memahami SQL: buka Query Tool pada database `postgres`, lalu jalankan perintah berikut sebagai satu perintah di luar transaksi:

```sql
CREATE DATABASE aien_pipeline_lab;
```

Jika nama database sudah ada, gunakan nama baru untuk latihan ini. Akun yang membuat database memerlukan hak `CREATEDB`; minta bantuan instruktur bila tidak memilikinya.

Klik kanan database latihan yang baru dibuat, lalu pilih **Query Tool**. Jalankan:

```sql
SELECT current_database(), current_user, version();
```

**Hasil yang diharapkan:** nama database latihan Anda, username PostgreSQL Anda, dan informasi versi PostgreSQL.

**Mengapa langkah ini penting?** Query selalu berjalan pada database yang terhubung ke tab Query Tool. Memilih database di panel kiri belum tentu mengubah koneksi tab yang sudah terbuka.

### Langkah 3: Inisialisasi Skema dan Dataset

Ekstrak paket praktikum. Di Query Tool yang terhubung ke database latihan, buka berkas:

```text
initial/init.sql
```

Jalankan seluruh berkas sebagai satu script. Jangan hanya menjalankan baris di posisi kursor. Tunggu sampai transaksi selesai dan hasil jumlah tabel muncul.

#### Yang terjadi di belakang layar

1. Script membuat schema `lab_raw`, `lab_cleansed`, dan `lab_curated` jika belum tersedia.
2. Script membuat tabel sumber `customers`, `orders`, dan `shipments` pada `lab_raw`.
3. Script memuat dataset menggunakan `INSERT`, sehingga Anda tidak perlu mengimpor CSV secara manual.
4. Script menampilkan jumlah baris awal untuk pemeriksaan.

**Perilaku saat diulang:** script mengosongkan dan memuat ulang **tiga tabel `lab_raw`** pada database yang sedang terhubung. Jalankan hanya pada database latihan. Script tidak menghapus database atau schema lainnya.

#### Hasil yang diharapkan

| Tabel | Jumlah baris |
|---|---:|
| customers | 6 |
| orders | 22 |
| shipments | 18 |

Tabel orders memiliki **20 ID unik**. Dua baris tambahan merupakan salinan identik yang sengaja dimasukkan untuk latihan deduplikasi.

Raw sudah memakai tipe `date` dan `numeric`. Validasi parsing file sumber belum menjadi tugas pada sesi ini.

### Langkah 4: Pemeriksaan Koneksi dan Kesehatan Dataset

Buka dan jalankan:

```text
starter/check_connection.sql
```

Script menampilkan database aktif serta jumlah baris sumber. Pastikan hasilnya sesuai Langkah 3.

Untuk melihat contoh data:

```sql
SELECT *
FROM lab_raw.orders
ORDER BY order_id
LIMIT 5;
```

**Penjelasan:** `SELECT` menentukan kolom, `FROM` menentukan tabel, `ORDER BY` mengurutkan hasil, dan `LIMIT` membatasi baris yang ditampilkan.

### Langkah 5: Cara Menjalankan Latihan

1. Buka `starter/demo.sql` dan ikuti demonstrasi instruktur per blok query.
2. Buka `workspace/queries.sql`, lalu simpan salinan kerja jika diperlukan.
3. Ganti token seperti `__KOLOM_ID__` dengan SQL yang sesuai. Token tersebut adalah ruang jawaban, bukan sintaks SQL.
4. Sorot satu blok query lengkap sampai tanda `;`, lalu jalankan seleksi tersebut.
5. Selesaikan blok secara berurutan. View pada tahap berikutnya bergantung pada view tahap sebelumnya.
6. Catat hasil dan alasan keputusan pada `workspace/jawaban.md`. File Markdown ini dapat diedit sebagai teks biasa.

**Jangan menjalankan seluruh template saat masih terdapat token yang belum diisi.**

### Troubleshooting Isu Umum

| Gejala | Penyebab yang mungkin | Langkah perbaikan |
|---|---|---|
| Connection refused | Layanan belum aktif, host/port tidak sesuai | Periksa layanan PostgreSQL dan parameter koneksi. |
| Password authentication failed | Username atau password tidak sesuai | Gunakan kredensial instalasi atau konfirmasi kepada panitia. |
| Permission denied | Akun tidak memiliki hak pada database latihan | Minta hak yang diperlukan kepada instruktur. |
| Relation does not exist | Setup belum dijalankan atau koneksi salah | Periksa `current_database()`, jalankan setup, lalu ikuti urutan view. |
| Syntax error dekat `__...__` | Masih ada placeholder | Isi token, kemudian jalankan ulang blok terkait. |
| Current transaction is aborted | Ada error dalam transaksi | Jalankan `ROLLBACK;`, perbaiki, lalu ulang transaksi tersebut. |
| Data awal berbeda | Raw sudah berubah atau database berbeda | Periksa koneksi. Muat ulang `initial/init.sql` pada database latihan jika memang ingin kembali ke data awal. |

## Layout Direktori Praktikum

```text
Hands_On_Database_AI_Part_1/
├── readme.md                      # Materi dalam format Markdown
├── readme.html                    # Materi untuk dibaca di browser
├── initial/
│   └── init.sql                   # Setup schema dan data awal
├── starter/
│   ├── check_connection.sql       # Pemeriksaan koneksi dan jumlah sumber
│   └── demo.sql                   # Query demonstrasi instruktur
├── workspace/
│   ├── queries.sql                # WORKSPACE PESERTA: query yang dilengkapi
│   └── jawaban.md                 # Jawaban desain dan refleksi
├── data/
│   ├── customers.csv              # Salinan dataset untuk inspeksi
│   ├── orders.csv
│   └── shipments.csv
├── test/
│   ├── check_pipeline.sql         # Pemeriksaan hasil PASS / FAIL
│   └── assert_pipeline.sql        # Pemeriksaan yang berhenti jika gagal
├── solution/                      # Hanya pada paket lengkap instruktur
│   ├── queries.sql                # Solusi lengkap dengan quarantine
│   ├── template_terisi.sql         # Kunci token template peserta
│   ├── checkpoint_cleansed.sql     # Bantuan saat peserta tertinggal
│   ├── tantangan_opsional.sql
│   └── expected/                  # CSV hasil referensi
└── instruktur/                    # Hanya pada paket lengkap instruktur
    └── panduan.md                 # Rundown, kunci, dan rubrik
```

CSV adalah salinan dataset yang sama dengan isi script setup. Mengimpor CSV lagi setelah setup akan berisiko menggandakan data.

## Kontrak Data dan Kamus Kolom

Snapshot latihan bertanggal **31 Januari 2026**. Semua tanggal memakai kalender lokal yang sama. Satu pesanan pada lab memiliki maksimal satu pengiriman.

| Tabel | Kolom | Makna |
|---|---|---|
| customers | customer_id | Identitas pelanggan. |
| customers | segment | Kategori pelanggan, retail atau business setelah cleaning. |
| orders | order_id | Identitas pesanan. |
| orders | customer_id | Referensi pelanggan yang melakukan order. |
| orders | order_date | Tanggal checkout, menjadi acuan waktu prediksi. |
| orders | promised_date | Tanggal janji tiba yang diketahui saat checkout. |
| orders | order_value | Nilai pesanan dalam rupiah, harus positif. |
| orders | destination_city | Kota tujuan yang dicatat saat checkout. |
| shipments | shipment_id | Identitas pengiriman. |
| shipments | order_id | Referensi pesanan, unik pada shipments dalam lab. |
| shipments | delivered_date | Tanggal tiba aktual, dapat NULL jika belum tercatat. |

**Asumsi waktu:** segment tidak berubah selama periode latihan. Pada sistem nyata, perubahan segment atau janji pengiriman memerlukan data historis yang sesuai waktu prediksi.

**Aturan target:** `is_late = 1` jika tanggal tiba aktual melewati tanggal janji. Tiba tepat pada tanggal janji menghasilkan `is_late = 0`. Label hanya dibuat ketika tanggal tiba valid dan sudah diketahui.

## Konsep Guided Hands-on

### Konsep 1: OLTP, OLAP, dan Kebutuhan AI

**OLTP** mendukung aktivitas operasional seperti menyimpan order baru atau memperbarui satu pengiriman. **OLAP** mendukung analisis data, misalnya merangkum tingkat keterlambatan per kota.

| Aktivitas KirimKita | Kebutuhan |
|---|---|
| Pelanggan menyelesaikan checkout | Menyimpan transaksi dengan konsisten. |
| Tim operasi memperbarui status pengiriman | Mengubah record tertentu. |
| Analis membandingkan keterlambatan antarkota | Membaca dan merangkum banyak transaksi. |
| Tim AI menyiapkan dataset training | Menggabungkan histori pada grain yang sesuai. |

Dalam lab, satu PostgreSQL melayani kedua contoh pola penggunaan. Istilah OLTP dan OLAP menjelaskan beban kerja, bukan nama produk database.

### Konsep 2: Grain, Relasi, dan Desain Skema

**Grain** menjelaskan arti satu baris. Sebelum menulis JOIN, tentukan grain tiap tabel.

| Tabel | Grain | Primary key pada model bisnis | Foreign key pada model bisnis |
|---|---|---|---|
| customers | Satu pelanggan | customer_id | Tidak ada pada contoh ini |
| orders | Satu pesanan | order_id | customer_id ke customers |
| shipments | Satu pengiriman | shipment_id | order_id ke orders |

Relasi customer ke orders adalah **satu ke banyak**. Relasi order ke shipments pada lab adalah **satu ke nol atau satu**.

Primary key mengenali baris secara unik dan tidak NULL. Foreign key menyatakan referensi ke key tabel lain. Raw sengaja tidak memaksakan constraint bisnis agar masalah data dapat diperiksa; key pada tabel di atas adalah **desain yang diharapkan**, bukan constraint fisik pada raw.

#### Normalisasi dan denormalisasi

Normalisasi memisahkan entitas untuk mengurangi pengulangan atribut dan inkonsistensi. Atribut pelanggan disimpan di customers, sementara kejadian order disimpan di orders.

Untuk kebutuhan AI, kita menggabungkan atribut yang relevan menjadi satu dataset. Bentuk denormalisasi ini memudahkan konsumsi data, tetapi pipeline tetap harus menjaga grain dan konsistensi hasil.

Contoh baris hasil yang dituju:

| order_id | order_value | destination_city | segment | promised_days |
|---|---:|---|---|---:|
| O001 | 112500 | BANDUNG | retail | 3 |

#### Mengapa JOIN dapat menggandakan baris?

Jika satu order cocok dengan dua record di tabel kanan, JOIN menghasilkan dua baris untuk order tersebut. Dalam model dengan beberapa paket per order, tentukan aturan agregasi pengiriman sebelum membentuk dataset satu baris per order.

`DISTINCT` tidak menyelesaikan semua konflik: dua baris dengan ID sama dan isi berbeda tetap merupakan dua baris berbeda.

### Konsep 3: Layer Raw, Cleansed, dan Curated

| Layer | Tanggung jawab | Implementasi lab |
|---|---|---|
| Raw | Menyimpan sumber, termasuk masalahnya | Tabel pada `lab_raw` |
| Cleansed | Menyeragamkan dan menerapkan aturan kualitas | View pada `lab_cleansed` |
| Curated | Menyusun data sesuai kebutuhan AI | View pada `lab_curated` |

**Schema** merupakan namespace pengelompokan objek database. **View** menyimpan definisi query; view biasa tidak membuat salinan fisik data. Pembagian ini mengajarkan layer secara logis dalam satu database.

Serving menyajikan hasil kepada aplikasi atau konsumen. Archive menyimpan histori sesuai kebijakan retensi. Keduanya dibahas secara konseptual pada sesi ini dan tidak memerlukan tool atau implementasi tambahan.

#### Kebijakan cleaning pada lab

| Masalah | Kebijakan |
|---|---|
| Salinan order identik | Pertahankan satu baris pada cleansed. |
| Customer tidak dikenal | Keluarkan dari dataset siap pakai, pertahankan raw untuk investigasi. |
| order_value nol atau negatif | Keluarkan dari dataset siap pakai. |
| Kota memakai casing/spasi berbeda | Terapkan TRIM lalu UPPER. |
| Kota kosong | Gunakan kategori UNKNOWN. |
| Tanggal tiba sebelum tanggal order | Keluarkan record pengiriman dari hasil bersih. |
| Tanggal tiba NULL | Pertahankan sebagai outcome belum diketahui, jangan buat label. |

Pada pembahasan, instruktur memperlihatkan **quarantine**, yaitu kumpulan record bermasalah beserta alasan penolakannya. Jangan mengubah nilai negatif atau tanggal invalid dengan tebakan.

### Konsep 4: SQL untuk Memeriksa dan Membersihkan Data

#### 4.1 Menghitung baris dan ID unik

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT order_id) AS unique_orders
FROM lab_raw.orders;
```

Hasil `22` dan `20` menunjukkan ada pengulangan order_id. Selisih ini belum cukup untuk menentukan cara memperbaiki konflik; periksa isi baris yang berulang.

#### 4.2 GROUP BY dan HAVING

```sql
SELECT order_id, COUNT(*) AS jumlah
FROM lab_raw.orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

`GROUP BY` membentuk kelompok per ID, sedangkan `HAVING` memilih kelompok yang jumlahnya melebihi satu. Berbeda dari `WHERE` yang memfilter baris sebelum agregasi.

#### 4.3 LEFT JOIN dan NULL

```sql
SELECT o.order_id, c.customer_id
FROM lab_raw.orders o
LEFT JOIN lab_raw.customers c
    ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

LEFT JOIN mempertahankan baris kiri yang tidak cocok. NULL pada kolom kanan membantu menemukan referensi customer yang hilang. Gunakan `IS NULL`, bukan `= NULL`.

#### 4.4 Normalisasi teks

```sql
SELECT COALESCE(NULLIF(UPPER(TRIM('  bandung  ')), ''), 'UNKNOWN');
```

Hasilnya `BANDUNG`.

```sql
SELECT COALESCE(NULLIF(UPPER(TRIM('   ')), ''), 'UNKNOWN');
```

Hasilnya `UNKNOWN`.

`TRIM` menghapus spasi tepi, `UPPER` menyeragamkan huruf, `NULLIF` mengubah string kosong menjadi NULL, dan `COALESCE` memberikan nilai pengganti. Rangkaian ini belum menyelesaikan typo atau alias nama kota.

#### 4.5 Perhitungan tanggal dan kondisi

```sql
SELECT DATE '2026-01-04' - DATE '2026-01-01' AS promised_days;
```

Hasilnya `3`. `CASE WHEN ... THEN ... ELSE ... END` digunakan untuk menerjemahkan aturan terlambat menjadi label angka.

### Konsep 5: Fitur, Label, dan Data Leakage

Fitur merupakan masukan untuk prediksi. Label merupakan outcome yang ingin diprediksi. Pemilihan fitur harus mengikuti **waktu prediksi**, bukan sekadar ketersediaan kolom pada dataset historis.

| Kolom | Peran |
|---|---|
| order_value | Calon fitur yang diketahui saat checkout. |
| destination_city | Calon fitur berupa snapshot tujuan saat checkout. |
| segment | Calon fitur, dengan asumsi tetap selama lab. |
| promised_days | Calon fitur dari janji yang diketahui saat checkout. |
| order_id | Metadata untuk pelacakan. |
| order_date | Metadata untuk waktu dan pembagian dataset. |
| delivered_date | Outcome untuk membentuk label, tidak masuk fitur checkout. |
| is_late | Label target. |

Memakai tanggal tiba aktual untuk prediksi saat checkout membocorkan informasi dari masa setelah prediksi. Dataset dapat terlihat sangat mudah diprediksi tetapi gagal ketika aplikasi menerima order baru.

Tanggal tiba NULL tidak berarti pengiriman tepat waktu. Baris tersebut belum memiliki label valid dan masuk `pending_review`. Baris ini juga dapat mencakup outcome yang bermasalah, sehingga tidak otomatis berarti semua pesanan masih aktif dikirim.

## Challenge Hands-on SQL

Buka `workspace/queries.sql`. Selesaikan lima challenge berikut dalam **60 menit**. Huruf A–E pada template sesuai dengan Challenge 1–5.

### Challenge 1: Desain Skema dan Kebutuhan Data — 8 Menit

**Tugas:** isi bagian A pada `workspace/jawaban.md`.

1. Tentukan grain ketiga tabel.
2. Tandai primary key dan foreign key.
3. Jelaskan cardinality customers–orders dan orders–shipments.
4. Tuliskan satu aktivitas OLTP dan satu aktivitas OLAP dari kasus ini.
5. Jelaskan perubahan desain jika satu order dapat dikirim dalam beberapa paket.

**Output:** penjelasan skema dan relasi. Tidak perlu membuat diagram menggunakan aplikasi lain.

### Challenge 2: Profiling dan Diagnosis Data — 10 Menit

**Tugas:** lengkapi blok B pada template.

1. Bandingkan jumlah baris dengan jumlah order_id unik.
2. Temukan ID yang muncul lebih dari sekali.
3. Temukan order yang tidak memiliki pasangan customer.
4. Tambahkan query untuk order_value tidak positif.
5. Gabungkan orders dan shipments untuk menemukan tanggal tiba sebelum tanggal order.

**Checkpoint:** 22 baris raw dan 20 order_id unik. Catat ID bermasalah di bagian B lembar jawaban.

**Pertanyaan refleksi:** mengapa menemukan duplikasi key belum tentu berarti semua baris duplikat boleh dihapus sembarang?

### Challenge 3: Membentuk Layer Cleansed — 17 Menit

**Tugas:** lengkapi blok C dan jalankan berurutan: customers, orders, lalu shipments.

- Seragamkan segment dengan `LOWER(TRIM(...))`.
- Gunakan deduplikasi untuk salinan order identik.
- Pertahankan order dengan customer valid, nilai positif, dan tanggal janji yang valid.
- Seragamkan kota dan tangani string kosong.
- Pertahankan pengiriman dengan tanggal valid atau NULL.

**Checkpoint:**

| Hasil | Target |
|---|---:|
| lab_cleansed.orders | 18 baris |
| lab_cleansed.shipments | 17 baris |
| Order dengan kota UNKNOWN | 1 baris |

**Pertanyaan refleksi:** mengapa kota kosong bisa diberi kategori UNKNOWN, sedangkan tanggal tiba kosong tidak boleh diubah menjadi label 0?

Jika tertinggal, minta instruktur memberikan checkpoint cleansed, lalu lanjutkan Challenge 4 sendiri.

### Challenge 4: Menyusun Dataset Curated untuk AI — 15 Menit

**Tugas:** lengkapi blok D.

1. Buat `order_features` dari order dan customer bersih.
2. Hitung `promised_days` dari selisih tanggal janji dan checkout.
3. Buat `order_labels` dari pengiriman yang valid dan memiliki tanggal tiba.
4. Isi `is_late` dengan 1 jika pengiriman melewati janji, selain itu 0.
5. Gabungkan fitur dan label menjadi `training_dataset`.
6. Pisahkan fitur tanpa label valid ke `pending_review`.

**Checkpoint:** features 18 baris, training 15 baris, dan pending 3 baris.

**Contoh hasil training:**

| order_id | order_value | destination_city | promised_days | is_late |
|---|---:|---|---:|---:|
| O001 | 112500 | BANDUNG | 3 | 0 |
| O003 | 137500 | JAKARTA | 2 | 1 |

Tabel contoh menampilkan sebagian kolom. Hasil lengkap juga mencakup order_date dan segment.

### Challenge 5: Validasi dan Penjelasan Keputusan — 10 Menit

**Tugas:** buka `test/check_pipeline.sql` dan jalankan seluruh script setelah semua view tersedia.

1. Periksa bahwa seluruh status bernilai PASS.
2. Pastikan satu order hanya memiliki satu baris pada features.
3. Jelaskan mengapa delivered_date tidak masuk fitur.
4. Jelaskan mengapa tiga baris pending belum boleh masuk dataset training ini.
5. Simpan query jawaban dan lengkapi refleksi pada lembar jawaban.

**Target minimum:** semua pemeriksaan lulus dan Anda dapat menjelaskan alasan pemisahan fitur, label, dan pending.

## Verifikasi & Pemeriksaan Solusi

### Hasil pemeriksaan yang diharapkan

| Pemeriksaan | Target |
|---|---:|
| raw_orders | 22 |
| raw_unique_orders | 20 |
| clean_orders | 18 |
| clean_shipments | 17 |
| features | 18 |
| training | 15 |
| late | 5 |
| pending | 3 |
| duplicate_features | 0 |
| unknown_city | 1 |
| null_labels | 0 |
| nonpositive_features | 0 |
| outcome_columns_in_features | 0 |

Script menampilkan kolom `pemeriksaan`, `aktual`, `target`, dan `status`. Pemeriksaan tersebut mencakup aturan inti lab, tetapi tidak membuktikan pipeline sudah lengkap untuk produksi. Pemeriksaan nama kolom outcome juga perlu dilengkapi peninjauan logika query untuk mencegah leakage yang berganti nama.

Untuk pemeriksaan yang berhenti saat menemukan ketidaksesuaian, jalankan `test/assert_pipeline.sql`. Jika lulus, PostgreSQL menyelesaikan blok `DO` tanpa error. Ini tidak memerlukan Python.

### Membandingkan dengan solusi

Setelah menyimpan pekerjaan Anda, instruktur dapat membagikan:

- `solution/template_terisi.sql` untuk melihat pengisian token.
- `solution/queries.sql` untuk solusi lengkap, termasuk view quarantine.
- `solution/expected/` untuk hasil CSV referensi.

Menjalankan solusi lengkap **mengganti definisi view latihan** pada database aktif. Simpan SQL jawaban sebelum melakukannya. Jalankan setup, solusi, lalu pemeriksaan bila ingin melakukan walkthrough referensi dari awal.

### Output yang dikumpulkan

1. `queries_NAMA.sql`: SQL jawaban peserta.
2. `jawaban_NAMA.md`: desain skema, temuan profiling, dan penjelasan keputusan.
3. Hasil `test/check_pipeline.sql`, diekspor sebagai CSV atau disimpan dari hasil query.
4. `training_dataset.csv`, diekspor dari query berikut melalui fitur **Save results to file** pada Data Output pgAdmin:

```sql
SELECT *
FROM lab_curated.training_dataset
ORDER BY order_id;
```

Sertakan header kolom pada ekspor CSV.

## Challenge Tambahan — Opsional, di Luar 60 Menit

- Buat view quarantine order dengan alasan customer tidak dikenal atau nilai tidak positif.
- Hitung jumlah order dan tingkat keterlambatan per kota. Sertakan ukuran sampel setiap kota.
- Pisahkan training_dataset berdasarkan order_date: sampai 10 Januari dan sesudah 10 Januari. Diskusikan alasan menggunakan waktu dalam evaluasi prediksi masa depan.
- Jelaskan aturan agregasi yang diperlukan jika satu order memiliki beberapa shipment.

Dataset hanya memiliki 15 baris berlabel, sehingga hasil kelompok tidak cukup untuk menarik kesimpulan bisnis atau menilai akurasi model nyata.

## Instruksi Pembersihan

1. Simpan query dan ekspor hasil yang diperlukan.
2. Tutup tab Query Tool dan putuskan koneksi server di pgAdmin bila sudah selesai.
3. Database latihan boleh tetap disimpan untuk mengulang materi. Tidak perlu menghentikan layanan PostgreSQL yang juga dipakai pekerjaan lain.
4. Jika panitia meminta database dihapus, pastikan hasil sudah tersimpan dan nama database benar, lalu lakukan penghapusan melalui instruktur. Tidak ada perintah penghapusan otomatis dalam paket ini.

Selamat! Anda telah menyelesaikan praktikum Database & Data Pipeline Fundamentals for AI. Anda memiliki SQL yang membentuk dataset, pemeriksaan kualitas, dan alasan pemilihan fitur yang sesuai waktu prediksi.

## Referensi

- [PostgreSQL 17: JOIN](https://www.postgresql.org/docs/17/tutorial-join.html)
- [PostgreSQL 17: CASE, COALESCE, dan NULLIF](https://www.postgresql.org/docs/17/functions-conditional.html)
- [PostgreSQL: constraints dan key](https://www.postgresql.org/docs/18/ddl-constraints.html)
- [PostgreSQL: schema](https://www.postgresql.org/docs/18/ddl-schemas.html)
- [pgAdmin: Query Tool](https://www.pgadmin.org/docs/pgadmin4/latest/query_tool.html)
- [scikit-learn: data leakage](https://scikit-learn.org/stable/common_pitfalls.html#data-leakage)

Contoh data, aturan kasus, dan angka checkpoint merupakan materi sintetis untuk praktikum ini. Penamaan layer merupakan konvensi kelas yang dapat berbeda dari implementasi organisasi.
