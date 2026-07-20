# Modul Linux Filesystem - Bab 3.3

## Directory Entry: Di Mana Nama File Disimpan?

------------------------------------------------------------------------

# Pendahuluan

Pada bab sebelumnya kita telah mempelajari bahwa inode menyimpan
metadata sebuah file, sedangkan Data Block menyimpan isi file. Akan
tetapi masih ada satu pertanyaan penting yang belum terjawab.

Apabila inode tidak menyimpan nama file, lalu mengapa kita dapat membuka
file menggunakan nama seperti `cat.jpg`, `notes.txt`, atau `main.c`?

Jawabannya terletak pada **Directory Entry**.

Konsep ini merupakan salah satu fondasi desain Unix. Dengan memahami
Directory Entry, kita akan mengerti bagaimana kernel menemukan file,
mengapa hard link dapat dibuat, serta mengapa menghapus sebuah file
tidak selalu langsung menghapus datanya.

------------------------------------------------------------------------

# Directory Juga Merupakan File

Bagi kernel Linux, directory juga merupakan objek filesystem sehingga
memiliki inode sendiri.

Perbedaannya dengan file biasa adalah isi Data Block milik directory
bukanlah data pengguna, melainkan daftar pasangan **nama → inode**.

Contoh konseptual:

``` text
Nama File        Inode
-----------------------
cat.jpg          1052
notes.txt        1060
main.c           1098
```

Daftar inilah yang disebut **Directory Entry**.

------------------------------------------------------------------------

# Bagaimana Kernel Mencari File?

Saat pengguna membuka:

``` text
/home/darrell/cat.jpg
```

kernel tidak langsung mengetahui inode `cat.jpg`.

Kernel akan:

1.  Membaca inode `/`.
2.  Membaca Directory Entry `/` untuk mencari `home`.
3.  Membaca Directory Entry `home` untuk mencari `darrell`.
4.  Membaca Directory Entry `darrell` untuk mencari `cat.jpg`.
5.  Mendapat nomor inode `cat.jpg`.
6.  Membaca inode tersebut.
7.  Menggunakan pointer pada inode untuk membaca Data Block.

Secara konseptual:

``` text
/
 ↓
Directory Entry
 ↓
home
 ↓
Directory Entry
 ↓
darrell
 ↓
Directory Entry
 ↓
cat.jpg
 ↓
Inode
 ↓
Data Block
```

------------------------------------------------------------------------

# Mengapa Nama File Tidak Disimpan di Inode?

Karena desain ini memungkinkan satu inode memiliki lebih dari satu nama.

``` text
laporan.txt ─┐
             ├──► Inode 1523
backup.txt ──┘
```

Konsep inilah yang menjadi dasar **Hard Link**.

------------------------------------------------------------------------

# Apa yang Terjadi Saat File Dibuat?

Filesystem secara konsep akan:

1.  Mencari inode kosong.
2.  Mencari Data Block kosong.
3.  Menulis metadata ke Inode Table.
4.  Menulis isi file ke Data Block.
5.  Menambahkan Directory Entry yang menghubungkan nama file dengan
    inode.

Barulah file dapat diakses menggunakan namanya.

------------------------------------------------------------------------

# Apa yang Terjadi Saat File Dihapus?

Saat menjalankan:

``` bash
rm cat.jpg
```

Filesystem pertama-tama menghapus Directory Entry yang menunjuk inode
tersebut.

Selanjutnya nilai link count pada inode dikurangi. Selama masih ada
Directory Entry lain yang menunjuk inode itu, isi file tetap ada. Jika
tidak ada lagi referensi, inode dan Data Block dapat dibebaskan untuk
dipakai kembali.

------------------------------------------------------------------------

# Ringkasan

Directory Entry adalah jembatan antara nama file dan inode.

Hubungannya dapat diringkas sebagai berikut.

``` text
Nama File
    │
    ▼
Directory Entry
    │
    ▼
Inode
    │
    ▼
Data Block
```

Dengan memahami Directory Entry, kita telah melengkapi tiga komponen
utama anatomi filesystem ext4:

-   Directory Entry menyimpan nama file.
-   Inode menyimpan metadata.
-   Data Block menyimpan isi file.

Bab berikutnya akan membahas Hard Link yang merupakan konsekuensi
langsung dari desain tersebut.
