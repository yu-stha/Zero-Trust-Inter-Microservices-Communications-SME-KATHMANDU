# Complete Results — Baseline vs Zero Trust

## Experiment Summary
- Date: June 2026
- Instance: m7i-flex.large (2 vCPU, 8GB RAM)
- K3s: v1.35.5+k3s1
- Application: Google Online Boutique (5 microservices + Redis)
- Zero Trust: Linkerd mTLS + Linkerd Authorization Policy

## Table 1 — Resource Overhead Comparison
| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU at rest | 0.287% | 0.173% | -0.114% |
| CPU under load | 0.883% | 1.39% | +0.507% |
| RAM at rest | 13.7% | 19.4% | +5.7% |
| RAM under load | 15.0% | 20.4% | +5.4% |
| Frontend CPU peak | 3.30% | 4.13% | +0.83% |
| P50 Latency | N/A | 1ms | +1ms |
| P95 Latency | N/A | 1ms | +1ms |
| P99 Latency | N/A | 1ms | +1ms |

## Table 2 — Attack Simulation Results
| Attack | From | To | Baseline | Zero Trust |
|--------|------|----|----------|------------|
| Attack 1 | frontend | paymentservice | PORT OPEN | ACCESS DENIED |
| Attack 2 | cartservice | paymentservice | PORT OPEN | ACCESS DENIED |
| Attack 3 | productcatalogservice | paymentservice | PORT OPEN | ACCESS DENIED |

## Table 3 — Setup Complexity
| Task | Time Required | Skill Level |
|------|--------------|-------------|
| K3s installation | 5 minutes | Beginner |
| Microservices deployment | 10 minutes | Beginner |
| Linkerd installation | 15 minutes | Intermediate |
| Linkerd sidecar injection | 5 minutes | Intermediate |
| Authorization policy writing | 30 minutes | Intermediate |
| Calico CNI setup | 45 minutes | Advanced |
| Total setup time | ~2 hours | Intermediate |

## Research Question Answers

### RQ1: CPU and RAM overhead of Zero Trust on K3s?
CPU overhead under load: +0.507 percentage points (0.883% to 1.39%)
RAM overhead: +5.7 percentage points (13.7% to 19.4%)
Latency overhead: 1ms P50/P95/P99 (negligible)
CONCLUSION: Overhead is acceptable for SME workloads on m7i-flex.large equivalent hardware.

### RQ2: How effectively does Zero Trust prevent lateral movement?
All 3 lateral movement attacks blocked — 100% effectiveness.
Linkerd Authorization Policy successfully prevented unauthorized
service-to-service communication at the application layer.
CONCLUSION: Zero Trust enforcement is fully effective against lateral movement.

### RQ3: Is Zero Trust feasible for a Kathmandu SME without a dedicated security engineer?
Setup time: approximately 2 hours for a junior engineer familiar with Kubernetes basics.
Main complexity: CNI configuration (Calico) requires troubleshooting experience.
Linkerd itself is straightforward to install and inject.
CONCLUSION: Level 1 Zero Trust is feasible but CNI setup requires careful attention.

### RQ4: Minimum viable Zero Trust for a Kathmandu SME?
See docs/tiered-roadmap.md for full Level 1 specification.
Minimum: K3s + Calico CNI + Linkerd mTLS + Linkerd Authorization Policy
on paymentservice and other sensitive services.
Cost: ~$0.10/hour on equivalent cloud instance.
