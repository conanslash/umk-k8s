Baik! Saya tambah **Langkah Deployment** lengkap dengan contoh manifest untuk deploy aplikasi dan servis yang guna MetalLB sebagai LoadBalancer.

---

# Langkah-langkah lengkap install & deployment MetalLB dalam Kubernetes

---

## Langkah 1: Pastikan Kubernetes ready

```bash
kubectl get nodes
```

---

## Langkah 2: Pasang MetalLB

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.13/config/manifests/metallb-native.yaml
```

Tunggu pod MetalLB siap:

```bash
kubectl get pods -n metallb-system
```

---

## Langkah 3: Konfigurasi IP Pool untuk MetalLB

Buat fail `metallb-config.yaml`:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  namespace: metallb-system
  name: my-ip-pool
spec:
  addresses:
  - 192.168.1.240-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  namespace: metallb-system
  name: l2-advertisement
spec:
  ipAddressPools:
  - my-ip-pool
```

Gantikan `192.168.1.240-192.168.1.250` ikut rangkaian anda.

Apply config:

```bash
kubectl apply -f metallb-config.yaml
```

---

## Langkah 4: Buat Deployment aplikasi (contoh: nginx)

Buat fail `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
        - containerPort: 80
```

Apply deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

---

## Langkah 5: Buat Service jenis LoadBalancer guna MetalLB

Buat fail `nginx-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

Apply service:

```bash
kubectl apply -f nginx-service.yaml
```

---

## Langkah 6: Semak IP LoadBalancer dari MetalLB

```bash
kubectl get svc nginx-service
```

Output akan tunjuk IP dari range yang anda konfigurasikan di MetalLB.

---

## Langkah 7: Uji akses aplikasi

Buka browser atau `curl` ke IP tersebut:

```bash
curl http://<IP_LOADBALANCER>
```

Anda akan dapat respon dari server nginx.

---

### Ringkasannya:

1. Pasang MetalLB dan tunggu siap.
2. Konfigurasi IP Pool.
3. Deploy aplikasi (nginx).
4. Deploy Service jenis LoadBalancer.
5. Dapatkan IP dan akses aplikasi.

---

Kalau nak saya buatkan contoh fail lengkap untuk anda simpan, saya boleh sediakan!
