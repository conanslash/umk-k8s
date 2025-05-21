Baik! Saya akan tunjukkan contoh lengkap menggunakan **NFS sebagai Persistent Volume (PV)** dan **Persistent Volume Claim (PVC)** yang digunakan dalam **Deployment** di Kubernetes.

---

## Contoh lengkap: NFS + PVC + Deployment

---

### 1. Persistent Volume (PV) guna NFS

Fail: `pv-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: ""
  nfs:
    path: /kebetest
    server: cohecbjvip.umk.edu.my   # Ganti dengan IP server NFS anda
```

---

### 2. Persistent Volume Claim (PVC)

Fail: `pvc-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs
```

---

### 3. Deployment guna PVC

Fail: `deployment-nfs.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-nfs-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-nfs
  template:
    metadata:
      labels:
        app: nginx-nfs
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: nfs-storage
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nfs-storage
        persistentVolumeClaim:
          claimName: pvc-nfs
```

---

### Langkah guna:

1. **Apply PV**

```bash
kubectl apply -f pv-nfs.yaml
```

2. **Apply PVC**

```bash
kubectl apply -f pvc-nfs.yaml
```

3. **Apply Deployment**

```bash
kubectl apply -f deployment-nfs.yaml
```

4. **Semak status**

```bash
kubectl get pv
kubectl get pvc
kubectl get deployments
kubectl get pods
```

---

### Nota Penting:

* NFS server mesti boleh diakses oleh semua node dalam cluster.
* `accessModes: ReadWriteMany` membolehkan banyak pod baca/tulis serentak pada storage yang sama.
* Jika guna cloud atau NFS yang ada authentication, anda mungkin perlu setup mount options atau secret.

---

Kalau anda mahu saya bantu tambah contoh ConfigMap untuk index.html yang disimpan dalam NFS, atau cara troubleshoot masalah mounting, beritahu saya!
