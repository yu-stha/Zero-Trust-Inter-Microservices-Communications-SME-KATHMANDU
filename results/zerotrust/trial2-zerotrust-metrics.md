# Trial 2 — Zero Trust Metrics (Linkerd + NetworkPolicy)
## Date: July 2026
## Instance: m7i-flex.large (2 vCPU, 8GB RAM) — Fresh Instance
## CNI: Calico v3.27.3
## Linkerd: edge-26.6.3 — injected into all pods (2/2)
## NetworkPolicy: deny-all + 8 allow rules applied
## Linkerd Authorization Policy: paymentservice protected

## CPU at Rest
| Metric | Value |
|--------|-------|
| CPU Utilisation (from requests) | 0.780% |
| CPU Utilisation (from limits) | 0.395% |
| Memory Utilisation (from requests) | 19.6% |
| Memory Utilisation (from limits) | 9.42% |

## Memory per Pod at Rest
| Pod | Baseline | Zero Trust | Difference |
|-----|----------|------------|------------|
| cartservice | 14.7 MiB | 18.0 MiB | +3.3 MiB |
| checkoutservice | 4.75 MiB | 9.66 MiB | +4.91 MiB |
| frontend | 5.06 MiB | 10.3 MiB | +5.24 MiB |
| paymentservice | 14.1 MiB | 17.8 MiB | +3.7 MiB |
| productcatalogservice | 4.56 MiB | 7.87 MiB | +3.31 MiB |
| redis-cart | 5.45 MiB | 8.75 MiB | +3.3 MiB |
| TOTAL | 48.6 MiB | 72.4 MiB | +23.8 MiB |

## CPU Under Load
| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU (from requests) | 0.921% | 1.48% | +0.559% |
| CPU (from limits) | 0.466% | 0.748% | +0.282% |
| Memory (from requests) | 14.0% | 20.4% | +6.4% |
| Frontend CPU peak | 3.51% | 4.28% | +0.77% |

## Linkerd mTLS Status
- All 6 services MESHED 1/1
- Success rate: 100%
- P50 Latency: 1ms
- P95 Latency: 1ms
- P99 Latency: 1ms

## Attack Simulation Results
| Attack | From | To | Result |
|--------|------|----|--------|
| Attack 1 | frontend | paymentservice:50051 | ACCESS DENIED |
| Attack 2 | frontend | cartservice:7070 | ACCESS DENIED |
| Attack 3 | frontend | productcatalogservice:3550 | ACCESS DENIED |

## Comparison Summary
| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU at rest | 0.291% | 0.780% | +0.489% |
| CPU under load | 0.921% | 1.48% | +0.559% |
| RAM at rest | 13.1% | 19.6% | +6.5% |
| RAM under load | 14.0% | 20.4% | +6.4% |
| Lateral movement | POSSIBLE | BLOCKED | 100% blocked |
