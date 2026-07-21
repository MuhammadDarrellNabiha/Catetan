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





