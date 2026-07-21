# Modul Linux Filesystem -- Bab 4.1

# Virtual Filesystem (VFS) -- Edisi Super Detail

## Tujuan Pembelajaran

Setelah menyelesaikan bab ini Anda diharapkan mampu:

-   Menjelaskan mengapa VFS diperlukan.
-   Memahami posisi VFS di dalam kernel Linux.
-   Menjelaskan hubungan antara system call, VFS, path resolution,
    mount, dan filesystem driver.
-   Memahami mengapa aplikasi tidak perlu mengetahui jenis filesystem
    yang digunakan.
-   Membangun model mental yang benar sebelum mempelajari mount.

------------------------------------------------------------------------

# 1. Latar Belakang

Linux mendukung banyak filesystem:

-   ext4
-   XFS
-   Btrfs
-   FAT32
-   exFAT
-   NTFS (melalui driver)
-   ISO9660
-   NFS
-   tmpfs

Setiap filesystem mempunyai struktur internal yang berbeda.

Sebagai contoh:

-   ext4 menggunakan inode, extent, block group, dan superblock.
-   FAT32 tidak memiliki inode dan menggunakan File Allocation Table.
-   XFS memiliki struktur metadata sendiri.
-   Btrfs menggunakan pendekatan copy-on-write.

Pertanyaannya:

Bagaimana mungkin satu program seperti `cat` atau `vim` dapat membuka
file pada semua filesystem tersebut tanpa mengetahui perbedaannya?

Jawabannya adalah Virtual Filesystem (VFS).

------------------------------------------------------------------------

# 2. Apa itu VFS?

Virtual Filesystem (VFS) adalah subsistem di dalam kernel Linux yang
menyediakan antarmuka umum untuk seluruh filesystem.

VFS bukan filesystem.

VFS tidak menyimpan file.

VFS tidak memiliki block.

VFS tidak memiliki inode fisik.

Tugas VFS adalah menjadi lapisan abstraksi sehingga aplikasi dapat
menggunakan semua filesystem melalui system call yang sama.

------------------------------------------------------------------------

# 3. Posisi VFS

Diagram konseptual:

User Space

↓

Program

↓

System Call

↓

====================== Linux Kernel ======================

↓

Virtual Filesystem (VFS)

↓

Filesystem Driver

↓

Block Layer

↓

Storage Device

Semua komponen mulai dari VFS sampai block layer masih berada di dalam
kernel.

------------------------------------------------------------------------

# 4. Mengapa VFS Dibutuhkan?

Tanpa VFS:

Setiap aplikasi harus memahami seluruh filesystem.

Misalnya:

cat

↓

if ext4

↓

pakai algoritma ext4

if FAT32

↓

pakai algoritma FAT

if XFS

↓

pakai algoritma XFS

Hal ini tidak praktis.

Dengan VFS:

Program cukup memanggil:

-   open()
-   read()
-   write()
-   close()
-   stat()
-   mkdir()
-   unlink()

VFS menangani sisanya.

------------------------------------------------------------------------

# 5. Hubungan dengan System Call

Misalnya:

cat /home/user/file.txt

Program tidak membaca disk.

Program hanya memanggil:

open()

Kernel menerima system call tersebut.

Lalu eksekusi masuk ke subsistem VFS.

Mulai titik inilah seluruh operasi filesystem dikendalikan oleh VFS.

------------------------------------------------------------------------

# 6. Hubungan dengan Path Resolution

Salah satu tugas utama VFS adalah melakukan path resolution.

Input:

/home/user/file.txt

Output:

inode dari file tujuan.

Secara konsep:

Path

↓

Directory Entry

↓

inode

↓

Directory Entry

↓

inode

↓

Directory Entry

↓

inode file

Jika terdapat symbolic link, VFS membaca isi symlink lalu mengulang path
resolution terhadap target.

------------------------------------------------------------------------

# 7. Hubungan dengan Filesystem Driver

Setelah path berhasil di-resolve, VFS mengetahui inode tersebut berasal
dari filesystem apa.

Contoh:

/home → ext4

/mnt/windows → NTFS

/media/usb → FAT32

VFS kemudian meneruskan operasi ke driver filesystem yang sesuai.

Driver inilah yang memahami format metadata filesystem tersebut.

------------------------------------------------------------------------

# 8. Siapa Melakukan Apa?

Aplikasi: - meminta membuka file.

Kernel: - menerima system call.

VFS: - melakukan path resolution. - mengelola abstraksi filesystem. -
memilih driver filesystem yang sesuai. - meneruskan operasi.

Filesystem Driver: - membaca metadata sesuai format filesystem. -
mencari lokasi data. - melakukan operasi baca/tulis.

Block Layer: - mengubah permintaan menjadi operasi I/O perangkat.

Storage: - mengembalikan data.

------------------------------------------------------------------------

# 9. Contoh Lengkap

Perintah:

cat /home/user/file.txt

Alur:

User Space

↓

cat

↓

open()

↓

Linux Kernel

↓

VFS

↓

Path Resolution

↓

Directory Entry (/)

↓

inode home

↓

Directory Entry (/home)

↓

inode user

↓

Directory Entry (/home/user)

↓

inode file

↓

driver ext4

↓

extent

↓

block

↓

SSD

↓

data kembali

------------------------------------------------------------------------

# 10. Mengapa Disebut Virtual?

Kata "Virtual" berarti VFS memberikan pandangan yang seragam.

Aplikasi merasa seluruh file berada pada satu sistem yang sama.

Padahal di belakang layar:

/boot mungkin ext4

/home mungkin XFS

/mnt/windows mungkin NTFS

/media/usb mungkin FAT32

Semuanya tetap diakses menggunakan open(), read(), write(), dan system
call lainnya.

------------------------------------------------------------------------

# 11. Hubungan dengan Mount

VFS juga menyimpan informasi mount.

Ketika path resolution menemukan mount point, VFS berpindah ke
filesystem yang terpasang pada mount tersebut lalu melanjutkan
pencarian.

Inilah alasan mount menjadi topik berikutnya setelah VFS.

------------------------------------------------------------------------

# 12. Kesalahan Umum

Salah: "VFS berada di luar kernel."

Benar: VFS adalah subsistem kernel.

Salah: "Driver ext4 melakukan semuanya."

Benar: VFS mengorkestrasi operasi umum, driver ext4 menangani detail
format ext4.

Salah: "Program berbicara langsung dengan ext4."

Benar: Program selalu melalui system call dan VFS.

------------------------------------------------------------------------

# Ringkasan Besar

Program

↓

System Call

↓

Linux Kernel

↓

VFS

↓

Path Resolution

↓

Filesystem Driver

↓

Block Layer

↓

Storage

## Poin Penting

-   VFS adalah bagian dari kernel Linux.
-   VFS menyediakan abstraksi untuk seluruh filesystem.
-   Semua operasi file melewati VFS.
-   VFS melakukan path resolution.
-   VFS memilih driver filesystem yang benar.
-   Driver filesystem memahami format metadata masing-masing.
-   VFS membuat aplikasi tidak perlu mengetahui jenis filesystem yang
    digunakan.
-   Konsep VFS menjadi dasar untuk memahami mount dan fstab.
