# Lab Setup Guide

Step-by-step instructions to build the Zero Trust microservices lab from a blank AWS EC2 instance. This guide reflects the exact sequence validated during thesis experimentation.

## Prerequisites

- AWS account with permission to launch EC2 instances
- An SSH key pair (create one in the EC2 console if you don't have one)
- Basic familiarity with a Linux terminal

## Recommended Instance Specification

| Setting | Value |
|---|---|
| Instance type | m7i-flex.large (2 vCPU, 8 GB RAM) |
| OS | Ubuntu Server 22.04 LTS |
| Storage | 30 GiB gp3 |
| Security group | See table below |

### Security Group Inbound Rules

| Type | Port | Source |
|---|---|---|
| SSH | 22 | Your IP |
| Custom TCP | 6443 | Your IP (Kubernetes API — needed only for remote client access) |
| Custom TCP | 30080 | Your IP (frontend NodePort, optional) |

Avoid `0.0.0.0/0` where possible; scope rules to your own IP.

## Step 1 — Launch and Connect

Launch the instance with the specification above, then connect:

```bash
ssh -i your-key.pem ubuntu@<instance-public-ip>
```

## Step 2 — Base Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git unzip python3-pip python3-venv
sudo hostnamectl set-hostname zero-trust-lab
exec bash
```

## Step 3 — Install K3s with Calico-Ready Networking

Standard K3s ships with Flannel, which does **not** enforce Kubernetes NetworkPolicy. Install K3s with Flannel disabled so Calico can take over networking and policy enforcement:

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none --disable-network-policy" sh -
```

Set up kubectl access:

```bash
sudo chmod 644 /etc/rancher/k3s/k3s.yaml
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown ubuntu:ubuntu ~/.kube/config
echo "export KUBECONFIG=/home/ubuntu/.kube/config" >> ~/.bashrc
source ~/.bashrc
kubectl get nodes
```

The node will show `NotReady` until Calico is installed — this is expected.

## Step 4 — Install Calico CNI

```bash
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml
kubectl apply -f calico.yaml
```

Wait roughly 2 minutes, then confirm:

```bash
kubectl get nodes
kubectl get pods -n kube-system | grep calico
```

Both `calico-node` and `calico-kube-controllers` should show `1/1 Running`, and the node should show `Ready`.

**Sanity check before proceeding** — confirm pod networking and DNS actually work:

```bash
kubectl create namespace boutique
kubectl run nettest --image=busybox:1.28 -n boutique --restart=Never -- sleep 3600
sleep 10
kubectl exec -n boutique nettest -- nslookup kubernetes.default.svc.cluster.local
kubectl delete pod nettest -n boutique
```

A successful DNS resolution confirms Calico is healthy. Do not proceed until this works — a broken CNI is the most time-consuming failure mode to diagnose later.

## Step 5 — Clone the Repository

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519 -N ""
cat ~/.ssh/id_ed25519.pub
```

Add the printed key to [GitHub SSH keys](https://github.com/settings/keys), then:

```bash
ssh -T git@github.com
git clone git@github.com:yu-stha/Zero-Trust-Inter-Microservices-Communications-SME-KATHMANDU.git
cd Zero-Trust-Inter-Microservices-Communications-SME-KATHMANDU
```

## Step 6 — Deploy the Microservices

```bash
kubectl apply -f configs/boutique-deploy.yaml
```

Watch until all 6 pods show `Running`:

```bash
kubectl get pods -n boutique -w
```

### Add the Currency Service

The base Online Boutique deployment omits `currencyservice` and points services at a stub instead. Deploy the real service:

```bash
cat > configs/currencyservice-deploy.yaml << 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: currencyservice
  namespace: boutique
spec:
  replicas: 1
  selector:
    matchLabels:
      app: currencyservice
  template:
    metadata:
      labels:
        app: currencyservice
    spec:
      serviceAccountName: currencyservice
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/currencyservice:v0.8.0
        ports:
        - containerPort: 7000
        env:
        - name: PORT
          value: "7000"
        - name: DISABLE_PROFILER
          value: "1"
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: currencyservice
  namespace: boutique
spec:
  selector:
    app: currencyservice
  ports:
  - port: 7000
    targetPort: 7000
EOF

kubectl create serviceaccount currencyservice -n boutique
kubectl apply -f configs/currencyservice-deploy.yaml
kubectl set env deployment/frontend -n boutique CURRENCY_SERVICE_ADDR=currencyservice:7000
kubectl set env deployment/checkoutservice -n boutique CURRENCY_SERVICE_ADDR=currencyservice:7000
```

**Note:** `DISABLE_PROFILER=1` is required. Without it, the currency service crash-loops trying to reach Google Cloud's profiler service, which is unavailable outside GCP.

### Create Named ServiceAccounts for All Services

Linkerd's mTLS identity model uses ServiceAccount names. Give every service its own, rather than sharing the `default` ServiceAccount:

```bash
for svc in frontend cartservice checkoutservice paymentservice productcatalogservice redis-cart; do
  kubectl create serviceaccount $svc -n boutique 2>/dev/null
  kubectl patch deployment $svc -n boutique -p "{\"spec\":{\"template\":{\"spec\":{\"serviceAccountName\":\"$svc\"}}}}"
done
```

## Step 7 — Install Linkerd

```bash
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
export PATH=$PATH:$HOME/.linkerd2/bin
echo 'export PATH=$PATH:$HOME/.linkerd2/bin' >> ~/.bashrc

linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check
```

All checks should pass before continuing.

## Step 8 — Install Linkerd Viz (Observability)

```bash
linkerd viz install | kubectl apply -f -
linkerd viz check
```

## Step 9 — Inject Linkerd Sidecars

```bash
kubectl annotate namespace boutique linkerd.io/inject=enabled
kubectl rollout restart deployment -n boutique
kubectl get pods -n boutique -w
```

Wait until all 7 pods show `2/2 Running` — the second container is the Linkerd proxy.

## Step 10 — Verify the App Works Before Applying Zero Trust

This step matters. Confirm the application is fully healthy on the open, unprotected baseline before adding any policy:

```bash
kubectl port-forward -n boutique svc/frontend 8081:80 > /tmp/pf.log 2>&1 &
sleep 3
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8081/
```

This must return `200` before proceeding. If it doesn't, resolve the underlying application issue first — applying Zero Trust on top of a broken application makes diagnosis significantly harder.

## Step 11 — Baseline Attack Demonstration (Optional but Recommended)

Before enforcing Zero Trust, demonstrate the vulnerability:

```bash
kubectl exec -n boutique deploy/frontend -c server -- nc -zv paymentservice 50051
kubectl exec -n boutique deploy/frontend -c server -- nc -zv cartservice 7070
```

Both should report the port open, demonstrating that `frontend` can currently reach services it has no legitimate reason to reach.

## Step 12 — Install Prometheus and Grafana (Optional, for Metrics)

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.scrapeInterval=15s \
  --set grafana.adminPassword=thesis2024 \
  --set alertmanager.enabled=false
```

Access Grafana:

```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Then browse to `http://localhost:3000` (username `admin`, password `thesis2024`).

## Step 13 — Apply Zero Trust

At this point you can either write NetworkPolicy and Linkerd AuthorizationPolicy YAML manually, or use the [KTMGuard tool](https://github.com/yu-stha/Zero-Trust-KtmGuard-Tool) to generate it automatically from observed traffic. See that repository's `SETUP.md` for instructions.

Once generated:

```bash
kubectl apply -f configs/deny-all.yaml
kubectl apply -f configs/allow-policies.yaml
kubectl apply -f configs/linkerd-auth-policy.yaml
```

Confirm no pods were disrupted:

```bash
kubectl get pods -n boutique
```

## Step 14 — Confirm Enforcement

Repeat the baseline attack from Step 11. Both connections should now fail:

```bash
kubectl exec -n boutique deploy/frontend -c server -- wget -qO- --timeout=3 http://paymentservice:50051/
```

Expect a timeout or `503` — this is a legitimate application-layer denial, not a tooling failure.

## Common Issues

| Symptom | Cause | Fix |
|---|---|---|
| `kubectl` reports "connection refused" after stopping/starting the instance | K3s config file permissions reset on reboot | `sudo chmod 644 /etc/rancher/k3s/k3s.yaml` then re-copy to `~/.kube/config` |
| Every pod-to-pod connection times out after a reboot | Calico's routing state can become stale after host-level restarts | `kubectl delete pod -n kube-system -l k8s-app=calico-node` and `-l k8s-app=calico-kube-controllers`, wait for recreation |
| A service crash-loops on startup | Missing `DISABLE_PROFILER=1` (Online Boutique images try to reach GCP's profiler by default) | Add the env var to the affected deployment |
| Public IP changes after stop/start | EC2 instances without an Elastic IP get a new public IP on every start | Re-check the IP in the AWS console after each restart; update any saved kubeconfig accordingly |

## Cost Note

An m7i-flex.large instance costs approximately $0.10–0.12/hour. Stop the instance when not actively working with it. A full lab session (setup through teardown) typically costs under $1.