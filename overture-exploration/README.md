# Overture Maps \<-> fiboa Schema and CLI

This is an experiment to explore whether
1. the fiboa Schema language can be used to simplify Overture's JSON Schemas
2. the fiboa CLI can somewhat work with Overture data
3. the fiboa CLI can handle large amounts of data
4. the fiboa approach could work for Overture

This experiments with the Overture Maps data, specifically we look at the
`buildings` theme and the sub type `building`.

## Data Download

First of all, we need to download Overture data:
`aws --no-sign-request s3 cp s3://overturemaps-us-west-2/release/2024-04-16-beta.0/theme=buildings/type=building/ /mnt/d/overture/`

Assuming we download to: `/mnt/d/overture/`

## Schema Creation

We also need to convert the JSON Schema into a fiboa Schema:
[building.yaml](building.yaml)

Checking for validity of the schema:
`fiboa validate-schema building.yaml`

Notes:
- `$comment`s, `description`s and `title`s were removed for simplicity
  but can be used in fiboa Schema if needed
- Fiboa schema has no support `unevaluatedProperties`.
  `unevaluatedProperties` is not needed as we don't support references or allOf/anyOf/oneOf etc.
- The chosen data types were guesses depending on the expected values

## Validate GeoParquet

Example:
`fiboa validate /mnt/d/overture/ -s building.yaml -f 0.2.0 --data --timer`

Identified issues:
  - theme: Required field is missing
  - type: Required field is missing
  - id: Nullability differs, is True but must be False
  - geometry: Nullability differs, is True but must be False
  - bbox: Nullability differs, is True but must be False
  - bbox: Validating bounding-box is not supported yet
  - version: Nullability differs, is True but must be False
  - version: Data type invalid, is int32 but must be uint32
  - update_time: Nullability differs, is True but must be False
  - update_time: Data type invalid, is string but must be date-time
  - names: Data type invalid, is struct<primary: string, common: map<string, string ('common')>, rules: list<element: struct<variant: string, language: string, value: string, between: list<element: double>, side: string>>> but must be object
  - level: Data type invalid, is int32 but must be int16
  - height: Data type invalid, is double but must be float
  - num_floors: Data type invalid, is int32 but must be uint8
  - min_height: Data type invalid, is double but must be float
  - min_floor: Data type invalid, is int32 but must be uint8
  - roof_direction: Data type invalid, is double but must be float
  - eave_height: Data type invalid, is double but must be float

Take aways:
- All fields seem to be generally nullable although not needed
- The "largest" data types are chosen instead of suitable smaller ones (e.g. int32 or double instead of int8 or float)
- unsigned data types are not chosen if the minimum is set to >= 0
- Timestamps are exported as strings instead of in the corresponding temporal data types
- The schema contains the fields `theme` and `type`, but they don't exist in the GeoParquet file


## Create GeoJSON from GeoParquet

Example:
`fiboa create-geojson /mnt/d/overture/part-00000-530a22ea-6b33-45e9-961b-567f661900b0-c000.zstd.parquet -o example`

Validate GeoJSON against Overture JSON Schema:

`npm install -g ajv-cli ajv-formats`

`ajv validate -s C:\Dev\overture-schema\schema\buildings\compiled-schema.yaml -d D:\overture\example\08bf2a40219b0fff0200c394dae731bd.json --spec=draft2020  -c ajv-formats`

## Create GeoParquet from GeoJSON

Example:
`fiboa create-geoparquet example/features.json -o example/features.parquet  -s building.yaml -f 0.2.0`

## Compare GeoParquet files

... file size, memory usage, better data types, ...
