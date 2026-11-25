# Keep bootstrap independent of the homelab

Recovery must work when the Proxmox host, its runners, Vaultwarden, or Entra sign-in is unavailable. Accept a documented administrator-laptop bootstrap and independently held recovery kit rather than requiring an always-running deployment or secrets platform.

## Consequences

- Store OpenTofu state outside Proxmox in a dedicated encrypted, versioned S3 backend with locking. Keep its bootstrap in code, provide an independently protected state-recovery path, and do not let an ordinary workload stack own its backend.
- Use SOPS with age for configuration secrets stored in Git. Keep plaintext secrets, private decryption keys, and infrastructure state out of Git; protect state and backups separately.
- Hold the decryption key, emergency credentials, and recovery instructions independently of the services they recover. Use short-lived cloud credentials where supported.
- Use Entra for normal human sign-in where supported, while retaining independently protected emergency administrative access. Neither federated login nor the password vault may be a prerequisite for its own recovery.
