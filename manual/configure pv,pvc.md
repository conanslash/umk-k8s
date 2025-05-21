
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
  name: php-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
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
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
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

### Langkah 5: Gunakan PVC dalam Deployment

Contoh penggunaan PVC dalam pod `php-app.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-app
  namespace: pengambilan
spec:
  replicas: 2
  selector:
    matchLabels:
      app: php-app
  template:
    metadata:
      labels:
        app: php-app
    spec:
      initContainers:
        - name: fix-permissions
          image: busybox
          command: ['sh', '-c', 'chmod 777 /uploads']
          volumeMounts:
            - name: upload-volume
              mountPath: /uploads
      containers:
        - name: php-container
          image: conanslash/php-pod-info:upload-0.2 # ← Replace with your Docker Hub username
          ports:
            - containerPort: 80
          env:
            - name: APP_ENV
              valueFrom:
                configMapKeyRef:
                  name: php-config
                  key: APP_ENV
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: php-secret
                  key: DB_PASSWORD
          volumeMounts:
            - name: upload-volume
              mountPath: /uploads
      volumes:
        - name: upload-volume
          persistentVolumeClaim:
            claimName: php-pvc
```

* `claimName` merujuk kepada nama PVC yang telah dibuat.

---

### **Nota tambahan:**

* Untuk storan sebenar di cloud (contoh AWS EBS, GCP PD), `storageClassName` akan berbeza.
* `hostPath` hanya sesuai untuk environment tempatan atau demo sahaja.

---