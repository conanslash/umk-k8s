---

## Contoh lengkap: NFS + PVC + Deployment

---
### Install HelM

```yaml
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner

helm install nfs-client nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
  --namespace nfs-provisioner --create-namespace \
  -f values.yaml
```

Fail:  `values.yaml`

```yaml
nfs:
  server: cohecbjvip.umk.edu.my
  path: /kebetest
  mountOptions:
    - vers=4
    - nolock
    - noatime
    - hard
    - timeo=600
    - retrans=2

storageClass:
  name: nfs-client
  defaultClass: true
```


### 1. Persistent Volume (PV) guna NFS

Fail: `pv-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteMany
  storageClassName: nfs-client
  nfs:
    path: /mnt/training/k8s
    server: 10.21.21.200
```

---

### 2. Persistent Volume Claim (PVC)

Fail: `pvc-nfs.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
  namespace: pengambilan
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  storageClassName: nfs-client
```

---

### 3. Deployment guna PVC

Fail: `php-app.yaml`

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
      securityContext:
        fsGroup: 1000  # Ensure this group has write access in the container
      containers:
        - name: php-container
          image: conanslash/php-pod-info:upload-0.4 # ← Replace with your Docker Hub username
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
            - name: nfs-storage
              mountPath: /uploads-nfs
      volumes:
        - name: upload-volume
          persistentVolumeClaim:
            claimName: php-pvc
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
