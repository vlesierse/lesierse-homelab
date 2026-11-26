# Off-site recovery assessment

Assessment date: 2026-11-25. This is an evaluation of a **new** recovery
path, not a deployed system or a recovery procedure. No off-site homelab copies
have been established. The [architecture overview](architecture.md) records the
current inventory, local backup gaps, and remaining dataset unknowns.

## Requirements and sizing

- Protect the entire identified home-video directory, future photos,
  Vaultwarden data, all Minecraft worlds and state, servarr configuration and
  library metadata, Jellyfin and related statistics, and the dashboard's
  irreplaceable state or recovery-critical configuration. Identify any other
  non-reproducible data before declaring coverage complete.
- Exclude replaceable downloaded films, TV, and music, caches, and temporary
  downloads; never exclude home videos or future photos merely because they
  are media.
- Laptop and other personal-device backups are outside this work; the owner
  reports a separate device-backup arrangement. That does not establish
  coverage of data stored only on the homelab.
- Target three days of local PBS retention and keep 7 daily, 4 weekly, and 3
  monthly off-site recovery points. A successful job by itself is not a usable
  recovery point.
- Protect backups from complete host loss, accidental deletion, and compromise
  of routine deployment credentials. The backup deletion authority and
  emergency access must be independent of routine deployment.
- The objectives are at most 24 hours of data loss and usable essential
  workloads within 24 hours after hardware and internet are available.
  Neither objective is proven.

Read-only measurements on 2026-11-25 found approximately 52.0 GiB allocated
in the identified video directory and 23.0 GiB used across the root
filesystems of the six running guests containing the identified workloads.
Their combined 75.0 GiB (80.53 GB) is an **illustrative input**, not the
protected-data total: guest roots include software and caches, other data
locations may be missing, and filesystem usage is not necessarily transferred
or stored size. The owner estimates about 2 GB of new or changed protected
data per month. Initial photo size, guest-root churn, compression,
deduplication, and retention overhead are unknown.

The latest available local PBS garbage-collection report recorded about
19,700 chunks occupying 33.0 GB; the report's timestamp was not established.
It is not a measurement of the proposed S3 datastore or its object versions.
The latest observed guest job excluded the shared volume.

The owner reports a 1 Gbps internet connection; sustained upload and download
rates have not been measured. Transferring 86.53 GB in 24 hours alone requires
about 8 Mbps continuously, before allowing time for provisioning, database
restoration, application validation, retries, or protocol overhead. This
calculation does not prove the recovery objective.

## Preferred PBS path

The owner prefers local PBS snapshots followed by an incremental S3 copy and,
after host loss, a fresh Proxmox and PBS installation that reconnects the
S3-backed datastore. The installed PBS version, 4.2.14-1, supports S3-backed
datastores. PBS stores deduplicated chunks and snapshot metadata in S3; each
snapshot references all chunks required for that recovery point. It is not a
single S3 object, and the whole datastore cannot be treated as an ordinary
file copy.

The proposed path is to encrypt backups at the PVE or PBS client, write them
to the local datastore, and sync the selected encrypted snapshots to a
separate S3-backed datastore. The exact sync configuration still needs
validation. Do not propagate locally vanished snapshots to the off-site
datastore: prune the latter independently for its longer history. Ensure
snapshots reach S3 before local pruning, and alert if expected backups or
syncs are missing. An encrypted-only filter skips unencrypted snapshots; it
must not silently turn an encryption or coverage mistake into an apparently
successful backup.

Guest snapshots can include whole root filesystems, not just the identified
protected directories. Select guest groups and exclude replaceable data
deliberately; confirm how to exclude caches without excluding protected data.
The shared volume is not included in the observed guest backups and needs a
separate, selectively scoped PBS client backup or another proven PBS-compatible
approach. Recreating the host also requires host configuration from code and
any independently held recovery prerequisites.

The S3-backed datastore requires a persistent local cache; Proxmox recommends
64-128 GiB. A fresh PBS installation can reuse the S3 datastore with the same
datastore name and a new cache, but only one PBS instance may operate on it at
a time. Recovery must establish that the former instance cannot access it
before taking over. The recovery kit needs the PBS encryption key, S3 access,
datastore details, and instructions independently of the lost host and Entra.
On 2026-11-25 the host root filesystem had approximately 45.2 GiB available;
the existing local PBS backing filesystem had approximately 3.4 TiB
available. Cache placement, its capacity limit, and competition with other
data need review. The existing S3 bucket and Entra registration are out of
scope.

The installed PBS S3 endpoint command requires an access key and secret key
and exposes no session-token option. Short-lived AWS authentication for this
backend has not been established. Any credential approach needs narrowly
scoped permissions, rotation, and independent recovery; do not put secrets in
Git, command history, or process arguments.

## Independent deletion protection remains open

The PBS S3 backend requires object deletion for maintenance. Proxmox staff
stated that Object Lock was not part of PBS's S3 implementation; the 4.2
documentation does not establish lock-aware maintenance. A fixed object
retention period alone is insufficient because an old chunk may still be
referenced by a new snapshot. A delete marker or newer object version can also
hide a locked version. Do not assume that enabling Object Lock or S3
versioning by itself protects a usable PBS recovery point.

An independently administered AWS Backup vault is one possible protection to
evaluate, not a selected or fully costed solution. AWS Backup for S3 requires
versioning and makes subsequent backups incrementally, but its compatibility
with a live PBS datastore, a consistent restore of all required chunks and
metadata, authority separation, and costs are unproven. It also charges
minimum 128 KB per backed-up S3 object, making object counts relevant.
Independently recoverable keys and permissions remain necessary. Do not
configure lifecycle expiration or lock policies until their interaction with
PBS retention and maintenance has been demonstrated safely.

For application consistency, evaluate SQLite's online backup API for the
Vaultwarden database and PostgreSQL's logical backup tools for the observed
statistics database. The vault guest did not have the `sqlite3` executable
available during discovery. Identify and validate the actual databases,
attachments, configuration, and Minecraft save/quiesce mechanism first. Avoid
executing backup actions stored in a remotely writable repository; any local
hooks must be independently controlled, fail explicitly, and be restore-tested.

## Conditional S3 cost scenario

This example uses AWS public on-demand prices observed on 2026-11-25 for
`eu-west-1`, S3 Standard, and ordinary internet transfer. It is not a quote,
budget approval, or evidence that the retention and recovery requirements work.
GB here means decimal gigabytes.

Assume the illustrative 80.53 GB guest-root and video baseline, 2 GB/month of
retained changes for three months, 20% additional billed storage for metadata
and maintenance, and incremental deduplication effective enough for those
assumptions to hold. The resulting 103.84 GB billed storage is not an upper
bound. Assume 100,000 tier-1 and 500,000 tier-2 requests per month and one
86.53 GB internet restore per quarter. Charge all restore traffic at the paid
rate without relying on shared free allowances:

| Component | Assumption | Estimated USD/month |
|---|---|---:|
| Storage | 103.84 GB at $0.023/GB-month | 2.39 |
| Requests | $0.005/1,000 tier-1 and $0.004/10,000 tier-2 | 0.70 |
| Restore exercise transfer | 86.53 GB at $0.09/GB, once per quarter | 2.60 |
| **Illustrative subtotal** | Sum of rounded line items | **5.69** |

At the ECB reference rate of USD 1.0935 per EUR on 2026-11-25, the subtotal
is approximately EUR 5.20/month. It excludes taxes, billing exchange-rate
differences, the separate state backend, alerting, any compute, deletion
protection, additional requests, recovery hardware, and unmeasured data or
retention/maintenance overhead. Consequently the EUR 25/month ceiling is
**not yet validated**. Actual cost and growth must be measured and monitored;
escalate a conflict rather than weakening coverage or retention.

For sensitivity only, adding a hypothetical 103.84 GB AWS Backup vault copy
at the observed `eu-west-1` warm-storage price of $0.05/GB-month adds about
$5.19/month. Restoring 86.53 GB from that vault once per quarter at
$0.02/GB adds about $0.58/month. Together with the subtotal above, this is
about $11.46 or EUR 10.48/month before the exclusions above and the vault's
minimum object size, source versions, additional S3 requests, events, and
restore requests. This does not establish that the vault approach works with
PBS or fits the ceiling.

## Decision and production gates

Before finalizing the PBS path, identify the active data and attachment
locations, including dashboard state and every Minecraft world. Evaluate the
application-consistency, retention, independent-deletion, key-recovery, and
fresh-install restore approach against those locations and the cost ceiling.
Record any remaining assumptions rather than treating this estimate as proof.

Before production use, verify retention, maintenance, and recovery after
deletion or compromise of a routine credential with synthetic data and
separate identities. Do not test destructive operations against live backups.
A full recovery exercise must then restore into an isolated environment using
independent keys and access, verify application usability, and measure data
age and elapsed time. Agree on a homelab-independent failure/overdue alert
path and measure actual billed storage, requests, state-backend costs, and
alert costs against the ceiling.

Live cloud or workload changes require explicit authorization after a
reviewed plan. Until these checks are complete, neither an off-site backup nor
the recovery objectives should be described as implemented.

## Sources

- Proxmox PBS documentation for [S3-backed datastores](https://pbs.proxmox.com/docs/storage.html#datastores-with-s3-backend),
  [sync jobs](https://pbs.proxmox.com/docs/managing-remotes.html#sync-jobs),
  [pruning and garbage collection](https://pbs.proxmox.com/docs/maintenance.html),
  and [incremental, deduplicated snapshots](https://pbs.proxmox.com/docs/technical-overview.html#snapshots).
- [Proxmox staff on Object Lock](https://forum.proxmox.com/threads/immutable-backups-on-s3-object-storage.169404/).
- [AWS Backup for S3](https://docs.aws.amazon.com/aws-backup/latest/devguide/s3-backups.html),
  [vault locks](https://docs.aws.amazon.com/aws-backup/latest/devguide/vault-lock.html),
  and [AWS Backup pricing](https://aws.amazon.com/backup/pricing/).
- [AWS S3 pricing](https://aws.amazon.com/s3/pricing/),
  [Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html),
  [versioning](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Versioning.html),
  and [lifecycle management](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html).
  Prices were also checked using the AWS Pricing API for S3 and data transfer.
- [ECB reference rates](https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html),
  [SQLite online backup](https://www.sqlite.org/backup.html), and
  [PostgreSQL SQL dumps](https://www.postgresql.org/docs/current/backup-dump.html).
