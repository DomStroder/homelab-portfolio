# Monitoring

This folder contains screenshots and documentation for the Grafana monitoring dashboard.

## What Goes Here

- grafana-dashboard.png — Live screenshot of the Grafana dashboard showing all containers reporting metrics

## Monitoring Stack

The monitoring pipeline works like this:

1. Proxmox Metric Server pushes data to InfluxDB (CT 107) at 192.168.1.18
2. Grafana (CT 108) at 192.168.1.19 pulls from InfluxDB as a data source
3. Dashboard panels show real-time and historical data for all containers

## Dashboard Panels

- CPU usage per container
- RAM usage per container
- Disk I/O
- Network traffic in and out
- Node temperature (if sensor data available)

                    ## Why This Matters

                    Proxmox has basic built-in metrics but they don't persist history. InfluxDB stores time-series data so you can see trends over time, not just the current moment. This is how production environments handle infrastructure monitoring — dedicated metrics pipeline, not just a live view.

## Status

Screenshot pending — will be added once the monitoring stack is deployed in Phase 1 Step 4.
