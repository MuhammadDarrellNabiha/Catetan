# Modul Linux Filesystem - Bab 3.1

## Mengapa Filesystem Dibutuhkan dan Jenis-Jenis Filesystem

------------------------------------------------------------------------

# Pendahuluan

Pada bab-bab sebelumnya kita telah mempelajari bahwa sebuah media
penyimpanan seperti HDD maupun SSD pada dasarnya hanyalah sekumpulan
**block** yang dapat dibaca dan ditulis oleh sistem operasi. Linux tidak
melihat SSD sebagai sebuah tempat yang telah memiliki folder, file,
maupun direktori. Yang dilihat oleh kernel hanyalah sebuah **block
device**, yaitu perangkat yang terdiri atas jutaan block penyimpanan.

Sebuah SSD yang baru keluar dari pabrik hanyalah ruang kosong. Memang
tersedia jutaan block yang dapat digunakan, tetapi belum ada aturan
mengenai bagaimana block tersebut digunakan. Sistem operasi belum
mengetahui block mana yang kosong, block mana yang telah digunakan,
bagaimana sebuah file diberi nama, atau bagaimana cara menemukan kembali
file yang telah disimpan.

Karena itulah dibutuhkan sebuah **filesystem**.

------------------------------------------------------------------------

# Mengapa Filesystem Dibutuhkan?

Bayangkan sebuah gudang yang benar-benar kosong. Gudang tersebut
memiliki ruangan yang sangat luas, tetapi belum memiliki rak, nomor
ruangan, daftar inventaris, maupun sistem pencatatan. Ketika seseorang
datang membawa sebuah kardus dan berkata, *"Tolong simpan kardus ini,"*
petugas gudang akan kebingungan. Kardus memang dapat diletakkan di mana
saja, tetapi beberapa hari kemudian tidak ada yang tahu letaknya, siapa
pemiliknya, ataupun apakah tempat tersebut sudah digunakan oleh kardus
lain.

Media penyimpanan tanpa filesystem berada pada kondisi yang sama. Linux
memang dapat menulis data ke block tertentu, tetapi tanpa aturan
penyimpanan, data tersebut tidak akan dapat dikelola dengan baik.

Filesystem merupakan **seperangkat aturan** yang mengatur bagaimana data
disimpan di dalam sebuah partisi. Filesystem bukanlah kumpulan file
ataupun folder, melainkan mekanisme yang memungkinkan file dan folder
dapat dibuat, ditemukan kembali, diubah, dan dihapus.

Dengan adanya filesystem, setiap block pada media penyimpanan memperoleh
fungsi yang jelas. Sebagian block digunakan untuk menyimpan metadata
filesystem, sebagian digunakan untuk metadata file, dan sisanya
digunakan untuk menyimpan isi file. Berkat struktur inilah kernel dapat
mengetahui lokasi sebuah file, ukuran file, pemilik file, permission,
hingga block mana yang masih kosong.

------------------------------------------------------------------------

# Apa yang Dilakukan mkfs?

Ketika menjalankan perintah:

``` bash
mkfs.ext4 /dev/nvme0n1p2
```

banyak orang mengira bahwa perintah tersebut hanya menghapus isi
partisi. Sebenarnya, `mkfs.ext4` melakukan sesuatu yang jauh lebih
penting, yaitu **menginisialisasi struktur filesystem ext4** pada
partisi tersebut.

Program tersebut mulai menulis berbagai metadata awal yang diperlukan
oleh ext4 agar kernel dapat mengenali dan menggunakan partisi tersebut.
Setelah proses ini selesai, partisi tersebut memiliki aturan penyimpanan
yang dipahami oleh Linux.

Dengan kata lain, `mkfs` tidak membuat file ataupun folder. `mkfs`
membangun fondasi yang memungkinkan file dan folder dapat disimpan
secara teratur.

------------------------------------------------------------------------

# Filesystem Bukan Isi File

Kesalahan yang sering terjadi adalah menganggap ext4, XFS, atau Btrfs
sebagai tempat penyimpanan file.

Sebenarnya filesystem bukanlah file ataupun folder. Filesystem merupakan
sistem pengelolaan data.

Analogi yang paling mudah adalah sebuah perpustakaan. Perpustakaan
bukanlah kumpulan buku, melainkan sistem yang mengatur bagaimana buku
diberi nomor, dikelompokkan, disusun di rak, dan dicari kembali ketika
dibutuhkan. Buku hanyalah isi perpustakaan, sedangkan sistem
pengelolaannya adalah perpustakaannya itu sendiri.

Filesystem memiliki fungsi yang sama terhadap block-block pada media
penyimpanan.

------------------------------------------------------------------------

# Mengapa Satu Disk Dapat Memiliki Beberapa Filesystem?

Filesystem dibuat **di dalam partisi**, bukan di seluruh disk.

Misalnya sebuah SSD memiliki tiga partisi:

-   EFI System Partition
-   Root Partition
-   Data Partition

Masing-masing partisi dapat diformat menggunakan filesystem yang berbeda
sesuai kebutuhan.

Sebagai contoh:

-   EFI System Partition menggunakan FAT32 karena merupakan persyaratan
    UEFI.
-   Root Partition menggunakan ext4 karena stabil dan umum digunakan
    pada Linux.
-   Data Partition dapat menggunakan XFS atau Btrfs apabila memiliki
    kebutuhan tertentu.

Karena setiap partisi berdiri sendiri, maka setiap partisi dapat
memiliki filesystem yang berbeda walaupun seluruhnya berada pada SSD
yang sama.

------------------------------------------------------------------------

# Jenis-Jenis Filesystem

Walaupun seluruh filesystem memiliki tujuan yang sama, yaitu mengatur
penyimpanan data, setiap filesystem dirancang dengan filosofi dan tujuan
yang berbeda.

## ext4

ext4 merupakan filesystem yang paling umum digunakan pada sistem Linux
modern. Filesystem ini merupakan penerus ext2 dan ext3 serta menjadi
pilihan bawaan pada banyak distribusi Linux seperti Debian, Ubuntu, dan
Linux Mint.

Keunggulan utama ext4 adalah kestabilannya. Filesystem ini telah
digunakan selama bertahun-tahun dan memiliki performa yang sangat baik
baik pada desktop maupun server. Dokumentasinya sangat lengkap dan
hampir seluruh distribusi Linux memberikan dukungan penuh terhadap ext4.

Bagi sebagian besar pengguna Linux, ext4 merupakan pilihan yang paling
aman.

------------------------------------------------------------------------

## Btrfs

Btrfs (B-tree File System) merupakan filesystem modern yang dirancang
dengan berbagai fitur tambahan.

Salah satu fitur utamanya adalah **snapshot**, yaitu kemampuan menyimpan
kondisi filesystem pada suatu waktu sehingga apabila terjadi kesalahan,
sistem dapat dikembalikan ke kondisi sebelumnya.

Selain snapshot, Btrfs juga mendukung kompresi data, checksum untuk
mendeteksi kerusakan data, serta teknologi Copy-on-Write (CoW) yang
memungkinkan perubahan data dilakukan dengan cara yang lebih aman.

Karena fitur-fitur tersebut, Btrfs banyak digunakan pada distribusi
seperti openSUSE dan beberapa distribusi Linux modern lainnya.

------------------------------------------------------------------------

## XFS

XFS dikembangkan oleh Silicon Graphics (SGI) dan saat ini menjadi
filesystem bawaan pada Red Hat Enterprise Linux.

Filesystem ini terkenal karena mampu menangani file berukuran besar dan
beban kerja tinggi dengan sangat baik. Oleh karena itu XFS sering
digunakan pada server, data center, maupun sistem penyimpanan berskala
besar.

Pada penggunaan desktop sehari-hari, perbedaan performanya dengan ext4
biasanya tidak terlalu terasa.

------------------------------------------------------------------------

## FAT32

FAT32 merupakan filesystem yang dikembangkan oleh Microsoft dan hingga
saat ini masih digunakan secara luas pada flashdisk, kartu SD, serta EFI
System Partition.

Keunggulan FAT32 adalah kompatibilitasnya yang sangat tinggi. Hampir
seluruh sistem operasi modern mampu membaca filesystem ini.

Kelemahannya adalah ukuran maksimum satu file sekitar 4 GB sehingga
kurang cocok digunakan untuk file berukuran besar.

------------------------------------------------------------------------

## exFAT

exFAT dikembangkan sebagai penerus FAT32.

Filesystem ini tetap mempertahankan kompatibilitas yang tinggi dengan
berbagai sistem operasi, tetapi menghilangkan batas ukuran file 4 GB
yang dimiliki FAT32.

Karena alasan tersebut, exFAT menjadi pilihan yang sangat baik untuk
media penyimpanan eksternal yang sering digunakan bergantian antara
Windows, Linux, dan macOS.

------------------------------------------------------------------------

## NTFS

NTFS merupakan filesystem utama yang digunakan oleh Windows.

Dibandingkan FAT32, NTFS mendukung permission, journaling, ukuran file
yang jauh lebih besar, serta berbagai fitur keamanan tambahan.

Linux juga mampu membaca dan menulis filesystem NTFS sehingga sangat
berguna pada sistem dual boot.

------------------------------------------------------------------------

## HFS+ dan APFS

Pada komputer Apple, filesystem yang digunakan berbeda dengan Linux
maupun Windows.

Versi macOS yang lebih lama menggunakan HFS+, sedangkan macOS modern
menggunakan APFS (Apple File System).

APFS dirancang khusus untuk SSD dan menyediakan berbagai fitur modern
seperti snapshot, enkripsi bawaan, serta pengelolaan ruang penyimpanan
yang lebih efisien.

------------------------------------------------------------------------

# Kesimpulan

Filesystem merupakan salah satu komponen terpenting dalam sistem
operasi. Tanpa filesystem, sebuah partisi hanyalah kumpulan block kosong
yang tidak memiliki struktur. Filesystem menyediakan aturan yang
memungkinkan kernel menyimpan, menemukan, memperbarui, serta menghapus
file secara teratur.

Setiap filesystem memiliki tujuan yang berbeda. ext4 mengutamakan
kestabilan, Btrfs menawarkan fitur modern, XFS berfokus pada performa
server, FAT32 dan exFAT mengutamakan kompatibilitas lintas sistem
operasi, sedangkan NTFS dan APFS dioptimalkan untuk ekosistem Windows
dan Apple.

Memahami konsep filesystem merupakan langkah penting sebelum mempelajari
struktur internal ext4 seperti **Superblock, Block Group, Inode,
Directory Entry, dan Data Block**, karena seluruh komponen tersebut
merupakan implementasi nyata dari aturan yang telah dibahas pada bab
ini.
