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




