# 🚩 Write-up Investigative Reversing 4 - picoCTF 2019

## 📋 Informasi Tantangan
- **Kategori:** Forensics  
- **Tingkat Kesulitan:** Medium
- **File yang Diberikan:**
  - `mystery` (file biner executable)
  - `Item01_cp.bmp` hingga `Item05_cp.bmp` (5 file gambar)

![Deskripsi Soal](images/soal.png)
![Petunjuk Soal](images/clue.png)

---

## 🔍 Enumerasi Awal

Dari enumerasi file, kita memiliki:
- **5 file gambar BMP:** `Item01_cp.bmp` hingga `Item05_cp.bmp`
- **1 file biner:** `mystery`

Karena ada file biner executable, fokus investigasi adalah melakukan reverse engineering pada file `mystery` untuk memahami bagaimana program berinteraksi dengan kelima gambar tersebut.

---

## ⚙️ Analisis Reverse Engineering

Menggunakan **Ghidra**, logika internal program `mystery` berhasil dianalisis dalam tiga tahap:

### 1. Fungsi `main()`
![Fungsi Main](images/funcMain.png)
Analisis menunjukkan bahwa program ini adalah sebuah **enkoder** yang:
- Membaca flag dari file `flag.txt` (yang ada di server)
- Memanggil fungsi `encodeAll()` untuk memulai proses penyembunyian data

### 2. Fungsi `encodeAll()`
![Fungsi encodeAll](images/edcodeAll.png)
Fungsi ini mengandung logika kunci:
- Loop yang berjalan **mundur** dari 5 ke 1
- Program memproses file gambar dalam urutan terbalik: **Item05 → Item04 → Item03 → Item02 → Item01**
- Setiap iterasi memanggil `encodeDataInFile()` untuk melakukan enkripsi pada satu gambar

### 3. Fungsi `encodeDataInFile()`
![Fungsi encodeDataInFile](images/encodeDataInFile.png)
Ini adalah inti mekanisme steganografi:
- **Header Offset:** Melewati **2019 byte** (0x7e3) pertama dari setiap gambar
- **Pola Enkripsi:** Menyembunyikan **10 karakter flag** per gambar dengan pola:
  - Sembunyikan 1 karakter dalam 8 byte menggunakan LSB
  - Lalu salin 4 byte biasa (di-skip)

---

## 💻 Skrip Solusi

Berdasarkan analisis reverse engineering, dibuat skrip Python untuk membalik proses enkripsi:

```python
# solveFinal.py

def solve():
    # Daftar file dalam urutan terbalik (sesuai analisis encodeAll)
    files = [
        "Item05_cp.bmp",
        "Item04_cp.bmp", 
        "Item03_cp.bmp",
        "Item02_cp.bmp",
        "Item01_cp.bmp"
    ]

    full_flag = ""
    
    for filename in files:
        print(f"[*] Processing {filename}...")
        with open(filename, 'rb') as f:
            # Lewati header BMP sebesar 2019 byte
            f.seek(2019)

            # Setiap file menyembunyikan 10 karakter
            for _ in range(10):
                char_byte = 0
                
                # Rekonstruksi 1 karakter dari 8 byte gambar
                for _ in range(8):
                    img_byte = f.read(1)
                    if not img_byte:
                        raise EOFError("File ended unexpectedly.")
                    
                    # Ekstrak LSB (Least Significant Bit)
                    lsb = ord(img_byte) & 1
                    char_byte = (char_byte << 1) | lsb
                
                # Balik urutan bit dan konversi ke karakter
                full_flag += chr(int(bin(char_byte)[2:].zfill(8)[::-1], 2))

                # Lewati 4 byte sesuai pola enkoder
                f.read(4)

    print("\n[+] Flag ditemukan!")
    print(full_flag)

if __name__ == '__main__':
    solve()
```

---

## 🏴‍☠️ Eksekusi dan Flag

Setelah menjalankan skrip `solveFinal.py`, program berhasil memproses semua file dalam urutan yang benar dan mengekstrak flag yang tersembunyi:

**Flag:** `picoCTF{N1c3_R3ver51ng_5k1115_0000000000b93ee6e2}`

---

## 📚 Teknik yang Dipelajari

1. **Reverse Engineering** dengan Ghidra
2. **Analisis File BMP** dan struktur header
3. **Steganografi LSB** (Least Significant Bit)
4. **Manipulasi File Biner** dengan Python
5. **Pemahaman Pola Enkripsi** custom

---

## 🎯 Kesimpulan

Tantangan ini mengajarkan pentingnya memahami alur program secara keseluruhan melalui reverse engineering. Kunci solusinya adalah:
- Mengidentifikasi urutan pemrosesan file yang **terbalik**
- Memahami pola **LSB encoding** yang digunakan
- Mengimplementasikan **decoder** yang membalik proses enkripsi
