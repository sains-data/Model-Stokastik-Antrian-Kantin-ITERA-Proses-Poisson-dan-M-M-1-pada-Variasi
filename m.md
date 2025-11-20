#  **Model Stokastik Antrian Kantin ITERA: Proses Poisson & M/M/2 pada Variasi Cuaca**

Repository ini berisi analisis lengkap sistem antrian di **Kantin Rumah Kayu ITERA** menggunakan pendekatan **pemodelan stokastik**, yaitu **Proses Poisson** untuk kedatangan pelanggan dan **Model M/M/2** untuk sistem pelayanan dua kasir. Data observasi dicatat per **slot 5 menit**, mencakup kondisi **hujan** dan **tidak hujan**, dan digunakan untuk menilai kinerja layanan, memeriksa kestabilan sistem, menganalisis intensitas kedatangan, serta merumuskan rekomendasi operasional berbasis data bagi pengelola kantin dan pihak kampus.

---

#  **Fitur Utama**

###  Estimasi Laju Kedatangan (λ)

* Estimasi λ harian dan λ per slot 5 menit
* Perbandingan **hujan vs tidak hujan**

###  Uji Poisson & Variabilitas

* Mean–variance test
* Rasio var/mean → cek kecocokan model Poisson
* Slot fluktuatif pada cuaca hujan

###  Model Antrian Stokastik M/M/2

* Perhitungan ρ, L, Lq, W, Wq
* Evaluasi kestabilan sistem
* Sistem stabil untuk dua kasir

###  Analisis Kapasitas (Scenario Planning)

* Target *Wq ≤ 5 menit*
* Kurva hubungan **kapasitas μ_total** terhadap **Wq**
* Kebutuhan kapasitas ideal (≈130 pelanggan/jam)

###  Visualisasi

* Profil λ(t) per slot
* Kurva Wq vs μ_total
* Perbandingan intensitas hujan vs tidak hujan

###  Insight Operasional

* Rekomendasi berbasis data untuk pengelolaan kantin
* Strategi pengurangan waktu tunggu
* Kebijakan kampus untuk peningkatan fasilitas

###  **Diagram Alir Analisis**

Tersedia dalam folder `/gambar/diagram_alir_analisis.png`

---

#  **Struktur Repository**

```text
  antrian-kantin-itera/
│
├── data/
│   └── data_kantin_per5menit.csv
│
├── R/
│   ├── 01_preprocessing.R
│   ├── 02_estimate_poisson.R
│   ├── 03_mm2_model.R
│   └── 04_scenario_capacity.R
│
├── gambar/
│   ├── lambda_per_slot.png
│   ├── kurva_wq_vs_mu_tidak_hujan.png
│   ├── utilisasi_kasir.png
│   ├── diagram_alir_analisis.png
│
└── README.md
```

---

#  **Ringkasan Hasil Utama**

## **1. Total Pelanggan per Hari (Data Agregat)**

| Tanggal    | Kondisi     | Total |
| ---------- | ----------- | ----- |
| 2025-11-11 | Tidak Hujan | 112   |
| 2025-11-12 | Tidak Hujan | 115   |
| 2025-11-13 | Tidak Hujan | 134   |
| 2025-11-18 | Hujan       | 83    |
| 2025-11-19 | Tidak Hujan | 115   |

---

## **2. Estimasi Laju Kedatangan (λ)**

| Kondisi     | λ_mean  | λ_sd  |
| ----------- | ------- | ----- |
| Hujan       | **83**  | NA    |
| Tidak Hujan | **119** | 10.10 |

**Penurunan kedatangan saat hujan = −30.25%**
Elasticity ≈ **−0.30**

---

## **3. Mean–Variance Test per Slot (Uji Poisson)**

| Kondisi     | mean_slot | var_slot | var/mean |
| ----------- | --------- | -------- | -------- |
| Hujan       | 6.92      | 23.17    | 3.35     |
| Tidak Hujan | 9.92      | 9.31     | 0.94     |

Interpretasi:

* **Tidak hujan** → mendekati Poisson (stabil)
* **Hujan** → overdispersed (lonjakan pelanggan tidak teratur)

---

## **4. Profil λ(t) per Slot 5 Menit**

File: `/gambar/lambda_per_slot.png`

Insight:

* Slot-slot awal dan pertengahan jam menunjukkan puncak kedatangan
* Pola hujan berbeda drastis → penting untuk penjadwalan kasir adaptif

---

## **5. Analisis M/M/2 (Kasir = 2, μ = 60 per kasir)**

### **Parameter**

* c = 2
* μ = 60 → total μ_total = 120 pelanggan/jam

### **Utilisasi Sistem**

| Kondisi     | λ   | μ_total | ρ    | Status                   |
| ----------- | --- | ------- | ---- | ------------------------ |
| Tidak Hujan | 119 | 120     | 0.99 | Stabil tapi hampir jenuh |
| Hujan       | 83  | 120     | 0.69 | Stabil dan longgar       |

---

## **6. Scenario Planning (Target Wq ≤ 5 menit)**

File: `gambar/kurva_wq_vs_mu_tidak_hujan.png`

Hasil:

* Target Wq tercapai saat kapasitas **≥130 pelanggan/jam**
* Saat ini μ_total = 120 → sedikit di bawah ideal, tapi masih aman

---

#  **Insight Operasional untuk Kantin ITERA**

## **1. Sistem Dua Kasir Sudah Optimal**

* Utilisasi 99% pada hari tidak hujan → **sangat sibuk namun lancar**
* Tidak terjadi stagnasi → mengonfirmasi observasi lapangan

## **2. Cuaca Hujan Menurunkan Beban Sistem 30%**

* Potensi melakukan **penjadwalan dinamis**

  * 2 kasir pada hari normal
  * 1 kasir + 1 staff fleksibel pada hari hujan

## **3. Slot Puncak Dapat Diintervensi**

Strategi:

* Jalur cepat (*express lane*) untuk menu standar
* Penempatan kasir paling berpengalaman pada slot puncak
* Fitur pre-order/QR ordering

## **4. Perencanaan Jangka Menengah**

Jika mahasiswa bertambah atau jam makan semakin padat:

* Tambah 1 kasir pada jam kritis
* Implementasi **self-service checkout**
* Meningkatkan efektivitas kapasitas hingga **≥130 pelanggan/jam**

## **5. Bukti Pendukung Kebijakan Kampus**

Analisis ini memberikan dasar kuantitatif bagi:

* Evaluasi fasilitas kantin
* Perencanaan jangka panjang
* Peningkatan kenyamanan mahasiswa
* Efektivitas manajemen operasional UMKM kampus

---

#  **Cara Menjalankan Analisis**

```r
source("R/01_preprocessing.R")
source("R/02_estimate_poisson.R")
source("R/03_mm2_model.R")
source("R/04_scenario_capacity.R")
```

---

#  **Tentang Proyek Ini**

Analisis ini disusun sebagai **Tugas Besar Pemodelan Stokastik**, menggunakan data nyata dari Kantin Rumah Kayu ITERA. Hasilnya diharapkan menjadi referensi operasional dan akademik dalam pengelolaan fasilitas kampus berbasis data.

