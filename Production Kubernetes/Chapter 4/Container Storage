# Penyimpanan di Kubernetes

## Pendahuluan
Kubernetes awalnya dirancang untuk menangani workload stateless, tetapi kini semakin banyak workload stateful seperti database dan message queue yang dijalankan di Kubernetes. Untuk mendukung workload ini, Kubernetes menyediakan berbagai opsi penyimpanan yang lebih tahan terhadap kegagalan aplikasi atau pemindahan workload ke node lain.

## Pertimbangan Penyimpanan
Sebelum memilih metode penyimpanan di Kubernetes, penting untuk memahami beberapa kebutuhan utama:
- **Access Modes** (Mode Akses)
- **Volume Expansion** (Perluasan Volume)
- **Dynamic Provisioning** (Penyediaan Dinamis)
- **Backup dan Recovery**
- **Block, File, dan Object Storage**
- **Data Sementara (Ephemeral Data)**
- **Memilih Penyedia Penyimpanan**

## Mode Akses Penyimpanan
1. **ReadWriteOnce (RWO)**: Hanya satu pod yang bisa membaca dan menulis ke volume.
2. **ReadOnlyMany (ROX)**: Banyak pod bisa membaca volume.
3. **ReadWriteMany (RWX)**: Banyak pod bisa membaca dan menulis volume.

Mode **RWO** adalah yang paling umum di Kubernetes, karena banyak penyedia cloud seperti AWS EBS dan Azure Disk hanya mendukung mode ini.

## Perluasan Volume
Seiring waktu, aplikasi dapat membutuhkan lebih banyak ruang penyimpanan. Kubernetes mendukung perluasan volume dengan langkah-langkah berikut:
1. Mengajukan permintaan penyimpanan tambahan melalui **PersistentVolumeClaim (PVC)**.
2. Memperbesar volume melalui penyedia penyimpanan.
3. Memperbesar sistem file agar dapat memanfaatkan ruang tambahan.

Fitur ini bergantung pada dukungan penyedia penyimpanan dan kompatibilitasnya dengan Kubernetes.

## Provisioning Volume
Terdapat dua metode provisioning:
- **Static Provisioning**: Volume dibuat secara manual sebelum digunakan oleh Kubernetes.
- **Dynamic Provisioning**: Volume dibuat otomatis oleh Kubernetes menggunakan **Container Storage Interface (CSI)**.

Dynamic provisioning lebih disukai jika tersedia driver yang sesuai dengan penyedia penyimpanan.

## Backup dan Pemulihan
Backup merupakan aspek penting dari penyimpanan. Kubernetes memiliki berbagai strategi backup, salah satu solusi yang populer adalah **Project Velero**. Velero dapat melakukan:
- Backup objek Kubernetes untuk migrasi atau pemulihan.
- Penjadwalan snapshot volume.
- Hooks untuk menjalankan perintah sebelum atau sesudah proses backup.

## Jenis Penyimpanan
Terdapat tiga jenis penyimpanan utama yang digunakan dalam Kubernetes:
1. **Block Storage**: Penyimpanan berbasis blok tanpa sistem file, cocok untuk performa tinggi.
2. **File Storage**: Penyimpanan berbasis sistem file seperti NFS, cocok untuk workload berbagi data.
3. **Object Storage**: Penyimpanan berbasis objek seperti Amazon S3, cocok untuk data tidak terstruktur.

Pemilihan jenis penyimpanan bergantung pada kebutuhan aplikasi dan performa yang diinginkan.

---
# Penyimpanan Ephemeral

Secara default, penyimpanan dalam container bersifat sementara dan akan hilang jika container restart. Namun, Kubernetes menyediakan `emptyDir` untuk penyimpanan sementara yang bertahan saat container restart dan dapat digunakan untuk berbagi data antar-container dalam satu Pod. Untuk mencegah konsumsi penyimpanan yang berlebihan, Kubernetes mendukung pembatasan total penyimpanan ephemeral dalam Namespace.

## Memilih Penyedia Penyimpanan

Banyak pilihan penyedia penyimpanan, mulai dari solusi yang dikelola sendiri seperti Ceph hingga layanan cloud seperti Google Persistent Disk dan Amazon EBS. Penting untuk memahami kemampuan penyedia penyimpanan dan bagaimana integrasinya dengan Kubernetes agar sesuai dengan kebutuhan aplikasi.

## Primitif Penyimpanan Kubernetes

Kubernetes menyediakan beberapa mekanisme penyimpanan:
- **PersistentVolume (PV):** Merepresentasikan penyimpanan yang dapat digunakan oleh Pods.
- **PersistentVolumeClaim (PVC):** Permintaan untuk menggunakan PV tertentu.
- **StorageClass:** Menentukan cara pembuatan dan pengelolaan PV secara dinamis.

### Persistent Volume dan Claim

Contoh definisi PV dengan penyimpanan lokal:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0
spec:
  capacity:
    storage: 30Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /mnt/fast-disk/pod-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - test-w
```

PVC yang mengklaim PV di atas:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc0
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
```

Menggunakan PVC dalam sebuah Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: fast-disk
      persistentVolumeClaim:
        claimName: pvc0
  containers:
    - name: ml-processor
      image: ml-processor-image
      volumeMounts:
        - mountPath: "/var/lib/db"
          name: fast-disk
```

Untuk otomatisasi, dapat digunakan **Local Persistence Volume Static Provisioner**.

> **Peringatan:** Jangan gunakan `hostPath` karena berisiko keamanan tinggi.

## StorageClass untuk Provisioning Dinamis

StorageClass memungkinkan pembuatan PV secara otomatis berdasarkan kebutuhan PVC. Contoh penggunaan AWS EBS:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
parameters:
  type: io2
  iopsPerGB: "17"
  fsType: ext4
```

Kubernetes mendukung berbagai penyedia seperti AWS EBS, Ceph, dan GCE PD. Saat ini, Kubernetes beralih ke **Container Storage Interface (CSI)** untuk integrasi penyimpanan yang lebih fleksibel.

# Container Storage Interface (CSI)

## Apa Itu CSI?
Container Storage Interface (CSI) adalah standar yang memungkinkan penyedia penyimpanan (storage providers) menyediakan block dan file storage ke workload Kubernetes. Implementasi CSI disebut "drivers" yang bertanggung jawab untuk menghubungkan Kubernetes dengan penyimpanan seperti Google Persistent Disks atau Ceph.

## Komponen CSI
CSI terdiri dari dua komponen utama:
1. **Controller Plug-in** - Mengelola pembuatan dan penghapusan volume di penyedia penyimpanan.
2. **Node Plug-in** - Mempersiapkan volume agar dapat digunakan oleh Pods dalam node.

### CSI Controller
CSI Controller bertanggung jawab atas manajemen volume persisten. Kubernetes tidak langsung berkomunikasi dengan CSI Controller, melainkan menggunakan eksternal controller berikut:
- **external-provisioner**: Membuat PersistentVolume dari PersistentVolumeClaim.
- **external-attacher**: Mengelola VolumeAttachment untuk menghubungkan atau melepas volume dari node.
- **external-resizer**: Mengelola perubahan ukuran volume.
- **external-snapshotter**: Mengelola pembuatan snapshot volume.

### CSI Node
Node plug-in menjalankan driver dalam mode "node mode" untuk menangani mount volume ke Pods. Sidecar yang umum digunakan dalam Pod CSI Node:
- **node-driver-registrar**: Mendaftarkan driver ke kubelet.
- **liveness-probe**: Memantau kesehatan driver.

## Implementasi Storage as a Service
Untuk menyediakan penyimpanan secara deklaratif dan dinamis, kita bisa menggunakan layanan cloud seperti AWS EBS. Berikut adalah langkah-langkah instalasi driver:

### Instalasi
1. **Konfigurasi Akses ke Penyedia Penyimpanan**
   - Opsi:
     - **Instance profile** (Hak akses universal, tidak memerlukan kredensial di Kubernetes).
     - **Identity service** (Misalnya, proyek `kiam` untuk kontrol akses yang lebih aman).
     - **Kredensial dalam Secret Kubernetes**:
       ```yaml
       apiVersion: v1
       kind: Secret
       metadata:
         name: aws-secret
         namespace: kube-system
       stringData:
         key_id: "AKIAEXAMPLE"
         access_key: "examplepassword"
       ```

2. **Deploy Komponen CSI**
   - **Controller**: Dipasang sebagai Deployment dengan leader-election untuk failover.
   - **Node Plug-in**: Dipasang sebagai DaemonSet pada setiap node.
   - **Verifikasi dengan**:
     ```sh
     kubectl get csinode
     ```
     Contoh output:
     ```sh
     NAME                                    DRIVERS AGE
     ip-10-0-0-205.us-west-2.compute.internal 1 97m
     ```

3. **Registrasi Driver dalam Kubernetes**
   - **CSINode Object**
     ```yaml
     apiVersion: storage.k8s.io/v1
     kind: CSINode
     metadata:
       name: ip-10-0-0-205.us-west-2.compute.internal
     spec:
       drivers:
       - name: ebs.csi.aws.com
         nodeID: i-0284ac0df4da1d584
     ```
   - **CSIDriver Object**
     ```yaml
     apiVersion: storage.k8s.io/v1
     kind: CSIDriver
     metadata:
       name: aws-ebs-csi-driver
     spec:
       attachRequired: true
       podInfoOnMount: false
       volumeLifecycleModes:
       - Persistent
     ```

## Kesimpulan
CSI memungkinkan Kubernetes menggunakan berbagai penyedia penyimpanan dengan cara yang fleksibel dan standar. Dengan CSI, storage provisioning bisa dilakukan secara dinamis sesuai kebutuhan workload tanpa intervensi administrator. Implementasi seperti AWS EBS menunjukkan bagaimana semua komponen CSI bekerja bersama untuk menyediakan Storage as a Service.

# Menyediakan Opsi Penyimpanan di Kubernetes

## 1. Membuat StorageClass
Untuk memberikan opsi penyimpanan kepada developer, kita harus membuat **StorageClass**. Dalam skenario ini, kita menyediakan dua jenis penyimpanan:
- **HDD (Hard Disk Drive) – default**: Lebih murah dan cocok untuk penyimpanan data yang tidak memerlukan kecepatan tinggi.
- **SSD (Solid State Drive) – performa tinggi**: Cocok untuk aplikasi yang membutuhkan kecepatan baca/tulis yang tinggi.

### Contoh YAML untuk StorageClass:
```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: default-block
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: st1
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: performance-block
provisioner: ebs.csi.aws.com
parameters:
  type: io1
  iopsPerGB: "20"
```
- `default-block` adalah StorageClass default yang menggunakan HDD (`type: st1`).
- `performance-block` menggunakan SSD (`type: io1`) dengan performa lebih tinggi.

## 2. Menggunakan Penyimpanan dengan PersistentVolumeClaim (PVC)
Developer dapat menggunakan StorageClass ini dengan membuat **PersistentVolumeClaim (PVC)**.

### Contoh PVC:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc0
spec:
  resources:
    requests:
      storage: 11Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  resources:
    requests:
      storage: 14Gi
  storageClassName: performance-block
```
- `pvc0` akan menggunakan `default-block` (HDD).
- `pvc1` akan menggunakan `performance-block` (SSD).

## 3. Cara Kubernetes Mengalokasikan Storage
- `pvc1` (SSD) akan langsung dibuat sebagai volume yang siap digunakan.
- `pvc0` (HDD) hanya akan dibuat jika ada **Pod** yang menggunakannya. Ini menghindari biaya untuk storage yang tidak terpakai.

Jika developer membuat dua **Pod** yang menggunakan **pvc0** dan **pvc1**, Kubernetes akan:
1. Membuat volume di AWS (EBS).
2. Menyambungkan volume ke **Node** tempat Pod dijalankan.
3. Memformat volume dan memasangnya di dalam **container**.
4. Volume tetap ada meskipun Pod berpindah Node.

## 4. Memperbesar Ukuran Penyimpanan (Resizing)
- Storage pada AWS EBS bisa diperbesar tanpa perlu menghentikan Pod.
- Setelah diperbesar, filesystem di dalamnya juga perlu diperluas agar ruang tambahan bisa digunakan.
- **Catatan:** Kubernetes **tidak** mendukung **pengecilan** ukuran storage.

## 5. Snapshot untuk Backup dan Restore
Snapshot bisa digunakan untuk backup data secara berkala.

### **Langkah-langkahnya:**
#### 1. Membuat VolumeSnapshotClass:
```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshotClass
metadata:
  name: default-snapshots
driver: ebs.csi.aws.com
deletionPolicy: Delete
```
#### 2. Membuat Snapshot dari PVC:
```yaml
apiVersion: snapshot.storage.k8s.io/v1beta1
kind: VolumeSnapshot
metadata:
  name: snap1
spec:
  volumeSnapshotClassName: default-snapshots
  source:
    persistentVolumeClaimName: pvc0
```
#### 3. Mengembalikan Data dari Snapshot:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-reclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: default-block
  resources:
    requests:
      storage: 600Gi
  dataSource:
    name: snap1
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```
- Jika PVC asli hilang atau rusak, kita bisa membuat PVC baru dari **snapshot** terakhir.

## Kesimpulan
- **StorageClass** memungkinkan kita menyediakan berbagai opsi penyimpanan sesuai kebutuhan.
- **PVC** digunakan untuk meminta penyimpanan dan akan dibuat sesuai dengan StorageClass yang dipilih.
- **Snapshot** memungkinkan backup dan pemulihan data dengan cepat.
- Developer bisa mengelola storage tanpa perlu memahami infrastruktur penyimpanan secara mendalam.




