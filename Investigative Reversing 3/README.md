# 🚩 Write-up Investigative Reversing 3 - picoCTF 2019

Selamat datang di repositori saya! Di sini saya mendokumentasikan perjalanan saya dalam menyelesaikan berbagai tantangan Capture The Flag (CTF).

---

## 📋 Daftar Isi

- [Deskripsi Soal](#deskripsi-soal)
- [Analisis](#analisis)
- [Solusi](#solusi)
- [Flag](#flag)
- [File Terkait](#file-terkait)

---

## 📝 Deskripsi Soal

- **Kategori:** Forensic
- **Challenge:** Investigative Reversing 3
- **Deskripsi:** Diberikan sebuah file biner (`mystery`) yang berisi gambar BMP dengan flag tersembunyi menggunakan teknik steganografi LSB.

![Deskripsi Soal](images/soal.png)
![Petunjuk Soal](images/clue.png)

---

## 🔍 Analisis

Analisis pada biner `mystery` menggunakan Ghidra menunjukkan bahwa program ini adalah sebuah **enkoder steganografi**. Ditemukan pola enkripsi LSB (Least Significant Bit) yang unik:

- Setelah 723 byte header
- 8 byte gambar digunakan untuk menyembunyikan 1 karakter flag
- Lalu 1 byte disalin biasa

![Logika di Ghidra](images/ghidra.png)

---

## 💡 Solusi

Solusinya adalah dengan membuat skrip Python untuk membalik proses tersebut. Skrip ini membaca `encoded.bmp`, melewati header, dan mengekstrak bit-bit tersembunyi sesuai pola yang ditemukan untuk merekonstruksi flag.

```python
# solve_ir3.py
def solve_ir3():
    with open('encoded.bmp', 'rb') as f:
        f.seek(723)  # Skip header
        flag = ""
        
        for i in range(50):
            char_byte = 0
            
            # Ekstrak 8 LSB dari 8 byte gambar
            for j in range(8):
                img_byte = f.read(1)
                lsb = ord(img_byte) & 1
                char_byte = (char_byte << 1) | lsb
            
            # Konversi byte ke karakter
            flag += chr(int(bin(char_byte)[2:].zfill(8)[::-1], 2))
            f.read(1)  # Skip 1 byte biasa
            
        print(flag)

if __name__ == "__main__":
    solve_ir3()
```

---

## 🏴 Flag

Setelah menjalankan skrip, flag yang didapatkan adalah:

`picoCTF{...jalankan skrip untuk mendapatkan flag...}`

![Hasil Eksekusi](images/hasil.png)

---

## 📁 File Terkait

- `solve_ir3.py` - Script solusi Python
- `encoded.bmp` - File gambar dengan flag tersembunyi
- `mystery` - File biner asli
- `writeup_ir3.md` - Dokumentasi lengkap write-up

---

## 📚 Teknik yang Dipelajari

- Reverse Engineering dengan Ghidra
- Steganografi LSB (Least Significant Bit)
- Manipulasi file biner dengan Python
- Analisis struktur file BMP

---
