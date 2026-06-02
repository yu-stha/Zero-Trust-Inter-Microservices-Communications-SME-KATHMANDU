# Zero Trust Inter-Microservices Communications — SME Kathmandu

## Project Overview
This repository contains the complete lab environment, configuration files, results, and documentation for a BSc Cybersecurity thesis investigating Zero Trust network policy enforcement for microservices-based applications in resource-constrained SME environments of Kathmandu Valley, Nepal.

## Research Questions
1. What is the CPU, RAM, and latency overhead of enforcing Zero Trust via Linkerd mTLS on a K3s cluster?
2. How effectively does mTLS combined with Kubernetes NetworkPolicy prevent lateral movement?
3. Is Zero Trust enforcement operationally feasible for a Kathmandu SME without a dedicated security engineer?

## Lab Environment
- Cloud: AWS EC2 t3.small (practice) / t2.medium (final thesis)
- OS: Ubuntu 22.04 LTS
- Kubernetes: K3s single-node cluster
- Application: Google Online Boutique (5 microservices)
- Service Mesh: Linkerd
- Metrics: Prometheus + Grafana

## Repository Structure
- configs/ — Kubernetes NetworkPolicy and Linkerd configuration files
- results/ — Benchmark data and screenshots from both scenarios
- scripts/ — Load generation scripts
- docs/ — Tiered adoption roadmap for Kathmandu SMEs

## Experiment Design
- Scenario 1: Baseline — no Zero Trust, lateral movement attack succeeds
- Scenario 2: Zero Trust enforced — same attack blocked

## Status
- [x] Phase 1 — EC2 Setup
- [x] Phase 2 — GitHub Repository
- [ ] Phase 3 — K3s Installation
- [ ] Phase 4 — Microservices Deployment
- [ ] Phase 5 — Prometheus + Grafana
- [ ] Phase 6 — Baseline Experiment
- [ ] Phase 7 — Zero Trust Enforcement
- [ ] Phase 8 — Zero Trust Experiment
- [ ] Phase 9 — Results Comparison
- [ ] Phase 10 — Tiered Roadmap
- [ ] Phase 11 — Final Cleanup
