# Homelab

The local and cloud environment managed as one homelab, including its services and recovery needs.

## Language

**Workload**:
A service, or related group of services, with its configuration and persistent data.
_Avoid_: VM when referring to the service rather than its host

**Reprovisioning**:
Recreating homelab infrastructure and service configuration without necessarily restoring persistent data.
_Avoid_: Recovery when data has not been restored

**Recovery**:
Restoring homelab infrastructure, service configuration, and the persistent data needed for usable **Workloads**.
_Avoid_: Reprovisioning when data restoration is also intended

**Essential workload**:
A **Workload** whose recovery takes priority because it provides access, credentials, or irreplaceable files.
_Avoid_: Critical workload

**Best-effort workload**:
A **Workload** whose recovery can wait until **Essential workloads** are usable.
_Avoid_: Disposable workload, unless its data can also be discarded

**Emergency access**:
An administrative access path available when normal identity or credential services are unavailable.
_Avoid_: Normal sign-in

**Protected data**:
Homelab data selected for backup because losing it is unacceptable or recreating it is impractical, regardless of its **Workload** recovery priority.
_Avoid_: Essential workload data when referring to all data requiring backup

**Recovery kit**:
The keys, credentials, and instructions held independently of the homelab services they are needed to recover.
_Avoid_: Backup when referring only to recovery prerequisites
