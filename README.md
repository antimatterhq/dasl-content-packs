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

Each preset must include a version file at `presets/{source}/{sourceType}/version.yaml`, following
the schema located at [schema/version.schema.yaml](./schema/version.schema.yaml).

All presets must also be listed in [presets/index.yaml](./presets/index.yaml), following the
schema located at [schema/index.schema.yaml](./schema/index.schema.yaml).

## Adding a Preset

1. Add the appropriate directories. For a preset with source "foo" and sourceType "bar", ensure the
   following directory structure is made: `presets/foo/bar`.

2. Create the `preset.yaml` file in the preset's directory, following the preset schema mentioned above.

3. Add `README.md` and, if needed, `icon.png` to the preset's directory.

4. Put these changes up for review.

5. After merging the preset to the repository, create the `version.yaml` file in the preset's directory,
   following the version schema mentioned above. Ensure the proper version numbering is used (starting
   at 1, incrementing by 1 for each new version). Ensure the 'changes' field contains an informative,
   human-readable summary of the changes. For an initial commit, this can be "Initial commit of <source
   and description>". Ensure the 'commit' field refers to the commit that added the `preset.yaml` file to
   the repository.

6. Add the preset to `presets/index.yaml` by placing the source and sourceType of the new preset at the
   bottom of the list.

7. Put these changes up for review. Once merged, the preset will be visible to the DASL PresetStore.