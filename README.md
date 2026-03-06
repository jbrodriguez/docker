# Docker Stack Notes (lan.jbrio.net)

## Summary
This stack uses Caddy as the reverse proxy with DNS-01 (Cloudflare) to issue HTTPS certificates for internal services under `*.lan.jbrio.net`. Services live in separate compose files and are included by `docker-compose.yml`.

## Cloudflare Token
- Token is stored in `./.env-caddy` as `CLOUDFLARE_API_TOKEN=...`
- Required permissions:
  - Zone:DNS:Edit
  - Zone:Zone:Read

## OpenWRT DNS Override
We route internal names to the Docker host by overriding the LAN subdomain:
- `*.lan.jbrio.net -> 192.168.2.246`

Command used:
```sh
uci -q del_list dhcp.@dnsmasq[0].address='/jbrio.net/192.168.2.246'
uci add_list dhcp.@dnsmasq[0].address='/lan.jbrio.net/192.168.2.246'
uci commit dhcp
/etc/init.d/dnsmasq restart
```

## Caddy
- Image: `ghcr.io/caddybuilds/caddy-cloudflare:latest`
- Caddyfile: `./caddy/Caddyfile`
- Data volumes: `caddy_data`, `caddy_config`

## Compose Structure
- Main file: `docker-compose.yml` uses `include:` to load service files
- Caddy is defined in `caddy.yml`
- Portainer is defined in `portainer.yml`
- Services are reachable via `https://<service>.lan.jbrio.net`

## Notable Routing Details
- Jellyfin uses host networking, so Caddy proxies it via:
  - `reverse_proxy host.docker.internal:8096`
- `proxy` network is used to allow Caddy to reach containers

## Removed
- Traefik has been removed (files deleted and labels stripped)
- `traefik/` directory and `.env-traefik` are no longer used
