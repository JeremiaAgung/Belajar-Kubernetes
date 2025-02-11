# Jaringan Pod dalam Kubernetes

## Pendahuluan
Sejak awal perkembangan jaringan komputer, kita telah berusaha untuk memfasilitasi komunikasi antar-host. Tantangan utama dalam jaringan meliputi:

- **Pemberian alamat unik untuk setiap host**
- **Routing paket antar jaringan**
- **Penyebaran rute yang diketahui**

Selama lebih dari satu dekade terakhir, **Software-Defined Networking (SDN)** telah berkembang pesat untuk mengatasi tantangan ini dalam lingkungan yang semakin dinamis. Contoh implementasi SDN termasuk **VMware NSX di pusat data** dan **Amazon VPC di cloud**.

Di Kubernetes, prinsip-prinsip ini tetap relevan. Namun, unit terkecil yang perlu kita kelola bukan lagi host, melainkan **Pod**. Oleh karena itu, kita perlu memastikan bahwa setiap Pod memiliki alamat IP yang dapat diakses dan dapat dirutekan.

Sebagian besar jaringan dalam Kubernetes berbasis perangkat lunak (**software-defined networks**) karena Pod berjalan sebagai perangkat lunak di dalam host.

## Konsep Dasar Jaringan Pod
Bab ini akan membahas konsep dasar yang perlu dipahami sebelum menerapkan jaringan Pod, termasuk:

1. **Manajemen Alamat IP (IPAM)**
2. **Protokol Routing**
3. **Enkapsulasi dan Tunneling**
4. **Routability Workload**
5. **Dukungan IPv4 dan IPv6**
6. **Keamanan dan Enkripsi Trafik Workload**
7. **Kebijakan Jaringan (Network Policy)**

Dengan memahami aspek-aspek ini, kita dapat memilih solusi jaringan yang tepat untuk platform Kubernetes yang digunakan.

---

## 1. Manajemen Alamat IP (IPAM)
Dalam Kubernetes, setiap **Pod** diberikan alamat IP unik. IP ini dapat bersifat **internal** (hanya dapat diakses di dalam kluster) atau **eksternal** (dapat dirutekan dari luar kluster).

Dengan model satu IP per Pod, kita tidak perlu khawatir dengan konflik port pada satu IP yang sama. Namun, pendekatan ini memiliki tantangan tersendiri:

- **Pods bersifat sementara** → Pod dapat dihentikan dan dibuat ulang kapan saja oleh Kubernetes, sehingga IP yang diberikan harus dapat dialokasikan kembali dengan cepat.
- **Efisiensi pengelolaan pool IP** → Pool alamat IP dalam kluster harus dikelola dengan baik agar tidak kehabisan alamat.

### Cara Kerja IPAM di Kubernetes
Manajemen alamat IP dalam Kubernetes diimplementasikan berdasarkan **Container Networking Interface (CNI)** yang digunakan. Saat membuat kluster, kita dapat menentukan **CIDR** untuk jaringan Pod dengan perintah berikut:

```sh
kubeadm init --pod-network-cidr 10.30.0.0/16
```

Perintah ini akan mengatur opsi `--cluster-cidr` pada **kube-controller-manager**. Kubernetes kemudian akan membagi **subnet** untuk setiap Node dalam kluster, biasanya dengan ukuran **/24**.

Contoh konfigurasi Node dengan subnet `10.30.0.0/24`:

```yaml
apiVersion: v1
kind: Node
metadata:
  labels:
    kubernetes.io/hostname: test
  name: master-0
spec:
  podCIDR: 10.30.0.0/24
  podCIDRs:
    - 10.30.0.0/24
```

Setiap **CNI plug-in** memiliki pendekatan berbeda dalam menangani **IPAM**:

- **Calico** → Menghormati konfigurasi CIDR dari Kubernetes.
- **Cilium** → Dapat menggunakan alokasi IP terpisah atau mengikuti CIDR dari Kubernetes.

Pemilihan **CIDR** harus diperhatikan agar tidak berbenturan dengan jaringan Node atau host dalam kluster.

---

## 2. Protokol Routing
Setelah Pod memiliki alamat IP, langkah selanjutnya adalah memastikan bahwa lalu lintas antar Pod dapat dirutekan dengan benar.

### Protokol Routing di Kubernetes
Salah satu protokol yang sering digunakan dalam Kubernetes adalah **Border Gateway Protocol (BGP)**. Protokol ini memungkinkan distribusi rute dalam kluster dan bahkan dapat digunakan untuk memberitahu jaringan eksternal tentang rute ke alamat IP Pod.

Contoh implementasi BGP di Kubernetes:

- **Calico** → Menggunakan daemon BGP yang berjalan di setiap Node untuk menyebarkan informasi rute.
- **Kube-Router** → Memanfaatkan BGP untuk mengatur rute secara dinamis.

Namun, ada beberapa tantangan dalam menerapkan BGP, terutama di lingkungan cloud seperti **AWS**. Beberapa kendala yang mungkin terjadi:

1. **Source/Destination Check** → AWS EC2 secara default menolak paket dengan tujuan IP yang tidak sesuai.
2. **Paket yang keluar dari subnet AWS akan gagal dirutekan**.

Dalam skenario ini, kita perlu menggunakan solusi seperti **tunneling**.

---

## 3. Enkapsulasi dan Tunneling
Ketika jaringan dasar (**underlay network**) tidak mendukung rute langsung ke Pod, kita dapat menggunakan metode **enkapsulasi**.

### Apa itu Enkapsulasi?
Enkapsulasi berarti membungkus paket jaringan dalam paket lain, sehingga dari perspektif jaringan fisik, hanya melihat paket luar.

Beberapa protokol yang digunakan dalam enkapsulasi:

- **VXLAN** (Virtual eXtensible LAN)
- **Geneve** (Generic Network Virtualization Encapsulation)
- **GRE** (Generic Routing Encapsulation)

Dalam Kubernetes, **VXLAN** adalah salah satu metode yang paling banyak digunakan. Diagram berikut menunjukkan cara kerja VXLAN:

```
+------------------------------------------------+
| Paket Luar (Header VXLAN)                      |
|  - Source IP: Node-1                           |
|  - Destination IP: Node-2                      |
+------------------------------------------------+
        |
        | VXLAN Enkapsulasi
        v
+------------------------------------------------+
| Paket Dalam (Header Asli)                      |
|  - Source IP: Pod-1                            |
|  - Destination IP: Pod-2                       |
+------------------------------------------------+
```

### Kelebihan dan Kekurangan Enkapsulasi
**Kelebihan:**
- Dapat digunakan di berbagai lingkungan tanpa perlu mengubah jaringan fisik.
- Memungkinkan komunikasi antar Pod di berbagai Node tanpa perlu konfigurasi routing tambahan.

**Kekurangan:**
- Overhead tambahan akibat pembungkusan paket.
- Proses enkapsulasi dan dekapsulasi dapat membebani CPU.
- Trafik jaringan menjadi lebih sulit untuk dianalisis dan di-debug.

---

## Kesimpulan
Dalam Kubernetes, memahami jaringan Pod sangat penting untuk memastikan komunikasi antar-workload berjalan dengan lancar. Beberapa poin utama yang perlu diperhatikan:

1. **IP Address Management (IPAM)** menentukan bagaimana alamat IP dialokasikan ke Pod.
2. **Routing protocols** seperti BGP memungkinkan rute IP Pod diketahui oleh jaringan lain.
3. **Encapsulation & tunneling** seperti VXLAN dapat digunakan jika routing langsung tidak memungkinkan.

# **Keterjangkauan Workload (Workload Routability)**

## **Jaringan Pod dalam Cluster**

Pada kebanyakan cluster Kubernetes, jaringan Pod bersifat internal. Ini berarti Pod dapat berkomunikasi langsung satu sama lain, tetapi klien eksternal tidak dapat langsung mengakses IP Pod. Mengingat IP Pod bersifat sementara (ephemeral), mengandalkan komunikasi langsung dengan IP Pod bukanlah praktik yang baik. Sebagai gantinya, lebih baik menggunakan mekanisme service discovery atau load balancing yang mengabstraksi IP yang mendasarinya.

Salah satu keuntungan besar dari jaringan Pod internal adalah bahwa jaringan ini tidak menggunakan ruang alamat (IP address space) yang berharga dalam organisasi Anda. Banyak organisasi mengelola ruang alamat dengan ketat untuk memastikan alamat tetap unik dalam perusahaan. Oleh karena itu, meminta ruang alamat sebesar **/16** (65.536 IP) untuk setiap cluster Kubernetes bisa menjadi permintaan yang berlebihan.

## **Menghubungkan Lalu Lintas Eksternal ke Pod**

Karena IP Pod tidak dapat langsung diakses dari luar, ada beberapa pola yang digunakan untuk menghubungkan lalu lintas eksternal ke Pod:

1. **Menggunakan Ingress Controller**
   - Ingress Controller biasanya ditempatkan pada jaringan host di beberapa node khusus.
   - Setelah paket masuk ke Ingress Controller, ia dapat merutekannya langsung ke IP Pod karena terhubung dengan jaringan Pod.
   - Beberapa penyedia cloud bahkan menyediakan integrasi dengan load balancer eksternal yang menangani semua ini secara otomatis.

2. **Membuat Pod Dapat Diakses secara Langsung**
   - **Menggunakan Plugin Jaringan Khusus**: Misalnya, AWS VPC CNI yang menetapkan beberapa IP sekunder ke setiap node dan mendistribusikannya ke Pod, sehingga setiap Pod dapat dirutekan seperti host EC2.
   - **Menggunakan Protokol Routing seperti BGP**: Beberapa plugin jaringan dapat menggunakan BGP untuk menyebarkan rute ke Pod, memungkinkan sebagian jaringan Pod dapat diakses dari luar tanpa mengekspos seluruh ruang IP.

📌 **Catatan Penting:** Hindari membuat jaringan Pod dapat diakses dari luar kecuali benar-benar diperlukan. Biasanya, ini hanya diperlukan untuk aplikasi lawas yang membutuhkan koneksi langsung dari klien ke backend tertentu. Sebaiknya, aplikasikan service discovery dan load balancing agar sesuai dengan paradigma jaringan Kubernetes.

---

# **IPv4 dan IPv6 dalam Kubernetes**

Sebagian besar cluster Kubernetes saat ini menggunakan **IPv4** secara eksklusif. Namun, ada permintaan yang meningkat untuk mendukung **IPv6**, terutama di industri telekomunikasi.

- Kubernetes mendukung **dual-stack** sejak versi **1.16** (masih dalam tahap alpha saat itu).
- Dengan dual-stack, Anda dapat mengkonfigurasi cluster untuk mendukung alamat **IPv4 dan IPv6** secara bersamaan.

### **Syarat untuk Menggunakan IPv6**
1. **Mengaktifkan fitur dual-stack** di kube-apiserver dan kubelet.
2. **Mengkonfigurasi kube-apiserver, kube-controller-manager, dan kube-proxy** agar mendukung ruang alamat IPv4 dan IPv6.
3. **Menggunakan plugin CNI yang mendukung IPv6**, seperti **Calico atau Cilium**.

📌 **Contoh Alokasi CIDR Dual-Stack di Node:**
```yaml
spec:
  podCIDR: 10.30.0.0/24
  podCIDRs:
    - 10.30.0.0/24
    - 2002:1:1::/96
```

Dalam skenario ini, plugin CNI bertanggung jawab untuk menentukan apakah sebuah Pod mendapatkan alamat IPv4, IPv6, atau keduanya.

---

# **Enkripsi Lalu Lintas Workload**

Secara default, lalu lintas antara Pod **tidak dienkripsi**. Ini berarti paket yang dikirim tanpa enkripsi (misalnya dalam format teks biasa) dapat disadap.

Beberapa plugin jaringan mendukung **enkripsi lalu lintas**, seperti:
- **Antrea** dengan **IPsec** melalui tunnel **GRE**.
- **Calico** dengan **WireGuard** untuk mengenkripsi lalu lintas antar node.

📌 **Poin yang Perlu Dipertimbangkan:**
- **Apakah lalu lintas antar host di pusat data sudah dienkripsi?**
- **Apakah layanan sudah menggunakan TLS?** Jika iya, apakah perlu enkripsi tambahan di tingkat jaringan?
- **Apakah Anda berencana menggunakan service mesh (mTLS)?** Jika ya, apakah enkripsi tambahan di CNI masih diperlukan?

Enkripsi memang menambah keamanan, tetapi juga dapat memperlambat kinerja karena overhead tambahan dalam proses enkripsi dan dekripsi paket.

---

# **Kebijakan Jaringan (Network Policy)**

Network Policy dalam Kubernetes mirip dengan firewall atau security group yang memungkinkan kita menentukan lalu lintas **ingress** (masuk) dan **egress** (keluar) yang diperbolehkan.

### **Fitur Penting NetworkPolicy**
- **Dideklarasikan di dalam Kubernetes** dan bergantung pada plugin CNI yang mendukungnya (misalnya, Calico atau Cilium).
- **Berlaku dalam satu Namespace** secara default.
- **Secara bawaan, Kubernetes mengizinkan semua lalu lintas**, tetapi setelah NetworkPolicy diterapkan, lalu lintas yang tidak diizinkan akan diblokir.

📌 **Contoh NetworkPolicy untuk Namespace `org-1`:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: team-netpol
  namespace: org-1
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
      - ipBlock:
          cidr: 10.40.0.0/24
      ports:
        - protocol: TCP
          port: 80
  egress:
    - to:
      - namespaceSelector:
          matchLabels:
            name: org-2
      - podSelector:
          matchLabels:
            app: team-b
      ports:
        - protocol: TCP
          port: 80
```
Penjelasan:
- Mengizinkan trafik masuk (ingress) dari subnet `10.40.0.0/24` pada port 80.
- Mengizinkan trafik keluar (egress) ke Namespace `org-2` dengan aplikasi `team-b` hanya di port 80.

Beberapa plugin jaringan menawarkan API tambahan untuk **kebijakan yang lebih kompleks**, seperti **resolusi IP berdasarkan DNS, aturan Layer 7, atau kebijakan global** di seluruh cluster. Namun, menggunakan API vendor ini membuat aturan jaringan tidak lagi portabel ke plugin lain.

---
# Container Network Interface (CNI) di Kubernetes

## 📌 Pengantar
CNI (Container Network Interface) adalah standar untuk mengelola jaringan dalam Kubernetes. CNI memungkinkan setiap Pod untuk terhubung ke jaringan dan berkomunikasi dengan Pod lain di dalam cluster.

---

## 📦 Instalasi CNI di Kubernetes
CNI biasanya diinstal sebagai **DaemonSet**, yang secara otomatis mendistribusikan konfigurasi jaringan ke setiap node dalam cluster.

📍 **Lokasi utama konfigurasi CNI:**
- **Binary plugin CNI:** `/opt/cni/bin/`
- **File konfigurasi CNI:** `/etc/cni/net.d/`

Jika CNI tidak dikonfigurasi dengan benar, Pod seperti **CoreDNS** akan mengalami status `Pending` karena tidak dapat terhubung ke jaringan:
```bash
kubectl get pods -n kube-system
```
```
kube-system   coredns-xxxxx-xxxxx   0/1     Pending   0     3m14s
```

Untuk mengecek error pada `kubelet`, gunakan perintah:
```bash
journalctl -f -u kubelet
```
```
Container runtime network not ready: NetworkReady=false
reason:NetworkPluginNotReady message: docker: network plugin is not ready: cni config uninitialized
```

---

## 🚀 Instalasi Cilium CNI
Cilium adalah CNI modern berbasis **eBPF** yang meningkatkan performa jaringan tanpa bergantung pada iptables.

### **1. Pasang Plugin Cilium CNI di Setiap Node**
```bash
cp /cni/loopback /opt/cni/bin/ || true
cp /opt/cni/bin/cilium-cni /opt/cni/bin/
```

### **2. Konfigurasi CNI**
```bash
cat > /etc/cni/net.d/05-cilium.conf <<EOF
{
  "cniVersion": "0.3.1",
  "name": "cilium",
  "type": "cilium-cni",
  "enable-debug": true
}
EOF
```

---

## 🔍 Perbandingan CNI: Calico vs Cilium

| CNI     | Keunggulan | Kekurangan |
|---------|------------|------------|
| **Calico** | - Mendukung BGP dan native routing.<br>- Mendukung NetworkPolicy Kubernetes.<br>- Fleksibel dalam IPAM dengan IPPool. | - Membutuhkan infrastruktur dengan BGP.<br>- Bergantung pada iptables (kecuali menggunakan eBPF). |
| **Cilium** | - Berbasis eBPF, lebih cepat dan scalable.<br>- Mendukung NetworkPolicy Kubernetes + fitur keamanan tambahan.<br>- Routing lebih efisien dengan eBPF. | - Memerlukan kernel Linux modern dengan eBPF.<br>- Tidak semua fitur jaringan didukung oleh eBPF. |

---

## 🏆 Kesimpulan
✅ **Gunakan Calico** jika cluster Kubernetes Anda menggunakan banyak subnet dan ingin fleksibilitas lebih dalam routing.<br>
✅ **Gunakan Cilium** jika Anda ingin performa tinggi dengan eBPF dan optimasi jaringan yang lebih efisien.<br>
✅ **Gunakan Flannel** jika Anda ingin solusi sederhana tanpa banyak konfigurasi tambahan.





