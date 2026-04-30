# METODE SECANT 

---

## Penjelasan Tiap Blok Kode

# IMPORT LIBRARY
```python
import sympy as sp
```
mengimpor library sympy (Symbolic Python). Library ini dibuat agar nantinya program bisa membaca input string seperti "x2 - 4".

---

## DEFINISI FUNGSI DAN TRANSLATE INPUT
```python
def metode_secant(fungsi_str, x0, x1, toleransi=1e-5, maks_iterasi=50):
    x = sp.Symbol('x')
    
    try:
        ekspresi = sp.sympify(fungsi_str)
        f = sp.lambdify(x, ekspresi, 'math')
    except Exception as e:
        return False, f"Error saat membaca fungsi: {e}"
```
- def metode_secant(...): Membuat fungsi komputasi utama yang menerima 5 inputan user: teks fungsi, dua tebakan awal, toleransi, dan maksimal iterasi.
- sp.Symbol('x'): Memberitahu program bahwa huruf 'x' adalah sebuah variabel matematika, bukan sekadar huruf biasa.
- sp.sympify(): Ini adalah "penerjemah". Tugasnya mengubah teks inputan dari pengguna (string) menjadi struktur ekspresi matematika agar bisa dipahami komputer.
- sp.lambdify(): Mengubah ekspresi tersebut menjadi sebuah fungsi Python (disimpan di variabel f) yang siap dieksekusi atau dimasukkan angka (misal: f(2)).Blok try-except di sini berguna untuk menangkap error.
