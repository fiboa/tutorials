# Working with fiboa CLI

## Intro & Setup
1. You will need to have **Python 3.9** or any later version installed.
2. Run `pip install fiboa-cli` in the CLI to install the validator.
   You need version **0.3.6** for this tutorial.
3. Try running `fiboa` in the CLI to test whether it works.
4. Add e.g. `--help` to show documentation for all options.
    This works for all commands.
    Examples:
   - `fiboa --help`
   - `fiboa describe --help`
   - `fiboa validdate --help`

## fiboa describe

1. Example datasets are available on https://beta.source.coop/fiboa/ 
2. To get a first feeling of the dataset, you can read the description of the dataset
3. On the CLI, you can use `fiboa describe`
4. Copy URL from the README or use a local file, e.g. 
   https://data.source.coop/fiboa/de-sh/Feldbloecke_2024.parquet 
5. Provides information about GeoParquet, Collection, Schema/Columns, data excerpt (10 entries), example:
   - `fiboa describe https://data.source.coop/fiboa/de-sh/Feldbloecke_2024.parquet`
6. Get more insights by printing the JSON metadata for GeoParquet and fiboa Collection. Add `-j`
7. Show a larger excerpt of data. Add `-n 100`
8. Show specific columns.Can be used multiple times. Add e.g. `-c id -c area` to show only the columns id and area.
9. You could also load the GeoParquet file into QGIS, but it won’t give you information about fiboa version and extensions.

## fiboa in a notebook

We can also use the Python code that powers the CLI to run some analysis in a Jupyter Notebook.
This shows the advantage of fiboa. You can now work across multiple datasets and analyse them easily.

- [Open the Jupyter Notebook](./load-fiboa.ipynb)

**NOTE:** The fiboa CLI Python code is not really a public API or library yet.
The imports and methods may change at any time.
This is a proof-of-concept and not really meant for production use cases yet!

## fiboa validate

1. Validates GeoJSON or GeoParquet files according to the schemas, including extensions.
   By default only checks metadata, it doesn’t check actual values in the cells.
   Example:
   - `fiboa validate https://data.source.coop/fiboa/de-sh/Feldbloecke_2024.parquet`
   
2. Add `--data` to also validate the values in the cells.

3. It may show messages such as “No schema defined”, which means that the column is neither defined by the fiboa core spec, but also not part of the extensions that have been defined in the `fiboa_extensions` array in the collection. Thus, the column is not validated at all.

5. This should be enough for most cases, but there are four other options to customize validation for certain use cases, usually development:
   - `-s /path/to/other-schema.yaml`:
     
     fiboa Schema to validate against. Can be a local file or a URL.
     Important for spec development, e.g. when a new version hasn’t been released yet.
   - `-e https://example.com/schema.yaml,/path/to/schema.yaml`:
     
     Maps a remote fiboa extension schema url to a local file. First the URL, then the local file path. Separated with a comma.
     Important for extension development, e.g. when the new version hasn’t been released yet.
   - `-f 0.1.0`:
     
     The fiboa version to validate against if you want to test against other versions than the one that is specified in the Collection.
   - `-c /path/to/collection.json`:
     
     Points to the collection that defines the fiboa version and extensions.
     Should usually not be needed unless you don’t follow the best practice to either link to the collection or embed it into the data.
   
## fiboa create-geoparquet

1. Assuming we have two fiboa compliant GeoJSON files in a folder named features, we can simply run
   - `fiboa create-geoparquet ./features -o test.parquet`

2. It complains about three missing schemas for the properties `flik`, `nutz_code`, and `nutz_txt`.
   - For `nutz_code` and `nutz_txt` there is no extension (yet?), but there is one for `flik` at <https://github.com/fiboa/flik-extension>
   - The extension identifier for `flik` is: <https://fiboa.github.io/flik-extension/v0.1.0/schema.yaml>
   - We can use the `-e` parameter to specify which extension to use:
     - `fiboa create-geoparquet ./features -o test.parquet -e https://fiboa.github.io/flik-extension/v0.1.0/schema.yaml`

3. We can also validate it:
   - `fiboa validate test.parquet --data`

   We get an error reported, something is wrong with one of the FLIK identifiers.

4. With the describe command from before we can inspect the resulting file and have a look:
    - `fiboa describe test.parquet`

   Alright, the identifier of the malformed row is 2713, so let’s fix it in GeoJSON.

5. We can run the commands again:
    1. `fiboa create-geoparquet ./features -o test.parquet -e https://fiboa.github.io/flik-extension/v0.1.0/schema.yaml`

    2. `fiboa validate test.parquet --data`

   Great, now it looks good!

6. If we want to provide more metadata for the Collection, we could use STAC and provide a collection that I’ve prepared:
    - `fiboa create-geoparquet ./features -o test.parquet -c ./features/collection.json`

7. We can also inspect it again, this time we ask for the collection details specifically:
    - `fiboa describe test.parquet -j`

## fiboa create-geojson

1. If you want to go the other way around and create GeoJSON from GeoParquet, this is also possible. We could just try to extract the Features (and Collection) from the GeoParquet file we just created:
    - `fiboa create-geoparquet ./features -o test.parquet -c ./features/collection.json`

2. That’s hard to read, I want the files to be better formatted, so I add `-i 2`

3. We got a FeatureCollection, but I want individual features again. So I add `-f`

4. If you just want to export a certain number of features for testing purposes, add for example `-n 123` to export a maximum of 123 features.

5. We can compare the resulting files now. The sort of the properties has changed, but that doesn’t matter. And it added a link to the collection metadata. But everything else is exactly the same.

## Other tutorials
- [Write a converter with fiboa CLI](../cli-convert/README.md) (April, 25th). There we'll use:
  - `fiboa convert`
- [How to create an extension](../create-extension/README.md) (April, 29th). There we'll use:
  - `fiboa validate-schema` - to validate the schemas we create for the core specification and extensions
  - `fiboa rename-extension` - to update an extension template with different ids, names, etc.