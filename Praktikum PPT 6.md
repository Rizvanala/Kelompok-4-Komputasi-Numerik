## Metode Integrasi Romberg

---

## Langkah-Langkah

Input: fungsi `f(x)`, batas `a` dan `b`, ordo maksimum, dan toleransi. Program membangun tabel R(n,m) dengan dua rumus:

**Langkah 1, Trapezoidal bertingkat (kolom pertama):**
```
R(n, 0) = h/2 * [f(a) + 2*Σf(xi) + f(b)],   h = (b-a) / 2^n
```

**Langkah 2, Ekstrapolasi Richardson (sisa tabel):**
```
R(n, m) = [4^m * R(n, m-1) - R(n-1, m-1)] / (4^m - 1)
```

Berhenti saat salah satu kondisi terpenuhi:
- Toleransi: `|R(n,n) - R(n-1,n-1)| / |R(n,n)| < ε`
- Ordo maksimum tercapai

---

## Penjelasan Tiap Blok Kode

---

### IMPORT LIBRARY
```python
import math
import sys
```
`math` untuk fungsi matematika, `sys` untuk hentikan program saat error fatal.

---

### NAMESPACE FUNGSI AMAN
```python
SAFE_NAMESPACE = {
    "sin": math.sin, "cos": math.cos, "tan": math.tan,
    "exp": math.exp, "log": math.log, "sqrt": math.sqrt,
    "pi": math.pi, "e": math.e, ...
}
```
Whitelist fungsi yang boleh dipakai `eval()`, input berbahaya dari user otomatis ditolak.

---

### DEFINISI FUNGSI DAN TRANSLATE INPUT
```python
def parse_function(expr_str):
    expr = expr_str.replace("^", "**")
    def f(x):
        ns = dict(SAFE_NAMESPACE)
        ns["x"] = x
        try:
            result = eval(expr, {"__builtins__": {}}, ns)
            return float(result)
        except Exception as err:
            raise ValueError(f"Error evaluasi f({x}): {err}")
    return f
```
Mengubah string input jadi fungsi `f(x)`. `^` dikonversi ke `**`, error akan ke `try-except`.

```python
def parse_value(val_str):
    ns = dict(SAFE_NAMESPACE)
    return float(eval(val_str.replace("^", "**"), {"__builtins__": {}}, ns))
```
Mengubah input batas jadi angka, mendukung tanda seperti `pi` atau `2*pi`.

---

### METODE | TRAPEZOIDAL
```python
def trapezoidal(f, a, b, n):
    h = (b - a) / n
    total = f(a) + f(b)
    for i in range(1, n):
        total += 2.0 * f(a + i * h)
    return (h / 2.0) * total
```
Trapezoidal standar, error `O(h²)`. Jadi dasar kolom pertama Romberg sekaligus pembanding di tabel akhir.

---

### METODE | SIMPSON 1/3 DAN 3/8
```python
def simpson_1_3(f, a, b, n):   # n harus genap
def simpson_3_8(f, a, b, n):   # n harus kelipatan 3
```
Hanya untuk tabel perbandingan. Error `O(h⁴)`, lebih akurat dari Trapezoidal tapi masih di bawah Romberg `O(h^(2n+2))`.

---

### ALGORITMA INTI | METODE ROMBERG
```python
def romberg(f, a, b, max_order=10, toleransi=1e-8):
    R = []
    jumlah_eval = 0
    for n in range(max_order):
        baris = [0.0] * (n + 1)
        h = (b - a) / (2 ** n)
```
`R` tabel 2D yang tumbuh tiap iterasi. Tiap baris `n`, `h` dibagi dua sehingga subinterval berlipat ganda.

**Kolom pertama R(n,0):**
```python
        if n == 0:
            baris[0] = (h / 2.0) * (f(a) + f(b))
        else:
            baris[0] = 0.5 * R[n-1][0] + h * jumlah_baru
```
`n=0` pakai 2 titik saja. Baris berikutnya hanya hitung titik baru, titik lama tidak dihitung ulang. Jadi jauh lebih efisien.

**Ekstrapolasi Richardson:**
```python
        for m in range(1, n + 1):
            faktor = 4 ** m
            baris[m] = (faktor * baris[m-1] - R[n-1][m-1]) / (faktor - 1)
```
Tiap kolom `m` mengeliminasi satu suku error. `m=1` setara Simpson `O(h⁴)`, `m=2` setara `O(h⁶)`, dst.

**Cek konvergensi:**
```python
        if n >= 1:
            error_rel = abs((R[n][n] - R[n-1][n-1]) / R[n][n])
            if error_rel < toleransi:
                return R, True, n, jumlah_eval
```
Berhenti otomatis saat selisih dua diagonal berturutan sudah di bawah toleransi.

---

### TABEL | OUTPUT ROMBERG
```python
    header = f"  {'n':>3} | {'Subint':>7} | "
    for m in range(n_final + 1):
        header += f"{'m=' + str(m):>14}"
```
Cetak tabel R(n,m) lengkap. Nilai diagonal ditandai `[nilai]*` sebagai hasil terbaik tiap baris.

---

### TABEL | KONVERGENSI
```python
    for n in range(n_final + 1):
        val = R[n][n]
        if n >= 1:
            err = abs((val - val_lama) / val) * 100
            status = "KONVERGEN ✓" if err < 0.001 else "iterasi..."
```
Tampilkan perubahan nilai diagonal `R(n,n)` tiap iterasi beserta error relatifnya.

---

### USER INTERFACE & INPUT VALIDASI
```python
def main():
    while True:
        expr = input("\n  Masukkan f(x) : ").strip()
        if not expr:
            print("  [!] Fungsi tidak boleh kosong.")
            continue
        f = parse_function(expr)
        f(1.0)
        break
```
Fungsi diuji dulu dengan `f(1.0)` sebelum masuk algoritma, error sintaks langsung ketahuan di sini.

```python
    try:
        a = parse_value(input("  Batas bawah a  : ").strip())
        b = parse_value(input("  Batas atas  b  : ").strip())
        max_order = int(ordo_str) if ordo_str else 8
        toleransi = float(tol_str) if tol_str else 1e-8
    except ValueError:
        print("  [!] Input tidak valid!")
```
Default ordo `8` dan toleransi `1e-8` jika tidak diisi. `try-except` tangkap input non-angka.

---

### FINAL | EXECUTION & SIMPULAN
```python
    R, konvergen, n_final, jumlah_eval = romberg(f, a, b, max_order, toleransi)
    hasil_akhir = R[n_final][n_final]

    cetak_tabel_romberg(R, n_final)
    cetak_konvergensi(R, n_final)
    cetak_perbandingan(f, a, b, hasil_akhir, n_final)
    cetak_hasil_akhir(hasil_akhir, a, b, n_final, jumlah_eval, konvergen, toleransi)
```
`romberg()` kembalikan tabel, status konvergensi, ordo terakhir, dan jumlah evaluasi. Hasil akhir dari `R[n_final][n_final]`, pojok kanan-bawah tabel, nilai paling akurat. Empat fungsi cetak semua output secara berurutan.

```python
if __name__ == "__main__":
    main()
```
`main()` hanya jalan saat file dieksekusi langsung, bukan saat di-import.

---

## Contoh Output Program

```
=================================================================
       PROGRAM INTEGRASI ROMBERG - METODE NUMERIK
       Tugas Praktikum #3 - Komputasi Numerik
=================================================================

[INFO] Mengapa Romberg lebih baik dari Trapezoidal?
  - Trapezoidal memiliki error O(h^2): butuh banyak interval
    untuk akurasi tinggi.
  - Romberg menggunakan Ekstrapolasi Richardson untuk
    mengeliminasi suku error secara bertahap:
      Kolom 0 (m=0) : error O(h^2)  <- Trapezoidal biasa
      Kolom 1 (m=1) : error O(h^4)  <- Seperti Simpson 1/3
      Kolom 2 (m=2) : error O(h^6)
      Kolom m       : error O(h^(2m+2))

=================================================================
  TABEL ROMBERG  R(n, m)
=================================================================
    n |  Subint |            m=0           m=1           m=2
  ----------------------------------------------------------
    0 |       1 |   [1314.000000]*           ---            ---
    1 |       2 |     936.000000   [810.000000]*           ---
    2 |       4 |     841.500000     810.000000   [810.000000]*

  Keterangan: [nilai]* = diagonal (hasil ekstrapolasi terbaik)
              Nilai pojok kanan-bawah = HASIL AKHIR

=================================================================
  KONVERGENSI PER ITERASI
=================================================================
    Iter |    Nilai R(n,n) |   Error Relatif | Status
  --------------------------------------------------------
       0 |   1314.00000000 |          (awal) | -
       1 |    810.00000000 |    62.22222222% | iterasi...
       2 |    810.00000000 |     0.00000000% | KONVERGEN ✓

=================================================================
  PERBANDINGAN METODE (n = 6 subinterval)
=================================================================
  Metode                         |          Hasil |   Selisih vs Romberg
  ----------------------------------------------------------------------
  Trapezoidal (n=6)              |   824.00000000 | 14.00000000 (1.7284%)
  Trapezoidal (n=12)             |   813.50000000 | 3.50000000 (0.4321%)
  Simpson 1/3 (n=6)              |   810.00000000 | 0.00000000 (0.0000%)
  Simpson 3/8 (n=6)              |   810.00000000 | 0.00000000 (0.0000%)
  Romberg (ordo=2)               |   810.00000000 |          (referensi)

=================================================================
  HASIL AKHIR
=================================================================
  Integral f(x) dx dari x=1.0 sampai x=7.0

  Hasil Romberg     = 810.0000000000
  Ordo tercapai    = 2
  Subinterval      = 4
  Evaluasi f(x)    = 5 kali
  Toleransi        = 1e-08
  Status           = KONVERGEN ✓
=================================================================
```

**Contoh kedua — f(x) = sin(x) dari 0 sampai π (nilai eksak = 2.0):**

```
=================================================================
  TABEL ROMBERG  R(n, m)
=================================================================
    n |  Subint |       m=0      m=1      m=2      m=3      m=4      m=5
  -----------------------------------------------------------------------
    0 |       1 |  [0.000000]*    ---      ---      ---      ---      ---
    1 |       2 |    1.570796  [2.094395]* ---      ---      ---      ---
    2 |       4 |    1.896119   2.004560 [1.998571]* ---     ---      ---
    3 |       8 |    1.974232   2.000269  1.999983 [2.000006]* ---    ---
    4 |      16 |    1.993570   2.000017  2.000000  2.000000 [2.000000]* ---
    5 |      32 |    1.998393   2.000001  2.000000  2.000000  2.000000 [2.000000]*

=================================================================
  KONVERGENSI PER ITERASI
=================================================================
    Iter |    Nilai R(n,n) |   Error Relatif | Status
  --------------------------------------------------------
       0 |      0.00000000 |          (awal) | -
       1 |      2.09439510 |   100.00000000% | iterasi...
       2 |      1.99857073 |     4.79464495% | iterasi...
       3 |      2.00000555 |     0.07174071% | iterasi...
       4 |      1.99999999 |     0.00027777% | KONVERGEN ✓
       5 |      2.00000000 |     0.00000027% | KONVERGEN ✓

=================================================================
  HASIL AKHIR
=================================================================
  Hasil Romberg     = 2.0000000000   (eksak: 2.0)
  Ordo tercapai    = 5
  Evaluasi f(x)    = 33 kali
  Status           = KONVERGEN ✓
=================================================================
```

---

## Kode Lengkap

```python
import math
import sys

SAFE_NAMESPACE = {
    "sin": math.sin, "cos": math.cos, "tan": math.tan,
    "asin": math.asin, "acos": math.acos, "atan": math.atan,
    "exp": math.exp, "log": math.log, "log10": math.log10,
    "sqrt": math.sqrt, "abs": abs, "pi": math.pi, "e": math.e,
    "sinh": math.sinh, "cosh": math.cosh, "tanh": math.tanh,
}

def parse_function(expr_str):
    expr = expr_str.replace("^", "**")
    def f(x):
        ns = dict(SAFE_NAMESPACE)
        ns["x"] = x
        try:
            result = eval(expr, {"__builtins__": {}}, ns)
            return float(result)
        except Exception as err:
            raise ValueError(f"Error evaluasi f({x}): {err}")
    return f

def parse_value(val_str):
    ns = dict(SAFE_NAMESPACE)
    try:
        return float(eval(val_str.replace("^", "**"), {"__builtins__": {}}, ns))
    except Exception:
        raise ValueError(f"Nilai '{val_str}' tidak valid.")

def trapezoidal(f, a, b, n):
    h = (b - a) / n
    total = f(a) + f(b)
    for i in range(1, n):
        total += 2.0 * f(a + i * h)
    return (h / 2.0) * total

def simpson_1_3(f, a, b, n):
    if n % 2 != 0:
        n += 1
    h = (b - a) / n
    total = f(a) + f(b)
    for i in range(1, n):
        koef = 4 if i % 2 != 0 else 2
        total += koef * f(a + i * h)
    return (h / 3.0) * total

def simpson_3_8(f, a, b, n):
    if n % 3 != 0:
        n = n + (3 - n % 3)
    h = (b - a) / n
    total = f(a) + f(b)
    for i in range(1, n):
        koef = 2 if i % 3 == 0 else 3
        total += koef * f(a + i * h)
    return (3.0 * h / 8.0) * total

def romberg(f, a, b, max_order=10, toleransi=1e-8):
    R = []
    jumlah_eval = 0
    for n in range(max_order):
        baris = [0.0] * (n + 1)
        h = (b - a) / (2 ** n)
        if n == 0:
            baris[0] = (h / 2.0) * (f(a) + f(b))
            jumlah_eval += 2
        else:
            titik_lama = 2 ** (n - 1)
            jumlah_baru = 0.0
            for k in range(1, titik_lama + 1):
                jumlah_baru += f(a + (2 * k - 1) * h)
                jumlah_eval += 1
            baris[0] = 0.5 * R[n - 1][0] + h * jumlah_baru
        for m in range(1, n + 1):
            faktor = 4 ** m
            baris[m] = (faktor * baris[m - 1] - R[n - 1][m - 1]) / (faktor - 1)
        R.append(baris)
        if n >= 1:
            nilai_lama = R[n - 1][n - 1]
            nilai_baru = R[n][n]
            if abs(nilai_baru) > 1e-15:
                error_rel = abs((nilai_baru - nilai_lama) / nilai_baru)
            else:
                error_rel = abs(nilai_baru - nilai_lama)
            if error_rel < toleransi:
                return R, True, n, jumlah_eval
    return R, False, max_order - 1, jumlah_eval

GARIS = "=" * 65
GARIS_TIPIS = "-" * 65

def cetak_tabel_romberg(R, n_final):
    print("\n" + GARIS)
    print("  TABEL ROMBERG  R(n, m)")
    print(GARIS)
    header = f"  {'n':>3} | {'Subint':>7} | "
    for m in range(n_final + 1):
        header += f"{'m=' + str(m):>14}"
    print(header)
    print("  " + "-" * (16 + 14 * (n_final + 1)))
    for n in range(n_final + 1):
        num_interval = 2 ** n
        baris_str = f"  {n:>3} | {num_interval:>7} | "
        for m in range(n_final + 1):
            if m <= n:
                val = R[n][m]
                if m == n:
                    baris_str += f"  [{val:>10.6f}]*"
                else:
                    baris_str += f"  {val:>12.6f} "
            else:
                baris_str += f"  {'---':>12} "
        print(baris_str)
    print()
    print("  Keterangan: [nilai]* = diagonal (hasil ekstrapolasi terbaik)")
    print("              Nilai pojok kanan-bawah = HASIL AKHIR")

def cetak_konvergensi(R, n_final):
    print("\n" + GARIS)
    print("  KONVERGENSI PER ITERASI")
    print(GARIS)
    print(f"  {'Iter':>6} | {'Nilai R(n,n)':>15} | {'Error Relatif':>15} | Status")
    print("  " + "-" * 56)
    for n in range(n_final + 1):
        val = R[n][n]
        if n == 0:
            print(f"  {n:>6} | {val:>15.8f} | {'(awal)':>15} | -")
        else:
            val_lama = R[n - 1][n - 1]
            if abs(val) > 1e-15:
                err = abs((val - val_lama) / val) * 100
            else:
                err = abs(val - val_lama) * 100
            status = "KONVERGEN ✓" if err < 0.001 else "iterasi..."
            print(f"  {n:>6} | {val:>15.8f} | {err:>14.8f}% | {status}")

def cetak_perbandingan(f, a, b, hasil_romberg, n_final):
    print("\n" + GARIS)
    print("  PERBANDINGAN METODE (n = 6 subinterval)")
    print(GARIS)
    hasil = {}
    hasil["Trapezoidal (n=6)"]   = trapezoidal(f, a, b, 6)
    hasil["Trapezoidal (n=12)"]  = trapezoidal(f, a, b, 12)
    hasil["Simpson 1/3 (n=6)"]   = simpson_1_3(f, a, b, 6)
    hasil["Simpson 3/8 (n=6)"]   = simpson_3_8(f, a, b, 6)
    hasil[f"Romberg (ordo={n_final})"] = hasil_romberg
    print(f"\n  {'Metode':<30} | {'Hasil':>14} | {'Selisih vs Romberg':>20}")
    print("  " + "-" * 70)
    for nama, val in hasil.items():
        selisih = abs(val - hasil_romberg)
        if nama.startswith("Romberg"):
            print(f"  {nama:<30} | {val:>14.8f} | {'(referensi)':>20}")
        else:
            persen = (selisih / abs(hasil_romberg)) * 100 if abs(hasil_romberg) > 1e-15 else 0
            print(f"  {nama:<30} | {val:>14.8f} | {selisih:>10.8f} ({persen:.4f}%)")

def cetak_hasil_akhir(hasil, a, b, n_final, jumlah_eval, konvergen, toleransi):
    print("\n" + GARIS)
    print("  HASIL AKHIR")
    print(GARIS)
    print(f"  Integral f(x) dx dari x={a} sampai x={b}")
    print(f"\n  Hasil Romberg     = {hasil:.10f}")
    print(f"  Ordo tercapai    = {n_final}")
    print(f"  Subinterval      = {2**n_final}")
    print(f"  Evaluasi f(x)    = {jumlah_eval} kali")
    print(f"  Toleransi        = {toleransi}")
    print(f"  Status           = {'KONVERGEN ✓' if konvergen else 'Batas ordo tercapai'}")
    print(GARIS + "\n")

def main():
    print("""
╔══════════════════════════════════════════════════════════════╗
║         PROGRAM INTEGRASI ROMBERG - METODE NUMERIK          ║
║         Tugas Praktikum #3 - Komputasi Numerik (PPT-06)     ║
╚══════════════════════════════════════════════════════════════╝
    Petunjuk Input Fungsi:
    - Gunakan 'x' sebagai variabel.
    - Pangkat gunakan tanda sisipan (^) atau bintang ganda (**), misal: x^3 atau x**3
    - Perkalian gunakan bintang (*), misal: 2*x
    - Fungsi matematika: sin(x), cos(x), tan(x), exp(x), log(x), sqrt(x)
    - Konstanta: pi (untuk π), e (untuk bilangan Euler)
    - Contoh: x^3 + 2*x^2 - x + 1
    """)

    while True:
        try:
            expr = input("Masukkan fungsi f(x) = ").strip()
            if not expr:
                print("[!] Fungsi tidak boleh kosong. Silakan coba lagi.\n")
                continue
            f = parse_function(expr)
            f(1.0)
            break
        except Exception as err:
            print(f"[!] Fungsi tidak valid: {err}\n")

    try:
        a_str = input("Masukkan batas bawah a              : ").strip()
        b_str = input("Masukkan batas atas  b              : ").strip()
        a = parse_value(a_str)
        b = parse_value(b_str)
        if b <= a:
            print("\n[ERROR] Batas atas harus lebih besar dari batas bawah.")
            return
        ordo_str  = input("Masukkan ordo maksimum (default=8)  : ").strip()
        tol_str   = input("Masukkan toleransi error (default=1e-8): ").strip()
        max_order = int(ordo_str)   if ordo_str  else 8
        toleransi = float(tol_str)  if tol_str   else 1e-8
    except ValueError:
        print("\n[ERROR] Input tidak valid! Pastikan Anda memasukkan angka yang benar.")
        return

    print("\n  Menghitung...\n")

    try:
        R, konvergen, n_final, jumlah_eval = romberg(f, a, b, max_order, toleransi)
    except Exception as err:
        print(f"\n[ERROR] Gagal menghitung: {err}")
        return

    hasil_akhir = R[n_final][n_final]

    cetak_tabel_romberg(R, n_final)
    cetak_konvergensi(R, n_final)
    cetak_perbandingan(f, a, b, hasil_akhir, n_final)
    cetak_hasil_akhir(hasil_akhir, a, b, n_final, jumlah_eval, konvergen, toleransi)

    if konvergen:
        print(f"[KESIMPULAN - BERHASIL]")
        print(f"Integral ditemukan setelah {n_final + 1} iterasi Romberg")
        print(f"Nilai Integral ≈ {hasil_akhir:.10f}")
    else:
        print("[KESIMPULAN - BATAS ORDO TERCAPAI]")
        print(f"Hasil terbaik  ≈ {hasil_akhir:.10f}")

    lagi = input("\nHitung fungsi lain? (y/n) : ").strip().lower()
    if lagi == 'y':
        main()

if __name__ == "__main__":
    main()
```
