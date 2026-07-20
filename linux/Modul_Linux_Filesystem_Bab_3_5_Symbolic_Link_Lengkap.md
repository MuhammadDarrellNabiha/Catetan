# Modul Linux Filesystem -- Bab 3.5

# Symbolic Link (Soft Link)

## Edisi Lengkap

> **Tujuan bab:** Setelah menyelesaikan bab ini, Anda diharapkan mampu
> menjelaskan bagaimana Symbolic Link bekerja di level filesystem,
> mengapa desainnya berbeda dengan Hard Link, bagaimana kernel mengikuti
> symlink ketika membuka sebuah file, serta memahami alasan Linux sangat
> bergantung pada mekanisme ini.

------------------------------------------------------------------------

# 1. Pendahuluan

Pada bab sebelumnya kita mempelajari Hard Link.

Hard Link bekerja dengan **menambahkan Directory Entry baru yang
menunjuk ke inode yang sama**.

Akibatnya:

-   inode tetap satu,
-   Data Block tetap satu,
-   metadata tetap satu,
-   Link Count bertambah.

Desain tersebut sangat efisien, tetapi memiliki keterbatasan.

Misalnya:

-   tidak bisa lintas filesystem,
-   umumnya tidak boleh dibuat ke directory,
-   inode target harus berada pada filesystem yang sama.

Unix membutuhkan mekanisme yang lebih fleksibel.

Solusinya adalah **Symbolic Link (Soft Link).**

------------------------------------------------------------------------

# 2. Filosofi Symbolic Link

Bayangkan Anda mempunyai rumah.

Hard Link seperti membuat **dua pintu menuju rumah yang sama**.

``` text
Pintu A ─────┐
             │
             ▼
         Rumah
             ▲
             │
Pintu B ─────┘
```

Sedangkan Symbolic Link bukan pintu kedua.

Ia hanyalah secarik kertas yang bertuliskan:

> "Rumah yang Anda cari berada di Jalan Mawar No. 10."

Artinya symlink **tidak menunjuk rumah secara langsung**.

Ia hanya menyimpan informasi mengenai **bagaimana cara mencapai rumah
tersebut**.

------------------------------------------------------------------------

# 3. Definisi

Secara teknis:

> Symbolic Link adalah sebuah file khusus yang isi datanya berupa **path
> menuju file atau directory lain**.

Perhatikan kalimat di atas.

Symlink tetap merupakan **file**.

Karena merupakan file, symlink juga mempunyai:

-   inode sendiri,
-   permission,
-   owner,
-   timestamp,
-   ukuran file.

Yang membedakan hanyalah **isi Data Block**.

Jika file biasa menyimpan:

``` text
Hello World
```

atau

``` text
gambar JPEG
```

maka symlink menyimpan sesuatu seperti:

``` text
/home/darrell/cat.jpg
```

atau

``` text
../config/settings.conf
```

------------------------------------------------------------------------

# 4. Struktur Internal

Misalkan terdapat file:

``` text
cat.jpg
```

Strukturnya:

``` text
Directory Entry
      │
      ▼
Inode 501
      │
      ▼
Data Block
(gambar)
```

Sekarang dibuat:

``` bash
ln -s cat.jpg foto
```

Yang terjadi bukan:

``` text
foto
     │
     ▼
Inode 501
```

melainkan:

``` text
Directory Entry

foto
 │
 ▼

Inode 900
 │
 ▼

Data Block

"cat.jpg"
```

Perhatikan bahwa inode 900 **bukan inode milik gambar**.

Ia adalah inode milik symlink.

Isi datanya hanyalah teks.

------------------------------------------------------------------------

# 5. Apa yang Terjadi Saat Kernel Membuka Symlink?

Misalnya pengguna menjalankan:

``` bash
cat foto
```

Kernel tidak langsung membaca gambar.

Urutannya:

1.  Path `foto` ditemukan.
2.  Kernel membuka inode symlink.
3.  Kernel mengetahui bahwa tipe inode adalah **symbolic link**.
4.  Kernel membaca isi Data Block symlink.
5.  Isi Data Block adalah path target.
6.  Kernel melakukan **path resolution lagi** menggunakan path tersebut.
7.  Setelah inode target ditemukan, barulah Data Block file asli dibaca.

Artinya membuka symlink membutuhkan satu langkah tambahan dibanding
membuka file biasa.

------------------------------------------------------------------------

# 6. Mengapa Symlink Bisa Lintas Filesystem?

Sekarang perhatikan dua SSD berikut.

``` text
SSD A

/home

SSD B

/mnt/storage
```

Symlink dapat berisi:

``` text
/mnt/storage/video.mp4
```

Kernel hanya membaca teks tersebut.

Kemudian kernel melakukan pencarian ulang.

Karena hanya berupa path, target boleh berada:

-   di partition lain,
-   di SSD lain,
-   bahkan pada filesystem yang berbeda.

Hard Link tidak bisa melakukan hal ini karena ia membutuhkan inode yang
sama.

Padahal inode tidak pernah berlaku lintas filesystem.

------------------------------------------------------------------------

# 7. Broken Symbolic Link

Karena symlink hanya mengetahui path, target dapat hilang.

Misalnya:

``` bash
ln -s cat.jpg foto
rm cat.jpg
```

Yang tersisa:

``` text
foto
 │
 ▼

"cat.jpg"
```

Tetapi:

``` text
cat.jpg
```

sudah tidak ada.

Ketika kernel mencoba melakukan path resolution:

``` text
cat.jpg
```

hasilnya gagal.

Inilah yang disebut **Broken Symbolic Link**.

Perhatikan bahwa symlink **tidak rusak**.

Yang hilang hanyalah targetnya.

------------------------------------------------------------------------

# 8. Hard Link vs Symbolic Link

## Hard Link

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

## Symbolic Link

``` text
Directory Entry
        │
        ▼
   Inode Symlink
        │
        ▼
"/home/darrell/cat.jpg"
        │
        ▼
Kernel melakukan Path Resolution
        │
        ▼
   Inode Target
        │
        ▼
     Data Block
```

Perbedaan terbesar:

-   Hard Link berbagi inode.
-   Symlink hanya mengetahui alamat tujuan.

------------------------------------------------------------------------

# 9. Mengapa Linux Sangat Banyak Menggunakan Symlink?

Contoh:

``` text
/bin -> /usr/bin
/lib -> /usr/lib
```

Ketika sebuah program lama mencoba membuka:

``` text
/bin/bash
```

Kernel membaca symlink.

Isi symlink:

``` text
/usr/bin
```

Kernel kemudian melanjutkan pencarian menuju lokasi sebenarnya.

Dengan cara ini:

-   aplikasi lama tetap berjalan,
-   struktur filesystem dapat berubah,
-   kompatibilitas tetap terjaga.

------------------------------------------------------------------------

# 10. Praktik

Buat file:

``` bash
echo "Linux" > cat.txt
```

Buat symlink:

``` bash
ln -s cat.txt link.txt
```

Periksa inode:

``` bash
ls -li
```

Anda akan melihat inode `cat.txt` dan `link.txt` berbeda.

Kemudian:

``` bash
rm cat.txt
```

Lalu:

``` bash
cat link.txt
```

Kernel akan gagal menemukan target karena symlink hanya menyimpan path.

------------------------------------------------------------------------

# 11. Ringkasan

Symbolic Link bukanlah file kedua dan bukan pula Hard Link.

Ia adalah **file khusus** yang menyimpan **path menuju objek lain**.

Saat symlink dibuka, kernel harus:

1.  membuka inode symlink,
2.  membaca path target,
3.  melakukan path resolution kembali,
4.  menemukan inode target,
5.  baru membaca isi file.

Inilah alasan symlink jauh lebih fleksibel dibanding Hard Link, tetapi
juga dapat menjadi **broken** apabila targetnya dihapus.
