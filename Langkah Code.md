# Model-Stokastik-Antrian-Kantin-ITERA-Proses-Poisson-dan-M-M-c-pada-Variasi-Cuaca
Penelitian ini menganalisis antrian Kantin Rumah Kayu ITERA dengan Proses Poisson dan model M/M/2 untuk mempelajari dampak cuaca terhadap kedatangan pelanggan, utilisasi kasir, dan waktu tunggu, serta memberikan rekomendasi peningkatan efisiensi layanan berbasis data.




* **Proses Poisson & Non-Homogen**
* **Birth–Death / M/M/2**
* Insight kebijakan yang bisa "dijual" ke institusi.

Saya mulai dari `df` yang sudah di buat.

---

## 0. Setup awal

```r
library(tidyverse)

# Data hasil observasi (sudah di buat)
df <- tribble(
  ~Tanggal, ~Slot_Waktu, ~Jumlah, ~Kondisi,
  "2025-11-11","11.50 - 11.55",6,"Tidak Hujan",
  "2025-11-11","11.55 - 12.00",7,"Tidak Hujan",
  "2025-11-11","12.00 - 12.05",12,"Tidak Hujan",
  "2025-11-11","12.05 - 12.10",14,"Tidak Hujan",
  "2025-11-11","12.10 - 12.15",8,"Tidak Hujan",
  "2025-11-11","12.15 - 12.20",9,"Tidak Hujan",
  "2025-11-11","12.20 - 12.25",9,"Tidak Hujan",
  "2025-11-11","12.25 - 12.30",7,"Tidak Hujan",
  "2025-11-11","12.30 - 12.35",13,"Tidak Hujan",
  "2025-11-11","12.35 - 12.40",11,"Tidak Hujan",
  "2025-11-11","12.40 - 12.45",9,"Tidak Hujan",
  "2025-11-11","12.45 - 12.50",7,"Tidak Hujan",
  "2025-11-12","11.50 - 11.55",10,"Tidak Hujan",
  "2025-11-12","11.55 - 12.00",11,"Tidak Hujan",
  "2025-11-12","12.00 - 12.05",12,"Tidak Hujan",
  "2025-11-12","12.05 - 12.10",12,"Tidak Hujan",
  "2025-11-12","12.10 - 12.15",11,"Tidak Hujan",
  "2025-11-12","12.15 - 12.20",13,"Tidak Hujan",
  "2025-11-12","12.20 - 12.25",5,"Tidak Hujan",
  "2025-11-12","12.25 - 12.30",5,"Tidak Hujan",
  "2025-11-12","12.30 - 12.35",9,"Tidak Hujan",
  "2025-11-12","12.35 - 12.40",7,"Tidak Hujan",
  "2025-11-12","12.40 - 12.45",9,"Tidak Hujan",
  "2025-11-12","12.45 - 12.50",11,"Tidak Hujan",
  "2025-11-13","11.50 - 11.55",6,"Tidak Hujan",
  "2025-11-13","11.55 - 12.00",8,"Tidak Hujan",
  "2025-11-13","12.00 - 12.05",12,"Tidak Hujan",
  "2025-11-13","12.05 - 12.10",13,"Tidak Hujan",
  "2025-11-13","12.10 - 12.15",8,"Tidak Hujan",
  "2025-11-13","12.15 - 12.20",9,"Tidak Hujan",
  "2025-11-13","12.20 - 12.25",13,"Tidak Hujan",
  "2025-11-13","12.25 - 12.30",12,"Tidak Hujan",
  "2025-11-13","12.30 - 12.35",14,"Tidak Hujan",
  "2025-11-13","12.35 - 12.40",13,"Tidak Hujan",
  "2025-11-13","12.40 - 12.45",14,"Tidak Hujan",
  "2025-11-13","12.45 - 12.50",12,"Tidak Hujan",
  "2025-11-19","11.50 - 11.55",8,"Tidak Hujan",
  "2025-11-19","11.55 - 12.00",8,"Tidak Hujan",
  "2025-11-19","12.00 - 12.05",4,"Tidak Hujan",
  "2025-11-19","12.05 - 12.10",7,"Tidak Hujan",
  "2025-11-19","12.10 - 12.15",16,"Tidak Hujan",
  "2025-11-19","12.15 - 12.20",11,"Tidak Hujan",
  "2025-11-19","12.20 - 12.25",12,"Tidak Hujan",
  "2025-11-19","12.25 - 12.30",12,"Tidak Hujan",
  "2025-11-19","12.30 - 12.35",7,"Tidak Hujan",
  "2025-11-19","12.35 - 12.40",13,"Tidak Hujan",
  "2025-11-19","12.40 - 12.45",14,"Tidak Hujan",
  "2025-11-19","12.45 - 12.50",3,"Tidak Hujan",
  "2025-11-18","11.50 - 11.55",15,"Hujan",
  "2025-11-18","11.55 - 12.00",4,"Hujan",
  "2025-11-18","12.00 - 12.05",3,"Hujan",
  "2025-11-18","12.05 - 12.10",2,"Hujan",
  "2025-11-18","12.10 - 12.15",2,"Hujan",
  "2025-11-18","12.15 - 12.20",8,"Hujan",
  "2025-11-18","12.20 - 12.25",15,"Hujan",
  "2025-11-18","12.25 - 12.30",12,"Hujan",
  "2025-11-18","12.30 - 12.35",7,"Hujan",
  "2025-11-18","12.35 - 12.40",6,"Hujan",
  "2025-11-18","12.40 - 12.45",7,"Hujan",
  "2025-11-18","12.45 - 12.50",2,"Hujan"
)
```

Penjelasan singkat: memuat data observasi asli ke dalam R menggunakan `tribble` agar formatnya rapi dan minim error.

---

## 1. Preprocessing: tanggal, slot jadi numerik, dan ringkasan harian

```r
df_clean <- df %>%
  mutate(
    Tanggal = as.Date(Tanggal),                 # ubah ke Date
    slot_id = row_number(),                     # index urut (opsional)
    slot_index = rep(1:12, times = nrow(df) / 12)  # 12 slot per hari
  )

# Ringkasan total per hari dan kondisi
daily_summary <- df_clean %>%
  group_by(Tanggal, Kondisi) %>%
  summarise(
    total = sum(Jumlah),
    .groups = "drop"
  )

daily_summary
```

Penjelasan:

* `Tanggal` diubah ke tipe `Date`.
* `slot_index` 1–12 dipakai sebagai "waktu diskrit" dalam satu jam.
* `daily_summary` menghitung **total pelanggan per hari** untuk setiap kondisi cuaca.

---

## 2. Estimasi laju kedatangan λ per hari dan per kondisi (Poisson Process)

```r
# Asumsi jendela observasi = 1 jam (12 slot x 5 menit)
daily_lambda <- daily_summary %>%
  mutate(
    lambda_per_hour = total / 1     # pelanggan per jam
  )

daily_lambda

# Rata-rata λ per kondisi cuaca
lambda_by_cond <- daily_lambda %>%
  group_by(Kondisi) %>%
  summarise(
    lambda_mean = mean(lambda_per_hour),
    lambda_sd   = sd(lambda_per_hour),
    .groups = "drop"
  )

lambda_by_cond
```

Penjelasan:

* Karena tiap hari kamu mengamati 1 jam penuh, `lambda_per_hour` = total pelanggan pada jam itu.
* `lambda_by_cond` memberikan **laju kedatangan rata-rata** untuk kondisi hujan vs tidak hujan.
* Ini langsung terkait materi **Proses Poisson**: λ sebagai parameter kedatangan.

---

## 3. Cek "kepoissonan" data slot 5 menit (mean vs varians)

```r
# Cek mean dan varians jumlah pelanggan per slot (5 menit) per kondisi
slot_stats <- df_clean %>%
  group_by(Kondisi) %>%
  summarise(
    mean_slot   = mean(Jumlah),
    var_slot    = var(Jumlah),
    ratio_var_mean = var(Jumlah) / mean(Jumlah),
    .groups = "drop"
  )

slot_stats
```

Penjelasan:

* Untuk Proses Poisson, **mean ≈ variance**.
* `ratio_var_mean` dekat 1 → konsisten dengan asumsi Poisson.
* Ini menghubungkan langsung ke teori **distribusi Poisson** dan **proses Poisson** di modul.

---

## 4. Poisson Non-Homogen (NHPP) via λ(t) per slot waktu

Di sini kita lihat bahwa λ berubah sepanjang jam (baru sangat stokastik):

```r
slot_lambda <- df_clean %>%
  group_by(slot_index, Kondisi) %>%
  summarise(
    rata_slot = mean(Jumlah),
    .groups = "drop"
  )

slot_lambda
```

Visualisasinya:

```r
ggplot(slot_lambda, aes(x = slot_index, y = rata_slot, color = Kondisi)) +
  geom_line(linewidth = 1) +
  geom_point(size = 2) +
  scale_x_continuous(breaks = 1:12) +
  labs(
    title = "Profil Intensitas Kedatangan per Slot (λ(t))",
    x = "Slot ke- (5 menit)",
    y = "Rata-rata pelanggan per slot"
  ) +
  theme_minimal()
```

Penjelasan:

* `slot_lambda` adalah **estimasi intensitas λ(t)** per 5 menit.
* Karena nilai λ(t) berbeda antar slot, ini mendekati **Poisson Non-Homogen (NHPP)**: λ bukan konstan, tetapi fungsi dari waktu diskrit.
* Insight: pihak kampus bisa melihat **slot mana yang menjadi "critical window"** dengan intensitas tertinggi.

---

## 5. Menghitung rasio penurunan λ akibat hujan (Elasticity)

```r
lambda_normal <- lambda_by_cond$lambda_mean[lambda_by_cond$Kondisi == "Tidak Hujan"]
lambda_hujan  <- lambda_by_cond$lambda_mean[lambda_by_cond$Kondisi == "Hujan"]

elasticity <- (lambda_hujan - lambda_normal) / lambda_normal
elasticity
```

Penjelasan:

* `elasticity` negatif, sekitar –0.30 → **hujan menurunkan laju kedatangan sekitar 30 persen**.
* Ini bisa dibahas sebagai "demand sensitivity" terhadap cuaca, sangat kuat untuk argumen ke rektorat.

---

## 6. Model M/M/2 (Birth–Death Process) berdasarkan λ hasil observasi

Sekarang kita pakai materi **proses kelahiran-kematian / M/M/2** dengan 2 kasir.

```r
# Cek nilai lambda_normal dan lambda_hujan
cat("Lambda Normal:", lambda_normal, "pelanggan/jam\n")
cat("Lambda Hujan:", lambda_hujan, "pelanggan/jam\n")

# Service rate berdasarkan observasi: 1.5 menit per transaksi
mu_kasir <- 60 / 1.5  # 40 pelanggan/jam (1.5 menit/transaksi)
c_kasir <- 2    # 2 kasir

cat("Service rate per kasir:", round(mu_kasir, 1), "pelanggan/jam (1.5 menit/transaksi)\n")
cat("Jumlah kasir:", c_kasir, "\n")

# Hitung rho untuk kondisi normal
rho_normal <- lambda_normal / (c_kasir * mu_kasir)
rho_hujan <- lambda_hujan / (c_kasir * mu_kasir)

cat("Utilisasi (rho) kondisi normal:", round(rho_normal, 3), "(", round(rho_normal*100, 1), "% )\n")
cat("Utilisasi (rho) kondisi hujan:", round(rho_hujan, 3), "(", round(rho_hujan*100, 1), "% )\n")

# Analisis kestabilan sistem
if (rho_normal < 1) {
  cat("Sistem STABIL dengan 2 kasir\n")
} else {
  cat("Sistem OVERLOAD dengan 2 kasir\n")
  
  # Analisis dengan jumlah kasir yang lebih realistis
  c_grid_realistis <- tibble(c = 2:6) %>%  # coba 2 sampai 6 kasir
    rowwise() %>%
    mutate(
      kapasitas = c * mu_kasir,
      rho_normal = lambda_normal / kapasitas,
      rho_hujan = lambda_hujan / kapasitas,
      stabil_normal = rho_normal < 1,
      stabil_hujan = rho_hujan < 1,
      utilisasi_normal = round(rho_normal * 100, 1),
      utilisasi_hujan = round(rho_hujan * 100, 1)
    ) %>%
    ungroup()
  
  print(c_grid_realistis)
  
  # Cari jumlah kasir minimum untuk sistem stabil
  c_min_stabil <- c_grid_realistis %>%
    filter(stabil_normal == TRUE) %>%
    slice(1)
  
  cat("Jumlah kasir minimum untuk sistem stabil:", c_min_stabil$c, "\n")
  
  # Sekarang hitung dengan jumlah kasir yang stabil
  c_kasir_stabil <- c_min_stabil$c
  
  mm2_normal <- mmc_metrics(lambda_normal, mu = mu_kasir, c = c_kasir_stabil) %>%
    mutate(Kondisi = "Tidak Hujan")
  
  mm2_hujan <- mmc_metrics(lambda_hujan, mu = mu_kasir, c = c_kasir_stabil) %>%
    mutate(Kondisi = "Hujan")
  
  mm2_compare <- bind_rows(mm2_normal, mm2_hujan) %>%
    mutate(
      W_min  = W  * 60,   # konversi ke menit
      Wq_min = Wq * 60,   # konversi ke menit
      utilisasi_percent = round(rho * 100, 1)
    ) %>%
    select(Kondisi, lambda, mu, c, utilisasi_percent, L, Lq, W_min, Wq_min)
  
  print(mm2_compare)
}

# Analisis performa dengan 2 kasir (kondisi aktual)
cat("\n=== ANALISIS DENGAN 2 KASIR (KONDISI AKTUAL) ===\n")

if (rho_normal < 1 & rho_hujan < 1) {

  mm2_actual_normal <- mmc_metrics(lambda_normal, mu = mu_kasir, c = 2) %>%
    mutate(Kondisi = "Tidak Hujan")

  mm2_actual_hujan <- mmc_metrics(lambda_hujan, mu = mu_kasir, c = 2) %>%
    mutate(Kondisi = "Hujan")

  mm2_actual_compare <- bind_rows(mm2_actual_normal, mm2_actual_hujan) %>%
    mutate(
      W_min  = W  * 60,
      Wq_min = Wq * 60,
      utilisasi_percent = round(rho * 100, 1)
    ) %>%
    select(Kondisi, lambda, mu, c, utilisasi_percent, L, Lq, W_min, Wq_min)

  print(mm2_actual_compare)

} else {
  cat("Tidak bisa menghitung metrik M/M/2 karena sistem TIDAK STABIL (rho >= 1) untuk 2 kasir.\n")
}
```

Penjelasan:

* Menghubungkan **λ hasil data** dengan **μ asumsi kasir** untuk menurunkan ρ, L, Lq, W, Wq.
* Model **M/M/2** lebih realistis karena ada 2 kasir di lapangan.
* Ini persis materi **birth–death processes**, **M/M/c queue**, dan **continuous-time Markov chain** pada state jumlah pelanggan di sistem.

---

## 7. Analisis sensitivitas kebijakan: berapa kasir yang dibutuhkan agar Wq ≤ target

Ini bagian yang powerfull buat rekomendasi kebijakan.

Misal: kampus ingin **rata-rata waktu tunggu Wq ≤ 5 menit** saat tidak hujan.

```r
target_Wq_min <- 5    # target 5 menit

# Analisis sensitivitas dengan multiple servers
c_grid <- tibble(c = 1:5) %>%
  rowwise() %>%
  mutate(
    lambda = lambda_normal,
    mu     = mu_kasir,               # service rate per kasir
    rho    = lambda / (c * mu),      # utilisasi sistem
    a      = lambda / mu,            # λ / μ

    # Denominator P0 (probabilitas sistem kosong)
    # P0 = 1 / [ Σ_{n=0}^{c-1} (a^n / n!) + (a^c / (c! * (1 - rho))) ]
    P0_denom = sum(a^(0:(c - 1)) / factorial(0:(c - 1))) +
               (a^c / (factorial(c) * (1 - rho))),

    P0 = ifelse(rho < 1, 1 / P0_denom, NA_real_),

    # Lq = (P0 * a^c * rho) / (c! * (1 - rho)^2)
    Lq = ifelse(
      rho < 1,
      P0 * (a^c * rho) / (factorial(c) * (1 - rho)^2),
      NA_real_
    ),

    # Wq = Lq / lambda (jam), lalu dikonversi ke menit
    Wq     = ifelse(rho < 1, Lq / lambda, NA_real_),
    Wq_min = Wq * 60
  ) %>%
  ungroup()

# Cari jumlah kasir minimum untuk mencapai target
c_needed <- c_grid %>%
  filter(!is.na(Wq_min)) %>%              # hanya yang stabil
  filter(Wq_min <= target_Wq_min) %>%     # sudah memenuhi target
  slice(1)

if (nrow(c_needed) == 0) {
  c_min_wq <- c_grid %>%
    filter(!is.na(Wq_min)) %>%            # hanya yang stabil
    filter(Wq_min == min(Wq_min, na.rm = TRUE)) %>%
    slice(1)
  
  cat("Dengan service rate", round(c_grid$mu[1], 2), "pelanggan/jam per kasir:\n")
  cat("Tidak ada jumlah kasir (1–5) yang mencapai Wq ≤", target_Wq_min, "menit\n")
  cat("Wq minimum yang dapat dicapai:", round(c_min_wq$Wq_min, 2), "menit\n")
  cat("Dengan", c_min_wq$c, "kasir\n")
} else {
  cat("Jumlah kasir yang dibutuhkan:", c_needed$c, "kasir\n")
}

c_grid
```

Penjelasan:

* Kita cari **berapa jumlah kasir** yang dibutuhkan agar rata-rata waktu tunggu tidak lebih dari 5 menit.
* Hasil `c_needed` bisa diterjemahkan menjadi rekomendasi staffing yang optimal.

---

## 8. Visualisasi Jumlah kasir dan Utilitas

```r
{r}
# ==========================================
# Analisis M/M/c + Visualisasi 
# ==========================================

library(tibble)
library(dplyr)
library(ggplot2)

target_Wq_min <- 5    # target 5 menit

# --------- 1) Hitung c_grid (kode kamu) ---------
c_grid <- tibble(c = 1:5) %>%
  rowwise() %>%
  mutate(
    lambda = lambda_normal,
    mu     = mu_kasir,               # service rate per kasir
    rho    = lambda / (c * mu),      # utilisasi sistem
    a      = lambda / mu,            # λ / μ
    P0_denom = sum(a^(0:(c - 1)) / factorial(0:(c - 1))) +
               (a^c / (factorial(c) * (1 - rho))),
    P0 = ifelse(rho < 1, 1 / P0_denom, NA_real_),
    Lq = ifelse(rho < 1,
                P0 * (a^c * rho) / (factorial(c) * (1 - rho)^2),
                NA_real_),
    Wq     = ifelse(rho < 1, Lq / lambda, NA_real_),
    Wq_min = Wq * 60
  ) %>%
  ungroup()

# --------- 2) Info jumlah kasir minimum ---------
c_needed <- c_grid %>%
  filter(!is.na(Wq_min)) %>%
  filter(Wq_min <= target_Wq_min) %>%
  slice(1)

if (nrow(c_needed) == 0) {
  c_min_wq <- c_grid %>%
    filter(!is.na(Wq_min)) %>%
    filter(Wq_min == min(Wq_min, na.rm = TRUE)) %>%
    slice(1)
  cat("Tidak ada c (1–5) yang mencapai Wq <=", target_Wq_min, "menit\n")
  cat("Wq minimum:", round(c_min_wq$Wq_min, 2), "menit dengan c =", c_min_wq$c, "\n\n")
} else {
  cat("Jumlah kasir minimum agar Wq <=", target_Wq_min,
      "menit adalah c =", c_needed$c, "\n\n")
}

print(c_grid)

# --------- 3) Siapkan data untuk plot ---------
c_grid_plot <- c_grid %>%
  mutate(
    rho_percent = rho * 100,
    status = if_else(is.na(Wq_min),
                     "Tidak stabil (rho ≥ 1)",
                     "Stabil (rho < 1)")
  )

# Data hanya yang stabil untuk plot Wq (supaya tidak ada NA)
c_grid_stable <- c_grid_plot %>%
  filter(!is.na(Wq_min))

# --------- 4) Plot Wq_min vs c (tanpa NA) ---------
ggplot(c_grid_stable, aes(x = factor(c), y = Wq_min, fill = status)) +
  geom_col(width = 0.6) +
  geom_hline(yintercept = target_Wq_min, linetype = "dashed") +
  geom_text(aes(label = round(Wq_min, 1)),
            vjust = -0.3, size = 3) +
  labs(title = "Waktu Tunggu Rata-rata di Antrian (Wq)",
       x = "Jumlah kasir (c)",
       y = "Wq (menit)",
       fill = "Status") +
  theme_minimal()

# --------- 5) Plot utilisasi vs c (semua c) ---------
ggplot(c_grid_plot, aes(x = factor(c), y = rho_percent, fill = status)) +
  geom_col(width = 0.6) +
  geom_hline(yintercept = 100, linetype = "dashed") +
  geom_text(aes(label = paste0(round(rho_percent, 1), "%")),
            vjust = -0.3, size = 3) +
  labs(title = "Utilisasi Sistem (rho)",
       x = "Jumlah kasir (c)",
       y = "Utilisasi (%)",
       fill = "Status") +
  theme_minimal()
```

Penjelasan:

* Grafik ini sangat efektif untuk menunjukkan ke kampus:
  "Dengan jumlah kasir saat ini (2), waktu tunggu adalah X menit. Jika ditambah menjadi Y kasir, Wq turun menjadi Z menit."
* Ini insight yang "terasa" secara operasional, tidak hanya matematis.

---

## 9. Insight kuat untuk institusi (ringkasan konsep)

Dari seluruh analisis, beberapa insight penting yang bisa ditulis di laporan:

1. **Poisson & NHPP**

   * Kedatangan pelanggan per slot 5 menit konsisten dengan asumsi **proses Poisson/NHPP**: rata-rata dan varians jumlah pelanggan per slot berada pada orde yang sama.
   * Profil ( \lambda(t) ) per slot menunjukkan adanya **jam puncak** di tengah interval pengamatan. Slot inilah yang menjadi **window kritis** yang harus diintervensi (misalnya dengan penambahan kasir sementara, pengaturan alur antrean, atau sistem pre-order).

2. **Dampak hujan**

   * Pada kondisi hujan, laju kedatangan pelanggan ( \lambda ) turun sekitar **30 persen** dibanding kondisi tidak hujan. Jadi hujan bukan sekadar “lebih sepi”, tapi memang terjadi **penurunan demand yang signifikan**.
   * Konsekuensinya, **pendapatan harian UMKM berpotensi turun**, sehingga stok bahan dan persiapan produksi perlu disesuaikan dengan prediksi cuaca.

3. **M/M/c, kondisi lapangan (2 kasir), dan utilisasi**

   * Di lapangan terdapat **2 kasir**, dengan waktu layanan rata-rata sekitar **1,5 menit/pelanggan**. Ini setara dengan kapasitas pelayanan per kasir (\mu \approx 40) pelanggan/jam, sehingga kapasitas total dua kasir adalah sekitar **80 pelanggan/jam**.
   * Dari hasil estimasi, laju kedatangan pada jam sibuk sekitar ( \lambda \approx 119 ) pelanggan/jam. Dengan rumus
     [
     \rho = \frac{\lambda}{c\mu},
     ]
     diperoleh:

     * **c = 2 kasir (kondisi aktual)** → ( \rho \approx 1{,}49 ) atau sekitar **149 persen**. Artinya, **secara teori M/M/2 tidak stabil** di jam puncak: laju kedatangan lebih besar daripada kapasitas layanan, sehingga antrian cenderung menumpuk.
     * **c = 3 kasir** → ( \rho \approx 0{,}99 ) (hampir 100 persen): sistem mulai stabil tetapi kasir bekerja pada beban yang sangat tinggi.
     * **c = 4–5 kasir** → ( \rho \approx 74–60 persen ): sistem stabil dengan buffer yang lebih aman, antrian jauh lebih terkendali.
   * Grafik utilisasi yang dihasilkan memperlihatkan pola ini secara visual: batang untuk c = 1–2 berada jauh di atas 100 persen (overload), sedangkan c = 4–5 berada di zona aman.

4. **Analisis sensitivitas staffing**

   * Dengan menetapkan target **SLA waktu tunggu rata-rata di antrian** (misalnya ( W_q \le 5 ) menit), model M/M/c memungkinkan kita menghitung **jumlah kasir minimal yang diperlukan pada jam puncak**.
   * Untuk parameter ( \lambda \approx 119 ) dan ( \mu \approx 40 ), perhitungan menunjukkan bahwa:

     * **2 kasir (kondisi saat ini)** belum cukup untuk mencapai target SLA di jam sibuk.
     * **Penambahan kasir menjadi sekitar 4–5 orang** pada window puncak jauh lebih realistis untuk menurunkan waktu tunggu ke kisaran beberapa menit.
   * Hasil ini dapat diterjemahkan menjadi:

     * **Staffing fleksibel**: 2 kasir pada jam sepi, naik menjadi 3–5 kasir saat jam puncak.
     * **Sistem shift kasir** yang khusus diaktifkan di slot waktu dengan λ tertinggi.
     * **Optimalisasi layout** (jalur antrean, pemisahan antrean “pesan + bayar” dan “bayar saja”) untuk mengurangi waktu layanan efektif per pelanggan.

5. **Rekomendasi berbasis data**

   * **Operasional**

     * Menambahkan kasir sementara atau mengaktifkan kasir cadangan saat jam puncak agar utilisasi turun dari >140 persen menjadi di bawah 100 persen.
     * Mengatur kembali prosedur pelayanan (misalnya kasir khusus non-tunai atau khusus top-up) untuk mempercepat rata-rata layanan.
   * **Teknologi**

     * Mengembangkan atau memanfaatkan **sistem pre-order** (pesan dulu, bayar di kasir cepat) untuk meratakan beban layanan kasir sepanjang jam operasional.
   * **Manajemen**

     * Menyusun **kebijakan staffing adaptif** yang mempertimbangkan:

       * pola kedatangan ( \lambda(t) ) (hari dan jam sibuk),
       * serta prediksi cuaca (hujan vs tidak hujan),
         sehingga jumlah kasir tidak berlebihan di jam sepi tetapi **cukup kuat menahan lonjakan antrian** di jam puncak.
