# Modul Linux Filesystem - Bab 3.2

## Anatomi Filesystem ext4

------------------------------------------------------------------------

# Pendahuluan

Pada bab sebelumnya telah dijelaskan bahwa filesystem merupakan
seperangkat aturan yang mengatur bagaimana data disimpan di dalam sebuah
partisi. Akan tetapi, sampai pada tahap tersebut kita baru memahami
tujuan filesystem, belum memahami bagaimana filesystem bekerja secara
internal.

Pertanyaan yang kemudian muncul adalah bagaimana kernel mengetahui
lokasi sebuah file, bagaimana metadata file disimpan, mengapa sebuah
file memiliki owner dan permission, serta bagaimana sistem dapat
mengetahui block mana yang kosong dan mana yang telah digunakan.

Untuk menjawab seluruh pertanyaan tersebut kita harus melihat struktur
internal sebuah filesystem, khususnya ext4. Walaupun filesystem lain
memiliki implementasi yang berbeda, konsep yang dipelajari pada bab ini
akan sangat membantu dalam memahami cara kerja filesystem modern secara
umum.

------------------------------------------------------------------------

# Gambaran Umum Struktur ext4

Ketika sebuah partisi diformat menggunakan `mkfs.ext4`, partisi tersebut
tidak langsung dipenuhi oleh file pengguna. Sebagian ruang dialokasikan
untuk menyimpan struktur internal filesystem yang akan digunakan kernel
untuk mengelola seluruh data.

Secara konseptual, struktur ext4 dapat digambarkan sebagai berikut.

``` text
Filesystem ext4
│
├── Superblock
│
├── Block Group 0
│   ├── Block Bitmap
│   ├── Inode Bitmap
│   ├── Inode Table
│   └── Data Blocks
│
├── Block Group 1
│   ├── Block Bitmap
│   ├── Inode Bitmap
│   ├── Inode Table
│   └── Data Blocks
│
├── Block Group 2
│   ├── Block Bitmap
│   ├── Inode Bitmap
│   ├── Inode Table
│   └── Data Blocks
│
└── ...
```

Masing-masing komponen memiliki tanggung jawab yang berbeda sehingga
filesystem dapat bekerja secara efisien.

------------------------------------------------------------------------

# Superblock

Superblock merupakan metadata utama dari sebuah filesystem. Ketika
kernel melakukan proses mount, salah satu struktur pertama yang dibaca
adalah superblock.

Superblock tidak menyimpan isi file. Sebaliknya, superblock menyimpan
informasi mengenai filesystem itu sendiri, seperti jenis filesystem yang
digunakan, ukuran block, jumlah inode, jumlah block, ukuran filesystem,
status filesystem, UUID, dan berbagai parameter penting lainnya.

Dengan kata lain, superblock dapat dianggap sebagai identitas sebuah
filesystem. Tanpa superblock, kernel tidak mengetahui bagaimana cara
membaca partisi tersebut.

Pada ext4 tidak hanya terdapat satu superblock. Selain superblock utama,
ext4 juga menyimpan beberapa salinan (backup superblock) pada Block
Group tertentu. Tujuannya adalah agar filesystem masih dapat dipulihkan
apabila superblock utama mengalami kerusakan.

------------------------------------------------------------------------

# Mengapa ext4 Menggunakan Block Group?

Apabila seluruh inode ditempatkan di satu lokasi dan seluruh data block
berada di lokasi lain, maka setiap kali sebuah file diakses sistem harus
berpindah-pindah ke berbagai area disk.

Pada HDD mekanik, perpindahan ini menyebabkan waktu pencarian (seek
time) menjadi lebih lama.

Untuk mengatasi masalah tersebut, ext4 membagi filesystem menjadi
beberapa bagian yang disebut **Block Group**.

Setiap Block Group memiliki metadata dan area penyimpanannya sendiri
sehingga data yang saling berhubungan dapat ditempatkan berdekatan.

Pendekatan ini meningkatkan efisiensi akses data dan membuat filesystem
lebih mudah dikelola.

------------------------------------------------------------------------

# Block Bitmap

Setiap Block Group memiliki sebuah Block Bitmap.

Block Bitmap merupakan peta yang menunjukkan block mana yang masih
kosong dan block mana yang telah digunakan.

Secara konseptual bitmap dapat digambarkan sebagai berikut.

``` text
1 = digunakan
0 = kosong

111111110001011000111
```

Ketika kernel ingin menyimpan isi sebuah file baru, filesystem tidak
perlu memeriksa seluruh block satu per satu. Filesystem cukup melihat
bitmap untuk menemukan block kosong yang tersedia.

Dengan cara ini proses alokasi block menjadi jauh lebih cepat.

------------------------------------------------------------------------

# Inode Bitmap

Selain block, filesystem juga harus mengetahui inode mana yang masih
tersedia.

Untuk tujuan tersebut setiap Block Group memiliki Inode Bitmap.

Cara kerjanya sama seperti Block Bitmap, hanya saja objek yang dipantau
adalah inode.

``` text
1 = inode digunakan
0 = inode kosong

1111111000011100
```

Saat sebuah file baru dibuat, ext4 akan mencari inode kosong melalui
Inode Bitmap sebelum metadata file ditulis ke Inode Table.

------------------------------------------------------------------------

# Inode Table

Inode Table merupakan kumpulan inode yang dimiliki oleh Block Group
tersebut.

Inode adalah struktur data yang menyimpan metadata sebuah objek
filesystem.

Metadata yang disimpan antara lain meliputi:

-   UID dan GID pemilik file.
-   Permission.
-   Ukuran file.
-   Timestamp.
-   Jumlah hard link.
-   Pointer menuju Data Block tempat isi file disimpan.

Hal yang sangat penting untuk dipahami adalah **inode tidak menyimpan
nama file**.

Nama file akan dipelajari pada bab berikutnya ketika membahas Directory
Entry.

Perlu diingat pula bahwa setiap objek filesystem memiliki satu inode.
Objek tersebut tidak hanya berupa file biasa, tetapi juga direktori,
symbolic link, device file, socket, FIFO, dan objek filesystem lainnya.

Karena inode dialokasikan saat filesystem dibuat, jumlah inode pada ext4
bersifat terbatas. Pada penggunaan desktop jumlah inode biasanya
mencapai jutaan sehingga hampir tidak pernah habis. Namun pada server
yang menyimpan jutaan file kecil, inode dapat habis walaupun ruang
penyimpanan masih tersedia.

------------------------------------------------------------------------

# Data Blocks

Data Block merupakan bagian yang benar-benar menyimpan isi file.

Apabila terdapat file bernama `cat.jpg`, maka byte-byte penyusun gambar
tersebut akan disimpan di dalam Data Block.

Data Block tidak mengetahui nama file maupun pemilik file. Data Block
hanya menyimpan data mentah (raw data).

Hubungan antara metadata file dan Data Block dijembatani oleh inode.

Dengan demikian inode dapat dianggap sebagai "peta" yang menunjukkan
lokasi Data Block yang berisi isi file.

------------------------------------------------------------------------

# Hubungan Antar Komponen

Ketika sebuah file dibuat, proses yang terjadi secara konseptual adalah
sebagai berikut.

1.  Filesystem mencari inode kosong melalui Inode Bitmap.
2.  Filesystem mencari block kosong melalui Block Bitmap.
3.  Metadata file ditulis ke Inode Table.
4.  Isi file ditulis ke Data Block.
5.  Hubungan antara inode dan nama file akan dicatat melalui Directory
    Entry (dibahas pada bab berikutnya).

Dengan mekanisme tersebut filesystem dapat mengetahui lokasi setiap
file, siapa pemiliknya, berapa ukurannya, dan block mana yang digunakan
untuk menyimpan isinya.

------------------------------------------------------------------------

# Ringkasan

Filesystem ext4 tidak hanya terdiri atas area penyimpanan data, tetapi
juga berbagai struktur metadata yang bekerja sama untuk mengelola
seluruh isi partisi.

Superblock menyimpan informasi mengenai filesystem secara keseluruhan.
Block Group membagi filesystem menjadi kelompok-kelompok yang lebih
kecil agar pengelolaan data menjadi lebih efisien. Block Bitmap dan
Inode Bitmap digunakan untuk melacak sumber daya yang masih tersedia.
Inode Table menyimpan metadata setiap objek filesystem, sedangkan Data
Block menyimpan isi file yang sebenarnya.

Dengan memahami hubungan antara kelima komponen tersebut, kita telah
memiliki pondasi yang kuat untuk mempelajari konsep berikutnya, yaitu
**Directory Entry**. Pada bab selanjutnya akan dijelaskan mengapa inode
tidak menyimpan nama file dan bagaimana Linux menghubungkan nama file
dengan inode sehingga pengguna dapat mengakses file menggunakan nama
seperti `cat.jpg` atau `notes.txt`.
