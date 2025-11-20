# Model Stokastik Antrian Kantin ITERA: Proses Poisson dan M/M/2 pada Variasi Cuaca

Repository ini berisi analisis sistem antrian di Kantin Rumah Kayu ITERA menggunakan **pemodelan stokastik**. Kedatangan pelanggan dimodelkan sebagai **Proses Poisson**, sedangkan sistem layanan dimodelkan sebagai **antrian M/M/2** (dua kasir aktif, masing-masing rata-rata 1 menit per transaksi ≈ 60 pelanggan/jam). Data observasi dicatat per **slot 5 menit** pada jam makan siang, untuk dua kondisi cuaca: **hujan** dan **tidak hujan**.

Tujuan utama proyek ini adalah:
- mengestimasi **laju kedatangan** pelanggan (λ) dan intensitas per slot waktu,
- mengevaluasi **kinerja antrian** (utilisasi kasir, waktu tunggu, panjang antrian),
- serta memberikan **rekomendasi kebijakan kapasitas** yang realistis bagi pengelola kantin dan pihak kampus.

---

## 1. Latar Belakang

Kantin kampus merupakan fasilitas strategis bagi mahasiswa dan staf. Pada jam makan siang, antrian yang tidak terkelola dengan baik dapat menurunkan kenyamanan, mengurangi waktu istirahat efektif, dan pada akhirnya berdampak pada produktivitas akademik. Observasi di Kantin Rumah Kayu ITERA menunjukkan bahwa:

- data kedatangan pelanggan tersedia per 5 menit,
- terdapat **2 kasir** yang melayani,
- waktu pelayanan rata-rata sekitar **1 menit per pelanggan**,
- secara kasat mata antrian **ramai tetapi tidak macet**.

Hal ini mendorong kebutuhan untuk menganalisis antrian secara kuantitatif menggunakan **pemodelan stokastik**.

---

## 2. Model Stokastik yang Digunakan

### 2.1 Proses Poisson

- Kedatangan pelanggan dimodelkan sebagai **Proses Poisson** dengan laju λ (pelanggan per jam).
- Laju λ diestimasi dari data agregat per jam dan per slot 5 menit.
- Dicek pula **rasio var/mean** sebagai indikator apakah asumsi Poisson (var ≈ mean) cukup wajar.

### 2.2 Antrian M/M/2 (Birth–Death Process)

- Sistem dilayani oleh **2 kasir** (c = 2).
- Waktu pelayanan tiap kasir ~ **Eksponensial** dengan laju μ = 60 pelanggan/jam (≈ 1 menit).
- Total kapasitas sistem: **c × μ = 120 pelanggan/jam**.
- Dihitung metrik klasik antrian:
  - ρ : tingkat utilisasi kasir
  - L : rata-rata pelanggan dalam sistem
  - Lq: rata-rata pelanggan dalam antrian
  - W : rata-rata waktu dalam sistem
  - Wq: rata-rata waktu menunggu

### 2.3 Analisis Kapasitas (Scenario Planning)

Sebagai tambahan, disusun skenario analisis **kapasitas minimum** untuk memenuhi target waktu tunggu tertentu (misal Wq ≤ 5 menit), dengan memvariasikan μ_total dan melihat pengaruhnya terhadap Wq.

---

## 3. Ringkasan Data Observasi

### 3.1 Agregasi Total Pelanggan per Hari

Data diambil untuk beberapa hari pengamatan pada jam 11.50–12.50 WIB.

| Tanggal     | Kondisi     | Total pelanggan/jam |
|------------|------------|---------------------|
| 2025-11-11 | Tidak hujan | 112                 |
| 2025-11-12 | Tidak hujan | 115                 |
| 2025-11-13 | Tidak hujan | 134                 |
| 2025-11-18 | Hujan       | 83                  |
| 2025-11-19 | Tidak hujan | 115                 |

### 3.2 Estimasi Laju Kedatangan (λ) per Kondisi

| Kondisi     | λ_mean (pelanggan/jam) | λ_sd |
|------------|------------------------|------|
| Hujan      | 83                     | NA   |
| Tidak hujan| 119                    | 10.10|

Laju kedatangan **turun sekitar 30% ketika hujan**, dengan elasticity:

\[
\text{Elasticity} = \frac{\lambda_{\text{hujan}} - \lambda_{\text{normal}}}{\lambda_{\text{normal}}}
≈ -0.30
\]

### 3.3 Mean–Variance per Slot 5 Menit

| Kondisi     | mean_slot | var_slot | var/mean |
|------------|-----------|----------|----------|
| Hujan      | 6.92      | 23.17    | 3.35     |
| Tidak hujan| 9.92      | 9.31     | 0.94     |

Interpretasi singkat:
- **Tidak hujan:** var/mean ≈ 1 → cukup sesuai asumsi Poisson (kedatangan relatif stabil).
- **Hujan:** var/mean ≫ 1 → kedatangan lebih “loncat-loncat” dan tidak stabil.

### 3.4 Profil λ(t) per Slot

Dari rata-rata per slot (slot 1–12, masing-masing 5 menit), diperoleh grafik intensitas kedatangan λ(t):

- File: `gambar/lambda_per_slot.png`

Grafik ini menampilkan slot-slot kritis, misalnya:
- Slot awal (11.50–11.55) & slot sekitar tengah jam cenderung paling padat,
- Pola berbeda antara **hujan** dan **tidak hujan**, yang penting untuk pengaturan shift kasir.

---

## 4. Ringkasan Hasil Antrian M/M/2

### 4.1 Parameter Dasar

- Jumlah kasir: **c = 2**
- Service rate per kasir: **μ = 60 pelanggan/jam**
- Kapasitas total: **c × μ = 120 pelanggan/jam**
- Laju kedatangan:
  - λ_normal ≈ **119 pelanggan/jam**
  - λ_hujan ≈ **83 pelanggan/jam**

### 4.2 Utilisasi Sistem

\[
\rho = \frac{\lambda}{c \mu}
\]

| Kondisi     | λ   | c×μ | ρ (utilisasi) | Kestabilan |
|------------|-----|-----|---------------|------------|
| Tidak hujan| 119 | 120 | ≈ 0.99        | Stabil, hampir jenuh |
| Hujan      | 83  | 120 | ≈ 0.69        | Stabil dan longgar   |

Artinya:
- Pada **hari tidak hujan**, kasir bekerja **sangat sibuk** tetapi sistem masih stabil, konsisten dengan observasi “ramai namun tidak macet”.
- Pada **hari hujan**, kapasitas kasir lebih dari cukup; sistem jauh dari jenuh.

### 4.3 Analisis Kapasitas Target Wq ≤ 5 Menit

Dengan memvariasikan kapasitas pelayanan total μ (pelanggan/jam), diperoleh:

- File grafik: `gambar/kurva_wq_vs_mu_tidak_hujan.png`

Ringkasan:
- Untuk λ ≈ 119, target **Wq ≤ 5 menit** tercapai ketika **μ_total ≈ 130 pelanggan/jam**.
- Saat ini kapasitas total 2 kasir adalah **120 pelanggan/jam**, sedikit di bawah rekomendasi teoritis, tetapi masih dapat diterima karena tidak semua hari berada di puncak 119 pelanggan/jam.

---

## 5. Insight Operasional untuk Kantin ITERA

1. **Kapasitas Saat Ini Cukup, Namun Sangat Sibuk pada Hari Normal**  
   Dengan 2 kasir, utilisasi ~0.99 pada hari tidak hujan. Ini menjelaskan mengapa antrian terlihat padat tetapi masih mengalir. Jika jumlah mahasiswa atau intensitas pengunjung meningkat sedikit saja, sistem berisiko memasuki zona sangat jenuh.

2. **Cuaca Hujan Mengurangi Beban Sistem ±30%**  
   Kedatangan turun dari 119 menjadi 83 pelanggan/jam. Ini membuka peluang penjadwalan dinamis:
   - jumlah kasir penuh saat cuaca cerah,
   - pengaturan shift lebih fleksibel saat hujan.

3. **Slot Waktu Kritis Dapat Menjadi Fokus Intervensi**  
   Dari profil λ(t), beberapa slot 5 menit memiliki lonjakan pelanggan yang tinggi. Kantin bisa:
   - membuka “jalur cepat” untuk menu tertentu,
   - menempatkan kasir yang lebih berpengalaman pada slot puncak,
   - atau mendorong pre-order / pemesanan online sebelum jam puncak.

4. **Rekomendasi Kapasitas Jangka Menengah**  
   Jika jumlah mahasiswa meningkat atau jam operasional dipersempit, disarankan mempertimbangkan:
   - menambah **1 kasir tambahan pada jam puncak**, atau
   - menambah **teknologi self-service / QR ordering** sehingga efektif meningkatkan μ_total mendekati ≥130 pelanggan/jam untuk menjaga Wq di sekitar 5 menit.

5. **Basis Bukti untuk Kebijakan Kampus**  
   Hasil ini tidak hanya berbentuk angka, tetapi juga memberikan narasi kuat bahwa:
   - kantin sudah dikelola cukup efisien,
   - namun sistem berada dekat batas kapasitas,
   - sehingga keputusan penambahan fasilitas bisa dipertimbangkan berbasis data, bukan sekadar persepsi.

---

## 6. Struktur Repository

```text
📦 antrian-kantin-itera/
│
├── data/
│   └── data_kantin_observasi.csv        # dataset observasi per 5 menit
│
├── R/
│   └── analisis_poisson_mm2.R          # script utama analisis stokastik
│
├── gambar/
│   ├── lambda_per_slot.png             # profil λ(t) per slot
│   ├── kurva_wq_vs_mu_tidak_hujan.png  # kurva Wq vs kapasitas μ
│   └── diagram_alir_analisis.png       # diagram alir penelitian
│
└── README.md
