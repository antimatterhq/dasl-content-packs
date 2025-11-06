# Cloudflare presets

This preset directory contains Cloudflare data sources ([Cloudflare logs](https://developers.cloudflare.com/logs/)). Currently supported data sources:

- [gateway_dns](https://developers.cloudflare.com/logs/logpush/logpush-job/datasets/account/gateway_dns/)

## Gateway DNS

Assumed data format: json lines (or multiline)

Silver tables:
- cloudflare_gateway_dns

Gold tables:
- dns_activity
