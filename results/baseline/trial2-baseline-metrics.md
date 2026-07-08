# Trial 2 — Baseline Metrics (No Zero Trust)
## Date: July 2026
## Instance: m7i-flex.large (2 vCPU, 8GB RAM) — Fresh Instance
## CNI: Calico v3.27.3
## Linkerd: NOT installed
## NetworkPolicy: NOT applied

## CPU at Rest
| Metric | Value |
|--------|-------|
| CPU Utilisation (from requests) | 0.291% |
| CPU Utilisation (from limits) | 0.147% |
| Memory Utilisation (from requests) | 13.1% |
| Memory Utilisation (from limits) | 6.32% |

## Memory per Pod at Rest
| Pod | Memory Usage |
|-----|-------------|
| cartservice | 14.7 MiB |
| checkoutservice | 4.75 MiB |
| frontend | 5.06 MiB |
| paymentservice | 14.1 MiB |
| productcatalogservice | 4.56 MiB |
| redis-cart | 5.45 MiB |
| TOTAL | ~48.6 MiB |

## CPU Under Load
| Metric | Value |
|--------|-------|
| CPU Utilisation (from requests) | 0.921% |
| CPU Utilisation (from limits) | 0.466% |
| Memory Utilisation (from requests) | 14.0% |
| Frontend CPU peak | 3.51% |

## Attack Simulation Results
| Attack | From | To | Result |
|--------|------|----|--------|
| Attack 1 | frontend | paymentservice:50051 | LATERAL MOVEMENT POSSIBLE |
| Attack 2 | frontend | cartservice:7070 | LATERAL MOVEMENT POSSIBLE |
| Attack 3 | frontend | productcatalogservice:3550 | LATERAL MOVEMENT POSSIBLE |
