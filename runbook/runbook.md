# Datacenter Runbook — Homelab Infrastructure

This runbook documents the full architecture of my Proxmox homelab, including the reasoning behind each design decision, the static IP map, and the troubleshooting commands I use to keep everything running. It's written the way I'd want to find it if I came back to this setup after six months away.

---

## Why This Is Built the Way It Is

**Why Proxmox instead of a regular Linux server?**
Proxmox lets me run multiple isolated services on a single piece of hardware using LXC containers. Each container has one job. If something breaks in one container it doesn't affect anything else. That's how real data centers handle service isolation — dedicated roles, not everything crammed onto one machine.

**Why LXC containers instead of VMs?**
LXC containers share the host kernel, which means they use far less RAM and storage than full virtual machines. For a home lab on a mini PC with limited resources, containers let me run eight services simultaneously without the machine grinding to a halt. In production environments you'd see both VMs and containers — containers for lightweight services, VMs for anything that needs full OS isolation.

**Why does Pi-hole handle DHCP instead of the AT&T router?**
The AT&T fiber router is locked down. It doesn't give you real control over DNS settings, which means you can't reliably point DNS to a custom server. The workaround is to disable DHCP on the AT&T router entirely and hand that job to Pi-hole. Now Pi-hole controls both what IP addresses get handed out to devices AND where those devices send their DNS queries. Every device on the network goes through Pi-hole automatically without any manual configuration.

**Why Nginx Proxy Manager as a reverse proxy?**
Without NPM, if you want to expose any service to the internet you have to open a direct port to that specific container. That means the container is directly reachable from outside. With NPM as the single entry point, only ports 80 and 443 are forwarded on the router — NPM receives all traffic and routes it internally to the right container. Nothing internal is ever directly exposed. This is standard DMZ architecture.

**Why InfluxDB and Grafana for monitoring?**
Proxmox has built-in metrics but they're basic and don't persist history. InfluxDB stores time-series metrics and Grafana visualizes them. The combination gives me a live dashboard showing CPU, RAM, temperatures, and disk I/O across all containers — and the history stays so I can see trends over time, not just the current snapshot.

**Why are the DMZ containers firewalled off from the LAN?**
Any container that faces the internet is a potential attack surface. If something in CT 105 or CT 106 got compromised, I don't want that attacker to be able to reach my other containers or devices on the network. The three-rule firewall stack prevents lateral movement — the compromised container can reach the internet but nothing else on the LAN.

---

## Static IP Map

All static IPs are assigned in the 192.168.1.10–.19 range. The AT&T router's DHCP pool starts at 192.168.1.61, so there's no overlap.

| Device/Service | Container ID | Static IP | Role |
|---|---|---|---|
| Proxmox Host | — | 192.168.1.10 | Bare-metal hypervisor |
| Pi-hole | CT 100 | 192.168.1.11 | DNS filtering + DHCP |
| Tailscale | CT 101 | 192.168.1.12 | Remote access VPN |
| Jellyfin | CT 102 | 192.168.1.13 | Media server |
| Reserved | CT 103 | 192.168.1.14 | Future use |
| Reserved | CT 104 | 192.168.1.15 | Future use |
| Web/DMZ | CT 105 | 192.168.1.16 | Future web app |
| Nginx Proxy Manager | CT 106 | 192.168.1.17 | Reverse proxy / entry point |
| InfluxDB | CT 107 | 192.168.1.18 | Metrics database |
| Grafana | CT 108 | 192.168.1.19 | Monitoring dashboard |
| AT&T Router | — | 192.168.1.254 | WAN gateway |

---

## Firewall Rules

Applied to CT 105 and CT 106 only. NOT applied to Pi-hole or Tailscale.

| Rule | Direction | Action | Target | Why |
|---|---|---|---|---|
| 1 (TOP) | Outbound | ACCEPT | AT&T Gateway IP | Allows internet access out |
| 2 | Outbound | DROP | 192.168.1.0/24 | Stops lateral movement to LAN |
| 3 | Inbound | DROP | 192.168.1.0/24 | Blocks LAN pivot attacks |

**Pi-hole exception:** Pi-hole needs full LAN access to hand out DHCP leases and answer DNS queries from every device on the network. Firewall rules would break the whole household.

**Tailscale exception:** Tailscale is the remote access overlay. It needs to talk to both the internet and the LAN to function. Lock it down and you lose remote access entirely.

---

## Troubleshooting Commands

Run these from the Proxmox host shell unless noted otherwise.

### General

```bash
# List all containers and their status
pct list

# Start a container
pct start [ID]

# Stop a container
pct stop [ID]

# Restart a container
pct restart [ID]

# Open a shell inside a container
pct enter [ID]

# Ping test from Proxmox host (confirms internet connectivity)
ping 8.8.8.8 -c 4
```

### Pi-hole (CT 100)

```bash
# Restart Pi-hole DNS service
pct exec 100 -- pihole restartdns

# Check Pi-hole status
pct exec 100 -- pihole status

# Update Pi-hole blocklists
pct exec 100 -- pihole -g

# View Pi-hole logs (last 50 lines)
pct exec 100 -- tail -n 50 /var/log/pihole.log
```

### Tailscale (CT 101)

```bash
# Check Tailscale connection status
pct exec 101 -- tailscale status

# Check Tailscale IP
pct exec 101 -- tailscale ip
```

### Firewall

```bash
# Compile and check firewall rules (shows what's active)
pve-firewall compile

# Check firewall status
pve-firewall status
```

### Grafana / InfluxDB

```bash
# Check if InfluxDB is running (CT 107)
pct exec 107 -- systemctl status influxdb

# Check if Grafana is running (CT 108)
pct exec 108 -- systemctl status grafana-server

# Restart InfluxDB
pct exec 107 -- systemctl restart influxdb

# Restart Grafana
pct exec 108 -- systemctl restart grafana-server
```

---

## Common Issues and Fixes

**Household internet stops working**
Pi-hole went down. Fix:
```bash
pct restart 100
# Wait 30 seconds, then test DNS on a device
```

**Can't reach Proxmox dashboard remotely**
Tailscale is down. Fix:
```bash
pct restart 101
pct exec 101 -- tailscale status
```

**Grafana dashboard shows no data**
Either InfluxDB is down or the Proxmox Metric Server connection dropped. Fix:
```bash
pct restart 107
pct restart 108
# Then check Proxmox: Datacenter > Metric Server > verify InfluxDB entry
```

**Can't reach NPM or a proxied service**
```bash
pct restart 106
# Check that port 80/443 are still forwarded on AT&T router
```

---

## Port Forwarding Reference (AT&T Router)

| External Port | Internal IP | Internal Port | Service |
|---|---|---|---|
| 80 | 192.168.1.17 | 80 | Nginx Proxy Manager (HTTP) |
| 443 | 192.168.1.17 | 443 | Nginx Proxy Manager (HTTPS) |

---

## Rollback Reference

If an IP migration breaks something:
1. Identify the last container that was changed
2. Revert its IP back to the previous value
3. If Pi-hole is unreachable, go to Proxmox console at 192.168.1.10 directly
4. Run `pct restart 100`
5. Confirm household DNS is working before continuing
