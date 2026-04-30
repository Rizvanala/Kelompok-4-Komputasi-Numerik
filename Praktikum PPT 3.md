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
- def metode_secant(...): Membuat fungsi komputasi yang menerima 5 inputan user: teks fungsi, dua tebakan awal, toleransi, dan maksimal iterasi.
- sp.Symbol('x'): Memberitahu program bahwa huruf 'x' adalah sebuah variabel matematika.
- sp.sympify(): Adalah "Penerjemah". Tugasnya mengubah teks inputan dari pengguna (string) menjadi ekspresi matematika agar bisa dipahami komputer.
- sp.lambdify(): Mengubah ekspresi tersebut menjadi sebuah fungsi Python (disimpan di variabel f) yang siap dieksekusi atau dimasukkan angka (misal: f(2)).Blok try-except di sini berguna untuk menangkap error.

---

# TABEL ITERASI
```python
print("\n" + "="*85)
    print(f"{'Iterasi':>7} | {'x(i-1)':>12} | {'x(i)':>12} | {'f(x_i)':>14} | {'x(i+1)':>14} | {'Error':>12}")
    print("="*85)
```

Untuk user interface (UI). Mencetak batas dan judul kolom dalam bentuk tabel untuk data iterasi. Tanda >7 atau >12 berfungsi untuk meratakan teks ke kanan (right-align).

---

# MAIN LOOPING DAN ALGORITMA SECANT
```python
for i in range(1, maks_iterasi + 1):
        try:
            f_x0 = f(x0)
            f_x1 = f(x1)
            
            if f_x1 - f_x0 == 0:
                return False, f"Dihentikan: Pembagian dengan nol..."
```

- for i in range(...): Memulai iterasi dari 1 sampai batas maksimal.
- Menghitung nilai fungsi pada tebakan pertama dan kedua.
- Pencegah Error Fatal: Rumus Secant mengharuskan pembagian dengan `(f(x_1) - f(x_0))`. Jika kedua nilai fungsi ini sama, hasil pengurangannya adalah 0. Jika pembagian dengan nol, program akan berhenti (return False) proses iterasi.

**Formula Secant dan Menghitung Error:**
```python
x2 = x1 - f_x1 * ((x1 - x0) / (f_x1 - f_x0))
            f_x2 = f(x2)
            error = abs(x2 - x1)
```

**Print Baris Data dan Cek Konvergen**
```python
print(f"{i:>7} | {x0:>12.6f} | {x1:>12.6f} | {f_x1:>14.6f} | {x2:>14.6f} | {error:>12.6f}")
            
            if error < toleransi:
                print("="*85)
                return True, (x2, i, f_x2)
                
            x0 = x1
            x1 = x2
```

---

## USER INTERFACE 
```python
def main():
    print(....)

    while True:
        fungsi_input = input("Masukkan fungsi f(x) = ")
        if fungsi_input.strip() == "":
            print("Fungsi tidak boleh kosong. Silakan coba lagi.\n")
            continue
        break
```

Tempat user memberikan input. Jika user tidak memberikan input (hanya enter), maka keluar peringatan. Jika sudah ada input dari user, break akan menghentikan siklus ini dan lanjut ke baris berikutnya.


***5 inputan user: teks fungsi, dua tebakan awal, toleransi, dan maksimal iterasi***
```python
try:
        x_awal = float(input("Masukkan tebakan awal pertama (x0) : "))
        x_kedua = float(input("Masukkan tebakan awal kedua (x1)   : "))
        tol = float(input("Masukkan nilai toleransi (e.g. 0.0001): "))
        maks_iter = int(input("Masukkan maksimal iterasi (e.g. 50)  : "))
    except ValueError:
        print("\n[ERROR] Input tidak valid! Pastikan Anda memasukkan angka...")
        return
```
- Meminta user input parameter matematis
- float(...) mengubah input teks menjadi angka desimal, dan int(...) untuk angka bulat.
- Blok try-except ValueError digunakan untuk berjaga-jaga jika user memasukkan input huruf saat diminta memasukkan angka (ex: input kata "dua" bukan angka "2").

---

## FINAL EXECUTION & SIMPULAN
```python
sukses, hasil = metode_secant(fungsi_input, x_awal, x_kedua, tol, maks_iter)

    if sukses:
        akar, total_iterasi, nilai_fungsi = hasil
        print("\n[KESIMPULAN - BERHASIL]")

    else:
        print("\n[KESIMPULAN - GAGAL/PERINGATAN]")
        print(hasil)

if __name__ == "__main__":
    main()
```

- sukses, hasil = ...: Menjalankan algoritma dengan data dari user. Fungsi metode_secant akan mengembalikan dua nilai: status (True/False) dan datanya.
- Blok if sukses: Jika algoritma berjalan lancar tanpa pembagian nol atau error aneh, data hasil akan diproses (akar, total_iterasi, nilai_fungsi) lalu diprint.
- Jika gagal (misal karena limit iterasi habis atau salah ketik rumus), program akan mencetak status peringatannya.if __name__ == "__main__"


