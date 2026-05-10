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
│   │   ├── model_promethee.joblib                   ← model terekspor (auto-generated)
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
| C1   | Kepemilikan KIP SMA    | 0.3   | Benefit |
| C2   | Status DTKS            | 0.2   | Benefit |
| C3   | Desil                  | 0.2   | Benefit |
| C4   | Penghasilan Orang Tua  | 0.1   | Benefit |
| C5   | Status Orang Tua       | 0.1   | Benefit |
| C6   | Prestasi               | 0.1   | Benefit |
| **Total** |                   | **1.0** |       |

Semua kriteria bertipe **Benefit** — semakin tinggi nilai, semakin layak mahasiswa menerima beasiswa.

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
- Model yang terlatih dalam `output/model/model_promethee.joblib`
- Konfigurasi dalam `output/model/scaler_config.json`
- Visualisasi dalam folder `output/figures/`
- Hasil validasi dalam `output/validasi_result.json`
- Gambar Confusion Matrix dalam `output/figures/confusion_matrix.png`
