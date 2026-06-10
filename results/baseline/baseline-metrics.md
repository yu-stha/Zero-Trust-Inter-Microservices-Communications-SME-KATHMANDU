# Baseline Metrics — No Zero Trust

## Environment
- Date: June 2026
- Instance: m7i-flex.large (2 vCPU, 8GB RAM)
- K3s version: v1.35.5+k3s1
- Namespace: boutique
- Condition: At rest, no load, no Zero Trust

## CPU at Rest
| Metric | Value |
|--------|-------|
| CPU Utilisation (from requests) | 0.287% |
| CPU Utilisation (from limits) | 0.145% |

## Memory at Rest (per pod)
| Pod | Memory Usage |
|-----|-------------|
| cartservice | 18.3 MiB |
| paymentservice | 12.5 MiB |
| frontend | 5.29 MiB |
| checkoutservice | 4.76 MiB |
| productcatalogservice | 4.67 MiB |
| redis-cart | 5.46 MiB |
| **TOTAL** | **~51 MiB** |

## Memory Utilisation
| Metric | Value |
|--------|-------|
| Memory Utilisation (from requests) | 13.7% |
| Memory Utilisation (from limits) | 6.62% |

## Lateral Movement Attack Results
| Attack | From | To | Port | Result |
|--------|------|----|------|--------|
| Attack 1 | frontend | paymentservice | 50051 | PORT OPEN |
| Attack 2 | frontend | productcatalogservice | 3550 | PORT OPEN |
| Attack 3 | frontend | cartservice | 7070 | PORT OPEN |
| Attack 4 | cartservice | paymentservice | 50051 | PORT OPEN |
| Attack 5 | productcatalogservice | paymentservice | 50051 | PORT OPEN |

## Conclusion
All services reachable from any other service.
No access control, no encryption, no verification.
Lateral movement fully possible in baseline state.

## CPU and Memory Under Load
(to be recorded after load test)
| Metric | Value |
|--------|-------|
| CPU Utilisation under load | TBD |
| Memory Utilisation under load | TBD |

## Updated Under Load Numbers
| Metric | At Rest | Under Load |
|--------|---------|------------|
| CPU Utilisation (from requests) | 0.287% | 0.883% |
| CPU Utilisation (from limits) | 0.145% | 0.448% |
| Memory Utilisation (from requests) | 13.7% | 15.0% |
| Memory Utilisation (from limits) | 6.62% | 7.24% |

## Per Pod CPU Under Load
| Pod | CPU Usage | CPU Requests % |
|-----|-----------|----------------|
| frontend | 0.00332 | 3.32% |
| redis-cart | 0.00154 | 2.20% |
| cartservice | 0.0000468 | 0.0468% |
| checkoutservice | 0.000105 | 0.105% |
| paymentservice | 0 | 0% |
| productcatalogservice | 0.0000238 | 0.0238% |
