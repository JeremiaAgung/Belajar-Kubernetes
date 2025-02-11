# Model Deployment Kubernetes

## 1. Menjalankan Kubernetes di Produksi
- **Langkah pertama:** Instalasi dan manajemen upgrade Kubernetes.
- **Tantangan utama:** Kubernetes terkait erat dengan infrastruktur, sehingga instalasi harus mempertimbangkan kebutuhan sistem.

## 2. Managed Service vs Roll Your Own
### ✅ Managed Service
- Mengurangi beban teknis dengan menggunakan layanan cloud seperti AWS EKS, GKE, atau AKS.
- Menghemat waktu dan tenaga dalam manajemen control plane.
- **Kekurangan:** Ketergantungan pada vendor (vendor lock-in).

### ✅ Roll Your Own
- Lebih fleksibel dan bebas dari batasan vendor.
- Memungkinkan konfigurasi penuh sesuai kebutuhan bisnis.
- **Kekurangan:** Membutuhkan tim berpengalaman untuk mengelola Kubernetes.

## 3. Otomatisasi
- **Mengurangi pekerjaan manual** dengan tools otomatisasi untuk deployment Kubernetes.
- **Dua pendekatan utama:**
  1. **Prebuilt Installer** – Menggunakan tool yang sudah ada untuk kemudahan instalasi.
  2. **Custom Automation** – Membangun sistem sendiri untuk fleksibilitas maksimal.

## 4. Arsitektur & Topologi Kubernetes
### 📌 Model deployment etcd
- **Dedicated vs Colocated** – Apakah etcd dijalankan di node terpisah atau digabung dengan control plane.
- **Containerized vs On-Host** – Apakah etcd dijalankan dalam container atau langsung di host.

### 📌 Cluster Tiers
- Pengelompokan cluster berdasarkan kebutuhan aplikasi dan Service Level Objective (SLO).

## 🎯 Kesimpulan
- **Gunakan managed service** jika ingin lebih mudah dan cepat dalam menjalankan Kubernetes.
- **Roll your own** jika ingin kendali penuh dan menghindari ketergantungan vendor.
- **Otomatisasi sangat penting** untuk mengelola Kubernetes dengan efisien.

# Node Pools & Cluster Federation in Kubernetes

## Node Pools
Node pools adalah cara untuk mengelompokkan node dalam satu cluster Kubernetes berdasarkan karakteristik atau peran tertentu.

### Jenis Node Pools
1. **Berdasarkan Karakteristik**
   - Contoh: GPU, tipe jaringan, rasio CPU ke RAM.
   - Digunakan untuk workload yang membutuhkan spesifikasi khusus.

2. **Berdasarkan Peran**
   - Contoh: Node pool khusus untuk ingress.
   - Menghindari konflik sumber daya dan meningkatkan isolasi.

### Keuntungan & Kekurangan
| **Keuntungan** | **Kekurangan** |
|---------------|--------------|
| Mengurangi jumlah cluster yang dikelola | Membutuhkan Node Selectors untuk workload |
| Mengurangi kompleksitas target workload | Perlu mengelola Node Taints |
| Lebih fleksibel dalam penggunaan sumber daya | Scaling menjadi lebih kompleks |

## Cluster Federation
Cluster federation adalah cara untuk mengelola banyak cluster Kubernetes secara terpusat dengan tetap mempertahankan isolasi antar cluster.

### Strategi Federation
- **Regional → Global** → Membagi federasi berdasarkan wilayah untuk efisiensi dan pengurangan latensi.

### Komponen Utama
1. **Management Cluster**
   - Cluster khusus untuk mengelola workload cluster lain.
   - Contoh: Cluster API untuk provisioning otomatis.

2. **Observability**
   - Pemantauan dengan Prometheus untuk melihat status cluster.
   - Mendukung model federasi untuk agregasi data dari berbagai cluster.

3. **Federated Deployment**
   - Menggunakan KubeFed atau CI/CD tools untuk mengelola deployment di berbagai cluster.
   - Memungkinkan deklarasi workload secara terpusat agar diterapkan di banyak cluster.

## Kesimpulan
- **Node Pools** membantu dalam manajemen cluster dengan mengelompokkan node berdasarkan karakteristik atau peran.
- **Cluster Federation** memungkinkan pengelolaan banyak cluster dengan lebih efisien melalui pendekatan terpusat.
- Implementasi keduanya harus mempertimbangkan skenario bisnis, skalabilitas, dan efisiensi operasional.

# Infrastruktur Kubernetes

## Pendahuluan
Kubernetes adalah sistem orkestrasi container yang membutuhkan infrastruktur IT yang memadai untuk dapat berjalan dengan optimal. Sebuah cluster Kubernetes dapat dijalankan di laptop menggunakan virtual machine atau container Docker, tetapi itu hanya untuk keperluan uji coba. Untuk produksi, berbagai komponen infrastruktur harus disiapkan agar Kubernetes dapat berjalan dengan stabil.

## Infrastruktur Dasar Kubernetes
Untuk menjalankan cluster Kubernetes di lingkungan produksi, diperlukan beberapa komponen utama:
- **Mesin (Nodes):** Bisa berupa mesin fisik (bare metal) atau mesin virtual.
- **Jaringan:** Koneksi yang stabil untuk komunikasi antar node.
- **Storage:** Sistem penyimpanan data yang dapat diakses oleh cluster.
- **Load Balancer:** Mengelola lalu lintas masuk ke dalam cluster.
- **Sistem Automasi:** Menghindari konfigurasi manual, lebih baik menggunakan sistem deklaratif untuk penyediaan sumber daya.

## Bare Metal vs Virtualisasi
Banyak yang mempertanyakan apakah layer virtualisasi masih diperlukan jika sudah menggunakan Kubernetes. Berikut adalah kelebihan dan kekurangan dari masing-masing pendekatan:

### **Keuntungan Virtualisasi**
- Dapat membuat dan mengkloning mesin dengan mudah.
- Menjalankan banyak sistem operasi dalam satu server.
- Optimalisasi penggunaan server dengan alokasi sumber daya yang fleksibel.
- Konfigurasi jaringan yang lebih mudah.
- Keamanan yang lebih baik dengan isolasi antar mesin virtual.

### **Tantangan Bare Metal**
- Memerlukan pengelolaan storage yang lebih kompleks.
- Tidak memiliki API Load Balancer seperti di cloud provider.
- Pengaturan keamanan lebih sulit dibandingkan dengan virtualisasi.
- Skalabilitas lebih rumit dibandingkan dengan virtualisasi.

Namun, dengan berbagai alat tambahan seperti **kube-vip**, **metallb**, dan **Ceph**, tantangan-tantangan ini dapat diatasi.

## Ukuran Cluster
Ketika merancang cluster Kubernetes, ada beberapa faktor yang perlu diperhatikan terkait ukuran cluster:

### **Keuntungan Cluster Besar**
- Pemanfaatan sumber daya lebih efisien.
- Mengurangi overhead dari control plane.
- Manajemen workload lebih sederhana.

### **Keuntungan Cluster Kecil**
- Dampak kegagalan lebih kecil.
- Lebih fleksibel dalam pengaturan multi-tenancy.
- Upgrade lebih mudah dengan mengganti cluster lama dengan yang baru.
- Memudahkan pengelolaan workload khusus seperti GPU atau memori tinggi.
  
---

# Infrastruktur Komputasi untuk Kubernetes

## 1. Mesin dalam Cluster Kubernetes
Setiap cluster Kubernetes memerlukan sejumlah mesin dengan spesifikasi tertentu. Berikut beberapa faktor yang perlu dipertimbangkan:
- **Jumlah inti CPU**
- **Kapasitas memori (RAM)**
- **Penyimpanan internal**
- **Kecepatan jaringan**
- **Dukungan perangkat keras khusus (GPU, TPU, dsb.)**

### Jenis Mesin dalam Cluster

#### a. Mesin etcd (Opsional)
- Digunakan jika menjalankan cluster **etcd** terpisah dari control plane.
- **Kriteria utama**:
  - Performa baca/tulis disk tinggi (gunakan SSD, bukan HDD).
  - Penyimpanan terdedikasi agar tidak berbagi sumber daya dengan sistem lain.
  - Latensi jaringan rendah antar mesin untuk replikasi cepat.

#### b. Node Control Plane (Wajib)
- Menjalankan komponen utama seperti **API Server, Scheduler, Controller Manager**.
- Untuk cluster besar, API Server dapat diskalakan secara **horizontal** atau **vertikal**.
- Scheduler dan Controller Manager hanya memiliki **satu pemimpin aktif**, sehingga hanya dapat ditingkatkan secara **vertikal**.

#### c. Worker Nodes (Wajib)
- Menjalankan pod dan aplikasi.
- Biasanya menggunakan mesin **general-purpose**.

#### d. Node dengan Optimasi Memori (Opsional)
- Untuk aplikasi yang membutuhkan memori besar.
- Contoh di AWS:
  - **M5:** 1 CPU : 4 GiB RAM.
  - **R5:** 1 CPU : 8 GiB RAM.

#### e. Node dengan Optimasi CPU (Opsional)
- Untuk aplikasi dengan kebutuhan CPU tinggi.
- Contoh di AWS:
  - **C5:** 1 CPU : 2 GiB RAM.

#### f. Node dengan Perangkat Keras Khusus (Opsional)
- Untuk beban kerja seperti AI/ML, gunakan node dengan GPU.
- Contoh di AWS: **p3, g4dn** dengan GPU NVIDIA.

---

## 2. Infrastruktur Jaringan dalam Kubernetes

### a. Keterjangkauan Jaringan
- Hindari mengekspos node langsung ke internet.
- Gunakan **bastion host** atau **jump box** untuk akses terbatas.
- Pastikan konektivitas ke layanan internal seperti **database, CI/CD, DNS internal**.

### b. Redundansi dan Ketersediaan (Availability Zones - AZs)
- **Gunakan minimal dua AZ**, tiga lebih baik.
- Distribusi control plane ke beberapa AZ meningkatkan ketahanan terhadap kegagalan.

### c. Load Balancing
- **Load balancer diperlukan untuk API Server Kubernetes**.
- Gunakan **ingress controller** untuk aplikasi yang memerlukan akses publik.

---

## 3. Strategi Otomasi Infrastruktur Kubernetes

### a. Alat Manajemen Infrastruktur
- **Terraform & AWS CloudFormation**:
  - Infrastruktur dapat dideklarasikan dalam file konfigurasi.
  - Cocok untuk lingkungan tetap.
  - Kurang fleksibel untuk infrastruktur yang sangat dinamis.

### b. Kubernetes Operators
- **Cluster API** memungkinkan otomatisasi pembuatan dan pengelolaan cluster Kubernetes.
- Lebih fleksibel dibanding Terraform karena dapat menangani kondisi variabel lebih baik.

---

## 4. Instalasi Mesin untuk Kubernetes

### a. Manajemen Konfigurasi
- Gunakan **Ansible, Chef, Puppet, atau SaltStack** untuk konfigurasi otomatis.
- **Kekurangan**:
  - Proses instalasi bisa lambat.
  - Bergantung pada ketersediaan repositori paket.

### b. Menggunakan Image Mesin (Machine Images)
- Metode lebih cepat dan lebih andal.
- Gunakan **Packer** untuk membuat image siap pakai.
- **Keuntungan**:
  - Mengurangi waktu join node ke cluster.
  - Tidak bergantung pada paket yang harus diunduh saat startup.
  - Lebih aman karena image dapat diuji sebelumnya.

**Kesimpulan**: **Gunakan prebuilt machine images** untuk cluster Kubernetes yang stabil dan siap produksi.

---

## 5. Perangkat Lunak yang Harus Diinstal pada Node

1. **Sistem Operasi**
   - Rekomendasi: **Ubuntu, RHEL/CentOS, atau Flatcar Container Linux**.
2. **Container Runtime**
   - Kubernetes memerlukan runtime seperti **Docker atau containerd**.

# Komponen Berbasis Kontainer

## Komponen Kontrol Plane dalam Manifes Statis
Manifes statis yang digunakan untuk membuat cluster harus mencakup komponen penting dari kontrol plane, yaitu:
- `etcd`
- `kube-apiserver`
- `kube-scheduler`
- `kube-controller-manager`

Anda dapat menyediakan manifes Pod tambahan jika diperlukan, tetapi batasi hanya pada Pod yang benar-benar harus dijalankan sebelum API Kubernetes tersedia atau didaftarkan ke dalam sistem federasi.

> Jika workload dapat diinstal melalui API server nanti, lakukan dengan cara tersebut.

Manifes statis hanya dapat dikelola dengan mengedit file manifes di disk mesin yang menjalankan cluster, yang kurang fleksibel dan sulit untuk diautomasi.

## Menggunakan `kubeadm`
Jika menggunakan `kubeadm` (yang sangat disarankan), manifes statis untuk kontrol plane, termasuk `etcd`, akan dibuat saat node kontrol plane diinisialisasi dengan perintah:
```sh
kubeadm init
```

Setiap konfigurasi tambahan untuk komponen ini dapat diteruskan ke `kubeadm` menggunakan file konfigurasi `kubeadm config`.

### Praktik Terbaik dalam Customisasi
- Hindari menyesuaikan manifes statis secara langsung dengan bootstrap utility.
- Jika sangat diperlukan, lakukan pembuatan manifes statis dan inisialisasi cluster secara terpisah menggunakan `kubeadm` untuk memungkinkan injeksi konfigurasi khusus.
- Jangan mengubah aset TLS yang dibuat sendiri oleh `kubeadm`, kecuali ada kebutuhan keamanan khusus untuk menggunakan CA perusahaan.

---

# Add-ons dalam Kubernetes

## Pengertian Add-ons
Add-ons adalah layanan tambahan yang dibangun di atas cluster Kubernetes. Add-ons biasanya mencakup:
- DNS cluster (`CoreDNS`)
- `kube-proxy`
- Plugin Container Network Interface (CNI)
- Monitoring dan logging
- Security policies

## Pengelolaan Add-ons
Add-ons harus dianggap sebagai bagian dari model deployment dan dikelola bersamaan dengan cluster Kubernetes. Beberapa pendekatan yang umum digunakan adalah:

### 1. Continuous Delivery Pipeline
Menggunakan alat seperti Jenkins untuk secara otomatis menginstal add-ons ke cluster baru.

### 2. Kubernetes Operator
Operator adalah cara yang lebih maju untuk mengelola add-ons, yang memungkinkan:
- Pengelolaan siklus hidup add-ons secara otomatis.
- Menyediakan sumber kebenaran pusat (centralized source of truth) untuk add-ons yang digunakan di cluster.

Jika memilih operator, pastikan tim memiliki pengalaman dengan pola operator Kubernetes untuk menghindari kompleksitas yang tidak perlu.

---

# Upgrade Cluster Kubernetes

## Versi Platform
- Dokumentasikan versi semua komponen platform, termasuk OS, runtime kontainer, Kubernetes, dan add-ons.
- Gunakan sistem penomoran versi yang jelas dan konsisten.

## Strategi Rollback
Karena kemungkinan adanya kegagalan selama upgrade, perlu disiapkan:
- Backup otomatis `etcd` dan sumber daya Kubernetes menggunakan Velero.
- Recovery otomatis untuk data persisten aplikasi.
- Playbook rollback yang terstruktur dan teruji.
- Cluster standby untuk failover pada aplikasi kritis.

## Pengujian Integrasi
Pengujian integrasi sangat penting untuk memastikan semua komponen bekerja dengan baik setelah upgrade.

### Penggunaan Sonobuoy untuk Pengujian
Sonobuoy dapat digunakan untuk menjalankan:
- Tes konformitas Kubernetes.
- Tes spesifik berdasarkan kebutuhan organisasi.
- Tes otomatis yang dijadwalkan menggunakan `CronJob` Kubernetes.

### Pengujian Integrasi Aplikasi
Pastikan workload yang berjalan di cluster diuji pada lingkungan yang menyerupai produksi sebelum diterapkan ke production.

---

# Kesimpulan
Dengan mengikuti prinsip-prinsip di atas, tim dapat:
- Mengelola cluster Kubernetes dengan lebih efisien.
- Menghindari kesalahan dalam deployment.
- Menjaga keberlanjutan dan keandalan sistem.

# Strategi Upgrade Kubernetes

Dalam panduan ini, kita akan membahas tiga strategi utama untuk melakukan upgrade pada platform berbasis Kubernetes:

1. **Penggantian Cluster (Cluster Replacement)**
2. **Penggantian Node (Node Replacement)**
3. **Upgrade di Tempat (In-Place Upgrade)**

Urutan ini disusun berdasarkan **biaya dan risiko**: dari yang memiliki biaya tertinggi tetapi risiko terendah hingga yang memiliki biaya terendah tetapi risiko tertinggi. Setiap strategi memiliki **kompromi** sendiri yang harus dipertimbangkan berdasarkan **anggaran, kebutuhan, dan toleransi risiko**.

## 1. Penggantian Cluster (Cluster Replacement)

**Metode ini memiliki biaya tertinggi tetapi risiko paling rendah.**

### Cara Kerja:
1. **Membuat cluster baru** dengan versi terbaru Kubernetes.
2. **Memigrasikan workload** dari cluster lama ke cluster baru.
3. **Menskalakan cluster baru** secara bertahap sambil mengurangi kapasitas cluster lama.
4. **Menghapus cluster lama** setelah semua workload berhasil dipindahkan.

### Tantangan:
- **Manajemen Traffic Ingress**: 
  - Gunakan **Global Service Load Balancer (GSLB)** atau **Reverse Proxy** agar traffic bisa dialihkan ke cluster baru.
- **Penyimpanan Persisten**:
  - Pastikan **storage** dapat diakses dari kedua cluster.
  - Untuk **cloud**, pastikan layanan penyimpanan bisa diakses dari kedua cluster.
  - Di **private data center**, sesuaikan pengaturan jaringan dan firewall.
- **Migrasi Workload**:
  - **Cara 1: Redeploy dari Source of Truth** (seperti GitOps).
  - **Cara 2: Menyalin Resource dari Cluster Lama ke Cluster Baru** menggunakan **Velero**.

---

## 2. Penggantian Node (Node Replacement)

**Strategi ini memiliki biaya dan risiko yang sedang.**

### Cara Kerja:
1. **Mengganti node secara bertahap** dengan yang baru.
2. **Memastikan kompatibilitas** workload dengan versi Kubernetes yang baru.
3. **Menggunakan Cluster API** untuk mengelola penggantian node secara otomatis.

### Cara Mengurangi Risiko:
- **Baca Release Notes Kubernetes** untuk memahami API yang berubah atau dihapus.
- **Uji coba di lingkungan development dan staging** sebelum diterapkan di production.
- **Minimalkan ketergantungan terhadap API Kubernetes** dalam aplikasi.
- **Gunakan image mesin yang telah diuji** untuk memastikan stabilitas.

### Urutan Penggantian Node:
1. **Mulai dari cluster etcd jika ada**: 
   - Gunakan metode **subtractively** (menghapus satu node lama, lalu menambahkan node baru).
2. **Ganti node Control Plane**:
   - Gunakan `kubeadm join --control-plane`.
3. **Ganti node Worker**:
   - Bisa dilakukan **satu per satu** atau **beberapa sekaligus** tergantung kapasitas cluster.
---

## 3. Upgrade di Tempat (In-Place Upgrade)

**Metode ini memiliki biaya terendah tetapi risiko tertinggi.**

### Cara Kerja:
1. **Upgrade dilakukan langsung di setiap node** tanpa menggantinya.
2. **Lakukan upgrade satu per satu** untuk mengurangi risiko downtime.
3. **Gunakan alat seperti Ansible** untuk mengotomatiskan proses.

### Tantangan dan Cara Mengatasinya:
- **Upgrade etcd**:
  - Upgrade satu node etcd pada satu waktu.
  - Jika menggunakan container, **pre-pull image** sebelum menghentikan node.
- **Upgrade Control Plane dan Worker Node**:
  - Jika menggunakan `kubeadm`, gunakan perintah upgrade bawaan.
  - Selalu **uji coba di staging sebelum production**.

---

## 4. Mekanisme Pemicu Upgrade

Bagaimana cara memulai proses upgrade secara otomatis?

- **GitOps**:
  - Menggunakan **repository kode** sebagai sumber kebenaran (source of truth).
  - Ketika ada perubahan di repository, upgrade otomatis dijalankan.
- **Cluster API**:
  - Menggunakan **Custom Resources di Kubernetes** untuk mendeklarasikan upgrade.
  - Operator Kubernetes akan menangani upgrade sesuai deklarasi tersebut.
- **Gabungan GitOps & Cluster API**:
  - Konfigurasi disimpan di Git, lalu dikelola oleh Cluster API untuk otomatisasi yang lebih baik.

---

## Kesimpulan

Ketika merancang model upgrade Kubernetes:
- **Otomatisasi adalah kunci utama.**
- **Pahami kebutuhan spesifik cluster Anda.**
- **Gunakan strategi yang paling sesuai** dengan anggaran dan toleransi risiko.
- **Selalu uji coba sebelum diterapkan di production.**
- **Kelola penyimpanan, networking, dan manajemen traffic dengan baik.**


