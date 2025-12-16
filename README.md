# k3s-gitops-lab

This repo contains the documentation and configuration for my **local GitOps lab** running on a **single-node k3s cluster**.

The goal is to practice a production-like workflow locally:
- Git as the source of truth
- GitOps controller (Mostly Argo CD, Flux if need be)
- Ingress-based access patterns
- DNS automation via `external-dns` (PowerDNS)
- TLS via `cert-manager` (internal CA for lab)

## Lab environment

### Kubernetes
- Distribution: **k3s**
- Install command:
    ```bash
    curl -sfL https://get.k3s.io | sh -s - --docker
    ```
- Default k3s components: `Traefik`, `CoreDNS`, `metrics-server`, `local-path` provisioner

### DNS Design
- Authoritative DNS: PowerDNS (running via Docker on the host)
- DNS zone: `k8s.abhinav.lan`
- DNS records are managed automatically by external-dns via the PowerDNS API
- Client devices use 192.168.0.10 as their DNS server

This setup mirrors how production environments typically work.

### GitOps philosophy used here
- Manual `kubectl apply` is avoided as much as possible
- Cluster bootstrap is declarative
- Changes flow: Git → controller → cluster

### Principles
- Prefer boring, well-understood components
- Avoid unnecessary abstractions
- Everything should be explainable and debuggable
- Documentation is as important as manifests

### Notes
- This is intentionally a single-node cluster.
- Some production concerns (HA, multi-AZ, autoscaling) are out of scope by design.
- The focus is on correctness, structure, and operational habits that transfer directly to real-world Kubernetes environments.
