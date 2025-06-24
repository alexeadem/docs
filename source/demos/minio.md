---
title: MinIO
---
## MinIO on QBO - Deploying High-Performance Object Storage

{% preview "minio.svg" %}

## **Introduction**

MinIO is a high-performance, S3-compatible object storage system, ideal for cloud-native applications. Running MinIO on **QBO** provides **scalability, multi-architecture support (arm64 & amd64), and high availability**.

QBO's **Kubernetes in Docker** approach enhances **performance and portability**, making it an excellent choice for deploying MinIO. With **bare-metal efficiency, direct resource access, and self-contained cloud environments**, QBO ensures seamless MinIO operation in **private, public, air-gapped, on-prem, or hybrid environments**.

In this **demo**, we will walk through setting up a **distributed MinIO deployment** on QBO, configuring **external access**, and testing **data uploads** using `mc` (MinIO Client).

{% youtube apm9bcOiOYQ %}

---

## **Step-by-Step Deployment Guide**

### **1. Check QBO Version**

Display QBO version information:

```bash
qbo version | jq .version[]?
```

### **2. Create or Retrieve Cluster**

Check if a cluster named `minio` exists, and create it if needed:

```bash
qbo get cluster minio | jq -e '.clusters[]?'
qbo add cluster minio -n 4 -i hub.docker.com/kindest/node:v1.32.0 | jq
```

### **3. Retrieve Node Information**

List all nodes in the cluster:

```bash
qbo get nodes minio | jq .nodes[]?
```

### **4. Configure Kubeconfig**

Set your Kubernetes configuration:

```bash
qbo get cluster minio -k | jq -r '.output[]?.kubeconfig | select( . != null)' > $HOME/.qbo/minio.conf
export KUBECONFIG=$HOME/.qbo/minio.conf
```

### **5. Verify Nodes**

Ensure the cluster is ready:

```bash
kubectl get nodes
```

### **6. Deploy Headless Service**

Apply the headless service required for MinIO internode communication:

```bash
kubectl apply -f distributed/minio-distributed-headless-service.yaml
```

### **7. Generate TLS Secrets**

Create and apply TLS secrets using ACME certificates retrieved from QBO:

```bash
qbo get acme -A | jq
kubectl get secret minio-tls -n default -o jsonpath="{.data.tls\.crt}" | base64 -d
kubectl get secret minio-tls -n default -o jsonpath="{.data.tls\.key}" | base64 -d
```

### **8. Create Access Credentials**

Use the cluster UUID as the MinIO password and store credentials:

```bash
UUID=$(qbo get cluster minio | jq -r '.clusters[]?.id')
kubectl create secret generic minio-creds --from-literal=MINIO_ACCESS_KEY=qbo --from-literal=MINIO_SECRET_KEY=$UUID --dry-run=client -o yaml | kubectl apply -f -
kubectl get secret minio-creds -o jsonpath='{.data.MINIO_ACCESS_KEY}' | base64 -d && echo
kubectl get secret minio-creds -o jsonpath='{.data.MINIO_SECRET_KEY}' | base64 -d && echo
```

### **9. Deploy MinIO StatefulSet**

Create the StatefulSet for distributed MinIO:

```bash
kubectl apply -f distributed/minio-distributed-statefulset.yaml
```

### **10. Expose MinIO via LoadBalancer**

Replace the default service and expose MinIO:

```bash
kubectl delete svc minio
kubectl apply -f minio-svc.yaml
```

### **11. Deploy Ingress Controller and Routes**

Set up ingress-nginx and apply MinIO ingress configs:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/baremetal/deploy.yaml
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec": {"type": "LoadBalancer"}}'
watch kubectl get job ingress-nginx-admission-patch -n ingress-nginx
kubectl apply -f minio-console-ingress.yaml
kubectl apply -f minio-s3-ingress.yaml
```

### **12. Restart and Inspect**

Restart the MinIO statefulset and verify:

```bash
kubectl rollout restart statefulset minio
watch kubectl get pods -n default
kubectl describe pod/minio-0 -n default
kubectl get pvc
```

### **13. Install MinIO Client**

Download and install `mc` (MinIO Client):

```bash
cd ~
curl -sSL -o mc https://dl.min.io/client/mc/release/linux-$(uname -m)/mc
mkdir -p ~/.local/bin
sudo install -m 555 mc ~/.local/bin/mc
```

### **14. Test DNS Resolution**

Confirm internal DNS works:

```bash
kubectl run busybox --image=busybox:latest --restart=Never -it --rm -- nslookup minio.default.svc.cluster.local
```

### **15. Add Local DNS Records (Optional for Development)**

If not using an external DNS provider, manually update `/etc/hosts`:

```bash
HOSTNAME=$(hostname | sed 's/^[^.]*\.//')
LB=$(kubectl get svc ingress-nginx-controller -n ingress-nginx --ignore-not-found -o json | jq -r '.spec.externalIPs[0] | select ( . != null)')

# Show recommended records
echo -e "\nAdd the following DNS records to your DNS provider or local /etc/hosts:\n"
echo -e "A\ts3.$HOSTNAME\t$LB"
echo -e "A\tconsole.$HOSTNAME\t$LB"

# Append to local hosts file (requires sudo)
LINE="$LB s3.$HOSTNAME console.$HOSTNAME"
TMP=$(mktemp)
grep -vE "s3\\.$HOSTNAME|console\\.$HOSTNAME" /etc/hosts > "$TMP"
echo "$LINE" >> "$TMP"
sudo cp "$TMP" /etc/hosts && rm "$TMP"
```

### **16. Configure Client and Upload Data**

Set the MinIO alias and access the admin interface:

```bash
HOSTNAME=$(hostname | sed 's/^[^.]*\.//')
mc alias set myminio https://s3.$HOSTNAME qbo $UUID
mc admin info myminio
```

Create a bucket and upload a test file:

```bash
mc mb myminio/mybucket
echo "Hello MinIO!" > testfile.txt
mc cp testfile.txt myminio/mybucket/
mc ls myminio/mybucket/
```

Access Information:

```bash
# USER: qbo
# PASSWORD: $UUID
# S3: https://s3.$HOSTNAME
# URL: https://minio.$HOSTNAME
```
{% preview 'Screenshot From 2025-06-24 10-24-14.png' %}
{% preview 'Screenshot From 2025-06-24 10-11-40.png' %}
---

## **Why MinIO on QBO? Performance and Portability**

### **Performance Benefits**

- **Bare-Metal Efficiency** - QBO eliminates the overhead of traditional virtualization, ensuring MinIO runs at peak performance.
- **Direct Resource Access** - Containers in QBO get direct access to CPU, RAM, and GPUs, improving performance for data-intensive workloads.

### **Portability Advantages**

- **Versatile Deployment** - MinIO runs seamlessly on **on-prem, cloud, hybrid, and air-gapped environments**.
- **Self-Contained Cloud** - QBO simplifies Kubernetes management, making MinIO deployments more portable and independent.

---

## **Conclusion**

By deploying MinIO on QBO, we achieve **scalable, high-performance object storage** with full Kubernetes integration.

QBO’s **Kubernetes in Docker architecture** enhances **performance** and **portability**, making it an ideal choice for cloud-native applications.

Start using MinIO on QBO today for an efficient, high-performance, and scalable storage solution.
