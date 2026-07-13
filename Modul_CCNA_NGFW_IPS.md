# Modul CCNA -- Next-Generation Firewall (NGFW) dan Intrusion Prevention System (IPS)

> **Level:** CCNA (dengan penjelasan mendekati level implementasi)\
> **Tujuan:** Memahami konsep, fungsi, dan mekanisme kerja NGFW beserta
> fitur-fitur utamanya.

------------------------------------------------------------------------

# 1. Next-Generation Firewall (NGFW)

## 1.1 Definisi

Next-Generation Firewall (NGFW) adalah firewall modern yang tidak hanya
memfilter paket berdasarkan **IP Address, Port, dan Protocol**, tetapi
juga mampu memahami **aplikasi**, **isi paket**, **URL**, **identitas
pengguna**, dan **ancaman keamanan**.

Firewall tradisional bekerja terutama pada Layer 3 dan Layer 4,
sedangkan NGFW mampu melakukan inspeksi hingga Layer 7 (Application
Layer).

------------------------------------------------------------------------

## 1.2 Mengapa NGFW Dibutuhkan?

Firewall tradisional hanya menjawab pertanyaan:

-   Dari IP mana paket berasal?
-   Menuju IP mana?
-   Menggunakan port berapa?
-   Menggunakan protokol apa?

Masalahnya, aplikasi modern hampir semuanya menggunakan HTTPS (TCP/443).

Contohnya:

-   YouTube
-   GitHub
-   WhatsApp
-   Microsoft Teams
-   Netflix

Semuanya menggunakan port yang sama.

Akibatnya firewall tradisional tidak dapat membedakan aplikasi maupun
ancaman yang berada di balik koneksi HTTPS.

NGFW dikembangkan untuk mengatasi keterbatasan tersebut.

------------------------------------------------------------------------

# 2. Arsitektur Pemrosesan Paket pada NGFW

Secara konseptual paket diproses seperti berikut.

``` text
Packet Masuk
      │
      ▼
Session Lookup
      │
      ▼
Firewall Policy
      │
      ▼
Application Detection
      │
      ▼
SSL/TLS Inspection (jika diaktifkan)
      │
      ▼
Deep Packet Inspection
      │
      ▼
IPS
      │
      ▼
URL Filtering
      │
      ▼
Logging
      │
      ▼
Forward / Drop
```

Setiap modul memiliki fungsi berbeda.

------------------------------------------------------------------------

# 3. Deep Packet Inspection (DPI)

## Definisi

Deep Packet Inspection (DPI) adalah teknik inspeksi paket yang
menganalisis tidak hanya header paket, tetapi juga payload (isi data).

## Tujuan

-   Mengidentifikasi aplikasi.
-   Mengidentifikasi jenis file.
-   Mendeteksi malware.
-   Mendeteksi exploit.
-   Menjadi dasar berbagai fitur NGFW.

## Cara Kerja

### Tahap 1 --- Paket diterima

NGFW menerima frame Ethernet.

``` text
Ethernet
   ↓
IP
   ↓
TCP/UDP
   ↓
Payload
```

------------------------------------------------------------------------

### Tahap 2 --- Parsing Header

Firewall membaca:

-   Source IP
-   Destination IP
-   Source Port
-   Destination Port
-   Protocol
-   TCP Flags
-   TTL

------------------------------------------------------------------------

### Tahap 3 --- Parsing Payload

Firewall melanjutkan inspeksi hingga payload.

Misalnya ditemukan:

    4D 5A

Magic Number tersebut menunjukkan file Windows PE (Executable).

Contoh lain:

  Magic Number   Jenis File
  -------------- --------------------
  %PDF           PDF
  PK             ZIP
  4D 5A          Windows Executable

Firewall tidak hanya mempercayai nama file, tetapi memeriksa struktur
sebenarnya.

------------------------------------------------------------------------

### Tahap 4 --- Signature Matching

Payload dibandingkan dengan database signature.

    Payload
        ↓
    Signature Database
        ↓
    Match?

Jika cocok dengan malware yang telah dikenal, firewall dapat:

-   Block
-   Quarantine
-   Alert

------------------------------------------------------------------------

## Kesimpulan DPI

DPI merupakan mesin analisis utama pada NGFW.

Namun DPI **bukan fitur akhir**, melainkan mekanisme yang mendukung
fitur lain seperti:

-   Application Awareness
-   IPS
-   Antivirus
-   Data Loss Prevention

------------------------------------------------------------------------

# 4. SSL/TLS Inspection

## Mengapa Dibutuhkan?

Sebagian besar trafik internet menggunakan HTTPS.

Tanpa SSL Inspection firewall hanya melihat data terenkripsi.

    ##########
    ##########
    ##########

Firewall tidak dapat mengetahui isi komunikasi.

------------------------------------------------------------------------

## Cara Kerja

Firewall bertindak sebagai proxy terpercaya.

    Client
        │
        ▼
    NGFW
        │
        ▼
    Website

### Langkah 1

Client meminta koneksi HTTPS.

### Langkah 2

Firewall membuat koneksi TLS baru ke server.

### Langkah 3

Firewall menerima data terenkripsi.

### Langkah 4

Firewall mendekripsi data.

### Langkah 5

Engine DPI, IPS, Antivirus, dan URL Filtering melakukan inspeksi.

### Langkah 6

Jika aman, firewall mengenkripsi ulang data dan mengirimkannya ke
client.

## Fungsi

-   Memungkinkan DPI membaca payload HTTPS.
-   Memungkinkan IPS memeriksa exploit dalam HTTPS.
-   Memungkinkan Antivirus memindai file download.
-   Memungkinkan inspeksi HTTP Header dan URL lengkap.

> Catatan: Agar browser mempercayai proses ini, organisasi harus
> memasang sertifikat CA milik firewall pada endpoint.

------------------------------------------------------------------------

# 5. Application Awareness

## Definisi

Application Awareness adalah kemampuan NGFW untuk mengenali aplikasi
yang sedang digunakan, meskipun aplikasi tersebut menggunakan port yang
sama.

## Tujuan

Administrator dapat membuat policy berdasarkan aplikasi, bukan hanya
berdasarkan port.

Contoh:

    Allow
    - GitHub
    - Microsoft Teams

    Block
    - TikTok
    - Torrent

------------------------------------------------------------------------

## Mekanisme Kerja

Application Awareness **bukan hasil dari DPI saja**.

NGFW menggunakan kombinasi beberapa teknik.

### 1. TLS Metadata

Firewall membaca informasi seperti:

-   SNI (Server Name Indication)
-   Sertifikat server

Misalnya:

    SNI = youtube.com

------------------------------------------------------------------------

### 2. DPI

Firewall membaca payload (jika tersedia).

------------------------------------------------------------------------

### 3. Signature Matching

Vendor memiliki ribuan signature aplikasi.

Firewall mencocokkan pola komunikasi dengan database tersebut.

------------------------------------------------------------------------

### 4. Behavioral Analysis

Firewall melihat perilaku trafik.

Contoh:

-   Pola streaming video
-   Pola VoIP
-   Pola sinkronisasi cloud

------------------------------------------------------------------------

### Hasil Akhir

Semua informasi digabungkan.

    TLS
    +
    DPI
    +
    Signature
    +
    Behavior
            │
            ▼
    Application Awareness

------------------------------------------------------------------------

# 6. URL Filtering

## Definisi

URL Filtering adalah kemampuan NGFW untuk mengontrol akses website
berdasarkan:

-   Domain
-   URL
-   Kategori Website

------------------------------------------------------------------------

## Tujuan

-   Memblokir website tertentu.
-   Membatasi akses berdasarkan kategori.
-   Mengurangi risiko phishing.
-   Mengurangi akses ke website berbahaya.

------------------------------------------------------------------------

## Cara Kerja

### HTTP

Firewall membaca:

    Host: facebook.com

------------------------------------------------------------------------

### HTTPS

Firewall membaca metadata TLS (misalnya SNI jika tersedia).

    SNI

    ↓

    facebook.com

------------------------------------------------------------------------

### Database Kategori

Firewall membandingkan domain dengan database vendor.

    facebook.com

    ↓

    Social Media

------------------------------------------------------------------------

### Policy

Administrator menentukan policy.

    Social Media

    ↓

    Block

Firewall kemudian:

    Drop Packet

------------------------------------------------------------------------

## Kategori Website

Contoh kategori:

-   Social Media
-   Gambling
-   Phishing
-   Malware
-   News
-   Streaming
-   Education
-   Finance

------------------------------------------------------------------------

# 7. Intrusion Prevention System (IPS)

## Definisi

Intrusion Prevention System (IPS) adalah sistem keamanan yang
menganalisis trafik jaringan untuk mendeteksi sekaligus **mencegah**
serangan secara real-time.

Berbeda dengan IDS yang hanya memberikan peringatan, IPS dapat langsung
mengambil tindakan.

------------------------------------------------------------------------

## Fungsi

-   Memblokir exploit.
-   Memblokir malware.
-   Memblokir port scanning tertentu.
-   Memblokir SQL Injection.
-   Memblokir Directory Traversal.
-   Memblokir Remote Code Execution.

------------------------------------------------------------------------

## Mekanisme Kerja IPS

### Tahap 1

Paket masuk.

------------------------------------------------------------------------

### Tahap 2

Payload dianalisis menggunakan DPI.

------------------------------------------------------------------------

### Tahap 3

Payload dibandingkan dengan database signature.

Contoh:

    ${jndi:ldap://...}

↓

Signature Log4Shell

↓

Block

------------------------------------------------------------------------

### Tahap 4

Jika tidak cocok dengan signature, IPS dapat menggunakan:

-   Anomaly Detection
-   Behavioral Analysis

Misalnya:

Biasanya server menerima:

100 request/detik

Tiba-tiba:

20.000 request/detik

IPS dapat mendeteksi adanya anomali.

------------------------------------------------------------------------

### Tahap 5

Firewall menjalankan aksi.

Contoh:

-   Drop Packet
-   Reset Connection
-   Block Source IP
-   Generate Log

------------------------------------------------------------------------

# 8. Hubungan Antar Fitur

    Packet Masuk
          │
          ▼
    SSL/TLS Inspection
          │
          ▼
    Deep Packet Inspection
          │
          ├───────────────┐
          ▼               ▼
    Application      IPS
    Awareness
          │
          ▼
    URL Filtering
          │
          ▼
    Firewall Policy
          │
          ▼
    Allow / Block

DPI merupakan salah satu fondasi utama. Fitur lain memanfaatkan hasil
analisis yang dilakukan oleh DPI bersama teknik lain seperti metadata
TLS, signature database, dan analisis perilaku.

------------------------------------------------------------------------

# Ringkasan

  ------------------------------------------------------------------------
  Fitur             Fungsi            Mekanisme Utama
  ----------------- ----------------- ------------------------------------
  NGFW              Firewall modern   Menggabungkan berbagai engine
                    Layer 7           keamanan

  DPI               Memeriksa isi     Parsing payload, signature matching
                    paket             

  SSL/TLS           Membuka trafik    Decrypt → Inspect → Encrypt
  Inspection        HTTPS untuk       
                    inspeksi          

  Application       Mengenali         DPI + TLS Metadata + Signature +
  Awareness         aplikasi          Behavior

  URL Filtering     Mengontrol akses  Domain/SNI → Database Kategori →
                    website           Policy

  IPS               Mencegah serangan DPI + Signature + Anomaly + Behavior
  ------------------------------------------------------------------------
