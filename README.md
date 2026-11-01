# PCD Assignment 3: Segmentasi Citra dengan Operasi Morfologi

Eksperimen pengolahan citra digital yang membandingkan dua metode segmentasi, yaitu **Otsu thresholding** dan **region growing**, serta menguji pengaruh operasi morfologi dasar (erosi, dilasi, *opening*, dan *closing*) terhadap kualitas masker biner yang dihasilkan.

## Tujuan

1. Membandingkan karakteristik segmentasi berbasis ambang global (Otsu) dan berbasis wilayah (*region growing*).
2. Mengevaluasi pengaruh operasi morfologi terhadap derau latar, lubang internal objek, dan objek yang saling bersentuhan.
3. Menentukan operasi morfologi yang paling sesuai untuk tiap jenis masker.

## Dataset

Tiga citra uji dengan karakteristik berbeda:

| Citra | Objek | Karakteristik |
|---|---|---|
| Gambar 1 | Koin | Permukaan logam berrelief dan pantulan cahaya |
| Gambar 2 | Biji kopi | Tekstur, gradasi warna, dan biji yang saling berdempetan |
| Gambar 3 | Daun di atas aspal | Latar bertekstur kasar yang memicu derau |

Citra dibaca dalam ruang warna RGB, lalu dikonversi ke *grayscale* 8-bit (rentang 0-255).

## Metodologi

### 1. Segmentasi

**Otsu thresholding.** Ambang global *T* dipilih dengan meminimalkan *within-class variance*. Untuk Gambar 1 dan 2 (objek lebih gelap daripada latar) aturan binarisasi diinversi, sedangkan Gambar 3 memakai aturan standar sehingga objek selalu menjadi *foreground*.

**Region growing.** *Seed* ditempatkan otomatis pada area inti objek menggunakan ambang kuantil ketat dari *T* Otsu, dengan ketetanggaan 8-arah. Piksel tetangga digabungkan jika memenuhi |Ip - Iq| < T. Nilai T ditentukan adaptif sebesar satu simpangan baku intensitas kelas objek hasil partisi Otsu, dengan batas tambahan |I_seed - Iq| < 1,8 x simpangan baku.

### 2. Operasi Morfologi

Operasi diterapkan pada masker biner hasil kedua metode segmentasi:

- **Erosi**: menyusutkan objek, menghapus derau kecil, tetapi memperbesar lubang.
- **Dilasi**: memperbesar objek dan menutup lubang kecil, tetapi turut membesarkan derau.
- **Opening** (erosi lalu dilasi): menghapus derau kecil dengan ukuran objek tetap terjaga.
- **Closing** (dilasi lalu erosi): menutup lubang dan celah dengan distorsi ukuran lebih kecil daripada dilasi.

Ukuran *structuring element* (SE) pada gambar hasil: **7x7**.

## Hasil Utama

### Parameter segmentasi

| Citra | T Otsu | T Region Growing | Jumlah seed |
|---|---|---|---|
| Gambar 1 (koin) | 156 | 30,14 | 133 |
| Gambar 2 (biji kopi) | 149 | 35,88 | 160 |
| Gambar 3 (daun) | 97 | 25,52 | 59 |

### Jumlah piksel *foreground*

| Citra | Metode | Tanpa morfologi | Erosi | Dilasi | Opening | Closing |
|---|---|---|---|---|---|---|
| Gambar 1 | Otsu | 139.696 | 75.996 | 159.222 | 132.838 | 153.902 |
| Gambar 1 | RG | 115.273 | 29.708 | 155.791 | 81.816 | 146.391 |
| Gambar 2 | Otsu | 153.496 | 97.710 | 178.720 | 150.154 | 158.410 |
| Gambar 2 | RG | 114.337 | 29.125 | 157.797 | 88.469 | 134.331 |
| Gambar 3 | Otsu | 62.175 | 33.123 | 143.240 | 50.870 | 72.574 |
| Gambar 3 | RG | 47.971 | 18.291 | 58.233 | 39.465 | 53.577 |

### Temuan

- **Otsu** menghasilkan masker yang padat, tetapi memuat derau latar pada citra daun (aspal bertekstur).
- **Region growing** memisahkan objek dari latar dengan bersih, tetapi masker berlubang pada permukaan bertekstur dan sangat bergantung pada sebaran *seed*.
- **Erosi** efektif membuang derau, tetapi sangat merusak masker yang sudah terfragmentasi (terutama RG).
- **Dilasi** menutup lubang, tetapi membesarkan derau dan menyatukan objek yang berdekatan.
- ***Opening*** paling cocok untuk masker berderau seperti hasil Otsu pada citra daun.
- ***Closing*** paling cocok untuk masker bersih yang berlubang seperti hasil RG.
