To simulate autoscaling for your PHP app Kubernetes deployment, you'll need to create a **Horizontal Pod Autoscaler (HPA)** resource that automatically adjusts the number of pod replicas based on metrics like CPU utilization.

Here’s how you can set it up:

---

### ✅ Prerequisites

1. Metrics server must be installed in your cluster:

   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. Your container image must support resource metrics (e.g., CPU usage).

---

### 🛠 Modify Deployment to Include Resource Requests and Limits

Update your `Deployment` to include `resources` under the `containers` section:

```yaml
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

Final container section should look like this:

```yaml
        - name: php-container
          image: conanslash/php-pod-info:upload-0.2
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
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

---

### ⚙️ Create HPA Resource

Now, create a `HorizontalPodAutoscaler` (save it as `php-app-hpa.yaml`):

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-app-hpa
  namespace: pengambilan
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Apply it:

```bash
kubectl apply -f php-app-hpa.yaml
```

---

### 🧪 Simulate Load


1. Create a pod to send requests:

    ```bash
    kubectl run -i --tty load-generator --image=busybox /bin/sh
    ```

2. In that shell, run:

    ```sh
    while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
    ```


Watch the HPA react:

```bash
kubectl get hpa -w
```

You’ll see the number of pods increase as CPU usage increases.
---

### 🔍 Monitor the Autoscaler

Run the following to watch autoscaler activity:

```bash
kubectl get hpa -n pengambilan -w
```

Or check pod scaling:

```bash
kubectl get pods -n pengambilan -l app=php-app
```

---
