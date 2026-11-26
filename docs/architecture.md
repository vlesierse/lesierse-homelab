# Homelab architecture

This document is the shareable overview of the homelab's observed state and intended direction. It is not a deployment specification: detailed configuration belongs in code, domain language in `GLOSSARY.md`, and durable architectural choices in `docs/adr/`.

## How to read this document

- **Observed** means confirmed by bounded read-only discovery on 2026-11-25 or on the later date specified below. Mutable facts must be refreshed during the next inventory.
- **Intended** means an agreed design direction, not something already deployed or proven.
- **Unknown** means the available discovery did not establish the fact. Do not infer a dependency or capability from proximity in a diagram.
- Workload names are shown as useful high-level labels. Hostnames, addresses, storage identifiers, cloud account/resource identifiers, DNS records, and secret values are intentionally omitted to keep this overview safe to share.

## Observed system

```mermaid
flowchart TB
    subgraph Homelab["Home homelab — observed"]
        PVE["Single Proxmox host"]
        subgraph Guests["LXC guests"]
            Share["100 · share — running"]
            Servarr["101 · servarr — running"]
            Vaultwarden["102 · vaultwarden — running"]
            Hermes["110 · hermes — running"]
            Jellyfin["201 · jellyfin — running"]
            Minecraft["202 · minecraft — running"]
            Runner["500 · GitHub runner — running"]
            Printer["210 · printer — stopped"]
            Plaiteo["501 · plaiteo runner — stopped"]
        end
        PBS["Proxmox Backup Server services\nrun on the Proxmox host"]
        SharedData["Large shared data filesystem\nmounted by share, servarr, and Jellyfin"]
        Datastore["PBS datastore"]
        LocalDisk["Host filesystem backing\nshared data and PBS datastore"]
        PVE --- Guests
        PVE --- PBS
        Share --- SharedData
        Servarr --- SharedData
        Jellyfin --- SharedData
        PBS --- Datastore
        SharedData --- LocalDisk
        Datastore --- LocalDisk
    end

    subgraph Cloud["Cloud — bounded discovery"]
        AWS["AWS · eu-west-1\nS3 resource observed;\nno Lambda functions observed"]
        Entra["Microsoft Entra\none application registration observed;\nclassified out of scope"]
    end
```

The diagram groups related components; it does not assert network connectivity or service-to-service dependencies beyond the shared filesystem mounts and storage relationships shown.

### Observed inventory

| Area | Observed state | Not established |
|---|---|---|
| Proxmox | One host; nine LXC guests: seven running and two stopped, as shown above. | Guest configuration ownership, external dependencies, and whether stopped guests remain in scope. |
| Guest services | Share provides file-sharing services; servarr runs Docker-managed media-management services; Vaultwarden, Hermes dashboard, Jellyfin, Minecraft controller, and GitHub runner services were observed. | Full service contracts, data consistency requirements, exposure, and recovery priority for each workload. |
| Storage | Share, servarr, and Jellyfin each have a Proxmox-managed mount of the same shared volume; none is marked for backup. Its backing file and the PBS datastore reside on the same host filesystem on a single disk; that filesystem reports approximately 3.5 TiB used, including the datastore. Guest root filesystems reside on a ZFS mirror. | Protected-data size and change rate, exact shared-data classification, and whether other storage or off-site copies exist. The filesystem usage is not the size of protected data. |
| Local backups | PBS 4.2.14-1 services run on the Proxmox host. A daily 01:00 snapshot job targets PBS and currently excludes runner guests 500 and 501. Its ten most recent runs report success, but only the latest included Vaultwarden; the preceding nine included the runners instead. The latest job completed backups of the other seven guests, but explicitly excluded the shared volume from all three guests that mount it. The PBS prune job is configured for keep-last 3, keep-daily 3, and keep-weekly 1; no PBS verification job is configured. A retained container-restore task reported success on 2026-06-29. No PVE-managed client encryption key file was found in the default location for this storage. | The observed datastore's root namespace lists only one Vaultwarden snapshot dated in 2026, following three dated in 2025. Snapshot entries and job success do not prove recoverability or rule out other copies. The historical restore's source and application usability are unverified. Application consistency, snapshot encryption, other key arrangements, notifications, and recent successful restore evidence remain unknown. The excluded shared volume is not covered by this guest-backup job. Same-host PBS on the same disk does not establish protection from host loss. |
| Network access | Tailscale was observed on the host and some guests. All seven running guests had at least one TCP listener bound to a wildcard address. On 2026-11-26 the Proxmox firewall reported `disabled/running`; four guests had a Tailscale interface and listed its DNS resolver alongside another private or loopback resolver. The owner reports that access is mainly through Tailscale and that no workload is intentionally public. | Wildcard listeners and the Proxmox firewall status alone do not establish internet exposure or the state of other firewalls. Full membership, access paths, DNS resolution, upstream firewall and forwarding rules, independently verified exposure, and which services depend on external network access remain unknown. |
| AWS | Bounded read-only discovery in `eu-west-1` found one S3 bucket and, after the owner removed a previously observed Lambda, no Lambda functions. The owner classified the existing bucket and former Lambda as out of scope. | Other service inventory, account recovery dependencies, and any homelab-owned resources not yet identified. The owner identified only this region and these services for initial discovery. |
| Entra | One application registration and service principals were observed in the requested tenant. The owner classified that registration as out of scope. An earlier listing was from a different tenant and is not evidence about the requested tenant. | Whether other enterprise applications or assignments are homelab-owned or required for recovery. |

These are inventory observations, not proof that backups are restorable or that recovery objectives are currently met. No credentials, environment values, detailed guest configuration, or application data are documented here.

### Protected-data inventory in progress

| Dataset | Current evidence and classification | Remaining gaps |
|---|---|---|
| Home videos | The owner confirmed the entire identified directory on the shared volume should be protected. A read-only measurement on 2026-11-25 found approximately 52.0 GiB allocated in that directory. The guest-backup job excludes the volume. | Measured change rate, consistency, and exact off-site sizing remain unknown. |
| Future photos | The owner identified a planned location on the shared volume; it did not exist on 2026-11-25. Photos are protected data when present. | Expected size, growth, and coverage when the location is created. |
| Vaultwarden | Credentials are an essential recovery concern. The owner reports configuration is on the guest disk. The working directory containing its standard database used approximately 116 KiB on 2026-11-25; the default attachment and Send directories beside it were absent. On 2026-11-26 the running process had that database and its WAL and shared-memory files open. The latest guest-backup job included its root filesystem, but the preceding nine jobs did not include this guest. No additional Proxmox-managed mount or NFS/CIFS/SSHFS mount was observed in this guest. | Verify any custom database, attachment, or Send locations, consistency requirements, and recovery procedure; open files and a guest snapshot alone do not establish complete or application-consistent coverage or rule out other external storage. |
| Minecraft | The owner confirmed all worlds and game state should be protected, even though the workload is best-effort. The guest root filesystem is included in the guest-backup job. Four candidate world directories, using approximately 11.9 MiB in total, were found under the controller installation on 2026-11-25; the entire installation used approximately 538 MiB. No additional Proxmox-managed mount or NFS/CIFS/SSHFS mount was observed in this guest. On 2026-11-26 a Bedrock server was running, with one candidate world directory under its working directory and three open file descriptors beneath that world; no process named `java` was observed. | Verify all world and configuration locations, consistency requirements, and recovery procedure; marker directories may include copies, and the measurements may not cover other external locations. |
| Servarr | The owner wants configuration and library metadata protected. The guest root filesystem is included in the guest-backup job, while the shared volume is excluded. Inspected Docker configuration bind mounts use a directory on the guest root filesystem, which used approximately 363 MiB on 2026-11-25. On 2026-11-26 database-like files and SQLite WAL/shared-memory sidecars were present for several media-management services. | Verify all application data locations, consistency requirements, and which metadata resides on the shared volume; the directory measurement may include non-protected data, and sidecar presence alone does not establish a consistent backup. |
| Jellyfin and related statistics | The owner identified catalog, user, and viewing-statistics data as protected. The running Jellyfin service uses a working directory on its guest root filesystem, which used approximately 1.5 GiB on 2026-11-25. On 2026-11-26 it had a database-like file and its WAL and shared-memory files open in that directory; a separate standard configuration directory was also present. A separate Jellystat service and PostgreSQL container were observed in servarr, with a database bind directory on its guest root filesystem. On 2026-11-26 the running database container's `postgres` and `pg_dump` utilities both reported version 15.2. Both root filesystems are included in the guest-backup job. | Verify which databases contain the required data, their full locations, consistency requirements, and recovery procedure; directory sizes do not establish protected-data size and utility availability does not prove a usable dump. |
| Hermes dashboard | The owner identified irreplaceable state or recovery-critical configuration. The running dashboard uses a working directory on its guest root filesystem, which used approximately 8.0 GiB on 2026-11-25. Two conventional cache directories under it used approximately 2.16 GiB on 2026-11-26. The root filesystem is included in the guest-backup job. | Identify the protected data and its actual location, size, consistency requirements, and recovery procedure; the working-directory measurement includes other data. |

The owner reports no existing off-site homelab copies or backup plan, despite the configured local job. Laptop and other personal-device backups are out of scope and handled separately; that does not establish coverage for data stored only on the homelab. Downloaded films, TV, and music can be replaced; the owner also excludes temporary downloads. These exclusions do not apply to home videos or future photos. No broader exclusion is implied. The owner estimates approximately 2 GB of new or changed protected data per month. The measurements above are not measurements of all protected data or its change rate; retained-history, request, and restore costs cannot yet be established.

The owner reports using an independent Bitwarden service for access credentials, rather than the Vaultwarden guest, and being able to access and recover it without Entra or the homelab. This has not been tested as part of a recovery exercise. A complete independently held recovery kit has not been established.

The owner reports no irreplaceable state or recovery-critical configuration for the printer and runners. The stopped guests remain untouched.

## Intended management and change flow

```mermaid
flowchart LR
    Repo["Git repository\ncode + architecture overview"]
    Actions["GitHub Actions\nchecks only, initially"]
    Laptop["Administrator laptop\nreviewed explicit plan/apply"]
    Tofu["OpenTofu\ninfrastructure resources"]
    Ansible["Ansible\nhost and workload configuration"]
    Proxmox["Proxmox and LXC workloads"]
    AWS["Classified homelab AWS resources"]
    Entra["Classified homelab Entra integrations"]
    State["Dedicated encrypted,\nversioned, locked S3 state"]
    Bootstrap["Separate backend bootstrap"]

    Repo --> Actions
    Repo --> Laptop
    Laptop --> Tofu
    Laptop --> Ansible
    Tofu --> Proxmox
    Tofu --> AWS
    Tofu --> Entra
    Tofu --> State
    Bootstrap --> State
    Ansible --> Proxmox
```

This is the agreed target, not a deployed implementation. Use one owner per resource or setting: OpenTofu for infrastructure resources and Ansible for host/workload configuration. Initially, applies are explicit from a laptop; the GitHub runner is not a bootstrap prerequisite. Adopt existing resources only after classification and reviewed plans show no unintended changes. UniFi management, physical-network redesign, tenant-wide identity management, and automatic failover are out of scope.

## Intended recovery boundary

```mermaid
flowchart LR
    Data["Guest backups + classified protected data\n(application-consistent exports\nand shared/bind-mounted data)"]
    Encrypt["PBS client-side encryption"]
    Local["Local PBS datastore\nshort retention"]
    Offsite["S3-backed PBS datastore\nlonger retention"]
    Kit["Independent recovery kit\nkeys + emergency access + instructions"]
    Code["Versioned configuration in Git"]
    State["S3 infrastructure state\nseparate from workload stacks"]
    Restore["Fresh Proxmox + PBS\nisolated restore and verification"]

    Data --> Encrypt --> Local --> Offsite --> Restore
    Kit --> Encrypt
    Kit --> Restore
    Code --> Restore
    State --> Restore
```

The owner wants short-lived local PBS backups for fast recovery and an incremental PBS copy in S3. After complete host loss, the intended path is to install Proxmox and PBS afresh, reconnect the S3-backed datastore, and restore the selected guests and protected data using independently held keys and access. A PBS snapshot in S3 comprises shared chunks and metadata, not one standalone file.

The local retention target is three days. The owner confirmed the longer off-site target remains 7 daily / 4 weekly / 3 monthly recovery points. Local pruning must not remove the longer off-site history. Exact protected datasets, client encryption, independent deletion protection, costs, and the fresh-install restore remain subject to validation; this path is not yet deployed. Recovery must not depend on the lost Proxmox host, its runner, Vaultwarden, or Entra sign-in.

A disposable test on 2026-11-26 synced two encrypted synthetic snapshots from a local PBS datastore to an S3-backed PBS datastore. Both remained off site after one was deleted locally and could be restored after reconnecting with a new cache. The test resources were removed. This used the existing PBS installation, not a fresh installation, and did not establish production coverage, retention schedules, deletion protection, key recovery, or application usability.

The other initial objectives are essential-workload recovery within 24 hours after replacement hardware and internet are available and no more than 24 hours of data loss. Additional recurring cost is capped at EUR 25/month without approval. These are design objectives, not verified capabilities or a high-availability SLA.

The [off-site recovery assessment](recovery-assessment.md) evaluates candidate mechanisms and a conditional cost scenario; it does not describe a deployed backup path.

## Keeping this overview current

Update this document in the same change whenever workload ownership, topology, dependencies, data locations, access, deployment, or backup/recovery paths change. During inventory and adoption, compare the diagram with the observed environment; distinguish observed state from intended state and leave unverified edges out. Refresh dated facts that may have drifted. Do not add secrets, credentials, private keys, personal data, or sensitive account/resource identifiers.
