# Regula Falsi

---

## Langkah-langkah Pembuatan

Pertama, saya tentukan dulu fungsi yang mau dicari akarnya, di sini saya pakai `f(x) = x³ - x - 2`. Terus saya tentukan interval `[a, b]` yang mengapit akar, dengan syarat `f(a)` dan `f(b)` harus berbeda tanda.

Setelah itu, saya implementasikan rumus Regula Falsi:

```
c = b - f(b) × (b - a) / (f(b) - f(a))
```

Di setiap iterasi, nilai `c` ini dihitung, terus dicek apakah `f(a) × f(c) < 0` atau tidak, untuk menentukan subinterval baru. Proses ini diulang sampai nilai error-nya lebih kecil dari toleransi yang sudah ditentukan (0.0001).

Terakhir, saya tambahkan visualisasi grafik pakai `matplotlib` , satu grafik untuk kurva fungsinya, satu lagi untuk grafik konvergensi error-nya.

---

## Penjelasan Tiap Blok Kode

### 1. Import Library
```python
import matplotlib.pyplot as plt
import numpy as np
```
Mengimpor dua library utama: `numpy` untuk perhitungan numerik (linspace, array), dan `matplotlib` untuk membuat grafik.

---

### 2. Definisi Fungsi f(x)
```python
def f(x):
    return x**3 - x - 2
```
Ini adalah fungsi matematika yang ingin dicari akarnya. Kalau mau ganti soal, cukup ubah bagian `return`nya.

---

### 3. Fungsi Inti Regula Falsi
```python
def regula_falsi(a, b, toleransi=0.0001, maks_iterasi=50):
```
Fungsi utama yang menjalankan metode Regula Falsi. Menerima input: batas kiri `a`, batas kanan `b`, toleransi error, dan maksimal iterasi.

**Cek syarat awal:**
```python
if f(a) * f(b) >= 0:
    raise ValueError(...)
```
Memastikan `f(a)` dan `f(b)` berbeda tanda , syarat wajib supaya metode inj bisa berjalan.

**Perhitungan titik c:**
```python
c = b - fb * (b - a) / (fb - fa)
```
Rumus inti Regula Falsi untuk menghitung titik potong garis dengan sumbu x.

**Update interval:**
```python
if fa * fc < 0:
    b = c
else:
    a = c
```
Menentukan subinterval baru: kalau akar ada di kiri `c`, maka `b` digeser ke `c`, begitu sebaliknya.

**Cek konvergensi:**
```python
if error is not None and error < toleransi:
    return c, tabel
```
Iterasi berhenti ketika selisih antara `c` sekarang dan `c` sebelumnya sudah lebih kecil dari toleransi.

---

### 4. Fungsi Grafik
```python
def buat_grafik(a_awal, b_awal, akar, tabel):
```
Membuat dua grafik sekaligus menggunakan `matplotlib`:

- **Grafik kiri** = menampilkan kurva `f(x)`, letak akar, interval awal, dan titik-titik `c` dari setiap iterasi.
- **Grafik kanan** = menampilkan grafik konvergensi error dalam skala logaritmik, sehingga penurunan error terlihat jelas.

```python
plt.savefig('regula_falsi_grafik.png', dpi=150, bbox_inches='tight')
```
Grafik otomatis disimpan sebagai file PNG di folder yang sama.

---

### 5. Blok Utama (main)
```python
if __name__ == "__main__":
    A         = 1
    B         = 2
    TOLERANSI = 0.0001
    MAKS_ITER = 50
```
Di sini kita atur nilai interval, toleransi, dan maksimal iterasi. **Kalau mau ganti soal, ubah di bagian ini.*

```python
akar, tabel = regula_falsi(A, B, TOLERANSI, MAKS_ITER)
buat_grafik(A, B, akar, tabel)
```
Memanggil fungsi Regula Falsi lalu langsung menampilkan grafiknya.

---

## Output Program

### Screenshot Output & Grafik

> 📷

>  <img width="560" height="429" alt="image" src="https://github.com/user-attachments/assets/1564ed87-1494-4163-ad54-5fe941df3b8c" />


>  <img width="1085" height="446" alt="Screenshot 2026-04-30 211833" src="https://github.com/user-attachments/assets/f5f314c6-7fff-47f0-85b2-8f902638e5c7" />


---
Cara Menjalankan

Program ini dijalankan menggunakan Google Colab.
Buka https://colab.research.google.com
Klik "+ New notebook"
buat kodenya di cell
Klik tombol ▶️ atau tekan Ctrl + Enter

Grafik akan otomatis tampil di bawah cell, dan tersimpan sebagai regula_falsi_grafik.png.

---

## Kode Lengkap

```python
import matplotlib.pyplot as plt
import numpy as np


def f(x):
    return x**3 - x - 2


def regula_falsi(a, b, toleransi=0.0001, maks_iterasi=50):
    if f(a) * f(b) >= 0:
        raise ValueError(f"f(a) dan f(b) harus berbeda tanda! f({a})={f(a):.6f}, f({b})={f(b):.6f}")

    tabel = []
    c_lama = None

    print("=" * 85)
    print(f"{'Iter':>4}  {'a':>12}  {'b':>12}  {'c (akar)':>14}  {'f(c)':>14}  {'Error':>14}")
    print("=" * 85)

    for i in range(1, maks_iterasi + 1):
        fa = f(a)
        fb = f(b)
        c  = b - fb * (b - a) / (fb - fa)
        fc = f(c)

        error = abs(c - c_lama) if c_lama is not None else None
        tabel.append({'iter': i, 'a': a, 'b': b, 'c': c, 'fa': fa, 'fb': fb, 'fc': fc, 'error': error})

        err_str = f"{error:.6e}" if error is not None else "    -     "
        print(f"{i:>4}  {a:>12.6f}  {b:>12.6f}  {c:>14.8f}  {fc:>14.8f}  {err_str:>14}")

        if error is not None and error < toleransi:
            print("=" * 85)
            print(f"\n✓ Konvergen setelah {i} iterasi.")
            print(f"  Akar ≈ {c:.8f}")
            print(f"  f(akar) = {fc:.10f}")
            print(f"  Error   = {error:.2e}\n")
            return c, tabel

        if fa * fc < 0:
            b = c
        else:
            a = c

        c_lama = c

    print("=" * 85)
    print(f"\n⚠ Maks iterasi ({maks_iterasi}) tercapai. Akar terakhir ≈ {c:.8f}\n")
    return c, tabel


def buat_grafik(a_awal, b_awal, akar, tabel):
    margin = abs(b_awal - a_awal) * 1.5
    x = np.linspace(a_awal - margin, b_awal + margin, 500)
    y = np.array([f(xi) for xi in x])

    fig, axes = plt.subplots(1, 2, figsize=(13, 5))
    fig.suptitle("Metode Regula Falsi", fontsize=14, fontweight='bold')

    ax1 = axes[0]
    ax1.plot(x, y, 'steelblue', linewidth=2, label='f(x)')
    ax1.axhline(0, color='black', linewidth=0.8, linestyle='--')
    ax1.axvline(akar, color='orangered', linewidth=1.5, linestyle=':', alpha=0.8)
    ax1.scatter([akar], [f(akar)], color='orangered', zorder=5, s=80, label=f'Akar ≈ {akar:.6f}')
    ax1.axvspan(a_awal, b_awal, alpha=0.08, color='green', label=f'Interval [{a_awal}, {b_awal}]')
    ax1.scatter([r['c'] for r in tabel], [r['fc'] for r in tabel], color='darkorange', s=30, alpha=0.6, zorder=4, label='Titik c iterasi')
    ax1.set_xlabel('x', fontsize=11)
    ax1.set_ylabel('f(x)', fontsize=11)
    ax1.set_title('Grafik Fungsi dan Proses Iterasi', fontsize=11)
    ax1.legend(fontsize=9)
    ax1.grid(True, alpha=0.3)
    ax1.set_ylim(max(-20, min(y)), min(20, max(y)))

    ax2 = axes[1]
    errors = [r['error'] for r in tabel if r['error'] is not None]
    iters  = [r['iter']  for r in tabel if r['error'] is not None]
    ax2.semilogy(iters, errors, 'o-', color='tomato', linewidth=2, markersize=5)
    ax2.set_xlabel('Iterasi', fontsize=11)
    ax2.set_ylabel('Error (skala log)', fontsize=11)
    ax2.set_title('Grafik Konvergensi Error', fontsize=11)
    ax2.grid(True, alpha=0.3, which='both')

    plt.tight_layout()
    plt.savefig('regula_falsi_grafik.png', dpi=150, bbox_inches='tight')
    print("  Grafik disimpan sebagai 'regula_falsi_grafik.png'")
    plt.show()


if __name__ == "__main__":
    A         = 1
    B         = 2
    TOLERANSI = 0.0001
    MAKS_ITER = 50

    print("\n╔══════════════════════════════════════╗")
    print("║   METODE REGULA FALSI                ║")
    print("║   Praktikum Metode Numerik           ║")
    print("╚══════════════════════════════════════╝\n")
    print(f"  Fungsi    : f(x) = x³ - x - 2")
    print(f"  Interval  : [{A}, {B}]")
    print(f"  Toleransi : {TOLERANSI}")
    print(f"  f({A}) = {f(A):.6f}")
    print(f"  f({B}) = {f(B):.6f}\n")

    try:
        akar, tabel = regula_falsi(A, B, TOLERANSI, MAKS_ITER)
        buat_grafik(A, B, akar, tabel)
    except ValueError as e:
        print(f"ERROR: {e}")
```

---
