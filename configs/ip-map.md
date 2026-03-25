# IP Address Reference Map

Quick reference for every device and service on the network.
Update this file any time a static IP changes.

## Reserved Static Range: 192.168.1.10 to 192.168.1.60

AT&T router DHCP pool starts at 192.168.1.61. No overlap possible.

## Infrastructure

| Device | IP | Notes |
|---|---|---|
| Proxmox Host | 192.168.1.10 | Dashboard: https://192.168.1.10:8006 |
| AT&T Router | 192.168.1.254 | WAN gateway |

## Containers

| Service | CT ID | IP | Access |
|---|---|---|---|
| Pi-hole | CT 100 | 192.168.1.11 | http://192.168.1.11/admin |
| Tailscale | CT 101 | 192.168.1.12 | CLI only |
| Jellyfin | CT 102 | 192.168.1.13 | http://192.168.1.13:8096 |
| Reserved | CT 103 | 192.168.1.14 | Future use |
| Reserved | CT 104 | 192.168.1.15 | Future use |
| Web/DMZ | CT 105 | 192.168.1.16 | via NPM |
| Nginx Proxy Manager | CT 106 | 192.168.1.17 | http://192.168.1.17:81 |
| InfluxDB | CT 107 | 192.168.1.18 | http://192.168.1.18:8086 |
| Grafana | CT 108 | 192.168.1.19 | http://192.168.1.19:3000 |

## Port Forwarding

| External | Internal | Service |
|---|---|---|
| 80 | 192.168.1.17:80 | NPM HTTP |
| 443 | 192.168.1.17:443 | NPM HTTPS |

Nothing else is forwarded externally. All traffic enters through NPM only.
