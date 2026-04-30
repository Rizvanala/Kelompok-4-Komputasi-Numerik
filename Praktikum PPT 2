# Metode Regula Falsi

Implementasi algoritma **Regula Falsi** (False Position Method) menggunakan Python, lengkap dengan tabel iterasi dan grafik fungsi.

Dibuat untuk memenuhi tugas **Praktikum 1 - Metode Numerik**.

---

## Apa itu Metode Regula Falsi?

Regula Falsi adalah metode numerik untuk mencari akar persamaan `f(x) = 0`. Metode ini bekerja dengan cara menarik garis lurus antara dua titik `[a, b]` yang mengapit akar (syarat: `f(a) × f(b) < 0`), lalu titik potong garis tersebut dengan sumbu-x dijadikan pendekatan akar baru.

**Rumus:**
```
c = b - f(b) × (b - a) / (f(b) - f(a))
```

Proses diulang sampai nilai error lebih kecil dari toleransi yang ditentukan.

---

## Fitur Program

- Menghitung akar persamaan dengan metode Regula Falsi
- Menampilkan tabel iterasi lengkap (a, b, c, f(c), error) di terminal
- Menghasilkan 2 grafik:
  - Grafik fungsi f(x) beserta titik-titik iterasi
  - Grafik konvergensi error (skala logaritmik)
- Menyimpan grafik sebagai file `regula_falsi_grafik.png`

---

## Cara Menjalankan

### 1. Install library yang dibutuhkan
```bash
pip install matplotlib numpy
```

### 2. Jalankan program
```bash
python regula_falsi_clean.py
```

---

## Struktur Kode

```
regula_falsi_clean.py
│
├── f(x)              → Definisi fungsi yang dicari akarnya
├── regula_falsi()    → Algoritma utama Regula Falsi + cetak tabel iterasi
├── buat_grafik()     → Membuat dan menyimpan grafik menggunakan matplotlib
└── if __name__...    → Program utama (parameter input di sini)
```

---

## Contoh Output

```
╔══════════════════════════════════════╗
║   METODE REGULA FALSI                ║
║   Praktikum Metode Numerik           ║
╚══════════════════════════════════════╝

  Fungsi    : f(x) = x³ - x - 2
  Interval  : [1, 2]
  Toleransi : 0.0001

=====================================================================================
Iter             a             b       c (akar)          f(c)           Error
=====================================================================================
   1      1.000000      2.000000    1.33333333    -0.96296296          -
   2      1.333333      2.000000    1.44827586    -0.39060811    1.149426e-01
   ...
✓ Konvergen setelah 8 iterasi.
  Akar ≈ 1.52137971
```

---

## Parameter yang Bisa Diubah

Di bagian bawah file (`if __name__ == "__main__":`), kamu bisa mengubah:

| Variabel | Keterangan | Default |
|----------|-----------|---------|
| `A` | Batas bawah interval | `1` |
| `B` | Batas atas interval | `2` |
| `TOLERANSI` | Batas error yang diijinkan | `0.0001` |
| `MAKS_ITER` | Jumlah maksimum iterasi | `50` |

Untuk mengganti fungsi, edit baris di `def f(x)`:
```python
def f(x):
    return x**3 - x - 2  # ganti sesuai soal
```

---

## Teknologi

- Python 3
- NumPy
- Matplotlib
