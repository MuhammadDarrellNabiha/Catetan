# CCNA 200-301 — Network Topology Architectures

> Blueprint CCNA 1.2
>
> * Two-Tier Architecture
> * Three-Tier Architecture
> * Spine-Leaf Architecture
> * WAN
> * SOHO
> * On-Premise
> * Cloud
> * Hybrid Cloud

---

# Network Topology Architecture

Network Topology Architecture adalah **desain atau cara suatu jaringan dibangun agar mampu memenuhi kebutuhan bisnis, memiliki performa yang baik, mudah dikelola, mudah dikembangkan (scalable), dan memiliki tingkat keandalan (reliability) yang tinggi.**

Berbeda dengan **Physical Topology** (Star, Bus, Ring, Mesh), Network Architecture menjelaskan bagaimana perangkat jaringan disusun, bagaimana trafik mengalir, dan di mana layanan ditempatkan.

---

# 1. Two-Tier Architecture

## Pengertian

Two-Tier Architecture merupakan desain jaringan yang hanya memiliki **2 layer**, yaitu:

* Access Layer
* Core Layer (Core + Distribution digabung)

Diagram:

```text
        Core / Distribution
               │
      ┌────────┴────────┐
      │                 │
   Access            Access
      │                 │
     PC                PC
```

Cisco menggabungkan Distribution dan Core menjadi satu perangkat agar biaya lebih murah dan konfigurasi lebih sederhana.

---

## Fungsi Access Layer

Access Layer merupakan layer yang langsung terhubung dengan endpoint.

Endpoint meliputi:

* PC
* Laptop
* Printer
* Access Point
* IP Phone
* CCTV

Tugas Access Layer:

* VLAN
* Access Port
* Trunk Port
* Port Security
* STP
* PoE

---

## Fungsi Core Layer

Pada Two-Tier, Core juga menjalankan fungsi Distribution.

Tugas Core:

* Inter-VLAN Routing
* Routing
* ACL
* QoS
* Policy
* High-Speed Switching
* Backbone Network

---

## Alur Paket

```text
PC
 │
Access
 │
Core
 │
Access
 │
PC
```

---

## Kelebihan

* Biaya lebih murah
* Konfigurasi sederhana
* Latency rendah
* Mudah dikelola

---

## Kekurangan

* Core bekerja lebih berat
* Kurang cocok untuk jaringan sangat besar
* Skalabilitas terbatas

---

## Cocok Untuk

* Sekolah
* Hotel
* Kantor kecil-menengah
* Rumah sakit kecil

---

# 2. Three-Tier Architecture

## Pengertian

Three-Tier Architecture memisahkan fungsi jaringan menjadi tiga layer.

* Access
* Distribution
* Core

Diagram:

```text
                Core
            ┌────┴────┐
            │         │
     Distribution  Distribution
         │   │         │   │
      Access Access  Access Access
         │     │        │     │
        PC    PC       PC    PC
```

---

## Access Layer

Layer yang langsung terhubung ke endpoint.

Fungsi:

* VLAN
* Access Port
* Trunk
* Port Security
* STP
* PoE

---

## Distribution Layer

Layer penghubung antara Access dan Core.

Tugas Distribution:

* Inter-VLAN Routing
* ACL
* QoS
* Route Filtering
* Policy Enforcement
* Load Balancing
* Route Summarization

Distribution mengumpulkan beberapa Access Switch sebelum diteruskan ke Core.

---

## Core Layer

Core adalah backbone jaringan.

Fokus utama:

* High-Speed Switching
* Fast Routing
* Redundancy
* Backbone

Core **tidak banyak menjalankan policy** agar forwarding paket berlangsung secepat mungkin.

---

## Alur Paket

```text
PC
 │
Access
 │
Distribution
 │
Core
 │
Distribution
 │
Access
 │
PC
```

---

## Kelebihan

* Sangat scalable
* Core tidak mudah overload
* Mudah troubleshooting
* Performa tinggi

---

## Kekurangan

* Mahal
* Konfigurasi kompleks
* Membutuhkan lebih banyak perangkat

---

## Cocok Untuk

* Enterprise
* Universitas
* Rumah sakit besar
* Bandara
* Pemerintahan

---

# Perbedaan Two-Tier vs Three-Tier

| Two-Tier              | Three-Tier                    |
| --------------------- | ----------------------------- |
| Access + Core         | Access + Distribution + Core  |
| Core menangani policy | Distribution menangani policy |
| Lebih murah           | Lebih mahal                   |
| Skalabilitas sedang   | Skalabilitas tinggi           |

---

# 3. Spine-Leaf Architecture

## Pengertian

Spine-Leaf Architecture adalah desain jaringan modern yang digunakan **di dalam data center**.

Tujuan:

* Latency rendah
* Bandwidth tinggi
* Mudah diskalakan
* Mengurangi bottleneck

---

## Komponen

### Spine Switch

Merupakan backbone jaringan.

Spine hanya terhubung ke seluruh Leaf.

Tidak pernah terhubung langsung ke server.

---

### Leaf Switch

Leaf merupakan Layer 3 Switch yang langsung terhubung ke:

* Server
* Storage
* Hypervisor
* Firewall
* Load Balancer

Diagram:

```text
           Spine1
          /      \
       Leaf1    Leaf2
       /   \      |
   Server Server Server
```

---

## Aturan Spine-Leaf

### Setiap Leaf harus terhubung ke seluruh Spine.

Contoh:

```text
          Spine1
         /  |   \
Leaf1 Leaf2 Leaf3
         \  |   /
          Spine2
```

---

### Spine tidak saling terhubung.

❌

```text
Spine1 ----- Spine2
```

---

### Leaf tidak saling terhubung.

❌

```text
Leaf1 ----- Leaf2
```

Komunikasi antar Leaf selalu melewati Spine.

---

## Alur Paket

```text
Server
 │
Leaf
 │
Spine
 │
Leaf
 │
Server
```

---

## Kelebihan

* Sangat scalable
* Semua jalur memiliki jumlah hop yang konsisten
* Redundansi tinggi
* Cocok untuk virtualisasi dan cloud
* Mengurangi bottleneck

---

## Kekurangan

* Mahal
* Konfigurasi lebih kompleks
* Biasanya menggunakan Layer 3 Switch berkecepatan tinggi

---

## Digunakan Untuk

* Data Center
* Private Cloud
* Public Cloud
* Virtualisasi
* Kubernetes Cluster

---

# Apa itu Bottleneck?

Bottleneck adalah kondisi ketika satu perangkat atau satu link memiliki kapasitas lebih kecil dibandingkan jumlah trafik yang harus dilewati.

Contoh:

```text
100 PC
 │
Access Switch
 │
1 Gbps  ← Bottleneck
 │
Core
```

Jika trafik mencapai 5 Gbps tetapi link hanya 1 Gbps, maka akan terjadi antrean paket, latency meningkat, throughput menurun, bahkan packet loss.

Spine-Leaf mengurangi bottleneck dengan menyediakan banyak jalur (ECMP/load balancing) sehingga trafik dapat dibagi ke beberapa Spine.

---

# 4. On-Premise

## Pengertian

On-Premise adalah model di mana seluruh server dan infrastruktur berada di lokasi perusahaan sendiri.

Diagram:

```text
Internet
    │
Firewall
    │
Core
    │
Server Farm
```

Semua server berada di kantor atau data center milik perusahaan.

---

## Kelebihan

* Kontrol penuh
* Data lebih aman
* Tidak bergantung internet untuk akses internal
* Cocok untuk data sensitif

---

## Kekurangan

* Investasi awal besar
* Maintenance dilakukan sendiri
* Membutuhkan ruang server
* Membutuhkan tim IT

---

## Cocok Untuk

* Bank
* Pemerintah
* Militer
* Perusahaan dengan regulasi ketat

---

# 5. Cloud

## Pengertian

Cloud adalah model di mana server berada di data center milik penyedia layanan cloud.

Diagram:

```text
Office
 │
Internet
 │
Cloud Provider
 │
Server
Database
Storage
```

Contoh layanan:

* AWS
* Microsoft Azure
* Google Cloud
* Oracle Cloud
* Alibaba Cloud

---

## Kelebihan

* Tidak perlu membeli server
* Skalabilitas tinggi
* Maintenance hardware dilakukan penyedia cloud
* Bayar sesuai penggunaan

---

## Kekurangan

* Bergantung pada koneksi internet
* Kontrol hardware terbatas
* Biaya dapat meningkat jika penggunaan besar

---

## Cocok Untuk

* Startup
* Website
* SaaS
* Aplikasi modern
* Development

---

# 6. Hybrid Cloud

Hybrid Cloud adalah kombinasi antara On-Premise dan Cloud.

Contoh:

```text
                Internet
                    │
               Cloud Server
                    │
                 VPN/IPsec
                    │
Firewall
    │
Core
    │
Local Database
```

Database tetap berada di kantor.

Website atau aplikasi berada di cloud.

---

## Kelebihan

* Fleksibel
* Data sensitif tetap lokal
* Mudah melakukan migrasi ke cloud
* Skalabilitas lebih baik

---

# Ringkasan

| Architecture | Digunakan Untuk       | Karakteristik                  |
| ------------ | --------------------- | ------------------------------ |
| Two-Tier     | Kantor kecil-menengah | Access + Core                  |
| Three-Tier   | Enterprise besar      | Access + Distribution + Core   |
| Spine-Leaf   | Data Center           | Leaf ↔ Semua Spine             |
| On-Premise   | Infrastruktur lokal   | Server di kantor               |
| Cloud        | Infrastruktur cloud   | Server di data center provider |
| Hybrid Cloud | Kombinasi             | Sebagian lokal, sebagian cloud |

---

# Poin Penting CCNA

* **Two-Tier** → Distribution dan Core digabung menjadi satu.
* **Three-Tier** → Access, Distribution, dan Core dipisahkan agar lebih scalable.
* **Spine-Leaf** → Digunakan di dalam data center modern. Semua Leaf terhubung ke semua Spine, sedangkan Leaf tidak saling terhubung dan Spine juga tidak saling terhubung.
* **On-Premise** → Server berada di lokasi perusahaan.
* **Cloud** → Server berada di data center penyedia cloud.
* **Hybrid Cloud** → Kombinasi On-Premise dan Cloud.
* **Bottleneck** → Titik kemacetan pada jaringan akibat kapasitas link atau perangkat lebih kecil dibandingkan trafik yang harus dilewatkan.
