# Cisco presets

This preset directory contains Cisco data sources ([IOS logs](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html/)). Currently supported data sources:

- [ios](https://www.cisco.com/c/en/us/td/docs/routers/access/wireless/software/guide/SysMsgLogging.html#wp1054470)

## Cisco IOS

Assumed data format: syslog

Silver tables:
- cisco_ios_iam_authentication
- cisco_ios_iam_authorize_session
- cisco_ios_network
- cisco_ios_process

Gold tables:
- authentication
- authorize_session
- network_activity
- process_activity