# Uptime Kuma GitOps Stack

Independent monitoring stack for `uptime.ambitiouscake.com`. It deliberately
runs outside the DVR VPN namespace so it can detect a Gluetun or DVR-stack
failure instead of failing with it.

## Portainer

- Compose path: `stack/docker-compose.yml`
- Persistent data: `/volume1/dkrcfg/uptime-kuma`
- Verified SQLite backups: `/volume1/dkrcfg/uptime-kuma-backups`
- Reverse-proxy target: `http://127.0.0.1:3001`
- Public URL: `https://uptime.ambitiouscake.com`

Create the data directory, copy the values from `stack/.env.sample` into the
Portainer stack environment, and deploy from this repository. Kuma uses host
networking so it can monitor the DVR stack's loopback-only published ports
directly. Uptime Kuma itself listens on port 3001; keep that port blocked at
the router and expose it only through the Synology reverse proxy.

## Synology reverse proxy

Uptime Kuma requires WebSocket forwarding for its dashboard and live status
updates. In DSM, open **Control Panel > Login Portal > Advanced > Reverse
Proxy**, edit the `uptime.ambitiouscake.com` rule, then under **Custom Header**
choose **Create > WebSocket**. Synology should add both of these request
headers:

- `Upgrade: $http_upgrade`
- `Connection: $connection_upgrade`

Keep the destination set to `http://127.0.0.1:3001`. A plain HTTP 200 response
is not a sufficient validation: the dashboard must also connect without the
"Cannot connect to the socket server" warning.

Uptime Kuma uses SQLite. The `uptime-kuma-backup` service creates a consistent
online backup every 24 hours, verifies it with `PRAGMA integrity_check`, writes
a SHA-256 checksum, and retains 30 days by default. Mirror both the live data
directory and `/volume1/dkrcfg/uptime-kuma-backups` with Synology Hyper Backup and
filesystem snapshots. Do not place `/app/data` on NFS.

Use encrypted off-NAS retention of at least 30 daily, 12 monthly, and 3 yearly
versions. Quarterly, restore one verified backup to a temporary Kuma instance
and confirm the dashboard and monitor configuration load. Keep SQLite unless
rebuilding from a clean MariaDB installation: Uptime Kuma does not officially
support in-place SQLite-to-MariaDB migration, and third-party conversion can
omit required indexes.
