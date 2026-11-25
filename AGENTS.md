## Agent skills

### Issue tracker

Issues and specs are tracked in GitHub Issues for `vlesierse/lesierse-homelab`. See `docs/agents/issue-tracker.md`.

### Domain docs

This is a single-context repository. See `docs/agents/domain.md`.

### Architecture documentation

`docs/architecture.md` is the shareable, sanitized view of the homelab's current architecture and recovery boundaries. Keep it aligned with observed reality: distinguish deployed facts from intended design and unknowns, and record the observation date for facts that can drift.

When a change affects workload ownership, topology, dependencies, data locations, access, deployment, or backup/recovery paths, update `docs/architecture.md` in the same change. Review it during inventory, adoption, and recovery work even if no update is needed. Never put secrets, credentials, private keys, personal data, or sensitive account/resource identifiers in this document.
