# Diagrams

This folder contains network topology diagrams for the homelab infrastructure.

## What Goes Here

- network-topology.png — Full network diagram showing all containers, IPs, firewall zones, and traffic flow

## How to View

Open network-topology.png to see the full architecture at a glance. The diagram shows:
- AT&T router at the top as the WAN entry point
- Proxmox host with all 8 containers mapped out
- Pi-hole labeled as DNS and DHCP server
- Nginx Proxy Manager as the DMZ entry point
- Firewalled Zone box around CT 105 and CT 106
- Tailscale overlay for remote access

## Status

Diagram pending — will be added once Phase 1 infrastructure is fully deployed and IPs are confirmed.
