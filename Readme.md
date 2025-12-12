# 📊 Analisis dan Optimasi Antrian Kantin Rumah Kayu  
## Pemodelan Stokastik dengan Proses Poisson dan M/M/2  

Repository ini berisi analisis lengkap menggunakan **Proses Poisson** dan **Model Antrian M/M/2** untuk memodelkan perilaku kedatangan dan layanan pelanggan di Kantin Rumah Kayu ITERA. Data diperoleh dari observasi nyata per 5 menit, dengan membandingkan kondisi **hujan** dan **tidak hujan**.

---

## 🎯 Latar Belakang  
Di lapangan terdapat **2 kasir aktif**, namun sering terjadi antrian tidak beraturan atau bergerombol, sehingga kapasitas layanan efektif menjadi jauh lebih rendah dibanding kapasitas teoritis. Hal ini menyebabkan kemacetan walaupun server sudah dua.

Analisis ini menghasilkan estimasi laju kedatangan, variabilitas slot waktu, tingkat stabilitas sistem, hingga rekomendasi peningkatan layanan kantin.

---

## 📈 Fitur Utama  

### 1. Estimasi Laju Kedatangan (λ) menggunakan Proses Poisson  
Rata-rata pelanggan/jam (λ):  
- **Tidak Hujan**: 119 pelanggan/jam  
- **Hujan**: 83 pelanggan/jam  
- **Penurunan karena hujan**: **–30.25%**

### 2. Uji Variabilitas Slot Waktu (5 menit per slot)  
Menghitung mean dan varians untuk 12 slot, dengan rasio var/mean sebagai indikator stabilitas proses:

| Kondisi      | Rasio var/mean | Makna |
|--------------|----------------|-------|
| Hujan        | 3.35           | Overdispersion tinggi, kedatangan sulit diprediksi |
| Tidak hujan  | 0.94           | Stabil dan mendekati Poisson murni |

### 3. Profil λ(t) per Slot  
- Slot hujan paling padat: **slot pertama** (15 pelanggan)  
- Slot tidak hujan paling padat: **slot ke‑4 sampai 7** (10–12 pelanggan)  
*(Gambar disimpan sebagai `gambar/lambda_per_slot.png`)*

### 4. Analisis Kestabilan Sistem M/M/2  
**Asumsi layanan:**  
- Observasi lapangan: 2 kasir aktif  
- Estimasi kapasitas layanan per kasir ≈ 30 pelanggan/jam  
- Total μ sistem = 60 pelanggan/jam  

**Hasil analisis:**  
| Kondisi      | λ   | μ_total | ρ    | Stabil?          |
|--------------|-----|---------|------|------------------|
| Tidak hujan  | 119 | 60      | 1.98 | ❌ Tidak stabil |
| Hujan        | 83  | 60      | 1.38 | ❌ Tidak stabil |

**Penyebab ketidakstabilan:**  
1. λ jauh lebih besar dari μ  
2. Mahasiswa sering antri bergerombol sehingga *service discipline* menjadi kacau  
3. Efisiensi kasir turun karena antrian tidak terbentuk dalam satu barisan yang rapi  

### 5. Penentuan μ Minimum Agar Sistem Stabil  
**Tujuan:** Wq ≤ 5 menit  
**Hasil perhitungan:**  
- Dibutuhkan μ_total ≈ **130 pelanggan/jam**  
- Jika satu kasir ≈ 30 pelanggan/jam → dibutuhkan **4–5 kasir**  
- Alternatif: **2 kasir + jalur cepat + order online/pickup**  
*(Gambar: `gambar/kurva_wq_vs_mu.png`)*

---

## 🔄 Diagram Alir Analisis Pemodelan  

1. Input data observasi 5 menit  
2. Agregasi per hari dan per slot  
3. Estimasi λ (Poisson)  
4. Analisis var/mean  
5. Estimasi λ(t) per slot  
6. Evaluasi M/M/2 (stabilitas 2 kasir)  
7. Simulasi tambahan μ  
8. Rekomendasi operasional  

---

## 📁 Struktur Repository  

```
📦 antrian-kantin-itera/
│
├── data/
│   └── data_kantin_observasi.csv
│
├── R/
│   └── model_poisson_mm2.R     (sudah menyesuaikan 2 kasir)
│
├── gambar/
│   ├── lambda_per_slot.png
│   ├── kurva_wq_vs_mu.png
│   └── diagram_alir.png
│
└── README.md
```

---

## 📊 Ringkasan Hasil Utama  

### 1. Total Pelanggan per Hari  
| Tanggal     | Kondisi    | Total |
|-------------|------------|-------|
| 2025‑11‑11  | Tidak hujan | 112   |
| 2025‑11‑12  | Tidak hujan | 115   |
| 2025‑11‑13  | Tidak hujan | 134   |
| 2025‑11‑18  | Hujan      | 83    |
| 2025‑11‑19  | Tidak hujan | 115   |

### 2. Estimasi λ per Kondisi  
| Kondisi     | λ_mean | λ_sd   |
|-------------|--------|--------|
| Hujan       | 83     | NA     |
| Tidak hujan | 119    | 10.10  |

### 3. Variabilitas Slot  
| Kondisi     | mean_slot | var_slot | var/mean |
|-------------|-----------|----------|----------|
| Hujan       | 6.92      | 23.17    | 3.35     |
| Tidak hujan | 9.92      | 9.31     | 0.94     |

**Interpretasi:**  
- Hujan membuat kedatangan tidak stabil  
- Tidak hujan lebih konsisten dan mendekati Poisson murni  

### 4. Elasticity Dampak Hujan  
Penurunan hingga **–30.25%**  
**Makna praktis:**  
- Hujan membuat kantin kehilangan ±36 pelanggan per jam  
- Dampak signifikan pada pendapatan dan kapasitas pelayanan  

### 5. Analisis M/M/2  
Walaupun kasir ada 2, sistem tetap tidak stabil karena:  
1. λ jauh melebihi μ_total  
2. Pola antrian tidak rapi  
3. Mahasiswa sering bergerombol → kasir tidak bisa optimal  
4. Jalur antri tidak berbentuk satu barisan  

**Akibatnya:**  
- Waktu tunggu meningkat  
- Antrian macet  
- Ruang makan menjadi padat dan chaos  

---

## 💡 Insight Operasional untuk ITERA  

### 1. Dua Kasir Tidak Cukup di Jam Puncak  
Dengan λ = 119 dan μ_total = 60 → ρ mendekati 2 → antrian pasti tidak stabil.

### 2. Solusi Realistis  
- **Tambah kasir** menjadi minimal **4 kasir**  
- **Bentuk jalur antrian satu garis** (*single line queue*)  
- Tambah papan petunjuk alur antrian  
- Terapkan opsi **order online/pickup**  
- Tambah kanopi agar hujan tidak menghambat akses  

### 3. Intervensi pada Slot Overload  
Slot paling padat dapat diberi:  
- Jalur cepat  
- Petugas pengarah antrian  
- Penataan ulang meja agar alur lebih jelas  

---

## 🧮 Teknologi  
- **R**  
- **tidyverse**, **ggplot2**, **dplyr**  
- *Math for queueing theory*

---

## ✨ Tentang Proyek  
Analisis ini dibuat sebagai **Tugas Besar Pemodelan Stokastik**.  
Tujuannya memberikan dasar ilmiah bagi perbaikan sistem antrian Kantin Rumah Kayu ITERA berdasarkan pendekatan **Proses Poisson** dan model antrian **M/M/2**.
