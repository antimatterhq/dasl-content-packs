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

---

## Using Your Own Custom Presets

In addition to the official presets managed in this repository, users can create and host their own
custom presets. There are two approaches: manually placing files in a Unity Catalog volume, or
installing via content packs.

### Prerequisites

To interact with Lakewatch programmatically (managing presets, creating datasources, etc.), you need
the `lakewatch` Python client installed in your Databricks notebook.

In the first cell of your Databricks notebook, run:

```python
%pip install lakewatch
```

Then restart the Python kernel and import the client:

```python
from lakewatch import Client

client = Client.for_workspace()
```

> **Note:** The `lakewatch` package is published to PyPI and includes the high-level SDK for
> preset operations, datasource CRUD, workspace configuration, and more. It automatically handles
> authentication when running inside a Databricks notebook.
>
> Auto-generated job notebooks (datasource pipelines, detection rules, exports) do **not** need
> this manual step — the dasl-apiserver injects the required `%pip install` automatically.

### Approach 1: Custom Presets in a Unity Catalog Volume

Host your own presets by placing them in a Unity Catalog volume and pointing your workspace at it.

#### Step 1: Check if a custom presets path is already configured

Before creating a new volume, check whether your workspace already has a custom presets path set.
If one exists, you should place your preset files there instead of configuring a new path (which
would overwrite the existing setting).

**Via the Lakewatch UI:**

Navigate to **Settings → Advanced** and look for the custom presets path field.

**Via Python client:**

```python
from lakewatch import Client

client = Client.for_workspace()
config = client.get_config()
print(config.dasl_custom_presets_path)
# If this prints a path (e.g., "/Volumes/my_catalog/my_schema/my_volume/presets"),
# use that existing path. If it prints None, you'll configure one in a later step.
```

#### Step 2: Create the directory structure

If a custom presets path already exists (from Step 1), place your files there. Otherwise, create
a new Unity Catalog volume with the same layout used by this repository:

```
/Volumes/<catalog>/<schema>/<volume>/presets/
├── index.yaml
└── <source>/
    └── <sourceType>/
        └── preset.yaml
```

For example, to add a custom preset for your internal app's authentication logs:

```
/Volumes/my_catalog/my_schema/my_volume/presets/
├── index.yaml
└── myapp/
    └── auth/
        └── preset.yaml
```

#### Step 3: Create or update `index.yaml`

The index file registers all your custom presets:

```yaml
presets:
  - source: "myapp"
    sourceType: "auth"
```

Each entry's `source` and `sourceType` correspond to the directory path: `<source>/<sourceType>/preset.yaml`.

#### Step 4: Place your `preset.yaml`

Copy your preset YAML into the appropriate directory. The preset's `name` field should follow the convention `<source>_<sourceType>` (e.g., `myapp_auth`).

#### Step 5: Configure your workspace to use the custom presets path

> **Skip this step** if your workspace already has a custom presets path configured (from Step 1)
> and you placed your files there.

**Python client:**

```python
from lakewatch import Client

client = Client.for_workspace()
config = client.get_config()
config.dasl_custom_presets_path = "/Volumes/my_catalog/my_schema/my_volume/presets"
client.put_config(config)
```

**REST API:**

```bash
curl -X PUT "${HOST}/ajax-api/2.0/dasl-apiserver/apis/workspace/v1/config" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}" \
  -d '{
    "apiVersion": "v1",
    "kind": "WorkspaceConfig",
    "spec": {
      "daslCustomPresetsPath": "/Volumes/my_catalog/my_schema/my_volume/presets",
      ...
    }
  }'
```

#### Step 6: Verify your preset is visible

**Python client:**

```python
presets = client.list_presets()
for p in presets.items:
    print(p.name)
# Should include: internal_myapp_auth
```

**REST API:**

```bash
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/datasources" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

> **Note:** Custom presets are automatically prefixed with `internal_` to distinguish them from
> official presets. You must use this prefix when referencing them (e.g., `internal_myapp_auth`).

### Approach 2: Installing Presets via Content Packs

Content packs bundle presets (and optionally detection rules) into installable packages. You can
install a preset from a content pack using the REST API:

```bash
curl -X POST "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/content-packs/${CONTENT_PACK_UUID}/install/preset" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

Installed content pack presets are stored in the workspace's custom presets location (same UC volume path).

---

## Preset Cache

Presets are cached after first retrieval to avoid repeatedly fetching from storage:

| Preset Type | Cache Duration |
|---|---|
| Official (Databricks-managed) presets | 12 hours |
| Custom (`internal_` prefixed) presets | 1 hour |

During active development, force a cache refresh after updating your preset files:

**Python client:**

```python
client.purge_preset_cache()
```

**REST API:**

```bash
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/presets/purge-cache" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

You can also pass `clear_cache=true` when listing presets to force a refresh:

```bash
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/datasources?clear_cache=true" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

---

## Inspecting Presets

### Listing All Available Presets

**Python client:**

```python
from lakewatch import Client

client = Client.for_workspace()
presets = client.list_presets()

for preset in presets.items:
    print(f"{preset.name}: {preset.title} ({preset.source}/{preset.source_type})")

# Check for presets that failed to load
for error in presets.errors:
    print(f"Error loading {error.name}: {error.error}")
```

**REST API:**

```bash
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/datasources" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

### Getting a Specific Preset

**Python client:**

```python
preset = client.get_preset("internal_myapp_auth")

# View autoloader config
print(preset.autoloader.format)

# View silver transform tables
for table in preset.silver.transform:
    print(f"Silver table: {table.name}")
    for field in table.fields:
        print(f"  {field.name}")

# View gold tables
for table in preset.gold:
    print(f"Gold table: {table.name} (from {table.input})")
```

**REST API:**

```bash
# Get the full preset specification
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/presets/datasource/internal_myapp_auth" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"

# Get just the summary metadata
curl "${HOST}/ajax-api/2.0/dasl-apiserver/apis/content/v1/presets/datasource/summary/internal_myapp_auth" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}"
```

---

## Creating a Datasource with a Custom Preset

Once your custom preset is deployed and visible (see above), you can create a datasource that uses it.

**Python client:**

```python
from lakewatch.types import DataSource

ds = DataSource(
    use_preset="internal_myapp_auth",
    source="myapp",
    source_type="auth",
    autoloader=DataSource.Autoloader(
        location="s3://my-security-logs/myapp/auth/"
    )
)

client.create_datasource("myapp_auth_logs", ds)
```

**REST API:**

```bash
curl -X POST "${HOST}/ajax-api/2.0/dasl-apiserver/apis/core/v1/datasources" \
  -H "Content-Type: application/json" \
  -H "Cookie: DBAUTH=${DBAUTH_TOKEN}" \
  -H "x-csrf-token: ${CSRF_TOKEN}" \
  -H "x-databricks-org-id: ${ORG_ID}" \
  -d '{
    "apiVersion": "v1",
    "kind": "DataSource",
    "metadata": {
      "name": "myapp_auth_logs"
    },
    "spec": {
      "source": "myapp",
      "sourceType": "auth",
      "usePreset": "internal_myapp_auth",
      "autoloader": {
        "location": "s3://my-security-logs/myapp/auth/"
      }
    }
  }'
```

---

## Testing Presets with PreviewEngine

The `PreviewEngine` validates your preset YAML and executes it against sample data in a Databricks notebook,
so you can verify transformations before deploying.

### Setup

```python
from pyspark.sql import SparkSession
from lakewatch import Client
from lakewatch.preset_development import PreviewEngine, PreviewParameters

spark = SparkSession.builder.getOrCreate()
client = Client.for_workspace()
```

### Loading Your Preset

```python
# From a file
with open("/Workspace/Users/you@company.com/presets/preset.yaml", "r") as f:
    preset_yaml = f.read()

# Or inline
preset_yaml = """
name: myapp_auth
author: Security Team
...
"""
```

### Choosing an Input Mode

`PreviewParameters` supports four input modes for supplying data to the engine.

#### Mode 1: Manual Input

Provide schema and data directly. Best for quick iteration with small, controlled datasets.

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, TimestampType

schema = StructType([
    StructField("timestamp", TimestampType(), True),
    StructField("user_email", StringType(), True),
    StructField("event_type", StringType(), True),
    StructField("status_code", IntegerType(), True),
    StructField("client_ip", StringType(), True),
])

data = [
    ("2024-06-15 10:30:00", "alice@example.com", "login",  200, "192.168.1.100"),
    ("2024-06-15 10:31:00", "bob@example.com",   "login",  401, "10.0.0.50"),
]

ds_params = (
    PreviewParameters(spark)
    .from_input()
    .set_data_schema(schema)
    .set_data(data)
)
```

#### Mode 2: Autoloader

Load data from cloud storage. The file format is read automatically from the preset's `autoloader.format`.

```python
ds_params = (
    PreviewParameters(spark, client)
    .from_autoloader()
    .set_autoloader_location("s3://my-bucket/security-logs/")
    .set_date_range("timestamp", "2024-06-01 00:00:00", "2024-06-02 00:00:00")
    .set_input_record_limit(50)
)
```

- `set_date_range(column, start, end)` — filters input by a timestamp column.
- `set_input_record_limit(n)` — caps the number of records loaded (default: 10).
- `set_pretransform_name("name")` — also test a Silver pretransform stage. Omit to skip pretransform.

#### Mode 3: Table

Read directly from a Unity Catalog table.

```python
ds_params = (
    PreviewParameters(spark, client)
    .from_table()
    .set_table("catalog.schema.my_raw_table")
    .set_input_record_limit(100)
)
```

#### Mode 4: SilverBronze (Multi-Table Joins)

Test presets that join multiple tables.

**Without autoloader** — skips bronze and reads from the first table:

```python
bronze_tables = [
    {"name": "catalog.schema.primary_table", "alias": "primary"},
    {
        "name": "catalog.schema.lookup_table",
        "alias": "lookup",
        "joinExpr": "primary.id = lookup.foreign_id",
        "joinType": "left"
    }
]

ds_params = (
    PreviewParameters(spark, client)
    .from_silverbronze_tables()
    .set_bronze_table_definitions(bronze_tables)
)
```

**With autoloader** — loads data via Auto Loader, runs preBronze, then joins:

```python
bronze_tables = [
    {"name": "autoloader_data"},  # First entry names the autoloader output
    {
        "name": "catalog.schema.enrichment_table",
        "alias": "enrich",
        "joinExpr": "autoloader_data.key = enrich.key",
        "joinType": "left"
    }
]

ds_params = (
    PreviewParameters(spark, client)
    .from_silverbronze_tables()
    .set_bronze_table_definitions(bronze_tables)
    .set_autoloader_location("s3://my-bucket/raw-data/")
)
```

### Running the Evaluation

```python
engine = PreviewEngine(spark, preset_yaml, ds_params)

engine.evaluate(
    gold_table_schema="my_catalog.my_gold_schema",  # Required if preset has gold transforms
    display=True,           # Render output in the notebook
    force_evaluation=False, # If True, collect all errors instead of stopping at first
    verbose=False           # If True, include full stack traces in error output
)
```

- `gold_table_schema` — `"catalog.schema"` where gold tables live. Required when the preset has gold
  transforms. The engine validates that each gold output is type-compatible with the corresponding
  Unity Catalog table.
- `force_evaluation` — by default the engine stops at the first error. Set to `True` to collect all
  errors across all stages.

### Accessing Results Programmatically

```python
bronze_df, pre_silver_df, silver_output_map, gold_output_map = engine.results()

# silver_output_map: {"table_name": DataFrame, ...}
# gold_output_map:   {"gold_name/silver_input": DataFrame, ...}

events_df = silver_output_map["auth_events"]
events_df.show()
events_df.printSchema()
```

### Storage Configuration

The engine needs temporary storage for autoloader schemas and checkpoints. By default this is resolved
from your workspace's `daslStoragePath`. To override:

```python
ds_params = (
    PreviewParameters(spark)
    .set_autoloader_temp_schema_location("/Volumes/catalog/schema/volume/schemas")
    .set_checkpoint_temp_location_base("/Volumes/catalog/schema/volume/checkpoints")
    .from_autoloader()
    .set_autoloader_location("s3://my-bucket/data/")
)
```

Temporary files are cleaned up automatically when the evaluation completes.

### Common Errors

| Error | Cause |
|---|---|
| `MissingSilverKeysError` | A gold table's `input` doesn't match any silver transform table name |
| `PreTransformNotFound` | The pretransform name passed to `set_pretransform_name()` doesn't exist in the preset |
| `DuplicateFieldNameError` | Two fields in the same stage share the same name |
| `MalformedFieldError` | A field spec has zero or more than one operation (`from`, `expr`, `literal`, etc.) |
| `InvalidLiteralError` | A `literal` value is not a string |
| `InvalidGoldTableSchemaError` | `gold_table_schema` is missing or malformed |
| `UnknownGoldTableError` | Gold table name doesn't exist in the specified Unity Catalog schema |
| `StageExecutionException` | SQL expressions failed during execution (use `force_evaluation=True` for full details) |

---

## API Reference

### REST API Endpoints (Presets)

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/apis/content/v1/datasources` | List all available preset templates |
| `GET` | `/apis/content/v1/presets/datasource/{name}` | Get full preset specification |
| `GET` | `/apis/content/v1/presets/datasource/summary/{name}` | Get preset summary metadata |
| `GET` | `/apis/content/v1/presets/purge-cache` | Purge preset cache |
| `POST` | `/apis/content/v1/content-packs/{uuid}/install/preset` | Install preset from content pack |

All endpoints are prefixed with `${HOST}/ajax-api/2.0/dasl-apiserver`.

### Python Client

Install: `%pip install lakewatch` (in a Databricks notebook)

```python
from lakewatch import Client

client = Client.for_workspace()

# Preset operations
client.list_presets()                          # List all presets
client.get_preset("internal_myapp_auth")       # Get specific preset
client.purge_preset_cache()                    # Purge preset cache

# Workspace config (for custom presets path)
client.get_config()                            # Get workspace config
client.put_config(config)                      # Update workspace config
```
