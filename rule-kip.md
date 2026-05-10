# rule-kip.md
# Aturan Pembuatan Notebook SPK Seleksi Penerima Beasiswa KIP-K
# Politeknik Negeri Jember — Metode PROMETHEE

---

## 1. Tujuan

Dokumen ini adalah aturan teknis untuk membuat **satu file notebook**
`spk_kipk_promethee.ipynb` yang mengimplementasikan Sistem Pendukung Keputusan
(SPK) seleksi penerima beasiswa KIP-K di Politeknik Negeri Jember menggunakan
metode **PROMETHEE II**.

Pipeline yang diimplementasikan dalam satu notebook:

```
Data Mentah → Preprocessing → Penentuan Kriteria → Pembobotan
    → Perhitungan PROMETHEE → Ranking → Validasi → Export Model
```

---

## 2. Struktur Path Proyek

```
spk-kipk/
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
├── spk_kipk_promethee.ipynb                         ← ★ FILE UTAMA (satu notebook lengkap)
├── rule-kip.md                                      ← dokumen aturan ini
└── requirements.txt                                 ← daftar library Python
```

### Aturan Path

- Semua path di dalam notebook ditulis relatif dari root proyek `spk-kipk/`.
- Konstanta path didefinisikan di **Cell 1 (Konfigurasi)** agar mudah disesuaikan.
- Folder `output/` dan sub-foldernya dibuat otomatis oleh notebook jika belum ada.

---

## 3. Isi File `requirements.txt`

```
pandas>=2.0
numpy>=1.24
openpyxl>=3.1
scikit-learn>=1.3
matplotlib>=3.7
seaborn>=0.12
joblib>=1.3
```

---

## 4. Kriteria & Bobot

| Kode | Nama Kriteria          | Bobot | Tipe    |
|------|------------------------|-------|---------|
| C1   | Kepemilikan KIP SMA    | 0.04  | Benefit |
| C2   | Status DTKS            | 0.17  | Benefit |
| C3   | Desil                  | 0.49  | Benefit |
| C4   | Penghasilan Orang Tua  | 0.03  | Benefit |
| C5   | Status Orang Tua       | 0.24  | Benefit |
| C6   | Prestasi               | 0.03  | Benefit |
| **Total** |                   | **1.0** |       |

Semua kriteria bertipe **Benefit** — semakin tinggi nilai, semakin layak
mahasiswa menerima beasiswa.

---

## 5. Normalisasi Sub-Kriteria (Skoring Teks → Numerik)

### C1 — Kepemilikan KIP SMA

| Sub-Kriteria                                                              | Nilai |
|---------------------------------------------------------------------------|-------|
| Memiliki KIP                                                              | 4     |
| Tidak memiliki KIP, tetapi termasuk keluarga penerima bantuan sosial lain | 3     |
| Tidak memiliki KIP, tetapi memiliki SKTM dari pihak berwenang             | 2     |
| Tidak memiliki semuanya                                                   | 1     |

### C2 — Status DTKS

| Sub-Kriteria            | Nilai |
|-------------------------|-------|
| Terdaftar dalam DTKS    | 3     |
| Penerima bantuan sosial | 2     |
| Belum terdaftar         | 1     |

### C3 — Desil

| Sub-Kriteria | Nilai |
|--------------|-------|
| Desil 1      | 4     |
| Desil 2      | 3     |
| Desil 3      | 2     |
| Desil > 4    | 1     |

### C4 — Penghasilan Orang Tua

| Rentang Penghasilan                  | Nilai |
|--------------------------------------|-------|
| ≤ Rp 500.000 s.d. ≤ Rp 1.000.000    | 5     |
| > Rp 1.000.000 s.d. ≤ Rp 2.000.000  | 4     |
| > Rp 2.000.000 s.d. ≤ Rp 3.000.000  | 3     |
| > Rp 3.000.000 s.d. ≤ Rp 4.000.000  | 2     |
| > Rp 4.000.000                       | 1     |

### C5 — Status Orang Tua

| Sub-Kriteria                      | Nilai |
|-----------------------------------|-------|
| Yatim Piatu                       | 4     |
| Yatim / Piatu                     | 3     |
| Orang tua sakit / tidak bekerja   | 2     |
| Orang tua masih bekerja           | 1     |

### C6 — Prestasi

| Sub-Kriteria    | Nilai |
|-----------------|-------|
| Internasional   | 5     |
| Nasional        | 4     |
| Provinsi / Kota | 3     |
| Sekolah         | 2     |
| Tidak ada       | 1     |

---

## 6. Struktur Notebook `spk_kipk_promethee.ipynb`

Notebook terdiri dari **9 cell** yang dijalankan secara berurutan dari atas ke bawah.
Setiap cell diawali sebuah cell Markdown sebagai heading tahap.

```
Cell 0  — Judul & Deskripsi (Markdown)
Cell 1  — Install Library & Konfigurasi Path / Kriteria
Cell 2  — [TAHAP 1] Load Data Mentah
Cell 3  — [TAHAP 2] Preprocessing
Cell 4  — [TAHAP 3] Penentuan Kriteria & Pembobotan
Cell 5  — [TAHAP 4] Perhitungan PROMETHEE II
Cell 6  — [TAHAP 5] Ranking Hasil
Cell 7  — [TAHAP 6] Validasi Model
Cell 8  — [TAHAP 7] Export Model & Hasil
```

---

## 7. Aturan Tiap Tahap dalam Notebook

---

### Cell 1 — Install Library & Konfigurasi

**Tujuan:** Satu tempat untuk semua konstanta yang mungkin perlu diubah.

Isi yang wajib ada:

```python
# Path data
DATA_PATH       = "data/raw/V_Data_Mahasiswa_Full_KIPK.xlsx"
DITETAPKAN_PATH = "data/raw/Mahasiswa_ditetapkan_Politeknik_Negeri_Jember_20241012.xlsx"
CLEAN_PATH      = "data/processed/data_clean.csv"

# Path output
OUT_RANKING     = "output/hasil_ranking.xlsx"
OUT_VALIDASI    = "output/validasi_result.json"
OUT_MODEL       = "output/model/model_promethee.joblib"
OUT_CONFIG      = "output/model/scaler_config.json"
OUT_FIG_DIST    = "output/figures/distribusi_kriteria.png"
OUT_FIG_FLOW    = "output/figures/promethee_flow.png"
OUT_FIG_CM      = "output/figures/confusion_matrix.png"

# Kuota penerima (None = otomatis dari ground truth historis yang cocok)
KUOTA = None

# Konfigurasi kriteria & bobot
CRITERIA = {
    "C1": {"name": "Kepemilikan KIP SMA",  "weight": 0.04, "type": "benefit"},
    "C2": {"name": "Status DTKS",           "weight": 0.17, "type": "benefit"},
    "C3": {"name": "Desil",                 "weight": 0.49, "type": "benefit"},
    "C4": {"name": "Penghasilan Orang Tua", "weight": 0.03, "type": "benefit"},
    "C5": {"name": "Status Orang Tua",      "weight": 0.24, "type": "benefit"},
    "C6": {"name": "Prestasi",              "weight": 0.03, "type": "benefit"},
}

# Mapping sub-kriteria teks → nilai numerik
SUB_CRITERIA_MAP = { "C1": {...}, "C2": {...}, ... }  # lihat Bagian 5
```

---

### TAHAP 1 — Load Data Mentah (Cell 2)

**Tujuan:** Membaca kedua file Excel ke dalam DataFrame.

Aturan:
- Baca `DATA_PATH` sebagai `df_raw`.
- Baca `DITETAPKAN_PATH` sebagai `df_label` (ground truth untuk validasi).
- Tampilkan `shape`, nama kolom, dan 5 baris pertama masing-masing file.
- Jika file tidak ditemukan, cetak pesan error yang jelas dan hentikan eksekusi.

---

### TAHAP 2 — Preprocessing (Cell 3)

**Tujuan:** Membersihkan dan mengubah data mentah menjadi matriks numerik siap hitung.

Langkah yang wajib dijalankan secara berurutan:

```
2a. Deteksi kolom kriteria secara fleksibel (toleransi variasi nama kolom)
2b. Hapus baris duplikat berdasarkan kolom NIM
2c. Tangani nilai kosong (missing values) secara konservatif:
      - Kolom identitas boleh diisi placeholder untuk tampilan
      - Kolom kriteria tidak boleh diisi modus sebelum scoring
      - Nilai kosong pada kriteria diberi skor terendah/default sesuai fungsi skoring
2d. Konversi kolom teks → nilai numerik sesuai tabel skoring (Bagian 5)
2e. Konversi kolom penghasilan (angka rupiah) → skor 1–5:
        ≤ 1.000.000            → 5
        > 1.000.000–2.000.000  → 4
        > 2.000.000–3.000.000  → 3
        > 3.000.000–4.000.000  → 2
        > 4.000.000            → 1
2f. Pastikan semua kolom kriteria C1–C6 bertipe numerik (int atau float)
2g. Simpan hasil ke CLEAN_PATH (data/processed/data_clean.csv)
```

Output cell: ringkasan jumlah duplikat dihapus, jumlah missing value per kolom,
dan preview `df_clean` 5 baris.

---

### TAHAP 3 — Penentuan Kriteria & Pembobotan (Cell 4)

**Tujuan:** Memvalidasi konfigurasi kriteria dan menampilkan gambaran data.

Aturan:
- Validasi bahwa total bobot semua kriteria = 1.0 (raise error jika tidak).
- Tampilkan tabel kriteria (kode, nama, bobot, tipe) sebagai DataFrame.
- Tampilkan statistik deskriptif `df_clean[["C1","C2","C3","C4","C5","C6"]]`.
- Tampilkan grafik distribusi nilai tiap kriteria (histogram 2×3 grid),
  simpan ke `OUT_FIG_DIST`.

---

### TAHAP 4 — Perhitungan PROMETHEE II (Cell 5)

**Tujuan:** Menghitung matriks preferensi, Leaving Flow, Entering Flow, dan Net Flow.

Implementasikan sebagai kelas Python `PROMETHEE` dengan metode:

```
__init__(weights, criteria)
    → simpan bobot dan kode kriteria sebagai atribut

preference_function(d) → float
    → Fungsi Preferensi Tipe I (Usual):
       return 1.0 jika d > 0, else 0.0

compute_preference_matrix(matrix: np.ndarray) → np.ndarray (n × n)
    → Untuk setiap pasang (i, j):
       π(a,b) = Σ_k [ w_k × P(f_k(a) − f_k(b)) ]

leaving_flow(pi: np.ndarray) → np.ndarray (n,)
    → Φ+(a) = 1/(n−1) × Σ_b π(a,b)

entering_flow(pi: np.ndarray) → np.ndarray (n,)
    → Φ−(a) = 1/(n−1) × Σ_b π(b,a)

net_flow(phi_plus, phi_minus) → np.ndarray (n,)
    → Φ(a) = Φ+(a) − Φ−(a)

rank(decision_matrix: pd.DataFrame) → (pd.DataFrame, np.ndarray)
    → Jalankan semua metode di atas secara berurutan
    → Kembalikan DataFrame dengan kolom tambahan:
       Phi_Plus, Phi_Minus, Net_Flow, Rank
    → Kembalikan juga matriks π untuk keperluan inspeksi
```

Output cell:
- Tampilkan 5×5 sudut kiri atas matriks `π`.
- Tampilkan tabel `[NIM, Nama, Phi_Plus, Phi_Minus, Net_Flow]` untuk 10 baris teratas.

---

### TAHAP 5 — Ranking Hasil (Cell 6)

**Tujuan:** Menampilkan dan menyimpan peringkat akhir seluruh mahasiswa.

Aturan:
- Urutkan berdasarkan `Net_Flow` descending (peringkat 1 = Net Flow tertinggi).
- Jika ada nilai Net Flow seri (tie), urutkan berdasarkan `Phi_Plus` descending.
- Tambahkan kolom `Layak`:

```python
df_result["Layak"] = df_result["Rank"] <= KUOTA
```

- Tampilkan tabel lengkap peringkat semua mahasiswa.
- Tampilkan dua grafik, simpan ke `OUT_FIG_FLOW`:
  - Bar chart horizontal Net Flow (20 mahasiswa teratas, warna hijau jika positif, merah jika negatif)
  - Scatter plot Phi+ vs Phi− dengan warna berdasarkan Net Flow (colormap RdYlGn)
- Simpan seluruh peringkat ke `OUT_RANKING`.

---

### TAHAP 6 — Validasi Model (Cell 7)

**Tujuan:** Mengukur kinerja sistem menggunakan ground truth dari data yang telah ditetapkan.

Aturan:
- Join `df_result` dengan `df_label` berdasarkan kolom `NIM`.
- Buat kolom label:
  - `y_true = 1` jika NIM ada di `df_label`, `y_true = 0` jika tidak.
  - `y_pred = 1` jika `Layak == True`, `y_pred = 0` jika tidak.
- Hitung dan tampilkan metrik:

```
Accuracy   = (TP + TN) / (TP + TN + FP + FN)
Precision  = TP / (TP + FP)
Recall     = TP / (TP + FN)
F1-Score   = 2 × (Precision × Recall) / (Precision + Recall)
```

- Tampilkan Confusion Matrix sebagai heatmap (seaborn `heatmap` dengan `annot=True`).
- Simpan heatmap Confusion Matrix ke `OUT_FIG_CM`
  (`output/figures/confusion_matrix.png`).
- Tampilkan Classification Report lengkap.
- Cetak kesimpulan otomatis:

```python
layak_implementasi = all(
    metric >= 0.70
    for metric in [accuracy, precision, recall, f1]
)
if layak_implementasi:
    print("Model LAYAK diimplementasikan")
else:
    print("Model BELUM layak - tinjau ulang bobot, preprocessing, atau nilai KUOTA")
```

Target metrik minimum agar model dinyatakan layak:

| Metrik    | Target Minimum |
|-----------|----------------|
| Accuracy  | ≥ 70%          |
| Precision | ≥ 70%          |
| Recall    | ≥ 70%          |
| F1-Score  | ≥ 70%          |

---

### TAHAP 7 — Export Model (Cell 8)

**Tujuan:** Menyimpan semua output agar dapat digunakan kembali tanpa menjalankan
ulang notebook.

Buat folder output otomatis sebelum menyimpan:

```python
import os
for folder in ["data/processed", "output/model", "output/figures"]:
    os.makedirs(folder, exist_ok=True)
```

File yang wajib diekspor:

| File                      | Path              | Isi                                               |
|---------------------------|-------------------|---------------------------------------------------|
| `model_promethee.joblib`  | `output/model/`   | Paket model PROMETHEE portabel (weights + criteria + metadata) |
| `scaler_config.json`      | `output/model/`   | Dictionary CRITERIA, SUB_CRITERIA_MAP, kuota, tie-breaker, dan catatan kalibrasi |
| `hasil_ranking.xlsx`      | `output/`         | Seluruh mahasiswa + Phi_Plus + Phi_Minus + Net_Flow + Rank + Layak |
| `validasi_result.json`    | `output/`         | Accuracy, Precision, Recall, F1, ROC-AUC, Average Precision, Confusion Matrix |
| `confusion_matrix.png`    | `output/figures/` | Heatmap Confusion Matrix hasil validasi |

Struktur `scaler_config.json`:

```json
{
  "criteria": {
    "C1": { "name": "Kepemilikan KIP SMA",  "weight": 0.04, "type": "benefit" },
    "C2": { "name": "Status DTKS",           "weight": 0.17, "type": "benefit" },
    "C3": { "name": "Desil",                 "weight": 0.49, "type": "benefit" },
    "C4": { "name": "Penghasilan Orang Tua", "weight": 0.03, "type": "benefit" },
    "C5": { "name": "Status Orang Tua",      "weight": 0.24, "type": "benefit" },
    "C6": { "name": "Prestasi",              "weight": 0.03, "type": "benefit" }
  },
  "sub_criteria_map": {
    "C1": { "Memiliki KIP": 4, "...": "..." },
    "C2": { "...": "..." },
    "...": "..."
  }
}
```

Struktur `validasi_result.json`:

```json
{
  "accuracy": 0.82,
  "precision": 0.79,
  "recall": 0.85,
  "f1_score": 0.81,
  "confusion_matrix": [[TN, FP], [FN, TP]],
  "layak_implementasi": true
}
```

Cetak konfirmasi path lengkap setiap file yang berhasil disimpan.

---

## 8. Aturan Umum Penulisan Notebook

- Setiap cell kode diawali komentar `# ── TAHAP N — NAMA TAHAP ──────────`.
- Setiap cell mencetak status di akhir, contoh: `✅ Preprocessing selesai`.
- Notebook **harus bisa dijalankan penuh** dengan Kernel → Restart & Run All
  tanpa error — tidak ada cell yang bergantung pada state cell yang dilewati.
- Semua konstanta yang mungkin perlu diubah (path, KUOTA, bobot) **hanya** ada
  di Cell 1, tidak tersebar di cell lain.
- Bahasa komentar dan output: **Bahasa Indonesia**.

---

*Dokumen ini adalah spesifikasi teknis untuk pembuatan `spk_kipk_promethee.ipynb`.*
*Ikuti urutan tahap dan aturan tiap cell secara ketat agar notebook dapat dijalankan*
*dari awal hingga akhir tanpa error.*
