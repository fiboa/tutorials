# Contributing

All contributions are subject to the
[fiboa Code of Conduct](https://github.com/fiboa/specification/blob/main/CODE_OF_CONDUCT.md).
For contributions, please follow the
[fiboa contributing guideline](https://github.com/fiboa/specification/blob/main/CONTRIBUTING.md).

## Running tests

You'll need to install Python >= 3.9 and pip to setup the test environment.
We use pipenv to execute the tests.

Start with the following command in the folder where this README is located:
`pip install pipenv --user`

Finally, you can run the tests as follows:

- To check the markdown run: `pipenv run test-docs`
- To check the fiboa schema run: `pipenv run test-schema`
- To check the examples run:
  - `pipenv run test-geojson` for GeoJSON
  - `pipenv run test-geoparquet` for GeoParquet
- To create a GeoParquet from the GeoJSON examples: `pipenv run create-geoparquet`
