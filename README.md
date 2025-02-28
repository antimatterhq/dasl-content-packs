# dasl-content-packs
This repository contains preset content pack definitions for processing various data sources,
grouped by source (e.g. AWS) and source type (e.g. Route53 within AWS). These define silver
table pre-transform and transform definitions, and gold table transform definitions.

## Layout and Schema
The preset content packs are stored using the pattern:
`presets/{source}/{sourceType}/preset.yaml`

For example, a preset named "aws_sec_lake_route53" with a source of "aws_sec_lake" and sourceType of
"route53" will be defined in `presets/aws_sec_lake/route53/preset.yaml`

The schema for a preset is located at [schema/preset.schema.yaml](./schema/preset.schema.yaml)

All presets must also be listed in [presets/index.yaml](./presets/index.yaml), following the
schema located at [schema/index.schema.yaml](./schema/index.schema.yaml)
