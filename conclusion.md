|----1st brainstorming----|

NGFW (Next-Generation Firewall) merupakan versi terbaru firewall yang di ciptakan untuk menutupi kekurangan firewall traditional, firewall ini berkerja dengan cara menambahkan beberapa fitur baru yang memungkinkan untuk melihat sebuah packet secara detail, berbeda dengan firewall traditional yang hanya melihat header header TCP/IP firewall next gen melakukan inspeksi hingga data payload, firewall ini juga memiliki database signature untuk mencocokan banyak pola komunikasi serta jenis serangan.

Fitur-Fitur NGFW

SSL-TLS Inspection digunakan untuk menganalisa jenis aplikasi dan pola komunikasi aplikasi dengan mendekripsi data payload yang terenkripsi
setelah data di dekripsi semua fitur lain bisa dijalankan DPI application Identification dsb, dalam proses nya ngfw bertindak sebagai MITM (man in the middle).


DPI (Deep Packet Inspection)

fitur ini berkerja dengan tiga langkah:
-parsing header
DPI engine melakukan parsing untuk mendapatkan informasi dari setiap heade
-parsing payload
DPI engine melakukan parsing untuk mendapatkan infromasi tentang application identity, file permssion, dsb
-signature matching
setelah mendapatakan informasi dari kedua proses tersebut, semua informasi di cocokan dengan signature dari database yang di milik ngfw.

Application Awerness merupakan fitur yang dapat membuat firewall mengetahuiaplikasi yang digunakan user, infromasi tentang aplikasi yang di gunakan user di dapat dari beberapa cara:
-TLS metadata
-DPI payload parsing
-signature matching
-behaveorail analysis
setelah informasi telah didapat semua informasi akan di proses di dan dikategorikan berdasarkan kategori yang telah di simpan di database, sebuah aplikasi yang tidak terdaftar maka akan di kategorikan unrated.

URL filtering  merupakan kemampuan NGFW untuk mengetahui url yang di akese user


IPS dan IDS

IPS (intrusion prevention system) sebuah system yang dimilik ngfw untuk mencegah sebuah serangan siber yang akan terjadi sebelum serangan siber itu benar benar terjadi, system ini berkerja dengan mem-block semua paket atau data yang kemungkinan akan menjadi bahaya di dalam keamanan jaringan, sebelumnya IPS sudah melakukan 2 Jenis Analisa
1. Behavorial analysis, IPS melakukan analisa dengan mencocokan kebiasaan paket dengan database firewall jika terdeteksi sebagai thread maka paket itulangsung di blok

2. anomaly detection, IPS melakukan analisa dengan melihat kelakukan jaringan yang aneh jika memang ada serangan siber.

IDS (intrusion detection System)
sama kayak IPS tapi beda nya dia hanya mendetect dan memberikan alret ke firewal tanpa melakukan action

Network Topology Architecture
two-tier-architecture
merupakan architecture jaringan yang desain nya menggabungkan distribution dan core layer kedalam layer yang sama. desain ini di buat dengan tujuan pengaplikasian kedalam jaringan kecil. desain jaringan ini juga sering disebut dengan collapsed-core-architecture.
three-tier-architecture
merupakan architecture jaringn standar yang dibuat cisco untuk mengaplikasikan sebuah jaringan
desain ini membagi layer jaringan menjadi 3 layer
core layer
distribution layer
access layer
spine-leaf-architecture
merupakan architecture jaringan yang digunakan khusus untuk komunikasi antar dua data center modern. desain ini dibuat karena data center memerlukan skalbilitas dan transfer kink yang sangat cepat desain jaringan ini memilik dua bagian utama yaitu 
spine switch adalah switch yang digunakan untuk menghubngkan dua leaf switch
leaf swithc adalah switch yang terhubng langsung dengan server atau data center
rule unik dalam desain ini adalah setiap leaf harus terhubung dengan spine leaf tidak boleh terhubng dengan leaf spine juga tidak boleh terhubng langsung dengan spine
on premise
desain jaringan yang menentukan tata letak server berada di dalam jaringan lan local atau di gedung perusahaan
Cloud
desain jaringa yang menentukan tata letak server berada di layanan penydeia server
Hybird
gabugnan antara cloude and on premise

|--------------2nd-Brainstorming------------|

OS Archittecture

dalam sebuah operating system terdapat arsitektur yang membedakan process berdasrakan level abstraksinya, level abstraksi ini dibuat dengan tujuan untuk mempermudah user dalam mengorganize tiap fungsi dari sebuah os

ada tiga level abstaraksi

level pertama merupakan hardaware,
hardware merupakan bentuk fisik dari bagian system yang sedang kita gunakancontoh montitor gpu memory cpu

level kedua adalah kernel space,
kernel space merupakan level abstraksi dimana semua code kernel dijalankan untuk mengarahkan hardware melakukan tugas nya, semua code kernal ini dijalankan di kernel mode yang artinya code ini dapet menginisialiasi semua fungsi hardware tanpa ada batas, ini merupakan kelebihan yang sangat mengerikankarena jika tidak sesuai dengan kemampuan hardware maka akan merusak hardware

level ketiga adalah user space, tempat diamana semua process yang berjalan dengan user mode dijalankan tempat ini adalah tempat dimana user mengarahkan perintah yang akan di teruskan ke kernel malalui systemcall

Systemcall

systemcall adalah sebuah interface yang menghubungkan kernel space dengan uuser space, tiap program/process yang membutuhkan hardware untuk berkerja harus mengirimkan systemcall ke kernel karena hanya kernel yang punya kendali ke hardwar, systemcall berbentuk sebuah function/request yang merupakan pintu masuk dan permintaan ke arah kernel

ada banyak jenis systemcall, namun dalam pembahasan kali ini kita belajar tentang fork() systemcall, systemcall ini dapat melakukan permintaan untuk kernel membuat sebuah process yang identik dengan process aslinya, duplikasiprocess ini akan menghasilkan dua process identik yang dinamai child process semua hasil child process akan dikirm ke process asli yang dinamai parent process untuk ditampilkan di monitor.

CPU Program and Process Flow

seperti yang kita bahas di OS architecture, CPU memiliki perilaku berbeda terhadap process yang berkerja di dalam user mode dan kernel mode, perbedaanini dapat menentukan flow yang akan dilakukan cpu dalam mengkseuki perintahdan menerapkan nya ke memori


|---------------3rd Brainstorming--------------------------------------|
Partition table
partition table adalah metadata yang digunakan linux untuk mengetahui identitas dari sebuah disk, pada awal nya disk yang baru keluar dari pabrik hanya berisi sektor sektor kosong yang sebenarnya bisa di isi linux namun jika linux kernel langsung mengsisi disk tersebut maka isis itu tidak akan berarturan, disini lah kegunaan metadata partition table, tujuan nya untuk memberitahu kernel tentang informasi dari sebuah disk sehingga disk bisa di partisi dan diisi secara teratur, biasanya isi dari pertition table adalah id disk,list partisi yang ada di sebuah disk, awal file bisa di tulis, akhir file bisa di tulis. partition table terletak di awal sectordisk supaya efi dapat membaca dan mendeteksi disk saat boot

GPT dan MBR
GPT merupakan singkatan dari GUID PARTITION TABLE , partition table ini menggunakan global uniqe identifier untuk mengidentifikasi sebuah disk, gpt merupakan partition table paling baru yang sangat fleksibel karena tidak memiliki aturan unik seperti mbr dalam pembuatan partisi nya karena disusun dengan kapasitas yang sangat besar yaitu 128 bit

MBR merupakan singkatan dari master boot record, merupakan partition table lama yang berdiri dari awal ibm pc partisi ini memiliki keterbatasan dalam membuat metadata yaitu pembatasan ukuran disk yaitu 2tb, mbr juga memiliki kelemahan yaitu tidak dapat membuat lebih dari 4 partisi. ia mamiliki dua struktur partisi yaitu primary partirion dan extended partition yang mewakili logical partition

filesystem
filesystem adalah aturan yang dibuat untuk mengaatur file yang di masukan kedalam sebuah disk atau partisi. jenis jenis filesystem :
ext4 (fourth extended filesystem) merupakan filsesystem yang diguakan oleh linux untuk di jadikan sebuah / dierctory bermula
xhs  merupakan filesystam yang digunakan untuk server beser akrena terkanel dengan kecepatan serta kompatibilitas nya yang sangat bagus
fat32 filesystem yang digunakan oleh EFI

anatomi filesystem ext4
Filesystem (ext4)
│
├── Block Group 0
│     ├── Primary Superblock
            superblock merupakan metadata dari sebuah filesystem, metadata            ini digunakan untuk mengetahui informasi seperti total block              di dalam filesystem dan juga jenis filesystemc
│     ├── Primary GDT
│     ├── Block Bitmap
            block bitmap merupkan indikator angka biner yang memberitahu k            ernel bahwa sebuah block sudah terisi oleh sebuha data 1 berar            -ti sudah terisi dan 0 belom terisi
│     ├── Inode Bitmap
            indode bitmap adalah indikator angka biner yang memberitahukan            kernel bahwa sebuah inode di inode table telah di gunakan.
│     ├── Inode Table
            apa itu inode,inode merupakan singkatan dari index node, yang             merupakan sebuah value yang biasanya sebuah integar dijadikan             sebagai identifier sebuah file dan directory, identifier ini              berbentuk metadata yang digunakan kernel untuk mengetahui di -            block mana sebauh fila bermula dan berakhir (Pointer) serta 
            metadata ini juga memberitahu bahwa di data block tersebut 
            merupkan sebuahfile ataupun folder, inode juga menyimpan 
            informasi tentang perrmission owner dan sebagainya. inode
            table adalah sebuah tempat yang digunakan untuk menyimpan
            inode.
│     └── Data Blocks
            data Blocks merupakan tempat data dari sebuah file disimpan
            dalam bentuk raw nya yaitu bilang biner 0 dan 1
            
│
├── Block Group 1
│     ├── Block Bitmap
│     ├── Inode Bitmap
│     ├── Inode Table
│     └── Data Blocks
│
├── Block Group N (yang dipilih sparse_super)
│     ├── Backup Superblock
│     ├── Backup GDT
│     ├── Block Bitmap
│     ├── Inode Bitmap
│     ├── Inode Table
│     └── Data Blocks


anatomi gpt partition table
SSD

├── GPT Header
├── Partition Entry Array
│
├── Partition 1
├── Partition 2
├── Partition 3
│
├── Backup Partition Entry Array
└── Backup GPT Header


directory-entry

ketika kernel melakukan paths-resolution kernel tidak menjelajahi setiap directory dengan membaca setiap inode dari inode table, melainkan dari informasi dari directory entry. directory entry merupakan data yang berasal dari sebuah directory yang isinya merupkan inode serta nama dari sebuah fileatau directory yang berada di dalam directory tersebut, dengan directory entry kernal akan lebih efisien dibandingkan kernel haru mengecek tiap inode dari awal sampai akhir dan menentukna path nya satu persatu.

hardlink

hardlink merupakan hasil dari command ln dan digunakan untuk menduplkat file entry kedalam inode yang sama namun di dalam directory entry terdapat dua nama yang mengarah ke sebuah inode. ketika kita membuat hardlink terdapat 1 pointer ata

symlink 
symlink merupkan hsil dari command ls -s yang membuat sebuah inode baru yang berisi path dari folder asli berada.

VFS
vfs(virtual filesystem) merupakan bagian kernel yang dapat memahami semua bahasa driver yang ada ntfs,xfs,ext4 dan sebagainya. vfs merupakan jembatan dari kernel ke driver karena kernel tidak mengerti tiap bahasa driver dari sebuah filesystem secara langsung, proses path resolutoion dilakukan oleh vfs.

mounting
proses mount adalah proses mengabungkan dua disk agar dapat posisi yang sama di mata kernel sehingga kernal dapat mengira bahwa disk yang telah di mount merupakan satu kesatuan dengan disk lainya. mounting berkerja dengan memansan sebuah point access di dalam disk utama yang ingin di gabungkan.

command partittining, formatiing, mounting

1. sudo fdisk -l dev/sdx
2. lsblk -o NAME,etc

3. sudo fdisk dev/sdx option: -n new partition
                                -g new gpt (guid partition table)
                                -m new mbr (master boot record)
                                -w write new partition 
4. sudo mkfs -t <type> <device>
5. sudo mkfs -t <type> -s size=<size> <device>
    sudo mkfs -t vfat -F 32 <device> ---> efi partiion or FAT filesystem

6. sudo mount /dev/sdx /mountpoint
7. verfikasi filesystem & Mount : -df -Th <device>
                                    -lsblk -o NAME,MOUNPOINT
8. cek UUID : blkid <device>
              -lsblk -o NAME,UUID | grep -i <device>
9. add /etc/fstab option=defaults dump=0 pass= <2> ---> for ext4 
                                                <0> ---> for xfs
                                                <1> ---> FAT32 || efi


                                    
10. findmnt <mountpoint>
    sudo mount -a ---> checking fstab format

11. done
