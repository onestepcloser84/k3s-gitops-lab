# Overview

This repository documents a **local, single-node Kubernetes GitOps lab** built on top of **k3s**, running directly on a desktop server. The lab exists to practice production-grade Kubernetes and GitOps workflows in a controlled, reproducible environment.

While the infrastructure is intentionally small, the **patterns, tooling, and operational mindset** are chosen to closely mirror real-world production clusters.

## Why this lab exists

In day-to-day work, many foundational systems are already in place:

* DNS is managed externally (e.g. Route53)
* Certificates are issued automatically
* GitOps controllers are already installed
* Platform components are maintained by dedicated teams

This lab intentionally **rebuilds those layers from scratch**, locally, in order to:

* Deepen understanding of how the pieces fit together
* Practice designing clean GitOps repository structures
* Gain confidence explaining architectural decisions
* Reduce reliance on assumptions or “black boxes”

The goal is not novelty, it is **fluency**.

## Scope and non-goals

### In scope

* GitOps-driven cluster management
* Declarative platform components (Ingress, DNS, TLS, observability)
* DNS automation using `external-dns`
* Internal certificate management using `cert-manager`
* Operational workflows such as upgrades, rollbacks, and drift correction

### Explicit non-goals

* High availability or multi-node clustering
* Cloud-provider-specific integrations
* Performance benchmarking or load testing
* Production traffic or exposure to the public internet

The lab optimizes for **clarity and correctness**, not scale.

## Design principles

1. **Git is the source of truth**
   All cluster state should be derivable from this repository.

2. **Prefer boring technology**
   Choose tools that are well understood and widely used.

3. **Separation of concerns**
   Host-level services, platform components, and applications are clearly separated.

4. **Inspectability over abstraction**
   Avoid hiding behavior behind excessive automation.

5. **Documentation is part of the system**
   Decisions are recorded alongside manifests.

## High-level architecture

At a high level, the lab consists of three layers:

1. **Host layer**

   * Ubuntu desktop server
   * Docker (for non-Kubernetes infrastructure such as DNS)

2. **Kubernetes platform layer**

   * k3s (single node)
   * Traefik for ingress
   * GitOps controller (Argo CD / Flux)
   * cert-manager
   * external-dns

3. **Application layer**

   * Sample workloads deployed via GitOps
   * Exposed via Ingress
   * Addressed using internal DNS names

## DNS philosophy

Unlike ad-hoc local setups that rely on wildcard DNS services or `/etc/hosts`, this lab uses:

* A real authoritative DNS server (PowerDNS)
* A private DNS zone (`k8s.abhinav.lan`)
* Automated record management via `external-dns`

This closely mirrors enterprise environments and allows DNS to be treated as **declarative infrastructure**, rather than a manual concern.

## How this repo should be read

The files in this repository are meant to be consumed in roughly this order:

1. Purpose, scope, and philosophy
2. DNS and ingress documentation
3. GitOps controller bootstrap
4. Platform services
5. Applications and examples

Each section builds on the previous one.

## Final note

This lab is intentionally iterative. The structure and tooling may evolve over time, but changes should be **intentional, documented, and reproducible**.

If something feels overly complex for a local setup, it is worth re-evaluating — the lab exists to build understanding, not to impress with complexity.
