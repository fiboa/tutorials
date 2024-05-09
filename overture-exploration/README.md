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
- Fiboa schema has no support for `patternProperties` and `unevaluatedProperties` yet.
  `patternProperties` could be added, `unevaluatedProperties` is not needed as we don't support references or allOf/anyOf/oneOf etc.
- The chosen data types were guesses depending on the expected values

## Validate GeoParquet

Example:
`fiboa validate /mnt/d/overture/ -s building.yaml -f 0.2.0 --data --timer`

## Create GeoJSON from GeoParquet

Example:
`fiboa create-geojson /mnt/d/overture/part-00000-530a22ea-6b33-45e9-961b-567f661900b0-c000.zstd.parquet -o example -f -i 2`

## Create GeoParquet from GeoJSON

Example:
``

Validate GeoJSON against Overture JSON Schema:

`npm install -g ajv-cli ajv-formats`

`ajv validate -s C:\Dev\overture-schema\schema\buildings\compiled-schema.yaml -d D:\overture\example\08bf2a40219b0fff0200c394dae731bd.json --spec=draft2020  -c ajv-formats`

## Compare GeoParquet files

... file size, memory usage, better data types, ...

## 