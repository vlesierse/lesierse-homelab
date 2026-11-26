# Off-site recovery assessment

Assessment date: 2026-11-26. This is an evaluation of a **new** recovery
path, not a deployed system or a recovery procedure. No off-site homelab copies
have been established. The [architecture overview](architecture.md) records the
current inventory, local backup gaps, and remaining dataset unknowns.

As of 2026-11-26, the owner narrowed the active work on issue #2 to costs and
isolated recovery testing. Remaining ownership, classification, dependency,
and workload-coverage inventory is deferred to [backlog issue
#6](https://github.com/vlesierse/lesierse-homelab/issues/6). The candidate
datasets and gaps below are cost-model context and known limitations, not a
claim that the deferred inventory is complete.

## Cost-model inputs and recovery-test scope

- The conditional cost scenario below uses the currently documented video
  directory and whole guest roots as sizing inputs. These are not a validated
  selection of all protected data; unknown datasets, incomplete workload
  classification, and changing data are deferred to #6.
- Exclude replaceable downloaded films, TV, and music, caches, and temporary
  downloads from the illustrative scenario; these assumptions do not establish
  final backup exclusions.
- Laptop and other personal-device backups are outside this work; the owner
  reports a separate device-backup arrangement. That does not establish
  coverage of data stored only on the homelab.
- Target three days of local PBS retention and keep 7 daily, 4 weekly, and 3
  monthly off-site recovery points. A successful job by itself is not a usable
  recovery point.
- Protect backups from complete host loss, accidental deletion, and compromise
  of routine deployment credentials. The backup deletion authority and
  emergency access must be independent of routine deployment.
- Use isolated, non-destructive tests with synthetic data or existing approved
  test artifacts. Do not modify production backups, services, or cloud
  resources without separate approval.
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

On 2026-11-26, the latest guest-root archive manifests in the observed
datastore's root namespace for all nine guests reported PBS client-encryption
mode `none`: seven snapshots were dated 2026-11-26 and the two excluded
runners' snapshots were dated 2026-11-24. Three sampled Vaultwarden archive
chunk headers also reported `none`. Encryption of other archives and
snapshots, or at the disk layer, remains unverified.

For the six running workload guest-root archives, a read-only comparison of
the 2026-11-25 and 2026-11-26 PBS indexes found 270 distinct chunks referenced
by the later archives but absent from all six earlier archives. The local
encoded chunk files totalled 496,870,477 bytes (473.9 MiB). This is a roughly
24-hour reference-set difference, not necessarily newly written bytes: chunks
may already occur in older snapshots or other groups. It includes whole guest
roots, excludes the shared volume, and does not measure protected-data churn,
monthly growth, retained history, or S3 costs.

The owner reports a 1 Gbps internet connection; sustained upload and download
rates have not been measured. Transferring 86.53 GB in 24 hours alone requires
about 8 Mbps continuously, before allowing time for provisioning, database
restoration, application validation, retries, or protocol overhead. This
calculation does not prove the recovery objective.

## Known locations and coverage gaps (backlog context)

These are observed or owner-identified locations, not a complete backup
selection. Paths are inside the named guests; bounded discovery examined
filesystem metadata, selected configuration values, and database schema names
without publishing private filenames, application records, or secrets.
Resolving the remaining location and coverage questions is deferred to #6;
this context informs the cost estimate and does not expand the active work.

| Dataset | Candidate location |
|---|---|
| Home videos | The entire `/data/media/videos` directory in share, on the shared volume. |
| Future photos | `/data/media/photos` in share, on the shared volume; absent on 2026-11-26. |
| Vaultwarden | `/var/lib/vaultwarden/db.sqlite3` and related files on the guest root filesystem; a standard-named RSA PEM file was also present. The standard admin configuration file was absent, with no config-file or dotenv override observed in the initial process environment. Custom attachment, Send, and database locations and independent key recovery remain unverified. |
| Minecraft | World markers under `/opt/crafty-controller` on the guest root filesystem; verify all worlds and any additional data. |
| Servarr | Docker configuration bind directories under `/app` on its guest root filesystem; check the shared `/data` mount for protected metadata. |
| Jellyfin and statistics | `/var/lib/jellyfin` and the standard `/etc/jellyfin` configuration directory on the Jellyfin guest; PostgreSQL bind directory `/app/jellystat/postgres` in servarr. Verify the effective configuration and all data locations. |
| Hermes dashboard | The service works from `/home/hermes` on the guest root filesystem. Three active SQLite databases and their sidecars were observed under `/home/hermes/.hermes`; one has table names containing generic configuration, link, session, and event terms. That directory used approximately 5.68 GiB and includes cache-named data. Complete protected state remains unverified. |

A same-filesystem scan of the shared volume outside the selected video and
photo directories found no NFO or database-like filenames among 1,206 regular
files on 2026-11-26. It did not follow symlinks or cross filesystem boundaries,
and does not rule out metadata in other formats or locations.

On 2026-11-26 the running Vaultwarden and Jellyfin processes each had a
database file and its WAL and shared-memory files open. The dashboard service
had three SQLite databases and their sidecars open. Copying only a main
database file would not establish a consistent backup. Read-only schema
inspection identified table-name categories for the dashboard and Jellyfin,
but ignored uncheckpointed WAL changes and did not verify complete content.
In the media-management guest, database-like files and SQLite WAL/shared-memory
sidecars were present for Bazarr, Lidarr, Prowlarr, Radarr, and Sonarr; their
presence alone does not prove current activity or backup consistency. The
running Jellystat PostgreSQL container's `postgres` and `pg_dump` utilities
both reported version 15.2; no dump was attempted. A Bedrock server was running
in the Minecraft guest, with one candidate world directory under its working
directory and three open file descriptors beneath that directory. Four world
markers were found in the controller's server tree; a same-filesystem scan
found none elsewhere on the guest root. A later three-marker size result used
whitespace-sensitive path parsing and suppressed errors; repeating the parsing
against current paths omitted the running world and reproduced the earlier
size. No process named `java` was observed; these observations do not establish
where every world or its configuration is stored.

On 2026-11-26, a streamed reconstruction and listing of the latest Vaultwarden
guest-root archive found entries for its standard database, its WAL and
shared-memory files, and the standard-named RSA PEM file. No files were written
to a restore destination; the listing does not prove a consistent, usable
restore or independent key recovery.

On the same date, the latest guest-root backup catalogs listed the three open
dashboard databases and their six sidecars, the open Jellyfin database and its
two sidecars, all four candidate Minecraft world markers, and all five
media-management configuration bind directories. The servarr catalog also
listed the statistics database's data directory, standard version marker, and
base directory. A later scan of regular files matching database-name patterns
in the five configuration directories matched their catalog entries; the scan
did not follow symlinks or cross filesystems.

Earlier scans also found two Sonarr database-like paths and a Lidarr WAL and
shared-memory pair that were not in the latest catalog; those paths were no
longer observed in the later scan. The Lidarr pair had modification times
after the snapshot, but its existence at snapshot time is unknown. These
changing filenames, catalog entries, and markers do not establish complete
coverage or application consistency.

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
separate S3-backed datastore. A narrow local-to-S3 sync configuration passed
the disposable pilot below; production coverage and scheduling still need
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
any independently held recovery prerequisites. Proxmox documents exclusion
patterns for container backups, while the PBS client skips nested mount points
unless explicitly included; the actual selections and future photo-directory
coverage must be verified rather than inferred from a successful job.

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

## Disposable PBS-to-S3 pilot

On 2026-11-26, the existing PBS 4.2.14-1 installation synced two client-side
encrypted snapshots of a generated 32 MiB file from a temporary local
datastore to a temporary S3-backed datastore. The unscheduled sync selected
only the test group, accepted encrypted snapshots only, and did not remove
vanished snapshots. It used a test-only S3 bucket and bucket-scoped IAM
credentials.

- The first sync reported 32.001 MiB in 10 chunks. After a 1,600-byte change,
  the second reported 5.301 MiB in three chunks. Initial S3 chunk objects
  totaled 33,555,308 bytes; newly added chunks totaled 5,558,537 bytes.
  This demonstrates reuse for this sample, not the production change rate,
  bandwidth, request count, or cost.
- Deleting the first local test snapshot and syncing again left both
  snapshots in the S3 datastore. This checks non-propagation for one
  deletion, not the three-day or 7 daily / 4 weekly / 3 monthly schedules.
- After detaching the test S3 datastore, removing its cache, and reconnecting
  it under the same name with a new cache, both snapshots restored with
  matching SHA-256 digests. A different test key was rejected. This reused
  the same PBS installation and did not test a fresh installation or
  independent key recovery.

The temporary PBS resources and directory, IAM credentials and user, and S3
bucket and objects were removed after the pilot. Only synthetic data was
used; no production backups or workloads were modified. The pilot did not
test application consistency, independent deletion protection, account
recovery, production costs, or the 24-hour objectives. No off-site
homelab backup was established.

## Independent deletion protection remains open

The PBS S3 backend requires object deletion for maintenance. Proxmox staff
stated that Object Lock was not part of PBS's S3 implementation; the 4.2
documentation does not establish lock-aware maintenance. A fixed object
retention period alone is insufficient because an old chunk may still be
referenced by a new snapshot. A delete marker or newer object version can also
hide a locked version. Do not assume that enabling Object Lock or S3
versioning by itself protects a usable PBS recovery point.

### Candidate comparison

All three candidates preserve the PBS datastore format. They require a
versioned S3 source and have not been tested with this PBS datastore; the
disposable pilot used an unversioned bucket. Every option also needs
independently recoverable PBS keys, cloud access, and a role boundary that
routine deployment credentials cannot bypass or change.

| Candidate | Isolation and recovery | Cost and operational trade-offs |
|---|---|---|
| S3 versioning only | Previous object versions remain in the same bucket. A separate administrator must reconstruct a coherent datastore at a known complete point, accounting for delete markers and PBS maintenance. | No separate vault, but each retained version is billed as a full object. Reconstruction, retained-history growth, and recovery time are unproven. |
| AWS Backup vault in the same account | A separately administered, locked vault can retain a copy of the bucket. Restore the copy to a new S3 bucket, then reconnect PBS. A separate role must control the vault, its key, and recovery. | Adds vault storage, restore, event, and S3 request costs. It is the simpler vault candidate, but PBS compatibility and the role boundary are unproven. |
| AWS Backup vault in another account | The protected copy has a stronger account boundary. AWS requires both accounts to belong to the same AWS Organization; restoring into the original account requires copying the recovery point back first. | Adds account, organization, source/destination vault, key, and copy management, plus destination storage and possible additional costs. This is stronger separation, not independence from AWS itself. |

AWS Backup Vault Lock in governance mode can be removed by an identity with
the necessary permissions. Compliance mode becomes immutable after a grace
period of at least three days, and locked recovery points cannot be deleted
before their retention ends. A configuration mistake can therefore cause
unavoidable ongoing costs. Lock mode, retention, and any production resources
require explicit review and approval; no lock is deployed for this recovery
path.

One candidate is S3 versioning with a routine PBS identity that cannot delete
object versions or change versioning, lifecycle, or access policies. A separate
administrator would have to reconstruct the entire datastore at a known
complete recovery point, including correctly handling delete markers and
concurrent maintenance. Individual recoverable objects do not prove that the
resulting PBS datastore is consistent or restorable. Version storage, restore
requests, recovery time, and protection from the routine deployment identity
also need evaluation; S3 charges for each full object version.

An independently administered AWS Backup vault is one possible protection to
evaluate, not a selected or fully costed solution. AWS Backup for S3 requires
versioning and makes subsequent backups incrementally, but its compatibility
with a live PBS datastore, a consistent restore of all required chunks and
metadata, authority separation, and costs are unproven. It also charges
minimum 128 KB per backed-up S3 object, making object counts relevant.
Its initial backup includes existing object versions and delete markers.
AWS documents point-in-time consistency for its S3 backups, but that does not
establish PBS-level consistency during sync or maintenance. Continuous
recovery points have a maximum 35-day window; cross-account or cross-region
copies are available, but copies of continuous backups do not support
point-in-time restore. Longer protection therefore needs periodic recovery
points and tested retention. AWS Backup does not restore bucket configuration,
so policies and settings must be recoverable separately. AWS documents that a
restore can report `COMPLETED` even when some objects failed, and that
restoring into an existing bucket does not overwrite existing objects or
remove extra ones. A recovery exercise must inspect such failures and verify
the PBS restore rather than relying on job status. AWS also documents a
subsequent continuous backup reporting `COMPLETED` when all objects failed to
be backed up.

Continuous backups depend on S3 EventBridge notifications: AWS documents that
disabling those notifications silently stops new changes being captured.
Any such design needs independent monitoring and protection of that dependency.
Periodic-only backups scan the bucket; AWS documents that combining periodic
snapshots with continuous backup can avoid those scans but depends on the
continuous recovery point.

AWS Backup also depends on the vault's KMS key and its permissions, separately
from the PBS client encryption key. Independently recoverable keys and
permissions remain necessary. Do not configure lifecycle expiration or lock
policies until their interaction with PBS retention and maintenance has been
demonstrated safely.

The age of the latest independently protected PBS snapshot must be measured
end to end. For example, a daily 01:00 PBS snapshot copied into a vault at
02:00 can be almost 25 hours old immediately before the next vault copy, even
when both jobs run daily. Frequency, lag, and failure alerts must jointly meet
the 24-hour data-loss objective.

For application consistency, evaluate SQLite's online backup API for
Vaultwarden and the media-management databases, and PostgreSQL's logical
backup tools for the observed statistics database. A database dump alone may
not capture PostgreSQL roles, extensions, or other restoration prerequisites.
The vault guest did not have the `sqlite3` executable available
during discovery. Jellyfin's documented built-in backup supports online
database backup, with metadata optional; the installed package version on
2026-11-26 was newer than the version that introduced it. Verify automation,
required configuration and metadata coverage, free-space requirements, and
restoration before relying on it. Jellyfin's manual backup instructions
require stopping the service. Crafty's documentation recommends stopping a
game server during its file-copy backup. The installed Bedrock server's
vendor documentation describes an online alternative: `save hold`, poll
`save query` until it returns the file list and lengths, copy only those
files and truncate the copies to the reported lengths, then `save resume`.
The server can keep running and modify files during this process; an ordinary
live directory copy is not equivalent. A solution must guarantee resume after
failure and verify complete world and configuration coverage and a usable
restore. Neither approach has been tested. No application backup, save
command, or service stop was performed.

Identify and validate the actual databases, attachments, configuration, and
Minecraft save/quiesce mechanism first. Avoid executing backup actions stored
in a remotely writable repository; any local hooks must be independently
controlled, fail explicitly, and be restore-tested.

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

## Deferred inventory gate

Before finalizing production coverage, identify the active data and attachment
locations, including dashboard state and every Minecraft world. Evaluate the
application-consistency, retention, independent-deletion, key-recovery, and
fresh-install restore approach against those locations and the cost ceiling.
This inventory work is deferred to #6; record assumptions rather than treating
the current cost estimate or synthetic tests as proof of coverage.

## Recovery-test and production gates

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
  [the backup client](https://pbs.proxmox.com/docs/backup-client.html),
  [catalog format](https://pbs.proxmox.com/docs/file-formats.html#proxmox-catalog-file-format-pcat1),
  [pruning and garbage collection](https://pbs.proxmox.com/docs/maintenance.html),
  and [incremental, deduplicated snapshots](https://pbs.proxmox.com/docs/technical-overview.html#snapshots).
- [Proxmox VE backup modes and exclusions](https://pve.proxmox.com/pve-docs/vzdump.1.html).
- [Proxmox staff on Object Lock](https://forum.proxmox.com/threads/immutable-backups-on-s3-object-storage.169404/).
- [AWS Backup for S3](https://docs.aws.amazon.com/aws-backup/latest/devguide/s3-backups.html),
  [S3 restores](https://docs.aws.amazon.com/aws-backup/latest/devguide/restoring-s3.html),
  [cross-account copies](https://docs.aws.amazon.com/aws-backup/latest/devguide/create-cross-account-backup.html),
  [backup encryption](https://docs.aws.amazon.com/aws-backup/latest/devguide/encryption.html),
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
- [Vaultwarden configuration template](https://github.com/dani-garcia/vaultwarden/blob/main/.env.template),
  [Jellyfin backup and restore](https://jellyfin.org/docs/general/administration/backup-and-restore/),
  and [Crafty backup manager](https://docs.craftycontrol.com/pages/user-guide/backup-manager/).
  The Bedrock online-backup protocol was also checked against the vendor
  how-to distributed beside the running server on 2026-11-26.
