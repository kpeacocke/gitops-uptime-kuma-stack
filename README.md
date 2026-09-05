# Uptime Kuma GitOps Stack

Independent monitoring stack for `uptime.ambitiouscake.com`. It deliberately
runs outside the DVR VPN namespace so it can detect a Gluetun or DVR-stack
failure instead of failing with it.

## Portainer

- Compose path: `stack/docker-compose.yml`
- Persistent data: `/volume1/dkrcfg/uptime-kuma`
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

Uptime Kuma v2 uses SQLite by default. Include its data directory in the daily
Synology Hyper Backup task and filesystem snapshots. Do not place `/app/data`
on NFS.
