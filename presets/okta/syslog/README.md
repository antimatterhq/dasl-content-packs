# Okta presets

This preset contains data processing instructions for the Okta syslog data source with Cribl packaging/metadata. 

## Exploration & Research

- [Okta Syslog API EventTypes](https://developer.okta.com/docs/reference/api/event-types/#catalog)
- [CSV File of EventTypes](https://developer.okta.com/docs/okta-event-types.csv)
- [CRIBL Okta Source Setup](https://docs.cribl.io/search/set-up-okta/#provider)
- [List of CRIBL Sources](https://docs.cribl.io/stream/sources/)
  - 30 different "up stream" sources - some of which contain multiple products
  - Okta is not listed as dedicated source - likely connected via some generic connector
- Seems to contain data from additional 3rd party source
  - [ZScaler](https://help.zscaler.com/zia/understanding-nanolog-streaming-service)
  - CrowdStrike
- examples from [OCSF](https://github.com/ocsf/examples/tree/main/mappings/markdown/Okta)

## Limitations
- currently only supports about 70 event classes present in the sample data
  - unsupported event types are being mapped to NULL
- some class-specific OCSF fields are currently not mapped
- uses `tempField` util for the `os` sub-object that is being reused across different objects

## Syslog
- `1004` different event types defined by the Okta API
- Assumed data format: jsonL
- gold tables written to
  - account_change
  - authentication
  - authorize_session
  - entity_management
  - user_access
  - group_management

