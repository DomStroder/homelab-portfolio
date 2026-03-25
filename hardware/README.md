# Hardware

This folder documents the physical infrastructure behind the homelab.

## Equipment

### Primary Server
- **Device:** Minisforum NAB6 Lite Mini PC
- **Role:** Proxmox VE bare-metal hypervisor host
- **Why this hardware:** Low power draw, fanless design, fits in a home environment, enough RAM and storage to run 8+ LXC containers simultaneously

### Network
- **ISP:** AT&T Fiber
- **Router:** AT&T provided gateway
- **Note:** AT&T router DHCP is disabled. Pi-hole (CT 100) handles all DHCP and DNS for the network because the AT&T gateway is too locked down to allow proper DNS customization.

## What Goes Here

- physical-setup.jpg — Photo of the Minisforum PC and router with organized cabling

## Why the Photo Matters

In data center work, cable management is a real skill. Organized, labeled cabling means faster troubleshooting, easier handoffs, and fewer mistakes under pressure. The photo here shows the physical setup is treated with the same discipline as the software configuration.

## Status

Photo pending — will be added once cabling is organized and setup is clean.
