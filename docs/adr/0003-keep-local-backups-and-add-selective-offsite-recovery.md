# Keep local backups and add selective off-site recovery

PBS currently runs on the Proxmox host, so its availability does not establish recovery after complete host loss. Retain PBS for convenient local restores and add a separate, client-side encrypted off-site recovery path for protected data instead of copying every guest or all media off site by default.

## Recovery objectives

- Cover complete host and local-disk loss, accidental deletion, and compromise of a deployment credential; separate deployment authority from authority to destroy retained backups.
- Prioritize access, credentials, and irreplaceable files. Media, games, printing, and runners can recover later, but their unique data is not automatically disposable.
- Initially target essential-workload recovery within 24 hours after replacement hardware and internet are available, with no more than 24 hours of data loss. These are homelab objectives, not a high-availability SLA or a verified capability.
- Initially retain 7 daily, 4 weekly, and 3 monthly off-site recovery points. Alert on failed or overdue backups, and prove recovery with isolated restores at least quarterly and after material recovery-path changes.

## Data and budget boundaries

- Protect personal files/photos, password-vault data, unique game state, and other non-reproducible application data. Explicitly exclude replaceable downloaded films/TV/music, caches, and temporary downloads; personally created or otherwise irreplaceable media remains protected.
- Cover shared and bind-mounted data explicitly. Use application-consistent exports or supported backup hooks where needed; a guest snapshot alone does not demonstrate coverage.
- Cap additional recurring costs at EUR 25/month without further approval, including off-site storage and recovery-exercise costs. Measure protected data and price storage, requests, retention, and recovery before selecting the off-site mechanism.
- If coverage, retention, or recovery objectives do not fit the budget, request a decision rather than silently excluding data, reducing retention, or exceeding the ceiling. Existing backup coverage, retention, storage backing, and recoverability remain unverified.
