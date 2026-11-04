# Laporan Proyek Tugas 2 Bagian B: Analisis Komparatif Algoritma Image Compression dan Benchmarking Kuantitatif

### Identitas Pengerjaan:
- **Mata Kuliah**: Pengolahan Citra Digital
- **Topik**: Kajian 10 Algoritma Kompresi Citra, Implementasi 3 Teknik Mandiri, dan Benchmarking Multi-Format
- **Penyusun**: Iman Dwi Satrio, Ahmad Zaidan
- **Tautan Repositori GitHub**: [tugas2-pengolahan-citra](https://github.com/imandwisatrio/tugas2-pengolahan-citra)
- **Berkas Notebook Terkait**: [`notebooks/02_image_compression.ipynb`](../notebooks/02_image_compression.ipynb)

---

## Daftar Isi Laporan
1. [Pendahuluan dan Latar Belakang](#1-pendahuluan-dan-latar-belakang)
2. [Kajian Teoretis Komprehensif 10 Teknik Image Compression](#2-kajian-teoretis-komprehensif-10-teknik-image-compression)
   - [Tabel Komparasi Ilmiah 10 Teknik Kompresi](#tabel-komparasi-ilmiah-10-teknik-kompresi)
3. [Kajian Khusus: Perlakuan & Rasterisasi Citra Vektor (SVG)](#3-kajian-khusus-perlakuan--rasterisasi-citra-vektor-svg)
4. [Metodologi dan Implementasi 3 Teknik Terpilih](#4-metodologi-dan-implementasi-3-teknik-terpilih)
5. [Tabel Hasil Benchmarking Kuantitatif Multi-Format](#5-tabel-hasil-benchmarking-kuantitatif-multi-format)
6. [Visualisasi Analitis Hasil Eksperimen](#6-visualisasi-analitis-hasil-eksperimen)
7. [Pembahasan Mendalam dan Analisis Trade-off](#7-pembahasan-mendalam-dan-analisis-trade-off)
8. [Kesimpulan dan Rekomendasi Format Industri](#8-kesimpulan-dan-rekomendasi-format-industri)
9. [Daftar Pustaka](#9-daftar-pustaka)

---

## 1. Pendahuluan dan Latar Belakang

Kompresi citra digital (*image compression*) merupakan proses reduksi jumlah bit yang diperlukan untuk merepresentasikan citra digital dengan meminimalkan redundansi data tanpa menurunkan kualitas informasi visual melampaui batas ambang yang dapat diterima. Redundansi pada citra mencakup:
1. **Redundansi Spasial/Korelasional**: Korelasi erat antar piksel yang bertetangga.
2. **Redundansi Spektral**: Korelasi antar pita frekuensi/kanal warna (misal RGB ke YCbCr).
3. **Redundansi Psikovisual**: Ketidakpekaan sistem visual manusia (*Human Visual System* / HVS) terhadap perubahan frekuensi spasial tinggi dan fluktuasi krominansi.
4. **Redundansi Entropi/Pengkodean**: Simbol-simbol dengan frekuensi kemunculan tinggi yang menggunakan panjang bit yang sama dengan simbol langka.

Kebutuhan akan efisiensi bandwidth transmisi jaringan dan efisiensi media penyimpanan menjadikan kompresi citra sebagai tulang punggung infrastruktur web, penginderaan jauh, rekam medis digital, dan aplikasi multimedia modern.

---

## 2. Kajian Teoretis Komprehensif 10 Teknik Image Compression

Agar mekanisme setiap algoritma mudah dipahami, kita dapat menggunakan analogi sederhana:

```
+----------------------------------------------------------------------------------+
|               ANALOGI SEDERHANA 10 TEKNIK IMAGE COMPRESSION                      |
+-----------------------------------+----------------------------------------------+
| 1. RLE                            | Seperti mencatat "100 piksel putih" alih-    |
|                                   | alih menulis kata "putih" 100 kali.          |
+-----------------------------------+----------------------------------------------+
| 2. Huffman Coding                 | Seperti alfabet Morse: huruf sering (E, A)   |
|                                   | diberi kode pendek, huruf langka (Z, Q)      |
|                                   | diberi kode panjang.                         |
+-----------------------------------+----------------------------------------------+
| 3. Arithmetic Coding              | Seperti merangkum seluruh kalimat buku ke    |
|                                   | dalam satu titik koordinat desimal di garis  |
|                                   | bilangan antara 0.0 dan 1.0.                 |
+-----------------------------------+----------------------------------------------+
| 4. LZW                            | Seperti membuat kamus singkatan gaul: kata-  |
|                                   | kata yang sering muncul diberi kode angka.   |
+-----------------------------------+----------------------------------------------+
| 5. DEFLATE (PNG)                  | Gabungan LZW dan Huffman: cari pola kalimat  |
|                                   | berulang lalu kompres frekuensi kodenya.     |
+-----------------------------------+----------------------------------------------+
| 6. JPEG (DCT)                     | Membagi gambar menjadi nada rendah (warna),  |
|                                   | sedang (pola), dan tinggi (noise/detail).    |
|                                   | Nada tinggi dibuang karena mata tak sadar.   |
+-----------------------------------+----------------------------------------------+
| 7. JPEG 2000 (DWT)                | Memperhalus gambar dengan gelombang wavelet; |
|                                   | kualitas turun anggun tanpa artefak kotak.   |
+-----------------------------------+----------------------------------------------+
| 8. WebP (VP8/VP8L)                | Memprediksi piksel tetangga lalu hanya       |
|                                   | mencatat sisa perbedaannya saja.             |
+-----------------------------------+----------------------------------------------+
| 9. Vector Quantization            | Seperti menyediakan palet cat terbatas;      |
|                                   | blok piksel dicocokkan ke warna terdekat.    |
+-----------------------------------+----------------------------------------------+
| 10. Fractal (IFS)                 | Menganggap gambar tersusun dari pola self-   |
|                                   | similar (seperti daun pakis atau kembang kol)|
|                                   | lalu hanya mencatat rumus fraktalnya.        |
+-----------------------------------+----------------------------------------------+
```

### Rincian 10 Algoritma:
1. **Run-Length Encoding (RLE)**: Menggantikan sekuens simbol berulang yang berurutan (*run*) menjadi pasangan $(v, L)$. Sangat efisien untuk gambar grafik dengan warna blok seragam (misal BMP, TIFF, PCX), namun bengkak pada foto natural bertekstur tajam. *Ref: Robinson & Cherry (1967), DOI: 10.1109/PROC.1967.5493*.
2. **Huffman Coding**: Pengkodean prefiks entropi optimal berdasar probabilitas kemunculan simbol menggunakan pohon biner biner Huffman. Simbol paling sering mendapat bit terpendek. *Ref: Huffman (1952), DOI: 10.1109/JRPROC.1952.273898*.
3. **Arithmetic Coding**: Merepresentasikan seluruh rangkaian aliran simbol ke dalam satu bilangan interval pecahan tunggal pada garis $[0, 1)$. Mampu mendekati batas teoritis Entropi Shannon secara presisi ($< 1$ bit/simbol). *Ref: Rissanen & Langdon (1979), DOI: 10.1147/rd.232.0149*.
4. **Lempel-Ziv-Welch (LZW)**: Algoritma kamus dinamis yang membangun leksikon pola string secara adaptif tanpa perlu mengirim tabel kamus terpisah. Menjadi fondasi format GIF dan TIFF. *Ref: Welch (1984), DOI: 10.1109/MC.1984.1659158*.
5. **DEFLATE (LZ77 + Huffman)**: Tulang punggung format PNG, mengombinasikan pencarian string referensi mundur (*backward distance-length*) LZ77 dengan kompresi entropi Huffman terbobot. *Ref: Deutsch (1996), RFC 1951, IETF*.
6. **JPEG (DCT-based)**: Mengonversi citra ke ruang $YC_bC_r$, mempartisi kanal luminansi ke blok $8 \times 8$, menerapkan 2D-DCT, membagi dengan matriks kuantisasi psikovisual, lalu mengurutkan zig-zag dan mengkodekan via RLE + Huffman. Menguasai web selama tiga dekade. *Ref: Wallace (1991), DOI: 10.1109/30.89988*.
7. **JPEG 2000 (DWT-based)**: Menggunakan dekomposisi piramidal *Discrete Wavelet Transform* (DWT CDF 9/7 untuk lossy, CDF 5/3 untuk lossless) dan pengkodean bit-plane EBCOT. Mengeliminasi efek *blocking* menjadi keburaman lembut (*graceful degradation*). *Ref: Skodras et al. (2001), DOI: 10.1109/79.952802*.
8. **WebP (VP8/VP8L)**: Dikembangkan Google berbasis codec VP8, menggunakan prediksi spasial intra-frame dari blok tetangga kiri dan atas sebelum mengompresi selisih residual menggunakan DCT modifikasi (lossy) atau transform ensemble LZ77 (lossless). Menghasilkan ukuran 25-34% lebih kecil dibanding JPEG pada kualitas setara. *Ref: Google WebP Specification (2010)*.
9. **Vector Quantization (VQ)**: Mengelompokkan vektor blok citra (misal $4 \times 4$) dan memetakannya ke indeks kode (*codevector*) terdekat dalam *codebook* optimal berdasar algoritma Linde-Buzo-Gray (LBG). Dekompresi instan hanya dengan pembacaan tabel (*table lookup*). *Ref: Gray (1984), DOI: 10.1109/MASSP.1984.1162229*.
10. **Fractal Compression (IFS)**: Memanfaatkan redundansi *self-similarity* antar-skala pada citra alami berdasar Teorema Kolase Banach. Citra didekomposisi ke blok domain besar dan dicocokkan ke blok *range* kecil melalui transformasi afin kontraktif. Waktu encoding sangat lambat namun dekompresi instan (*asymmetric compression*). *Ref: Barnsley & Hurd (1993), ISBN: 978-0120790661; Jacquin (1992), DOI: 10.1109/5.127835*.

---

### Tabel Komparasi Ilmiah 10 Teknik Kompresi

| No | Nama Teknik | Jenis Kompresi | Prinsip / Cara Kerja Utama | Rasio Kompresi Tipikal | Kualitas Hasil Rekonstruksi | Kompleksitas Komputasi | Contoh Format / Penggunaan Praktis | Referensi Kunci (DOI / ISBN) |
|:--:|:---|:---:|:---|:---:|:---:|:---:|:---|:---|
| **1** | **Run-Length Encoding (RLE)** | Lossless | Menghitung deret run nilai identik berurutan | $1.2:1 - 3:1$ (Grafik) | Bit-exact ($PSNR = \infty, SSIM = 1.0$) | O(N), Memori Minimal | BMP, PCX, TGA, Masking Mask | Robinson & Cherry (1967), DOI: [10.1109/PROC.1967.5493](https://doi.org/10.1109/PROC.1967.5493) |
| **2** | **Huffman Coding** | Lossless | Pengkodean entropi berbasis pohon frekuensi simbol | $1.5:1 - 2.5:1$ | Bit-exact ($PSNR = \infty, SSIM = 1.0$) | Waktu: O(N log N); Memori: O(K) | Tahap akhir JPEG, MP3, Gzip | Huffman (1952), DOI: [10.1109/JRPROC.1952.273898](https://doi.org/10.1109/JRPROC.1952.273898) |
| **3** | **Arithmetic Coding** | Lossless | Memetakan untaian simbol ke interval pecahan $[0, 1)$ | $1.8:1 - 3:1$ | Bit-exact ($PSNR = \infty, SSIM = 1.0$) | Waktu & Memori: Moderat | JPEG 2000, JBIG, H.264 (CABAC) | Rissanen & Langdon (1979), DOI: [10.1147/rd.232.0149](https://doi.org/10.1147/rd.232.0149) |
| **4** | **Lempel-Ziv-Welch (LZW)** | Lossless | Kamus dinamis menggantikan deret karakter dengan indeks | $1.5:1 - 4:1$ | Bit-exact ($PSNR = \infty, SSIM = 1.0$) | Waktu: Cepat; Memori: Moderat (4096 entri) | GIF, TIFF, UNIX compress | Welch (1984), DOI: [10.1109/MC.1984.1659158](https://doi.org/10.1109/MC.1984.1659158) |
| **5** | **DEFLATE (PNG)** | Lossless | Kombinasi pencarian string LZ77 dan Huffman terbobot | $2:1 - 5:1$ | Bit-exact ($PSNR = \infty, SSIM = 1.0$) | Waktu: Sedang; Memori: 32KB window | Portable Network Graphics (PNG), ZIP | Deutsch (1996), RFC 1951, IETF |
| **6** | **JPEG (DCT-based)** | Lossy | Partisi blok $8 \times 8$, transformasi 2D-DCT, kuantisasi | $10:1 - 30:1$ | Terkendali ($PSNR \approx 30 - 45\text{ dB}$) | Waktu: Sangat Cepat; Memori: Rendah | Standar Web, Kamera Digital, JPEG | Wallace (1991), DOI: [10.1109/30.89988](https://doi.org/10.1109/30.89988) |
| **7** | **JPEG 2000 (DWT)** | Hybrid (Lossless/Lossy) | Dekomposisi multi-resolusi DWT dan kuantisasi EBCOT | $20:1 - 50:1$ | Unggul, tanpa blocking artifact | Waktu: Berat; Memori: Tinggi | Arsip Medis DICOM, Sinema Digital DCI | Skodras et al. (2001), DOI: [10.1109/79.952802](https://doi.org/10.1109/79.952802) |
| **8** | **WebP (VP8/VP8L)** | Hybrid (Lossless/Lossy) | Prediksi intra-frame, transformasi DCT modifikasi / LZ77 | $15:1 - 40:1$ | Sangat Baik (PSNR > JPEG pada bit serupa) | Waktu: Sedang-Berat; Memori: Moderat | Standar Web Modern (Google Chrome/Android) | Google WebP Docs (2010), Chromium |
| **9** | **Vector Quantization (VQ)**| Lossy | Pemetaan blok piksel ke codevector dalam codebook LBG | $10:1 - 25:1$ | Rentan distorsi tepi (*edge degradation*) | Encoding: Sangat Berat; Decoding: Cepat | Video conferencing lama, Indeo Video | Gray (1984), DOI: [10.1109/MASSP.1984.1162229](https://doi.org/10.1109/MASSP.1984.1162229) |
| **10**| **Fractal (IFS)** | Lossy | Pemetaan kontraktif self-similarity antar-skala citra | $20:1 - 80:1$ | Halus pada perbesaran tinggi, resolusi independen | Encoding: Sangat Ekstrem; Decoding: Cepat | Microsoft Encarta, Riset Pola Alam | Jacquin (1992), DOI: [10.1109/5.127835](https://doi.org/10.1109/5.127835) |

---

## 3. Kajian Khusus: Perlakuan & Rasterisasi Citra Vektor (SVG)

Berkas **Scalable Vector Graphics (SVG)** merupakan representasi berbasis instruksi geometri matematis XML murni (garis, lingkaran, kurva kurva Bézier, warna isi). Karakteristik utamanya:
- **Ukuran Asli Sangat Kecil**: Berkas teks XML sumber hanya berukuran $\approx 894\text{ Bytes}$.
- **Kemandirian Resolusi**: Dapat diperbesar tanpa batas skala (*infinite zoom*) tanpa pecah.

Untuk memproses SVG menggunakan algoritma kompresi citra raster (RLE, JPEG, WebP), SVG **wajib dirasterisasi** terlebih dahulu ke matriks piksel diskrit 2D ($512 \times 512 \times 3$, `uint8`).
Akibatnya:
- Ukuran data mentah melonjak secara masif dari $894\text{ B}$ menjadi **$786.432\text{ Bytes}$ ($768\text{ KB}$)**.
- Ketika hasil kompresi (misal JPEG $\approx 13\text{ KB}$) dibandingkan terhadap berkas teks SVG asli ($0.89\text{ KB}$), Compression Ratio tampak $< 1.0$ (bengkak). Hal ini merupakan konsekuensi wajar dari konversi domain instruksi vektor ke representasi raster diskrit.
- Oleh karena itu, rasio kompresi objektif dievaluasi terhadap **baseline raster $512 \times 512$ murni ($768\text{ KB}$)**.

---

## 4. Metodologi dan Implementasi 3 Teknik Terpilih

Tiga algoritma kompresi citra diimplementasikan secara mandiri dan modular:
1. **Teknik 1: Lossless RLE (Run-Length Encoding)**:
   - Matriks citra 3D diratakan (*flattened*) per kanal warna.
   - Algoritma menelusuri piksel berurutan dan merekam tuple `(nilai_piksel, panjang_run)`.
   - Panjang run dienkode dalam format biner kompak `uint8` (jika run $> 255$, dipartisi ke potongan berikutnya).
2. **Teknik 2: Lossy JPEG DCT (Discrete Cosine Transform)**:
   - Konversi RGB ke $YC_bC_r$.
   - Partisi kanal ke blok-blok berukuran $8 \times 8$.
   - Transformasi 2D-DCT menggunakan basis cosinus ortonormal.
   - Kuantisasi menggunakan Matriks Standar Kuantisasi Luminansi JPEG dengan parameter kualitas $Q = 50$.
   - Pengurutan koefisien via pola zig-zag, diakhiri dengan kompresi entropi.
3. **Teknik 3: State-of-the-Art WebP (Predictive Coding)**:
   - Menggunakan modul kompresi WebP berbasis intra-prediction VP8 dengan parameter kualitas $Q = 50$.

---

## 5. Tabel Hasil Benchmarking Kuantitatif Multi-Format

Pengujian dilakukan terhadap 5 berkas citra uji beresolusi $512 \times 512$: `sample.bmp`, `sample.jpg`, `sample.png`, `sample.svg` (dirasterisasi), dan citra fotografi wajah riil `real_sample.png`.

Formulasi metrik kuantitatif:
- **Compression Ratio (CR)**: $\text{CR} = \frac{\text{Ukuran Asli (Bytes)}}{\text{Ukuran Terkompresi (Bytes)}}$
- **Space Saving (SS)**: $\text{SS} = \left(1 - \frac{\text{Ukuran Terkompresi}}{\text{Ukuran Asli}}\right) \times 100\%$
- **Peak Signal-to-Noise Ratio (PSNR)**: $\text{PSNR} = 10 \log_{10}\left(\frac{255^2}{\text{MSE}}\right)\text{ dB}$
- **Structural Similarity Index (SSIM)**: Mengukur kesamaan struktur, luminansi, dan kontras lokal.

### Tabel Rekapitulasi Hasil Eksekusi:

| Format Input | Teknik Kompresi | Ukuran Asli | Ukuran Kompresi | Compression Ratio | Space Saving (%) | PSNR (dB) | SSIM | Waktu Kompresi (ms) |
|:---|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **BMP** | **RLE (Lossless)** | 768.05 KB | 387.89 KB | **1.98x** | 49.50% | $\infty$ | 1.0000 | 12.4 ms |
| **BMP** | **JPEG DCT (Q=50)** | 768.05 KB | 17.65 KB | **43.52x** | 97.70% | 40.12 dB | 0.9678 | 4.8 ms |
| **BMP** | **WebP (Q=50)** | 768.05 KB | 2.71 KB | **283.41x** | 99.65% | 41.85 dB | 0.9782 | 8.2 ms |
| **PNG** | **RLE (Lossless)** | 45.21 KB | 387.89 KB | 0.12x (bengkak) | -757.96% | $\infty$ | 1.0000 | 11.9 ms |
| **PNG** | **JPEG DCT (Q=50)** | 45.21 KB | 17.65 KB | **2.56x** | 60.96% | 40.12 dB | 0.9678 | 4.6 ms |
| **PNG** | **WebP (Q=50)** | 45.21 KB | 2.71 KB | **16.68x** | 94.01% | 41.85 dB | 0.9782 | 7.9 ms |
| **JPEG** | **RLE (Lossless)** | 28.45 KB | 391.12 KB | 0.07x (bengkak) | -1274.76% | $\infty$ | 1.0000 | 12.1 ms |
| **JPEG** | **JPEG DCT (Q=50)** | 28.45 KB | 17.65 KB | **1.61x** | 37.96% | 40.12 dB | 0.9678 | 4.5 ms |
| **JPEG** | **WebP (Q=50)** | 28.45 KB | 2.71 KB | **10.50x** | 90.47% | 41.85 dB | 0.9782 | 8.1 ms |
| **SVG** *(Raster 512x512)* | **RLE (Lossless)** | 768.00 KB | 12.14 KB | **63.26x** | 98.42% | $\infty$ | 1.0000 | 8.5 ms |
| **SVG** *(Raster 512x512)* | **JPEG DCT (Q=50)** | 768.00 KB | 13.82 KB | **55.57x** | 98.20% | 42.65 dB | 0.9812 | 4.4 ms |
| **SVG** *(Raster 512x512)* | **WebP (Q=50)** | 768.00 KB | 2.54 KB | **302.36x** | 99.67% | 43.10 dB | 0.9850 | 7.6 ms |
| **Real DAiSEE** | **RLE (Lossless)** | 148.20 KB | 312.45 KB | 0.47x (bengkak) | -110.83% | $\infty$ | 1.0000 | 13.2 ms |
| **Real DAiSEE** | **JPEG DCT (Q=50)** | 148.20 KB | 18.25 KB | **8.12x** | 87.69% | 38.45 dB | 0.9412 | 4.9 ms |
| **Real DAiSEE** | **WebP (Q=50)** | 148.20 KB | 6.85 KB | **21.64x** | 95.38% | 39.80 dB | 0.9585 | 8.8 ms |

---

## 6. Visualisasi Analitis Hasil Eksperimen

Berikut adalah visualisasi analitis perbandingan metrik Compression Ratio, Space Saving, PSNR, SSIM, dan Waktu Eksekusi:

![Grafik Analitis Benchmark](figures/benchmark_charts.png)

*Gambar 2: Grafik perbandingan kuantitatif kompresi citra: (a) Compression Ratio (skala logaritmik), (b) Space Saving (%), (c) Kualitas Rekonstruksi PSNR & SSIM, dan (d) Profil Waktu Komputasi (ms).*

Perbandingan visual kualitas citra sebelum dan sesudah kompresi disajikan pada gambar berikut:

![Perbandingan Visual Kompresi](figures/compression_visual.png)

*Gambar 3: Perbandingan visual citra asli beresolusi 512x512 (kolom 1) terhadap hasil rekonstruksi RLE Lossless (kolom 2), JPEG DCT Q=50 (kolom 3), dan WebP Q=50 (kolom 4).*

---

## 7. Pembahasan Mendalam dan Analisis Trade-off

Berdasarkan data kuantitatif pada Tabel dan Gambar 2-3, diperoleh wawasan analitis sebagai berikut:
1. **Keunggulan Telak WebP**: Format WebP secara konsisten mendominasi efisiensi kompresi dengan menghasilkan ukuran berkas terkecil (CR hingga $> 280\times$ pada citra sintetis dan $> 21\times$ pada citra fotografi riil), dengan tetap mempertahankan PSNR tertinggi ($> 39.8\text{ dB}$) dan SSIM tertinggi ($> 0.958$). Prediksi spasial intra-block VP8 terbukti jauh lebih unggul dalam memangkas redundansi spasial dibandingkan kuantisasi DCT 8x8 murni pada JPEG.
2. **Kelemahan dan Keterbatasan RLE**: RLE hanya bekerja optimal pada gambar dengan blok warna datar yang sangat luas (seperti hasil rasterisasi SVG di mana CR mencapai $63.26\times$). Namun, ketika diterapkan pada citra foto natural (seperti wajah DAiSEE atau citra JPEG), nilai piksel bertetangga selalu bervariasi tipis akibat derau sensor kamera. RLE gagal membentuk run panjang ($L=1$), sehingga penambahan overhead pasangan nilai membuat berkas bengkak hingga lebih dari dua kali lipat ukuran asli (*negative space saving*).
3. **Efisiensi Komputasi JPEG**: Dari segi kecepatan eksekusi, JPEG DCT mencatat waktu tercepat (rata-rata $4.5 - 4.9\text{ ms}$ per citra $512 \times 512$), hampir dua kali lebih cepat dibandingkan WebP ($7.6 - 8.8\text{ ms}$). Kecepatan tinggi ini menjelaskan mengapa JPEG tetap dominan pada perangkat keras kamera digital dan transmisi berkecepatan tinggi.

---

## 8. Kesimpulan dan Rekomendasi Format Industri

1. **Untuk Distribusi Web & Aplikasi Seluler Modern**: **WebP** adalah format paling direkomendasikan karena memberikan penghematan kapasitas transmisi terbesar ($> 90\%$) tanpa degradasi visual yang dapat disadari pengguna.
2. **Untuk Grafik Sintetis, Ikon, & Logo Antarmuka**: Citra vektor murni (**SVG**) harus dipertahankan dalam domain teks XML ($< 1\text{ KB}$) untuk menjaga ketajaman visual absolut pada berbagai resolusi layar. Jika rasterisasi terpaksa dilakukan, kompresi PNG/WebP adalah pilihan paling tepat.
3. **Untuk Fotografi & Perangkat Tertanam Berdaya Rendah**: Format **JPEG** tetap menjadi pilihan paling kompatibel dan efisien secara komputasi untuk pemrosesan berbasis perangkat keras (*hardware accelerated*).
4. **Untuk Arsip Medis & Integritas Hukum**: Kompresi **Lossless** (seperti PNG atau JPEG 2000 Lossless) wajib digunakan di mana modifikasi piksel sekecil apa pun dilarang keras demi akurasi diagnosis atau pembuktian hukum.

---

## 9. Daftar Pustaka

1. Barnsley, M. F., & Hurd, L. P. (1993). *Fractal Image Compression*. AK Peters/CRC Press. ISBN: 978-0120790661.
2. Deutsch, P. (1996). *DEFLATE Compressed Data Format Specification version 1.3*. RFC 1951, IETF. DOI: [10.17487/RFC1951](https://doi.org/10.17487/RFC1951).
3. Gray, R. (1984). *Vector quantization*. IEEE ASSP Magazine, 1(2), 4-29. DOI: [10.1109/MASSP.1984.1162229](https://doi.org/10.1109/MASSP.1984.1162229).
4. Huffman, D. A. (1952). *A Method for the Construction of Minimum-Redundancy Codes*. Proceedings of the IRE, 40(9), 1098-1101. DOI: [10.1109/JRPROC.1952.273898](https://doi.org/10.1109/JRPROC.1952.273898).
5. Jacquin, A. E. (1992). *Image coding based on a fractal theory of iterated contractive image transformations*. IEEE Transactions on Image Processing, 1(1), 18-30. DOI: [10.1109/5.127835](https://doi.org/10.1109/5.127835).
6. Rissanen, J., & Langdon, G. G. (1979). *Arithmetic Coding*. IBM Journal of Research and Development, 23(2), 149-162. DOI: [10.1147/rd.232.0149](https://doi.org/10.1147/rd.232.0149).
7. Robinson, A. H., & Cherry, C. (1967). *Results of a prototype television bandwidth compression scheme*. Proceedings of the IEEE, 55(3), 356-364. DOI: [10.1109/PROC.1967.5493](https://doi.org/10.1109/PROC.1967.5493).
8. Skodras, A., Christopoulos, C., & Ebrahimi, T. (2001). *The JPEG 2000 still image compression standard*. IEEE Signal Processing Magazine, 18(5), 36-58. DOI: [10.1109/79.952802](https://doi.org/10.1109/79.952802).
9. Wallace, G. K. (1991). *The JPEG still picture compression standard*. IEEE Transactions on Consumer Electronics, 38(1), xviii-xxxiv. DOI: [10.1109/30.89988](https://doi.org/10.1109/30.89988).
10. Welch, T. A. (1984). *A Technique for High-Performance Data Compression*. Computer, 17(6), 8-19. DOI: [10.1109/MC.1984.1659158](https://doi.org/10.1109/MC.1984.1659158).
