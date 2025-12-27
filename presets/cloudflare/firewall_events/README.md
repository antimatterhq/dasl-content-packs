# Cloudflare presets

This preset directory contains Cloudflare data sources ([Cloudflare logs](https://developers.cloudflare.com/logs/)). Currently supported data sources:

- [firewall_events](https://developers.cloudflare.com/logs/logpush/logpush-job/datasets/zone/firewall_events/)

## Firewall Events

Assumed data format: json lines (or multiline)

Silver tables:
- cloudflare_firewall_events

Gold tables:
- network_activity
