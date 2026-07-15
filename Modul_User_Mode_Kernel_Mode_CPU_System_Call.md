# Modul: Memahami User Mode, Kernel Mode, CPU, System Call, dan Register (Linux x86-64)

## Pendahuluan

Modul ini merangkum konsep dasar bagaimana **CPU**, **kernel Linux**,
dan **program user** bekerja sama ketika sebuah program dijalankan.
Fokus utama modul ini adalah memahami:

-   User Mode vs Kernel Mode
-   Current Privilege Level (CPL)
-   Protection Ring
-   System Call
-   Register CPU (RAX)
-   Peran CPU dan Kernel
-   Hubungan dengan Hardware
-   Pengantar Kernel dan initramfs

------------------------------------------------------------------------

# 1. Gambaran Besar

Pada Linux terdapat tiga komponen utama:

``` text
Program User
Kernel
Hardware
```

Namun jika dilihat lebih detail, CPU selalu berada di tengah sebagai
pihak yang benar-benar mengeksekusi instruksi.

``` text
Program User
      │
      ▼
CPU
      │
      ▼
Kernel
      │
      ▼
Hardware
```

CPU adalah satu-satunya komponen yang benar-benar menjalankan instruksi.

Kernel hanyalah kumpulan kode yang berada di RAM.

------------------------------------------------------------------------

# 2. User Mode dan Kernel Mode

CPU memiliki dua mode utama.

## User Mode

Digunakan untuk menjalankan aplikasi biasa.

Contoh:

-   Bash
-   Chrome
-   Firefox
-   VSCode
-   Game

Pada mode ini program tidak memiliki hak penuh terhadap sistem.

Program tidak boleh:

-   mengakses hardware langsung
-   membuat process sendiri
-   memodifikasi page table
-   mengakses memory milik process lain

------------------------------------------------------------------------

## Kernel Mode

Digunakan untuk menjalankan kode kernel.

Kernel memiliki hak penuh terhadap seluruh resource sistem.

Kernel boleh:

-   membaca disk
-   mengakses RAM
-   mengendalikan device
-   membuat process
-   mengatur scheduler
-   mengelola filesystem

------------------------------------------------------------------------

# 3. CPL (Current Privilege Level)

CPL adalah singkatan dari:

**Current Privilege Level**

CPL merupakan status privilege yang sedang aktif pada CPU.

Pada arsitektur x86 terdapat konsep Protection Ring.

``` text
Ring 0 = Kernel
Ring 1 = Jarang digunakan
Ring 2 = Jarang digunakan
Ring 3 = User
```

Linux modern hampir selalu menggunakan:

``` text
Ring 0
Ring 3
```

Saat CPU menjalankan kode user:

``` text
CPL = 3
```

Saat CPU menjalankan kernel:

``` text
CPL = 0
```

Yang penting dipahami:

**CPL adalah milik CPU, bukan milik process.**

------------------------------------------------------------------------

# 4. CPU Selalu Menjalankan Instruksi

CPU selalu melakukan siklus berikut:

``` text
Fetch

↓

Decode

↓

Execute
```

Baik kode user maupun kernel selalu dijalankan oleh CPU.

Kernel tidak pernah berjalan sendiri.

------------------------------------------------------------------------

# 5. Mengapa Harus Ada Kernel?

Program user tidak boleh langsung mengakses hardware.

Misalnya program ingin membaca file.

Program tidak boleh melakukan:

``` text
Program

↓

SSD
```

Yang benar adalah:

``` text
Program

↓

System Call

↓

Kernel

↓

SSD
```

Kernel bertugas memeriksa:

-   permission
-   keamanan
-   resource
-   validitas request

------------------------------------------------------------------------

# 6. Apa itu System Call?

System Call adalah mekanisme agar program user dapat meminta layanan
kepada kernel.

Contoh:

-   read()
-   write()
-   fork()
-   execve()
-   mmap()
-   socket()

Semua layanan tersebut memerlukan kernel.

------------------------------------------------------------------------

# 7. Apa yang Dilakukan CPU Saat syscall?

Misalnya program menjalankan:

``` asm
mov rax, 0
syscall
```

CPU mengenali instruksi `syscall`.

CPU kemudian:

1.  berpindah dari CPL 3 ke CPL 0
2.  mengganti kernel stack
3.  melompat ke entry point kernel

CPU tidak memutuskan syscall tersebut valid atau tidak.

CPU hanya menjalankan mekanisme transisi.

------------------------------------------------------------------------

# 8. Siapa yang Mengambil Keputusan?

CPU tidak mengetahui:

-   read()
-   write()
-   open()
-   socket()

CPU hanya mengetahui cara menjalankan instruksi.

Kernel yang mengetahui arti syscall.

Urutan yang benar:

``` text
Program

↓

CPU

↓

syscall

↓

CPU masuk Kernel Mode

↓

Kernel membaca request

↓

Kernel memutuskan apa yang harus dilakukan
```

------------------------------------------------------------------------

# 9. Register CPU

CPU memiliki register.

Contohnya:

``` text
RAX
RBX
RCX
RDX
RDI
RSI
RSP
RBP
RIP
```

Register berada di dalam CPU.

Program hanya memberikan instruksi agar CPU mengubah isi register.

------------------------------------------------------------------------

# 10. Apa itu RAX?

RAX berasal dari sejarah register x86.

``` text
AX  = Accumulator

EAX = Extended Accumulator

RAX = 64-bit Accumulator Register
```

Pada Linux x86-64:

Saat syscall,

RAX digunakan untuk menyimpan nomor syscall.

Contoh:

``` text
RAX = 0

↓

read()
```

``` text
RAX = 1

↓

write()
```

``` text
RAX = 60

↓

exit()
```

Jadi:

RAX bukan PID.

RAX hanyalah register CPU.

------------------------------------------------------------------------

# 11. Dari Mana Kernel Mendapat Nilai RAX?

Misalnya:

``` asm
mov rax, 60
syscall
```

Yang terjadi:

``` text
Program

↓

CPU menjalankan mov

↓

RAX CPU = 60

↓

CPU menjalankan syscall

↓

CPU masuk Kernel Mode

↓

Kernel membaca register RAX milik CPU

↓

Kernel mengetahui bahwa syscall yang diminta adalah exit()
```

Kernel membaca register CPU.

Bukan membaca register milik program.

------------------------------------------------------------------------

# 12. Apakah Semua Program Harus Masuk Kernel Mode?

Tidak.

Kalau hanya menghitung:

``` c
a+b

loop

if

sorting

algoritma
```

CPU tetap berada di:

``` text
CPL = 3
```

Tidak ada syscall.

Kernel tidak dilibatkan.

Kernel hanya dipanggil ketika program membutuhkan layanan sistem.

------------------------------------------------------------------------

# 13. Kapan Kernel Dipanggil?

Kernel dipanggil ketika program membutuhkan resource sistem.

Contohnya:

-   membuka file
-   membuat process
-   membuka socket
-   meminta memory baru
-   mengakses device

Semua dilakukan melalui syscall.

------------------------------------------------------------------------

# 14. Hardware Juga Bisa Membuat CPU Masuk Kernel Mode

Selain syscall terdapat interrupt.

Contoh:

-   keyboard ditekan
-   mouse bergerak
-   timer
-   network card
-   SSD selesai membaca data

Urutannya:

``` text
Hardware

↓

Interrupt

↓

CPU

↓

Kernel

↓

Driver

↓

Program
```

Jadi CPU dapat masuk Kernel Mode karena:

-   syscall
-   interrupt
-   exception

------------------------------------------------------------------------

# 15. stdin dan Buffer

stdin adalah:

File Descriptor 0.

stdin bukan buffer.

Kernel memiliki buffer untuk sumber input.

Contohnya keyboard.

``` text
Keyboard

↓

Interrupt

↓

Kernel

↓

Buffer Terminal

↓

Program membaca stdin
```

Jika menggunakan pipe:

``` bash
echo "Halo" | cat
```

Maka stdin berasal dari pipe.

Jika menggunakan:

``` bash
cat < data.txt
```

Maka stdin berasal dari file.

stdin tetap FD 0.

Yang berubah hanyalah sumber inputnya.

------------------------------------------------------------------------

# 16. CPU vs Kernel

CPU bertugas:

-   fetch
-   decode
-   execute
-   berpindah mode
-   menjaga privilege hardware

Kernel bertugas:

-   process management
-   memory management
-   filesystem
-   scheduler
-   driver
-   networking
-   security

CPU menjalankan.

Kernel mengambil keputusan.

------------------------------------------------------------------------

# 17. Kernel Berada di Mana?

Kernel awalnya berada di disk.

Contoh:

``` text
/boot/vmlinuz
```

Bootloader memuat kernel ke RAM.

CPU kemudian mulai menjalankan kernel dari RAM.

------------------------------------------------------------------------

# 18. Apa itu initramfs?

initramfs bukan bagian dari kernel.

initramfs adalah filesystem sementara.

Biasanya berisi:

-   /init
-   BusyBox
-   driver storage
-   driver filesystem
-   utilitas awal

Bootloader biasanya memuat:

``` text
Kernel

+

initramfs
```

ke RAM.

Kernel kemudian menjalankan `/init` dari initramfs agar dapat memuat
driver dan melakukan mount root filesystem yang sebenarnya.

------------------------------------------------------------------------

# 19. Ringkasan Besar

``` text
Program User
      │
      ▼
CPU menjalankan instruksi
      │
      ▼
Jika tidak perlu kernel
      │
      ▼
Tetap User Mode (CPL = 3)

──────────────────────────────

Jika membutuhkan layanan sistem

Program

↓

Isi register (RAX, RDI, RSI, ...)

↓

syscall

↓

CPU masuk Kernel Mode

↓

Kernel membaca RAX

↓

Kernel menjalankan fungsi yang sesuai

↓

Hardware bila diperlukan

↓

CPU kembali ke User Mode

↓

Program melanjutkan eksekusi
```

------------------------------------------------------------------------

# Inti yang Harus Diingat

1.  CPU selalu menjalankan instruksi.
2.  Kernel hanyalah kode yang dijalankan CPU.
3.  Program user tidak boleh mengakses resource sistem secara langsung.
4.  Semua layanan kernel diakses melalui system call.
5.  CPU mengetahui cara berpindah ke Kernel Mode, tetapi kernel yang
    mengetahui arti dari setiap syscall.
6.  RAX adalah register CPU yang digunakan untuk menyimpan nomor syscall
    pada Linux x86-64.
7.  CPL adalah status privilege CPU saat ini.
8.  Linux umumnya menggunakan Ring 0 (kernel) dan Ring 3 (user).
9.  Interrupt dari hardware juga dapat menyebabkan CPU masuk ke Kernel
    Mode.
10. Kernel dan initramfs adalah dua komponen berbeda yang dimuat ke RAM
    saat proses boot.
