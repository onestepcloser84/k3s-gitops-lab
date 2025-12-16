# DNS with PowerDNS

This document describes the DNS architecture used in this lab, and the rationale behind choosing **PowerDNS + external-dns** instead of ad-hoc or wildcard-based DNS solutions.

DNS is treated as **first-class infrastructure**, managed declaratively and integrated into the GitOps workflow.

## Problem statement

Local Kubernetes labs often solve DNS in one of the following ways:

* `/etc/hosts` entries
* wildcard services like `nip.io` / `sslip.io`
* skipping DNS entirely and using NodePorts

While these approaches are convenient, they differ significantly from how DNS is handled in real production environments. In most professional setups:

* DNS is authoritative and centrally managed
* Records are created automatically
* Applications rely on stable, human-readable hostnames

This lab intentionally avoids shortcuts in order to practice **production-like DNS workflows**.

## Chosen approach

The lab uses:

* **PowerDNS Authoritative Server** running via Docker on the host
* A private DNS zone: `k8s.abhinav.lan`
* **external-dns** running inside Kubernetes
* DNS records created automatically from Kubernetes resources

This mirrors common enterprise patterns such as:

* Route53 + external-dns
* Cloud DNS + GitOps-managed ingress

## Why PowerDNS

PowerDNS was chosen because:

* It is a real, authoritative DNS server
* It has a stable and well-documented HTTP API
* It is natively supported by `external-dns`
* It supports SQL backends for easy inspection and backup

PowerDNS behaves similarly to managed DNS services, making it ideal for GitOps learning.

## Why external-dns

`external-dns` watches Kubernetes resources and reconciles DNS records automatically.

In this lab it:

* Watches `Ingress` resources (and optionally `Service` resources)
* Creates and updates DNS records in PowerDNS
* Deletes records when resources are removed

This allows DNS to be managed **indirectly via Kubernetes manifests**, instead of manual zone edits.

## DNS zone design

* Zone name: `k8s.abhinav.lan`
* Scope: private, internal-only
* Records are created dynamically per application

Example records:

```text
whoami.k8s.abhinav.lan    A   192.168.0.10
argocd.k8s.abhinav.lan    A   192.168.0.10
grafana.k8s.abhinav.lan   A   192.168.0.10
```

All records typically resolve to the Traefik ingress endpoint.

## Networking considerations

### Port usage

* PowerDNS listens on `192.168.0.10:53` (TCP/UDP)
* `systemd-resolved` continues to listen on `127.0.0.53`
* There is no port conflict because bindings are on different interfaces

### Client configuration

* Client devices are configured to use `192.168.0.10` as their DNS server
* Fallback resolvers still be configured for public DNS

## Security considerations

* PowerDNS API is protected using an API key
* The API is only reachable from trusted networks
* DNS is not exposed to the public internet

Secrets such as API keys are **never committed to Git**.

## Limitations

* This setup does not support public DNS or ACME challenges
* TLS certificates must be issued using an internal CA
* The `.lan` TLD is non-standard but widely supported for private networks

These limitations are acceptable and intentional for a local lab.

## Expected outcomes

With this setup in place:

* DNS records are created automatically when applications are deployed
* Ingress hostnames are stable and predictable
* DNS behaves similarly to production environments
* DNS changes are driven by GitOps workflows

## Next steps

Once DNS is operational:

1. Deploy `external-dns` via GitOps
2. Verify automatic record creation from Ingress resources
3. Add `cert-manager` with an internal CA
4. Expose applications securely using HTTPS

DNS is the foundation on which the rest of the platform is built.
