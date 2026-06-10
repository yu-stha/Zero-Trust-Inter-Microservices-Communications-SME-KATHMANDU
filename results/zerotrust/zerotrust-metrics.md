
## CPU and RAM at Rest — Zero Trust Enforced
| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU (from requests) | 0.287% | 0.173% | -0.114% |
| CPU (from limits) | 0.145% | 0.0878% | -0.057% |
| Memory (from requests) | 13.7% | 19.4% | +5.7% |
| Memory (from limits) | 6.62% | 9.34% | +2.72% |

## CPU and RAM Under Load — Zero Trust Enforced
| Metric | Baseline | Zero Trust | Difference |
|--------|----------|------------|------------|
| CPU (from requests) | 0.883% | 1.39% | +0.507% |
| CPU (from limits) | 0.448% | 0.703% | +0.255% |
| Memory (from requests) | 15.0% | 20.4% | +5.4% |
| Memory (from limits) | 7.24% | 9.82% | +2.58% |
| Frontend CPU peak | 3.30% | 4.13% | +0.83% |

## Summary
Zero Trust overhead under load:
- CPU increase: approximately 0.5 percentage points
- Memory increase: approximately 5.4 percentage points
- Lateral movement: completely blocked on all 3 attack attempts
- Linkerd latency: 1ms P50, 1ms P95, 1ms P99
