# Modul Linux Filesystem - Bab 3.4

## Hard Link: Satu Inode, Banyak Nama

------------------------------------------------------------------------

# Pendahuluan

Pada bab sebelumnya kita mempelajari bahwa sebuah nama file tidak
disimpan di dalam inode. Nama file disimpan oleh **Directory Entry**,
sedangkan inode hanya menyimpan metadata serta pointer menuju Data
Block.

Keputusan desain tersebut ternyata memiliki konsekuensi yang sangat
menarik, yaitu satu inode dapat direferensikan oleh lebih dari satu
Directory Entry. Konsep inilah yang disebut **Hard Link**.

Memahami Hard Link sangat penting karena konsep ini menjelaskan mengapa
proses seperti `rm`, `mv`, dan `ln` pada Linux bekerja seperti yang kita
lihat sehari-hari.

------------------------------------------------------------------------

# Apa Itu Hard Link?

Hard Link adalah **Directory Entry baru yang menunjuk ke inode yang
sudah ada**.

Perhatikan bahwa Hard Link **bukan salinan file**.

Misalnya terdapat file:

``` text
cat.jpg
    │
    ▼
Directory Entry
    │
    ▼
Inode 501
    │
    ▼
Data Block
```

Ketika menjalankan:

``` bash
ln cat.jpg backup.jpg
```

Filesystem **tidak membuat inode baru** dan **tidak menyalin Data
Block**.

Yang dibuat hanyalah Directory Entry baru.

``` text
cat.jpg
    │
    ├──────────────┐
    ▼              │
Inode 501 ◄────────┘
    ▲
    │
backup.jpg

        │
        ▼
     Data Block
```

Kini terdapat dua nama file, tetapi keduanya menunjuk ke inode yang
sama.

------------------------------------------------------------------------

# Link Count

Setiap inode memiliki sebuah field bernama **Link Count**.

Field ini menunjukkan berapa banyak Directory Entry yang masih menunjuk
inode tersebut.

Sebelum membuat Hard Link:

``` text
Inode 501

Link Count = 1
```

Setelah:

``` bash
ln cat.jpg backup.jpg
```

Menjadi:

``` text
Inode 501

Link Count = 2
```

Artinya inode tersebut memiliki dua nama.

------------------------------------------------------------------------

# Apa yang Terjadi Saat Salah Satu Nama Dihapus?

Misalkan kita menjalankan:

``` bash
rm cat.jpg
```

Yang dihapus hanyalah Directory Entry `cat.jpg`.

Directory Entry `backup.jpg` masih ada sehingga inode masih memiliki
satu referensi.

``` text
backup.jpg
      │
      ▼
   Inode 501
      │
      ▼
   Data Block
```

Link Count berubah menjadi:

``` text
1
```

Isi file tetap utuh.

Baru ketika Link Count menjadi nol dan tidak ada proses yang masih
membuka file tersebut, inode serta Data Block dapat dibebaskan untuk
digunakan kembali.

------------------------------------------------------------------------

# Hard Link Bukan Copy

Sering kali Hard Link disalahartikan sebagai hasil penyalinan file.

Padahal jika menjalankan:

``` bash
cp cat.jpg backup.jpg
```

Filesystem akan membuat inode baru dan Data Block baru.

Sebaliknya:

``` bash
ln cat.jpg backup.jpg
```

Tetap menggunakan inode yang sama.

Secara konsep:

``` text
cp

cat.jpg ─► inode 501 ─► Data Block A

backup.jpg ─► inode 700 ─► Data Block B
```

Sedangkan Hard Link:

``` text
ln

cat.jpg ─┐
          ├──► inode 501 ─► Data Block
backup.jpg┘
```

------------------------------------------------------------------------

# Mengapa Rename Sangat Cepat?

Misalnya menjalankan:

``` bash
mv cat.jpg foto.jpg
```

Selama pemindahan masih berada pada filesystem yang sama, kernel tidak
perlu memindahkan isi file.

Yang dilakukan hanyalah mengubah Directory Entry dari:

``` text
cat.jpg → inode 501
```

menjadi:

``` text
foto.jpg → inode 501
```

Inode dan Data Block tetap sama.

Karena itu operasi rename pada filesystem yang sama biasanya berlangsung
hampir seketika.

------------------------------------------------------------------------

# Keterbatasan Hard Link

Hard Link memiliki beberapa batasan.

-   Hard Link hanya dapat dibuat pada filesystem yang sama karena inode
    hanya berlaku di dalam satu filesystem.
-   Hard Link umumnya tidak digunakan untuk menghubungkan directory agar
    struktur pohon filesystem tidak membentuk siklus yang akan
    menyulitkan proses penelusuran.

------------------------------------------------------------------------

# Ringkasan

Hard Link bukanlah salinan file. Hard Link hanyalah Directory Entry
tambahan yang menunjuk ke inode yang sama.

Hubungannya dapat digambarkan sebagai berikut.

``` text
Directory Entry A
        │
        ├────────────┐
        ▼            │
     Inode 501 ◄─────┘
        ▲
        │
Directory Entry B
        │
        ▼
     Data Block
```

Karena inode yang digunakan sama, seluruh metadata dan isi file juga
sama. Menghapus salah satu nama tidak menghapus isi file selama masih
ada Directory Entry lain yang menunjuk inode tersebut.

Bab berikutnya akan membahas **Symbolic Link**, yang berbeda secara
mendasar dengan Hard Link walaupun sama-sama disebut "link".
