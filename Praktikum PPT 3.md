import sympy as sp

def metode_secant(fungsi_str, x0, x1, toleransi=1e-5, maks_iterasi=50):
    """
    Fungsi untuk mencari akar persamaan menggunakan Metode Secant.
    """
    x = sp.Symbol('x')
    
    try:
        ekspresi = sp.sympify(fungsi_str)
        
        f = sp.lambdify(x, ekspresi, 'math')
    except Exception as e:
        return False, f"Error saat membaca fungsi: {e}"

    print("\n" + "="*85)
    print(f"{'Iterasi':>7} | {'x(i-1)':>12} | {'x(i)':>12} | {'f(x_i)':>14} | {'x(i+1)':>14} | {'Error':>12}")
    print("="*85)

    for i in range(1, maks_iterasi + 1):
        try:
            f_x0 = f(x0)
            f_x1 = f(x1)
            
            if f_x1 - f_x0 == 0:
                return False, f"Dihentikan: Pembagian dengan nol terdeteksi pada iterasi {i} karena f(x(i)) == f(x(i-1))."
            
            # Rumus Metode Secant
            x2 = x1 - f_x1 * ((x1 - x0) / (f_x1 - f_x0))
            f_x2 = f(x2)
            
            # Error relatif
            error = abs(x2 - x1)
            
            
            print(f"{i:>7} | {x0:>12.6f} | {x1:>12.6f} | {f_x1:>14.6f} | {x2:>14.6f} | {error:>12.6f}")
            
            if error < toleransi:
                print("="*85)
                return True, (x2, i, f_x2)
                
            
            x0 = x1
            x1 = x2
            
        except Exception as e:
             return False, f"Terjadi kesalahan matematis saat iterasi: {e}"

    print("="*85)
    return False, f"Batas maksimum iterasi ({maks_iterasi}) tercapai tanpa mencapai toleransi."

def main():
    print("""
    ╔════════════════════════════════════════════════════════╗
    ║        PROGRAM PENCARI AKAR - METODE SECANT            ║
    ║        (Praktikum Metode Numerik - Tugas 2)            ║
    ╚════════════════════════════════════════════════════════╝
    Petunjuk Input Fungsi:
    - Gunakan 'x' sebagai variabel.
    - Pangkat gunakan double bintang (**), misal: x**3
    - Perkalian gunakan bintang (*), misal: 2*x
    - Fungsi trigonometri/log gunakan format: sin(x), cos(x), exp(x), log(x)
    """)

    while True:
        fungsi_input = input("Masukkan fungsi f(x) = ")
        if fungsi_input.strip() == "":
            print("Fungsi tidak boleh kosong. Silakan coba lagi.\n")
            continue
        break

    try:
        x_awal = float(input("Masukkan tebakan awal pertama (x0) : "))
        x_kedua = float(input("Masukkan tebakan awal kedua (x1)   : "))
        tol = float(input("Masukkan nilai toleransi (e.g. 0.0001): "))
        maks_iter = int(input("Masukkan maksimal iterasi (e.g. 50)  : "))
    except ValueError:
        print("\n[ERROR] Input tidak valid! Pastikan Anda memasukkan angka untuk tebakan, toleransi, dan iterasi.")
        return

    
    sukses, hasil = metode_secant(fungsi_input, x_awal, x_kedua, tol, maks_iter)


    if sukses:
        akar, total_iterasi, nilai_fungsi = hasil
        print("\n[KESIMPULAN - BERHASIL]")
        print(f"Akar persamaan ditemukan pada iterasi ke-{total_iterasi}")
        print(f"Nilai Akar (x) ≈ {akar:.8f}")
        print(f"Nilai f(x)     ≈ {nilai_fungsi:.10f}")
    else:
        print("\n[KESIMPULAN - GAGAL/PERINGATAN]")
        print(hasil)

if __name__ == "__main__":
    main()
