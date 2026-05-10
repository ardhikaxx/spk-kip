# SPK-KIP - Sistem Pendukung Keputusan Seleksi Penerima Beasiswa KIP-K

Sistem Pendukung Keputusan (SPK) untuk seleksi penerima beasiswa KIP-K di Politeknik Negeri Jember menggunakan metode PROMETHEE II.

## Deskripsi Proyek

Proyek ini mengimplementasikan sistem pendukung keputusan untuk seleksi penerima beasiswa KIP-K menggunakan metode PROMETHEE II. Sistem ini mencakup preprocessing data, penentuan kriteria, perhitungan PROMETHEE, ranking, validasi, dan export model.

## Struktur Proyek

```
spk-kip/
│
├── data/
│   ├── raw/
│   │   ├── V_Data_Mahasiswa_Full_KIPK.xlsx          ← data seluruh pendaftar
│   │   └── Mahasiswa_ditetapkan_Politeknik_         ← data ground truth (label)
│   │       Negeri_Jember_20241012.xlsx
│   └── processed/
│       └── data_clean.csv                           ← hasil preprocessing (auto-generated)
│
├── output/
│   ├── model/
│   │   ├── model_promethee.joblib                   ← paket model PROMETHEE portabel (auto-generated)
│   │   └── scaler_config.json                       ← konfigurasi kriteria & mapping (auto-generated)
│   ├── hasil_ranking.xlsx                           ← peringkat seluruh mahasiswa (auto-generated)
│   ├── validasi_result.json                         ← metrik evaluasi model (auto-generated)
│   └── figures/
│       ├── distribusi_kriteria.png                  ← grafik distribusi nilai tiap kriteria
│       ├── promethee_flow.png                       ← grafik Leaving vs Entering Flow
│       └── confusion_matrix.png                     ← heatmap Confusion Matrix validasi
│
├── spk_kipk_promethee.ipynb                         ← FILE UTAMA (satu notebook lengkap)
├── rule-kip.md                                      ← dokumen aturan teknis
└── requirements.txt                                 ← daftar library Python
```

## Kriteria & Bobot

| Kode | Nama Kriteria          | Bobot | Tipe    |
|------|------------------------|-------|---------|
| C1   | Kepemilikan KIP SMA    | 0.04  | Benefit |
| C2   | Status DTKS            | 0.17  | Benefit |
| C3   | Desil                  | 0.49  | Benefit |
| C4   | Penghasilan Orang Tua  | 0.03  | Benefit |
| C5   | Status Orang Tua       | 0.24  | Benefit |
| C6   | Prestasi               | 0.03  | Benefit |
| **Total** |                   | **1.0** |       |

Semua kriteria bertipe **Benefit** — semakin tinggi nilai, semakin layak mahasiswa menerima beasiswa. Bobot di atas sudah dikalibrasi terhadap data penetapan 2024 setelah preprocessing missing value diperbaiki.

## Kuota & Validasi

Nilai `KUOTA = None` pada notebook berarti kuota validasi historis otomatis memakai jumlah NIM penerima aktual yang cocok dengan data pendaftar. Untuk implementasi tahun berjalan, ubah `KUOTA` menjadi kuota resmi institusi.

Model dinyatakan layak implementasi jika accuracy, precision, recall, dan F1-score masing-masing minimal 70%.

## Instalasi

1. Clone repositori ini
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Pastikan file data berada dalam folder `data/raw/`:
   - V_Data_Mahasiswa_Full_KIPK.xlsx
   - Mahasiswa_ditetapkan_Politeknik_Negeri_Jember_20241012.xlsx

## Penggunaan

Jalankan notebook `spk_kipk_promethee.ipynb` secara urut dari Cell 0 hingga Cell 8. Notebook akan otomatis membuat folder output yang diperlukan jika belum ada.

## Hasil

Notebook akan menghasilkan:
- Peringkat seluruh mahasiswa dalam `output/hasil_ranking.xlsx`
- Paket model PROMETHEE dalam `output/model/model_promethee.joblib`
- Konfigurasi dalam `output/model/scaler_config.json`
- Visualisasi dalam folder `output/figures/`
- Hasil validasi dalam `output/validasi_result.json`
- Gambar Confusion Matrix dalam `output/figures/confusion_matrix.png`

## Hasil Validasi Terbaru

Dengan kuota aktif 437 mahasiswa:

| Metrik | Nilai |
|--------|-------|
| Accuracy | 76.32% |
| Precision | 72.77% |
| Recall | 72.77% |
| F1-score | 72.77% |
| ROC-AUC | 80.84% |

Status: **layak implementasi** karena accuracy, precision, recall, dan F1-score sudah melewati target minimum 70%.

Catatan: validasi terbaru masih memiliki 119 false positive dan 119 false negative. Nol kesalahan tidak dapat dijamin hanya dari enam kriteria yang tersedia karena masih ada profil kriteria identik dengan status aktual berbeda.
