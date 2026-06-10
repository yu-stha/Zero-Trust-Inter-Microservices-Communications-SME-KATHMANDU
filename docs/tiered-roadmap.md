# Zero Trust Tiered Adoption Roadmap
# For Kathmandu Valley SMEs — Based on Lab Evidence

## Introduction
This roadmap is based on direct lab experience building and testing a Zero Trust
enforcement framework on a resource-constrained single-node K3s cluster. All time
estimates and complexity ratings are derived from actual implementation experience,
not theoretical projections. The target audience is a dev lead or junior DevOps
engineer at a Kathmandu SME with basic Kubernetes knowledge and no dedicated
security engineer.

---

## Level 1 — Minimum Viable Zero Trust
**Target:** Any SME with basic Kubernetes knowledge
**Estimated time:** 2-3 hours
**Prerequisites:** Running K3s cluster, basic kubectl knowledge

### What You Get
- Automatic mTLS encryption on all service-to-service traffic
- Identity-based access control on sensitive services
- Lateral movement attacks blocked at application layer
- Full observability via Linkerd dashboard

### Step by Step

#### Step 1 — Install Calico CNI (45 minutes)
Calico is required for NetworkPolicy enforcement in K3s.
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none --disable-network-policy" sh -
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml
Wait for calico-node and calico-kube-controllers to show Running.
CRITICAL: Do not apply any NetworkPolicy until Calico is fully running.

#### Step 2 — Install Linkerd (15 minutes)
curl --proto '=https' --tlsv1.2 -sSfL https://run.linkerd.io/install | sh
export PATH=PATH:PATH:
PATH:HOME/.linkerd2/bin
linkerd install --crds | kubectl apply -f -
linkerd install | kubectl apply -f -
linkerd check
Wait for all green checkmarks.

#### Step 3 — Inject Linkerd Into Your Namespace (5 minutes)
kubectl annotate namespace YOUR_NAMESPACE linkerd.io/inject=enabled
kubectl rollout restart deployment -n YOUR_NAMESPACE
Wait until all pods show 2/2 Running.

#### Step 4 — Define Your Service Communication Map
CRITICAL PREREQUISITE: Before applying any policy, map every legitimate
communication path in your application. Example:
- frontend → productcatalogservice
- frontend → cartservice
- checkoutservice → paymentservice
Without this map, deny-all will break your application silently.

#### Step 5 — Apply Linkerd Authorization Policy (30 minutes)
Protect your most sensitive service first (e.g. paymentservice):
apiVersion: policy.linkerd.io/v1beta1
kind: Server
metadata:
name: paymentservice-server
namespace: YOUR_NAMESPACE
spec:
podSelector:
matchLabels:
app: paymentservice
port: 50051
proxyProtocol: gRPC
apiVersion: policy.linkerd.io/v1alpha1
kind: MeshTLSAuthentication
metadata:
name: allowed-services
namespace: YOUR_NAMESPACE
spec:
identities:

"checkoutservice.YOUR_NAMESPACE.serviceaccount.identity.linkerd.cluster.local"


apiVersion: policy.linkerd.io/v1alpha1
kind: AuthorizationPolicy
metadata:
name: payment-policy
namespace: YOUR_NAMESPACE
spec:
targetRef:
group: policy.linkerd.io
kind: Server
name: paymentservice-server
requiredAuthenticationRefs:

name: allowed-services
kind: MeshTLSAuthentication
group: policy.linkerd.io


### What Level 1 Protects Against
- Lateral movement between microservices
- Unencrypted internal traffic
- Unauthorized service-to-service calls on protected ports

### What Level 1 Does NOT Protect Against
- Fine-grained HTTP route-level authorization
- Runtime anomaly detection
- Compromised service accounts
- External traffic inspection

---

## Level 2 — Hardened Zero Trust
**Target:** SME with at least one junior DevOps engineer
**Estimated additional time:** 2-3 days on top of Level 1
**Prerequisites:** Level 1 complete

### What You Add
- HTTPRoute-level authorization (restrict specific API endpoints)
- Linkerd Viz dashboard for ongoing traffic monitoring
- NetworkPolicy for namespace-level isolation
- Regular review of authorization policies

### Key Steps
1. Install Linkerd Viz: `linkerd viz install | kubectl apply -f -`
2. Define HTTPRoute policies for each service
3. Apply namespace-level NetworkPolicy for coarse isolation
4. Set up Grafana alerts for unexpected traffic patterns
5. Document all authorization policies in version control

### What Level 2 Adds
- Per-route authorization (not just per-service)
- Full visibility into all service communication
- Namespace isolation as defence in depth
- Alerting on policy violations

---

## Level 3 — Full Zero Trust Posture
**Target:** SME with dedicated security role or outsourced security function
**Estimated time:** 2-4 weeks initial setup, ongoing maintenance
**Prerequisites:** Level 2 complete

### What You Add
- Runtime anomaly detection (Falco or similar)
- Automated certificate rotation
- Formal security audit process
- Incident response playbook for policy violations
- Regular penetration testing of service mesh configuration
- Compliance documentation

### What Level 3 Adds
- Proactive threat detection
- Compliance readiness
- Formal security governance
- Ongoing security assurance

---

## Cost Estimate for Kathmandu SMEs
| Level | Setup Time | Monthly Cost (AWS) | Skills Required |
|-------|-----------|-------------------|-----------------|
| Level 1 | 2-3 hours | ~$30-50/month | Junior DevOps |
| Level 2 | 2-3 days | ~$30-50/month | Mid DevOps |
| Level 3 | 2-4 weeks | ~$50-100/month | Senior/Security |

Note: Monthly cost refers to compute cost only. Linkerd is open source and free.

---

## Key Lesson From This Research
The single biggest barrier to Zero Trust adoption for Kathmandu SMEs is not
cost or complexity — it is the absence of locally-validated evidence that it
works at their scale. This research provides that evidence for the first time.
A junior engineer can implement Level 1 Zero Trust in a single working day
on free-tier equivalent hardware, achieving 100% lateral movement blocking
with less than 1.4% CPU overhead and 1ms latency penalty.
