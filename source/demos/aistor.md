---
title: MinIO 
---

## Setting up MinIO AIStore on QBO

{% preview "minio.svg" %}


## Deploying MinIO AIStore with QBO Container Engine (QCE)

**MinIO AIStore** is an S3-compatible object store optimized for AI workloads — enabling high-speed access to training data, model checkpoints, logs, and inference outputs.

In this guide, we deploy AIStore on an instance provisioned by the **QBO Container Engine (QCE)**. QCE runs directly on bare metal and leverages **Docker-in-Docker (DinD)** to manage isolated workloads — each with direct access to **NVIDIA GPUs** via the **NVIDIA Container Toolkit**.

### Why use MinIO AIStore on QCE?

- **GPU-native by default**: Every instance has direct access to attached NVIDIA GPUs  
- **Metal disk performance**: Local NVMe or SSD storage provides high-throughput access to data  
- **Simple and fast**: Deploy object storage in seconds with secure TLS and DNS configured automatically  
- **Optimized for AI**: Store models, datasets, and artifacts efficiently with S3-compatible APIs  
- **Portable infrastructure**: Deployable on-prem or across any QBO-powered environment  

This setup delivers a fast, production-grade object store for AI/ML workloads — with full control over data, performance, and deployment.


{% youtube 1InYU8S_Y1E %}

---


## Prerequisites
- A running QBO instance named `aistor_object_store`
- QBO CLI configured with access credentials
- Your QBO host issuing ACME certificates

---

## Step-by-Step Setup

### 1. Check QBO version and create the object store instance
```bash
# Get the QBO CLI/API version
qbo version | jq .version[]?

# Create an instance named aistor_object_store from the mc2 image
qbo add instance aistor_object_store -i registry.cloud.qbo.io/qbo/src/cloud/instance:mc2 | jq

# Retrieve details of the instance you just created
qbo get instance aistor_object_store | jq -e '.instances[]?'
```
### 2. Get the SSH configuration
```bash
qbo get instance aistor_object_store -k | jq -r '.output[]?.sshconfig | select( . != null)' > $HOME/.qbo/aistor_object_store.crt
chmod 600 $HOME/.qbo/aistor_object_store.crt
```

### 3. Retrieve instance IP address
```bash
ADDR=$(qbo get ipvs aistor_object_store | jq -r '.ipvs[]? | select(.node | contains("insta")) | .vip' | head -1)
```

### 4. Retrieve ACME TLS certificates from QBO
```bash
KEY=$(qbo get acme -A 2>/dev/null | jq -r .acmes[]?.privkey | openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt | base64 -w 0)
CRT=$(qbo get acme -A 2>/dev/null | jq -r .acmes[]?.fullchain | base64 -w 0)
```

### 5. Upload TLS certs to remote instance
```bash
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'mkdir -p ~/.minio/certs'
echo $KEY | base64 -d | ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'cat > ~/.minio/certs/private.key'
echo $CRT | base64 -d | ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'cat > ~/.minio/certs/public.crt'
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'chmod 600 ~/.minio/certs/private.key && chmod 644 ~/.minio/certs/public.crt'
```

### 6. Download MinIO binary and install license
```bash
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'curl -sSL https://dl.min.io/aistor/minio/release/linux-amd64/minio -o ~/minio && chmod +x ~/minio'
# License can be retrieved from: https://min.io/download?view=aistor
scp -O -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt ./minio.license qbo@$ADDR:minio.license
```

### 7. Create and upload the systemd service
> Note: The instance ID (UUID) is used as the MinIO qbo password
```bash
UUID=$(qbo get instance aistor_object_store | jq -r '.instances[]?.id')
cat > minio.service <<EOF
[Unit]
Description=MinIO Server
After=network.target

[Service]
User=qbo
Environment=MINIO_ROOT_USER=qbo
Environment=MINIO_ROOT_PASSWORD=$UUID
Environment=MINIO_LICENSE=/home/qbo/minio.license
ExecStart=/home/qbo/minio server /home/qbo/data --console-address :9001
Restart=always

[Install]
WantedBy=multi-user.target
EOF

scp -O -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt ./minio.service qbo@$ADDR:/tmp/minio.service
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'sudo mv /tmp/minio.service /etc/systemd/system/minio.service && sudo chown root:root /etc/systemd/system/minio.service'
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'sudo systemctl daemon-reload && sudo systemctl enable --now minio && sudo systemctl restart minio'
```


### 8. Verify MinIO is running
```bash
# Check systemd journal logs for MinIO
ssh -o StrictHostKeyChecking=no -i $HOME/.qbo/aistor_object_store.crt qbo@$ADDR 'sudo journalctl -t minio -n 20 --no-pager'

# Run health check — should return 200 on success
code=$(curl -s -o /dev/null -w '%{http_code}' https://aistor.$HOSTNAME:9000/minio/health/ready)

if [ "$code" = "200" ]; then
  echo -e "\033[32mMinIO is healthy: $code\033[0m"
else
  echo -e "\033[31mMinIO health check failed: $code\033[0m"
fi
```


### 9. Add to /etc/hosts for local DNS
```bash
HOSTNAME=$(hostname | sed 's/^[^.]*\.//')
echo "$ADDR aistor.$HOSTNAME" | sudo tee -a /etc/hosts
```

### 10. Install mc (MinIO Client) if needed
```bash
ARCH=$(uname -m)
[ "$ARCH" = "x86_64" ] && ARCH="amd64"
curl -sSL -o mc https://dl.min.io/client/mc/release/linux-$ARCH/mc
mkdir -p ~/.local/bin
sudo install -m 555 mc ~/.local/bin/mc
```

### 11. Configure `mc` and test upload
```bash
mc alias set myminio https://aistor.$HOSTNAME:9000 qbo $UUID
mc admin info myminio
mc mb myminio/mybucket
echo "Hello MinIO!" > testfile.txt
mc cp testfile.txt myminio/mybucket/
mc ls myminio/mybucket/
```

---

## Access Credentials
{% preview 'Screenshot From 2025-06-23 23-56-57.png' %}

<!-- {% preview 'Screenshot From 2025-06-23 23-56-50.png' %} -->

```
mc ls myminio/mybucket/
[2025-06-24 15:20:22 UTC]    13B STANDARD testfile.txt
USER     qbo                   
PASSWORD 0acc380e-6d10-49c7-945d-9f8f35549cd8
S3       https://aistor.cloud.levitas.dev:9000
URL      https://aistor.cloud.levitas.dev:9001
```
