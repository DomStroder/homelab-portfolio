# Homelab Portfolio — Dominic Stroder

I built this to learn by doing. Coming from a background in residential electrical and structured cabling, I wanted to understand how enterprise infrastructure actually works — not just read about it. So I built a small version of it at home.

This repo documents everything: the architecture decisions, the IP map, the firewall rules, and the troubleshooting steps I figured out along the way. It's meant to be a working reference, not just a showcase.

---

## The Setup

Running Proxmox VE on a Minisforum NAB6 Lite mini PC. Eight LXC containers, each with a specific job. The whole thing is designed the way a real environment would be — dedicated DNS server, reverse proxy as the only public entry point, firewall isolation around anything web-facing, and a monitoring pipeline so I can actually see what's happening.

One thing worth noting: the AT&T fiber router doesn't give you real control over DNS, so Pi-hole handles both DNS filtering and DHCP for the whole network. That means every device in the house runs through it. It's a small thing but it's the kind of problem you run into in the real world when consumer hardware doesn't do what you need it to do.

---

## Infrastructure

| Service | Container | IP | What It Does |
|---|---|---|---|
| Proxmox Host | — | 192.168.1.10 | Bare-metal hypervisor |
| Pi-hole | CT 100 | 192.168.1.11 | DNS filtering + DHCP server |
| Tailscale | CT 101 | 192.168.1.12 | Remote access VPN overlay |
| Jellyfin | CT 102 | 192.168.1.13 | Media server |
| Nginx Proxy Manager | CT 106 | 192.168.1.17 | Reverse proxy / single entry point |
| InfluxDB | CT 107 | 192.168.1.18 | Metrics database |
| Grafana | CT 108 | 192.168.1.19 | Monitoring dashboard |

Static IPs assigned in the 192.168.1.10–.19 range, sitting below the router's DHCP pool to avoid conflicts.

---

## Firewall Design

Web-facing containers (CT 105, CT 106) are isolated from the rest of the LAN using a three-rule stack:

1. ACCEPT outbound to the AT&T gateway — lets internet traffic out
2. 2. DROP outbound to 192.168.1.0/24 — blocks lateral movement to other LAN devices
   3. 3. DROP inbound from 192.168.1.0/24 — blocks anyone on the LAN from pivoting in
     
      4. Tailscale and Pi-hole are explicitly excluded from these rules. Tailscale needs LAN access to work as a remote access overlay. Pi-hole needs LAN access to serve DNS and DHCP to every device on the network.
     
      5. ---
     
      6. ## What's In This Repo
     
      7. - `/diagrams` — Network topology diagram showing the full architecture
         - - `/runbook` — Step-by-step documentation with troubleshooting commands and the reasoning behind each design decision
           - - `/monitoring` — Screenshot of the live Grafana dashboard
             - - `/hardware` — Photo of the physical setup
               - - `/configs` — IP reference map
                
                 - ---

                 ## Background

                 I have an Industrial Electrician degree from Mid-Del Technology Center and about two years of hands-on structured cabling and low-voltage electrical work. I'm currently working through CompTIA A+ (Core 1 and Core 2) with a target of Summer 2026.

                 This project is part of a deliberate career move toward data center operations. The homelab gives me something real to point to — not just certifications, but actual infrastructure I designed, built, and documented myself.

                 ---

                 ## Skills This Covers

                 Proxmox VE · LXC containers · Linux (Debian) · Network segmentation · Firewall rule design · Nginx reverse proxy · Pi-hole DNS/DHCP · Tailscale VPN · InfluxDB · Grafana · Structured cabling · Low-voltage electrical · Technical documentation
