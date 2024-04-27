# Future Extension Specification

- **Title:** Future
- **Identifier:** <https://fiboa.github.io/fututre-extension/v0.1.0/schema.yaml>
- **Property Name Prefix:** ftr
- **Extension Maturity Classification:** Proposal
- **Owner**: @m-mohr

This document explains the Future Extension to the
[Field Boundaries for Agriculture (fiboa) Specification](https://github.com/fiboa/specification).

This extension will provide information about the intended future use of a field.

- Examples:
  - [GeoJSON](examples/geojson/)
  - [GeoParquet](examples/geoparquet/)
- [Schema](schema/schema.yaml)
- [Changelog](./CHANGELOG.md)

## Properties

The properties in the table below can be used in these parts of fiboa documents:

- [ ] Collection
- [x] Feature Properties

| Property Name      | Type   | Description                                                  |
| ------------------ | ------ | ------------------------------------------------------------ |
| ftr:next_crop_type | string | **REQUIRED.** The crop type that is planned to be planted next. One of: `sugarcane`, `maize`, `rice`, `wheat`, `other` |
| ftr:plant_date     | date   | The date when the next crop type is planned to be planted.   |
| ftr:grow_time      | int16  | The rough number of days until the crop is meant to be harvested, must be > 0. |

## Contributing

See the [contributing guideline](CONTRIBUTING.md) for more details.
