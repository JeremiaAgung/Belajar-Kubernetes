# Container Runtime di Kubernetes

## Pendahuluan
Kubernetes adalah sistem orkestrasi kontainer yang tidak secara langsung membuat, memulai, atau menghentikan kontainer. Sebaliknya, Kubernetes mendelegasikan tugas tersebut ke **container runtime**, yaitu perangkat lunak yang bertanggung jawab untuk membuat dan mengelola kontainer di node klaster.

## Mengapa Kubernetes Tidak Memiliki Container Runtime Bawaan?
Kubernetes memilih menyediakan **antarmuka yang dapat dicolokkan** daripada mengimplementasikan container runtime sendiri karena:
- **Fleksibilitas**: Memungkinkan organisasi memilih runtime yang sesuai dengan kebutuhan mereka.
- **Inovasi**: Memfasilitasi pengembangan teknologi baru dalam ekosistem kontainer.
- **Standarisasi**: Mengadopsi standar industri seperti Open Container Initiative (OCI).

## Sejarah Container & Standarisasi OCI
Container telah mengubah pengembangan perangkat lunak dengan:
- **Isolasi yang lebih baik** dibandingkan VM.
- **Deployment yang lebih cepat dan efisien**.
- **Portabilitas tinggi** antar lingkungan.

Standarisasi melalui **Open Container Initiative (OCI)** menghasilkan spesifikasi untuk:
- **Runtime kontainer**
- **Format image kontainer**
- **Distribusi image**

## Container Runtime Interface (CRI)
Kubernetes menggunakan **Container Runtime Interface (CRI)** sebagai jembatan antara kubelet dan container runtime. CRI menentukan antarmuka yang harus diimplementasikan oleh container runtime agar kompatibel dengan Kubernetes.

## Memilih Container Runtime
Beberapa runtime yang kompatibel dengan Kubernetes:
- **containerd**: Runtime default Kubernetes, berbasis Docker.
- **CRI-O**: Dibuat khusus untuk Kubernetes dengan dukungan OCI.
- **gVisor**: Isolasi tambahan berbasis sandboxing.
- **Kata Containers**: Menyediakan keamanan berbasis VM dalam kontainer.

Kubernetes bergantung pada container runtime untuk menjalankan kontainer, memilih pendekatan modular dengan CRI agar tetap fleksibel dan inovatif. Pemilihan runtime bergantung pada kebutuhan spesifik platform yang digunakan.

# Sejarah Container dan Open Container Initiative (OCI)

## Awal Mula Container
Container berbasis Linux menggunakan dua fitur utama kernel:
- **Control Groups (cgroups)**: Mengontrol sumber daya CPU, memori, dll.
- **Namespaces**: Mengisolasi proses, filesystem, dan jaringan.

Walaupun fitur ini sudah ada sejak 2008, adopsi container baru meluas setelah munculnya **Docker**. Docker menyederhanakan penggunaan container dengan antarmuka CLI yang mudah, memungkinkan developer menjalankan aplikasi dalam lingkungan yang terisolasi dengan perintah sederhana seperti `docker run`.

## Manfaat Container
Container membawa perubahan besar dalam siklus pengembangan perangkat lunak:
- **Menyediakan lingkungan yang konsisten**: Menghindari masalah dependency antar sistem.
- **Mempermudah pengujian**: Lingkungan yang dapat direproduksi.
- **Meningkatkan efisiensi deployment**: Aplikasi dapat berjalan di mana saja selama ada Docker Engine.

Namun, popularitas container juga memicu tantangan baru: bagaimana memastikan kompatibilitas antara berbagai alat dan runtime container?

## Open Container Initiative (OCI)
Untuk mengatasi tantangan interoperabilitas, industri membentuk **Open Container Initiative (OCI)** pada tahun 2015, dengan tujuan standarisasi teknologi container. OCI mencakup tiga spesifikasi utama:
1. **OCI Runtime Specification**: Standar bagaimana container dijalankan.
2. **OCI Image Specification**: Standar format image container.
3. **OCI Distribution Specification**: Standar untuk distribusi image container.

OCI memungkinkan pengguna untuk berpindah antar produk dan solusi container dengan lebih mudah.

## OCI Runtime Specification
Spesifikasi runtime OCI mendefinisikan bagaimana container dibuat dan dijalankan dalam lingkungan yang kompatibel dengan OCI. Beberapa elemen utama dalam konfigurasi container meliputi:
- Filesystem root container.
- Perintah yang akan dijalankan dalam container.
- Variabel lingkungan.
- Batasan sumber daya.

Contoh konfigurasi container dalam OCI runtime:
```json
{
  "ociVersion": "1.0.1",
  "process": {
    "terminal": true,
    "user": {
      "uid": 1,
      "gid": 1
    },
    "args": [ "sh" ],
    "env": [ "PATH=/usr/bin" ]
  },
  "mounts": [
    { "destination": "/proc", "type": "proc", "source": "proc" }
  ]
}
```

### Siklus Hidup Container dalam OCI
1. **Creating**: Container sedang dibuat.
2. **Created**: Container telah selesai dibuat namun belum berjalan.
3. **Running**: Proses dalam container sedang berjalan.
4. **Stopped**: Proses dalam container telah selesai.

## Implementasi OCI dalam Ekosistem Container
**runc** adalah runtime container tingkat rendah yang mengimplementasikan OCI runtime spec. Beberapa runtime container lain seperti **Docker, containerd, dan CRI-O** menggunakan runc untuk menjalankan container sesuai standar OCI.

---

## OCI Image Specification  
OCI Image Specification mengatur bagaimana image container dibuat dan dikelola. Komponen utama dalam spesifikasi ini meliputi:  
- **Manifest**: Menyimpan metadata image, daftar layer, dan konfigurasi.  
- **Image Index**: Memungkinkan dukungan untuk berbagai arsitektur dalam satu image.  
- **Filesystem Layers**: Berisi perubahan pada sistem file dalam format TAR.  

Contoh manifest OCI:  
```json
{
  "schemaVersion": 2,
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "size": 7023,
    "digest": "sha256:b5b2b2c..."
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image.layer.v1.tar+gzip",
      "size": 32654,
      "digest": "sha256:9834876d..."
    }
  ]
}
```
Dengan spesifikasi OCI ini, image container dapat digunakan di berbagai runtime tanpa masalah kompatibilitas.

## Container Runtime Interface (CRI)  
CRI adalah antarmuka standar yang memungkinkan Kubernetes berinteraksi dengan berbagai runtime container tanpa ketergantungan langsung. CRI menggunakan gRPC dan memiliki dua layanan utama:  
1. **RuntimeService**: Mengelola lifecycle Pod dan container.  
2. **ImageService**: Mengelola image container, termasuk pull, list, dan delete.  

### Proses Menjalankan Pod dalam Kubernetes  
1. **Membuat sandbox** menggunakan `RunPodSandbox`.  
2. **Memeriksa ketersediaan image** dengan `ImageStatus`, lalu menarik image jika belum tersedia dengan `PullImage`.  
3. **Membuat container** dalam sandbox dengan `CreateContainer`.  
4. **Menjalankan container** menggunakan `StartContainer`.  

Dengan adanya CRI, Kubernetes dapat mendukung berbagai runtime seperti **containerd**, **CRI-O**, dan lainnya tanpa perubahan besar pada kode kubelet.

OCI dan CRI adalah fondasi utama dalam ekosistem container modern:  
- **OCI memastikan interoperabilitas image dan runtime container.**  
- **CRI memungkinkan Kubernetes mendukung berbagai runtime tanpa modifikasi besar.**  
Dengan standar ini, industri dapat mengembangkan solusi container yang lebih fleksibel dan efisien.

# Pemilihan Runtime Container dalam Kubernetes

## Pengantar
Dalam ekosistem Kubernetes, pemilihan runtime container menjadi fleksibel berkat Container Runtime Interface (CRI). Namun, dalam kebanyakan kasus, runtime container telah dipilih oleh distribusi Kubernetes yang digunakan, seperti dalam layanan Kubernetes terkelola atau proyek komunitas seperti Cluster API.


---

## 1. **Docker**
Docker adalah runtime yang populer di dunia container, namun Kubernetes tidak secara langsung menggunakan Docker karena fitur tambahan yang tidak diperlukan, seperti pembuatan image dan pengelolaan jaringan container.

- Kubernetes menggunakan **dockershim** sebagai perantara untuk kompatibilitas.
- **Docker CLI** dapat digunakan untuk inspeksi container dalam Kubernetes.
- Menggunakan **containerd** di balik layar untuk menjalankan container.

**Contoh Perintah**:
```sh
$ docker ps --format='{{.ID}}\t{{.Names}}' | grep nginx_default
```

---

## 2. **containerd**
containerd adalah runtime container yang lebih ringan dibandingkan Docker dan telah menjadi pilihan utama dalam berbagai distribusi Kubernetes seperti AKS, EKS, dan GKE.

- Mendukung **CRI** secara native dengan containerd CRI plug-in.
- Digunakan langsung oleh kubelet tanpa perantara seperti dockershim.
- Proses container berjalan di namespace `k8s.io`.

**Contoh Perintah**:
```sh
$ ctr --namespace k8s.io containers ls | grep nginx
```

---

## 3. **CRI-O**
CRI-O adalah runtime container yang dirancang khusus untuk Kubernetes dan digunakan oleh RedHat OpenShift.

- Implementasi CRI tanpa fitur tambahan di luar kebutuhan Kubernetes.
- Menggunakan **conmon** sebagai proses pemantau container.
- Tidak memiliki CLI sendiri, tetapi dapat digunakan dengan **crictl**.

**Contoh Perintah**:
```sh
$ crictl --runtime-endpoint unix:///var/run/crio/crio.sock ps --name nginx
```

---

## 4. **Kata Containers**
Kata Containers adalah runtime khusus yang menjalankan container dalam lightweight VM untuk isolasi yang lebih kuat.

- Menggunakan hypervisor seperti **QEMU, NEMU, atau AWS Firecracker**.
- Cocok untuk workload yang membutuhkan **isolasi tingkat VM**.
- Membutuhkan containerd sebagai runtime perantara.

**Contoh RuntimeClass untuk Kubernetes**:
```yaml
apiVersion: node.k8s.io/v1beta1
kind: RuntimeClass
metadata:
  name: kata-containers
handler: kata
```

**Menjalankan Pod dengan Kata Containers**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kata-example
spec:
  runtimeClassName: kata-containers
  containers:
    - name: nginx
      image: nginx
```

---

## 5. **Virtual Kubelet**
Virtual Kubelet bukan runtime container, tetapi bertindak sebagai kubelet virtual yang memungkinkan Kubernetes menjalankan workload di lingkungan alternatif seperti **Azure Container Instances** atau **AWS Fargate**.

---

## Kesimpulan
Pemilihan runtime container dalam Kubernetes tergantung pada:
1. **Dukungan dan pengalaman** dengan runtime tertentu.
2. **Kompatibilitas dan pengujian** dengan Kubernetes.
3. **Kebutuhan isolasi**, misalnya menggunakan VM berbasis runtime seperti Kata Containers.

## Ringkasan

**Runtime container** adalah komponen dasar dalam platform berbasis Kubernetes. Tanpa runtime container, tidak mungkin menjalankan workload dalam bentuk container.

Dalam bab ini, kita telah mempelajari bahwa Kubernetes menggunakan **Container Runtime Interface (CRI)** untuk berinteraksi dengan runtime container. Salah satu keuntungan utama dari CRI adalah sifatnya yang **modular**, sehingga memungkinkan penggunaan runtime container yang paling sesuai dengan kebutuhan.

### Contoh Runtime Container
Beberapa runtime container yang sering digunakan dalam praktik:
- **Docker**
- **containerd**
