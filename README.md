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
Portainer stack environment, and deploy from this repository. The port is
loopback-only; do not expose port 3001 at the router.

Uptime Kuma v2 uses SQLite by default. Include its data directory in the daily
Synology Hyper Backup task and filesystem snapshots. Do not place `/app/data`
on NFS.
