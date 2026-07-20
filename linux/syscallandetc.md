# Linux Internals #1 — System Call

> Catatan belajar mengenai konsep dasar **System Call**, hubungan antara User Space, Kernel Space, dan bagaimana sebuah program berkomunikasi dengan kernel Linux.

---

# Daftar Isi

1. Apa itu System Call?
2. Arsitektur Linux
3. User Space vs Kernel Space
4. Mengapa System Call Dibutuhkan?
5. Alur Kerja Sebuah System Call
6. Exit Status
7. libc (GNU C Library)
8. Hubungan Bash dengan System Call
9. Kesimpulan

---

# 1. Apa itu System Call?

## Definisi

**System Call** adalah **interface** yang digunakan oleh program di **User Space** untuk meminta layanan (service) dari **Kernel**.

Perhatikan kata **interface**.

Interface di sini **bukan GUI**, melainkan sebuah **batas komunikasi (communication boundary)** antara dua komponen yang berbeda.

Dalam kasus Linux:

```
User Space
      │
      ▼
System Call Interface
      │
      ▼
Kernel Space
```

Program **tidak pernah** mengakses kernel secara langsung.

Program hanya dapat meminta layanan melalui system call.

---

# 2. Arsitektur Linux

Secara sederhana Linux dibagi menjadi tiga bagian.

```
+------------------------------------+
|           User Space               |
|------------------------------------|
| Bash                               |
| Firefox                            |
| VS Code                            |
| cat                                |
| grep                               |
| ls                                 |
| Program C                          |
+------------------------------------+
               │
        System Call Interface
               │
+------------------------------------+
|          Kernel Space              |
|------------------------------------|
| Process Scheduler                  |
| Memory Manager                     |
| Virtual File System                |
| Network Stack                      |
| Device Driver                      |
| Security                           |
| IPC                                |
+------------------------------------+
               │
+------------------------------------+
|            Hardware                |
|------------------------------------|
| CPU                                |
| RAM                                |
| SSD                                |
| GPU                                |
| NIC                                |
| Keyboard                           |
+------------------------------------+
```

Program hanya berjalan di **User Space**.

Kernel berjalan di **Kernel Space**.

---

# 3. User Space vs Kernel Space

## User Space

Tempat semua aplikasi berjalan.

Contoh:

- Bash
- Firefox
- VS Code
- grep
- cat
- ls
- Program C

Program di User Space memiliki hak akses terbatas.

Program **tidak dapat** secara langsung:

- Membaca SSD
- Mengakses RAM Kernel
- Membuat Process
- Mengontrol CPU
- Mengakses Driver

---

## Kernel Space

Kernel Space memiliki hak akses penuh terhadap seluruh hardware.

Kernel bertugas mengelola:

- Process
- Memory
- File System
- Driver
- Network
- Security
- Scheduler

Semua operasi penting dilakukan oleh kernel.

---

# 4. Mengapa System Call Dibutuhkan?

Karena program biasa **tidak memiliki izin** untuk mengakses hardware secara langsung.

Misalnya program ingin membuka file.

Program tidak bisa melakukan:

```
Program
    │
    ▼
SSD
```

Yang benar adalah:

```
Program
    │
    ▼
System Call
    │
    ▼
Kernel
    │
    ▼
Filesystem
    │
    ▼
SSD
```

Program hanya meminta.

Kernel yang benar-benar melakukan pekerjaan.

---

# 5. Alur Kerja Sebuah System Call

Misalnya program memanggil:

```c
open("file.txt", O_RDONLY);
```

Yang terjadi:

```
Program

↓

System Call open()

↓

Kernel

↓

Virtual File System

↓

Filesystem Driver

↓

SSD

↓

Kernel

↓

Program
```

Kernel akan:

1. Memeriksa permission
2. Memastikan file ada
3. Membuka file
4. Mengembalikan File Descriptor

Jika gagal:

Kernel mengembalikan error.

---

# 6. Exit Status

## Definisi

Exit Status adalah **nilai integer** yang dikembalikan oleh suatu process kepada parent process ketika process tersebut selesai dijalankan.

Rentang nilainya:

```
0 - 255
```

Dalam Linux:

```
0
```

berarti

```
Success
```

Sedangkan selain 0 berarti proses selesai tetapi tidak berhasil sesuai definisi program tersebut.

Contoh:

```
grep
```

Exit Status:

```
0
```

Pattern ditemukan.

```
1
```

Pattern tidak ditemukan.

```
2
```

Terjadi error.

---

## Alur Exit Status

Misalnya:

```bash
grep ERROR sample.log
```

Yang terjadi:

```
Bash

↓

fork()

↓

grep

↓

grep selesai

↓

return 0

↓

Kernel menyimpan Exit Status

↓

Bash memanggil wait()

↓

Kernel memberikan Exit Status

↓

Bash menyimpan ke $?
```

---

## $?

Bash menyimpan Exit Status terakhir ke dalam:

```bash
$?
```

Contoh:

```bash
ls

echo $?
```

Output:

```
0
```

---

# 7. libc (GNU C Library)

## Apa itu libc?

libc adalah singkatan dari:

```
C Standard Library
```

Implementasi yang paling umum di Linux adalah:

```
glibc
```

libc menyediakan ribuan fungsi yang digunakan programmer.

Contohnya:

```
printf()
scanf()
malloc()
free()
fopen()
fclose()
strlen()
memcpy()
```

---

## Apakah semua fungsi libc adalah System Call?

Tidak.

Contoh:

```c
strlen("Linux")
```

Hanya menghitung jumlah karakter.

Kernel tidak terlibat.

Namun:

```c
printf("Hello");
```

Pada akhirnya akan memanggil:

```
write()
```

yang merupakan System Call.

---

## Contoh fopen()

```c
fopen("test.txt","r")
```

Bukan System Call.

Di dalam implementasinya nanti libc memanggil:

```
open()
```

yang merupakan System Call.

---

## Posisi libc

```
Program

↓

libc

↓

System Call

↓

Kernel

↓

Hardware
```

libc berada di User Space.

Kernel berada di Kernel Space.

---

# 8. Hubungan Bash dengan System Call

Bash hanyalah sebuah program biasa.

Misalnya:

```bash
cat sample.log
```

Yang sebenarnya terjadi:

```
Bash

↓

Menjalankan cat

↓

cat memanggil open()

↓

Kernel membuka file

↓

cat memanggil read()

↓

Kernel membaca data

↓

cat memanggil write()

↓

Kernel menampilkan ke terminal

↓

cat selesai

↓

exit()
```

Bash sendiri tidak memiliki akses langsung ke hardware.

Semua tetap melalui kernel.

---

# 9. Interface vs Request vs Function

Sering muncul pertanyaan:

"System Call itu interface atau function?"

Jawabannya tergantung sudut pandang.

## Dari sisi Arsitektur OS

System Call adalah:

> Interface

Karena menjadi penghubung antara User Space dan Kernel Space.

---

## Dari sisi Programmer C

System Call terlihat seperti:

```c
read()
write()
fork()
```

Karena dipanggil layaknya function.

---

## Dari sisi Kernel

System Call merupakan:

> Request

Program sedang meminta layanan kepada kernel.

Kernel boleh:

- Mengizinkan
- Menolak
- Mengembalikan Error

---

# 10. Kesimpulan

- Linux dibagi menjadi User Space, Kernel Space, dan Hardware.
- Program hanya berjalan di User Space.
- Program tidak boleh mengakses hardware secara langsung.
- Semua akses hardware harus melalui Kernel.
- System Call adalah interface yang menghubungkan User Space dengan Kernel Space.
- Program menggunakan System Call untuk meminta layanan kernel.
- Kernel menjalankan operasi yang diminta kemudian mengembalikan hasilnya.
- Exit Status adalah nilai yang dikembalikan process kepada parent process setelah selesai dijalankan.
- Bash menyimpan Exit Status terakhir pada variabel khusus `$?`.
- libc adalah pustaka standar C yang menyediakan fungsi-fungsi seperti `printf()`, `malloc()`, dan `fopen()`. Beberapa fungsi libc akan menggunakan System Call untuk berkomunikasi dengan kernel.

---

# Materi Selanjutnya

## Linux Internals #2

**CPU Mode & Privilege Level**

Materi yang akan dibahas:

- User Mode
- Kernel Mode
- Ring 0 dan Ring 3
- Mengapa User Space tidak boleh mengakses hardware
- Apa yang terjadi ketika instruksi `syscall` dijalankan
- Perpindahan dari User Mode ke Kernel Mode
- Bagaimana CPU berpindah konteks saat System Call terjadi
