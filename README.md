# Uptime Kuma GitOps Stack

Independent monitoring stack for `uptime.ambitiouscake.com`. It deliberately
runs outside the DVR VPN namespace so it can detect a Gluetun or DVR-stack
failure instead of failing with it.

## Portainer

- Compose path: `stack/docker-compose.yml`
- MariaDB data: Docker volume `uptime-kuma_mariadb-data`
- Legacy SQLite data retained at `/volume1/dkrcfg/uptime-kuma`
- Verified MariaDB dumps: `/volume1/dkrcfg/uptime-kuma-backups`
- Reverse-proxy target: `http://127.0.0.1:3001`
- Public URL: `https://uptime.ambitiouscake.com`

Copy the values from `stack/.env.sample` into the Portainer stack environment,
generate unique database passwords, and deploy from this repository. Uptime
Kuma publishes port 3001 only on NAS loopback; expose it only through the
Synology reverse proxy.

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

Uptime Kuma uses a dedicated MariaDB service. The `uptime-kuma-backup` service
creates a transactionally consistent compressed SQL dump every 24 hours,
validates the gzip stream, writes a SHA-256 checksum, and retains 30 days by
default. Mirror `/volume1/dkrcfg/uptime-kuma-backups` with Synology Hyper Backup
and filesystem snapshots. Back up the Docker volume as an additional recovery
layer, but restore from the verified SQL dumps rather than copying live database
files.

Use encrypted off-NAS retention of at least 30 daily, 12 monthly, and 3 yearly
versions. Quarterly, restore one verified dump to a temporary Kuma instance and
confirm the dashboard and monitor configuration load. The previous SQLite data
directory is intentionally retained for rollback; no unsupported schema
conversion is performed.
