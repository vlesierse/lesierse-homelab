# Adopt existing workloads with separate tool ownership

The homelab prioritizes reproducibility and recovery over high availability. Use OpenTofu for infrastructure resources and Ansible for host and workload configuration, assigning each resource or setting a single owner; preserve the existing LXC and Docker arrangements rather than introducing Kubernetes or replatforming during adoption.

## Consequences

- Preserve existing services and data. Classify dormant containers and cloud resources before adopting them; do not delete, migrate, or upgrade them implicitly.
- Manage homelab-specific AWS and Entra resources, service DNS, host/container networking, and Tailscale where supported. The Entra tenant, primary identity, ISP connection, physical network, and initial Proxmox installation are prerequisites; UniFi management is out of scope.
- Git defines desired state. Initially, GitHub Actions performs checks and an administrator explicitly applies from a laptop. Review adoption plans for unintended changes before applying; reconcile emergency manual changes into code afterward.
- Keep newly managed services private unless public exposure is explicitly approved. Preserve and document existing exposure during adoption rather than silently changing it.
