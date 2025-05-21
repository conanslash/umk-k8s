
---

### Langkah 1: Fahami konsep Persistent Volume (PV) dan Persistent Volume Claim (PVC)

* **Persistent Volume (PV)** ialah sumber storan yang disediakan oleh pentadbir kluster.
* **Persistent Volume Claim (PVC)** ialah permintaan storan oleh pengguna, seolah-olah seperti “booking” storan daripada PV.

---

### Langkah 2: Sediakan Persistent Volume (PV)

Contoh fail `pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: contoh-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

* `capacity.storage`: kapasiti storan, contoh 5Gi.
* `accessModes`: cara akses, contoh `ReadWriteOnce` bermaksud satu node boleh baca/tulis.
* `hostPath.path`: lokasi storan pada node (untuk demo atau local dev).

**Jalankan perintah:**

```bash
kubectl apply -f pv.yaml
```

---

### Langkah 3: Sediakan Persistent Volume Claim (PVC)

Contoh fail `pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: php-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: manual
```

* `requests.storage`: kapasiti storan yang diminta (5Gi).
* `storageClassName` mesti sama dengan PV yang sedia ada.

**Jalankan perintah:**

```bash
kubectl apply -f pvc.yaml
```

---

### Langkah 4: Semak PVC dan PV

Semak status PV:

```bash
kubectl get pv
```

Semak status PVC:

```bash
kubectl get pvc
```

Pastikan PVC sudah dalam status `Bound`, bermaksud PVC sudah berjaya mendapatkan PV.

---

### Langkah 5: Gunakan PVC dalam Pod

Contoh penggunaan PVC dalam pod `pod-pvc.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: contoh-pod
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: "/usr/share/nginx/html"
      name: contoh-volume
  volumes:
  - name: contoh-volume
    persistentVolumeClaim:
      claimName: contoh-pvc
```

* `claimName` merujuk kepada nama PVC yang telah dibuat.

**Jalankan perintah:**

```bash
kubectl apply -f pod-pvc.yaml
```

---

### Langkah 6: Semak Pod

Pastikan pod berjalan dengan:

```bash
kubectl get pods
```

---

### Langkah 7: Masuk ke dalam pod dan periksa storan

Masuk ke pod:

```bash
kubectl exec -it contoh-pod -- /bin/bash
```

Periksa folder `/usr/share/nginx/html` yang sudah dipasang volume dari PVC.

---

### **Nota tambahan:**

* Untuk storan sebenar di cloud (contoh AWS EBS, GCP PD), `storageClassName` akan berbeza.
* `hostPath` hanya sesuai untuk environment tempatan atau demo sahaja.

---

Kalau mahu, saya boleh sediakan contoh YAML lengkap atau terangkan langkah penggunaan `StorageClass` pula! Nak cuba?
