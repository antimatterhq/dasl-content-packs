# Crowdstrike presets

This preset directory contains a preset for handling of [Crowdstrike Falcon logs exposed via FDR](https://github.com/CrowdStrike/FDR).

## falcon

Assumed data format: json lines

Silver tables:
- `crowdstrike`

Gold tables:
- `authentication`
- `dns_activity`
- `file_activity`
- `kernel_extension_activity`
- `network_activity`
- `process_activity`
- `scheduled_job_activity`

