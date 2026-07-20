# Modul Linux Storage - Bab 2

# GPT dan MBR: Memahami Partition Table dari Dasar

> Tingkat: Fundamental → Intermediate\
> Tujuan: Memahami bagaimana sebuah disk dibagi menjadi beberapa
> partition sebelum filesystem dibuat.

------------------------------------------------------------------------

# Daftar Isi

1.  Mengapa Disk Harus Dipartisi?
2.  Apa itu Partition Table?
3.  Struktur Fisik Disk
4.  Master Boot Record (MBR)
5.  Keterbatasan MBR
6.  GUID Partition Table (GPT)
7.  Struktur GPT Secara Detail
8.  Protective MBR
9.  Primary Header
10. Partition Entry Array
11. Backup GPT
12. GUID, PARTUUID, dan UUID
13. Perbandingan MBR vs GPT
14. Hubungan GPT dengan Filesystem
15. Hubungan GPT dengan Proses Boot
16. Contoh Nyata menggunakan `lsblk`, `fdisk`, dan `gdisk`
17. Ringkasan

------------------------------------------------------------------------

# 1. Mengapa Disk Harus Dipartisi?

Saat sebuah SSD atau HDD baru keluar dari pabrik, media penyimpanan
tersebut hanyalah kumpulan sektor (sector).

Disk belum mengetahui:

-   sistem operasi berada di mana,
-   filesystem dimulai di mana,
-   ruang kosong berada di mana.

Agar sebuah disk dapat digunakan, kita membutuhkan **partition table**.

Partition table merupakan peta yang menjelaskan:

-   ada berapa partition,
-   di sektor berapa partition dimulai,
-   di sektor berapa partition berakhir,
-   partition tersebut digunakan untuk apa.

Tanpa partition table, sistem operasi tidak mengetahui bagaimana membagi
sebuah disk menjadi beberapa area yang berbeda.

------------------------------------------------------------------------

# 2. Apa itu Partition Table?

Partition Table adalah metadata yang berada di awal disk.

Metadata ini **bukan isi file**, melainkan informasi mengenai pembagian
ruang penyimpanan.

Contoh sederhana:

``` text
SSD 1 TB

+------------------------------------------------------+
| Partition Table                                      |
+------------------------------------------------------+
| EFI | Linux | Windows | Data                         |
+------------------------------------------------------+
```

Filesystem belum muncul di sini.

Partition table hanya mengatakan:

> "Partition pertama dimulai dari sektor sekian hingga sektor sekian."

------------------------------------------------------------------------

# 3. Struktur Fisik Disk

Secara konseptual:

``` text
Disk

Sector 0
Sector 1
Sector 2
...
Sector N
```

Sector adalah unit terkecil yang dapat dibaca atau ditulis perangkat
penyimpanan.

Modern SSD biasanya menggunakan sector logis 512 byte atau 4096 byte.

Partition table menyimpan alamat sector awal dan akhir setiap partition.

------------------------------------------------------------------------

# 4. Master Boot Record (MBR)

MBR merupakan format partition table klasik yang berasal dari awal era
IBM PC.

Layout sederhananya:

``` text
Sector 0

+-------------------------------+
| Boot Code (446 byte)          |
+-------------------------------+
| Partition Table (64 byte)     |
+-------------------------------+
| Signature 0x55AA (2 byte)     |
+-------------------------------+
```

Partition table hanya memiliki ruang untuk **empat entri**.

Karena itu MBR hanya mendukung:

-   maksimal 4 primary partition

atau

-   3 primary + 1 extended partition.

------------------------------------------------------------------------

# 5. Keterbatasan MBR

MBR memiliki beberapa keterbatasan mendasar.

## Maksimum 2 TiB

MBR menggunakan alamat 32-bit untuk sector.

Dengan sector 512 byte:

2\^32 × 512 byte ≈ 2 TiB.

Disk yang lebih besar tidak dapat dimanfaatkan sepenuhnya.

## Maksimum 4 Primary Partition

Jika membutuhkan lebih banyak partition, digunakan Extended Partition
yang berisi Logical Partition.

Skema ini cukup rumit dibanding GPT.

## Single Point of Failure

Seluruh informasi partition berada di Sector 0.

Jika sector tersebut rusak, informasi partition dapat hilang.

------------------------------------------------------------------------

# 6. GUID Partition Table (GPT)

GPT merupakan standar modern yang menjadi bagian dari spesifikasi UEFI.

Tujuan GPT:

-   menghilangkan batas 2 TiB,
-   mendukung jauh lebih banyak partition,
-   menyediakan redundansi metadata,
-   meningkatkan keandalan.

------------------------------------------------------------------------

# 7. Struktur GPT Secara Detail

Secara konseptual:

``` text
LBA 0     Protective MBR
LBA 1     Primary GPT Header
LBA 2-33  Partition Entry Array

...

Data Partition

...

Last-33   Backup Entry Array
Last-1    Backup GPT Header
```

GPT menyimpan salinan metadata di awal **dan** akhir disk.

------------------------------------------------------------------------

# 8. Protective MBR

Mengapa GPT masih memiliki MBR?

Jawabannya adalah kompatibilitas.

Program lama yang hanya mengenal MBR akan melihat sebuah partition besar
bertipe 0xEE.

Akibatnya program lama tidak mencoba menimpa GPT.

Karena fungsinya melindungi GPT, bagian ini disebut **Protective MBR**.

------------------------------------------------------------------------

# 9. Primary GPT Header

Header GPT menyimpan informasi penting seperti:

-   ukuran disk,
-   lokasi backup GPT,
-   jumlah partition entry,
-   ukuran setiap entry,
-   CRC checksum.

Checksum digunakan untuk mendeteksi kerusakan metadata.

------------------------------------------------------------------------

# 10. Partition Entry Array

Setiap partition memiliki sebuah entri.

Secara konsep setiap entri berisi:

-   Partition Type GUID
-   Unique Partition GUID (PARTUUID)
-   First LBA
-   Last LBA
-   Attribute Flags
-   Partition Name

Dengan demikian GPT mengetahui lokasi pasti setiap partition.

------------------------------------------------------------------------

# 11. Backup GPT

Di akhir disk terdapat salinan GPT.

Jika header utama rusak, utilitas seperti `gdisk` dapat menggunakan
backup untuk membantu proses pemulihan.

Ini merupakan salah satu keunggulan utama GPT dibanding MBR.

------------------------------------------------------------------------

# 12. GUID, PARTUUID, dan UUID

Ketiganya sering tertukar.

## GUID

Globally Unique Identifier.

Merupakan bilangan 128-bit yang dirancang agar unik secara global.

Contoh:

    550e8400-e29b-41d4-a716-446655440000

## PARTUUID

GUID milik sebuah partition.

Dibuat ketika partition dibuat.

## UUID Filesystem

UUID filesystem dibuat ketika filesystem dibuat menggunakan `mkfs`.

Misalnya:

``` bash
mkfs.ext4 /dev/nvme0n1p2
```

Perintah tersebut menghasilkan UUID ext4 baru.

Jadi:

-   PARTUUID mengidentifikasi partition.
-   UUID mengidentifikasi filesystem di dalam partition.

------------------------------------------------------------------------

# 13. Perbandingan MBR vs GPT

  -----------------------------------------------------------------------
  Fitur                         MBR                  GPT
  ----------------------------- -------------------- --------------------
  Maksimum ukuran disk          \~2 TiB              Sangat besar
                                                     (praktis jauh di
                                                     atas kebutuhan saat
                                                     ini)

  Primary partition             4                    Umumnya 128 (dapat
                                                     diubah)

  Backup metadata               Tidak                Ya

  Checksum                      Tidak                Ya

  Kompatibel UEFI               Tidak                Ya (standar modern)
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 14. Hubungan GPT dengan Filesystem

GPT **tidak menyimpan file**.

GPT hanya berkata:

``` text
Partition 2

Mulai : LBA X
Selesai : LBA Y
```

Kemudian filesystem (misalnya ext4) dibangun **di dalam** area tersebut.

``` text
Disk

GPT

Partition

└── ext4

    ├── Superblock
    ├── Block Groups
    ├── Inode
    └── Data Block
```

Jadi GPT berada **di luar filesystem**.

------------------------------------------------------------------------

# 15. Hubungan GPT dengan Boot

Saat komputer dinyalakan:

1.  Firmware (UEFI) membaca GPT.
2.  Firmware mencari EFI System Partition (ESP).
3.  EFI menjalankan bootloader.
4.  Bootloader memuat kernel Linux.
5.  Kernel membaca filesystem.

Perhatikan urutannya:

GPT → EFI Partition → Bootloader → Kernel → Filesystem.

------------------------------------------------------------------------

# 16. Contoh Nyata

Lihat struktur block device:

``` bash
lsblk
```

Lihat partition table:

``` bash
sudo fdisk -l
```

Lihat detail GPT:

``` bash
sudo gdisk -l /dev/nvme0n1
```

Melihat UUID:

``` bash
blkid
```

Contoh alur instalasi Arch Linux:

1.  Membuat GPT.
2.  Membuat EFI System Partition.
3.  Membuat Root Partition.
4.  Menjalankan `mkfs.fat` pada ESP.
5.  Menjalankan `mkfs.ext4` pada Root.
6.  Melakukan mount.
7.  Menginstal sistem.

------------------------------------------------------------------------

# 17. Ringkasan

Konsep hubungan seluruh komponen dapat diringkas sebagai berikut.

``` text
Physical Disk
      │
      ▼
Partition Table (GPT / MBR)
      │
      ▼
Partition
      │
      ▼
Filesystem (ext4, XFS, Btrfs, FAT32)
      │
      ▼
Superblock
      │
      ▼
Block Groups
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

**Inti yang perlu diingat**

-   Disk hanyalah media penyimpanan.
-   GPT/MBR membagi disk menjadi partition.
-   Filesystem dibangun di dalam partition.
-   Filesystem mengatur penyimpanan file menggunakan inode, directory
    entry, dan data block.
-   GPT dan filesystem adalah dua lapisan yang berbeda tetapi saling
    melengkapi.
