# Zero Trust Inter-Microservices Communications — SME Kathmandu

## Project Title
Design, Development and Evaluation of a Zero Trust Network Policy Enforcement Framework for Microservices-Based Applications in Resource-Constrained SME Environments of Kathmandu Valley, Nepal

## Degree
BSc (Hons) Cybersecurity and Ethical Hacking

---

## What This Project Is

This repository contains the complete lab environment, configuration files, results, and documentation for a BSc Cybersecurity thesis investigating Zero Trust network policy enforcement for microservices-based applications in resource-constrained SME environments of Kathmandu Valley, Nepal.

Modern Kubernetes deployments allow all services to communicate freely by default — no encryption, no verification, no access control. This creates a lateral movement vulnerability where an attacker who compromises one service can freely reach all other services. This project proves that vulnerability exists, then eliminates it using Linkerd service mesh and demonstrates the result with repeated attack simulations.

---

## Research Questions

1. What is the CPU, RAM, and latency overhead of enforcing Zero Trust via Linkerd mTLS on a K3s cluster constrained to SME-scale hardware?
2. How effectively does mTLS enforcement combined with Linkerd Authorization Policy prevent lateral movement between microservices?
3. Is Zero Trust enforcement operationally feasible for a Kathmandu SME without a dedicated security engineer?
4. What is the minimum viable Zero Trust configuration a resource-constrained Kathmandu SME can realistically implement?

---

## Lab Environment

| Component | Specification |
|-----------|--------------|
| Cloud | AWS EC2 m7i-flex.large |
| CPU | 2 vCPU |
| RAM | 8 GB |
| Storage | 20 GB gp3 |
| OS | Ubuntu 22.04 LTS |
| Kubernetes | K3s v1.35.5+k3s1 |
| CNI | Calico v3.27.3 |
| Service Mesh | Linkerd stable-2.x |
| Application | Google Online Boutique (5 services) |
| Monitoring | Prometheus + Grafana |

---

## Service Communication Map

    frontend → productcatalogservice
    frontend → cartservice
    frontend → checkoutservice
    checkoutservice → paymentservice
    checkoutservice → cartservice
    cartservice → redis-cart

Any communication outside these paths = lateral movement attack path.

---

## Experiment Design

### Scenario 1 — Baseline (No Zero Trust)
- Default K3s configuration
- No NetworkPolicy, no service mesh
- All pod-to-pod communication open, unencrypted, unverified
- Attack result: All attacks succeed — lateral movement fully possible

### Scenario 2 — Zero Trust Enforced
- Calico CNI for NetworkPolicy enforcement
- Linkerd mTLS injected into all pods (2/2 containers per pod)
- Linkerd Authorization Policy restricting paymentservice access
- Attack result: All attacks blocked — ACCESS DENIED

---

## Key Results

| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU at rest | 0.287% | 0.173% | -0.114% |
| CPU under load | 0.883% | 1.39% | +0.507% |
| RAM at rest | 13.7% | 19.4% | +5.7% |
| RAM under load | 15.0% | 20.4% | +5.4% |
| P50 Latency | N/A | 1ms | +1ms |
| Lateral movement | POSSIBLE | BLOCKED | 100% |

---

## Repository Structure

    zero-trust-sme-kathmandu/
    ├── README.md
    ├── configs/
    │   ├── boutique-deploy.yaml
    │   ├── deny-all.yaml
    │   ├── allow-policies.yaml
    │   ├── linkerd-auth-policy.yaml
    │   └── linkerd-notes.md
    ├── results/
    │   ├── baseline/
    │   │   ├── baseline-metrics.md
    │   │   └── screenshots/
    │   └── zerotrust/
    │       ├── zerotrust-metrics.md
    │       └── screenshots/
    ├── scripts/
    │   └── load-generator.sh
    └── docs/
        └── tiered-roadmap.md

---

## How to Reproduce This Experiment

1. Launch AWS EC2 m7i-flex.large with Ubuntu 22.04 LTS
2. Install K3s with Calico CNI
3. Install Linkerd and inject into boutique namespace
4. Deploy microservices: kubectl apply -f configs/boutique-deploy.yaml
5. Record baseline metrics from Grafana
6. Run baseline attack from frontend pod to paymentservice
7. Apply Linkerd auth policy: kubectl apply -f configs/linkerd-auth-policy.yaml
8. Repeat attack and observe ACCESS DENIED
9. Record Zero Trust metrics from Grafana

---

## Key Results Summary

Zero Trust enforcement via Linkerd mTLS is technically and operationally feasible for resource-constrained Kathmandu Valley SMEs. The CPU overhead of +0.507 percentage points under load and memory overhead of +5.7 percentage points are well within acceptable limits. All lateral movement attacks were blocked with 100% effectiveness. A junior engineer can implement Level 1 Zero Trust in 2-3 hours without a dedicated security engineer.

---

## Author
Yubin Shrestha
BSc (Hons) Cybersecurity and Ethical Hacking
Kathmandu Valley, Nepal, 2026
