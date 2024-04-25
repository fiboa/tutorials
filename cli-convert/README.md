# Convert custom data with fiboa CLI

**Note:** This tutorial is also covered in a video:
<https://www.youtube.com/watch?v=-SUDzug29Cg&list=PLENrKR4uOfvXH-bDf1ornXgO6NdEL25ZS&index=3>

This tutorial shows how to convert tabular/columnar datasets (e.g. Shapefiles, GeoPackages)
into a fiboa-compliant GeoParquet file.

We'll create a custom converter for this based on a pre-made template that is shipped with
the CLI source code and then we can run it at any time using the fiboa CLI.

The general high-level procedure here is:

- Install fiboa CLI from source
- Create a data survey if needed
- Copy and fill the template
- Add missing dependencies into a separate dependency group
- Publish the converter (update README and create PR)

A guide how to create a cloud-native fiboa dataset using fiboa CLI is
available at: <https://github.com/fiboa/data/blob/main/HOWTO.md>
It includes more steps than this tutorial as it also includes publishing and updating the
converted data on [Source Cooperative](https://source.coop) with additional data, e.g. README and PMTiles.

Having that said, let's create a converter based on the template.
This will guide you through the procedure step-by-step.

In this tutorial we'll convert the Thuringia, Germany dataset that has been surveyed
here: <https://github.com/fiboa/data-survey/blob/main/data/DE-TH.md>
Most of the information that we'll use here are coming directly from this survey.

## Intro & Setup

1. You will need to have **Python 3.9** or any later version installed.
2. You need the fiboa CLI source code from GitHub on your machine:
   <https://github.com/fiboa/cli>

   Ideally fork the repository and then clone it so that you can later create a PR for it.
   If you don't want to fork it and you have git installed,
   you can run `git clone https://github.com/fiboa/cli.git`
3. Switch into the folder that contains the source code, e.g. `cd cli`.
4. Install the CLI and its dependencies in editable dev mode: `pip install -e .`

### Create the converter

5. Copy the file `fiboa_cli/datasets/template.py` and rename it to something sensible.
   The filename will be the name that use to run the converter.
   If you name it 'xy_abc' you'll be able to run `fiboa convert xy_abc` in the CLI.
6. Open the newly created file in your favorite editor.
   You'll see a Python script with a lot of variables that should hopefully all be explained
   through the code comments.
7. Start filling the template:
    - Add two imports that we'll need later:
      ```py
      import re
      import pandas as pd
      ```
    - Set `URL` to `"https://www.geoproxy.geoportal-th.de/download-service/opendata/agrar/DGK_Thue.zip"`
      which is the download URL of the dataset we want to convert.
    - The following variables until `EXTENSIONS` are mostly for metadata pruposes.
      The data provided there will be written to the Collection metadata, in our case to a STAC Collection.
      I use example data below, but you could use anything you like.
       - `ID` => `"de_th"`
       - `TITLE` => `"Field boundaries for Thuringia, Germany"`
       - `DESCRIPTION` => 
          ```md
          """
          For use in the application procedure of the Integrated Administration and Control System (IACS), digital data layers are required that represent the current situation of agricultural use with the required accuracy. The field block is a contiguous agricultural area of one or more farmers surrounded by permanent boundaries. The field block thus contains information on the geographical location of the outer boundaries of the agricultural area. Reference parcels are uniquely numbered throughout Germany (Feldblockident - FBI). They also have a field block size (maximum eligible area) and a land use category.

          The following field block types exist:

          - Utilized agricultural area (UAA)
          - Landscape elements (LE)
          - Special use areas (SF)
          - Forest areas (FF)

          The field blocks are classified separately according to the main land uses of arable land (`AL`), grassland (`GL`), permanent crops (`DA`, `OB`, `WB`), including agroforestry systems with an approved utilization concept and according to the BNK for no "agricultural land" (`NW`, `EF` and `PK`) and others.

          Landscape elements (LE) are considered part of the eligible agricultural area under defined conditions. In Thuringia, these permanent conditional features are designated as a separate field block (FB) and are therefore part of the Thuringian area reference system (field block reference). They must have a clear reference to an UAA (agricultural land), i.e. they are located within an arable, permanent grassland or permanent crop area or border directly on it.

          To produce the DGK-Lw, (official) orthophotos from the Thuringian Land Registry and Surveying Administration (TLBG) and orthophotos from the TLLLR's own aerial surveys are interpreted. The origin of this image data is 50% of the state area each year, so that up-to-date image data is available for the entire Thuringian state area every year.
          """
          ```
       - `BBOX` => `[9.8778443239,50.2042330625,12.6531964048,51.6490678544]`
         I've copied the bounding box from <http://osmtipps.lefty1963.de/2008/10/bundeslnder.html>
       - `PROVIDER_NAME` = `"Thüringer Landesamt für Landwirtschaft und Ländlichen Raum"`
       - `PROVIDER_URL` = `"https://geomis.geoportal-th.de/geonetwork/srv/ger/catalog.search#/metadata/D872F2D6-60BC-11D6-B67D-00E0290F5BA0"`
       - `ATTRIBUTION` = `"© GDI-Th"`
       - `LICENSE` = `"dl-de/by-2-0"`
    - Now we actually get to the data descriptions.
      We need to create a mapping between the existing columns and the fiboa property names, identify which extensions apply, and check whether the data values need to be transformed.
      Proposed mapping for the columns:
      ```py
      COLUMNS = {
          'geometry': 'geometry', # fiboa Core field
          'BEZUGSJAHR': 'valid_year', # Custom field
          'FBI': 'flik', # FLIK extension
          'FBI_KURZ': 'id', # fiboa Core field
          'FB_FLAECHE': 'area', # fiboa Core field
          'FBI_VJ': 'flik_last_year', # Custom field
          'FB_FL_VJ': 'area_last_year', # Custom field
          'TK10': 'tk10', # Custom field
          'AFO': 'afo', # Custom field
        # Don't add LF, all values are 'LF' after the filter below
        # 'LF': 'lf',
          'BNK': 'bnk', # Custom field
          'KOND_LE': 'kond_le', # Custom field
          'AENDERUNG': 'change', # Custom field
          'GEO_UPDAT': 'determination_datetime' # fiboa Core field
      }
      ```
      The `LF` column only consists of the same value (see filters below) so we don't include
      it in the new file. If you don't list a field in the mapping, it will be omitted from the
      newly created file.
    - As we've used one extension, we also need to add the extension identifier to the list of extensions:
      `EXTENSIONS = ["https://fiboa.github.io/flik-extension/v0.1.0/schema.yaml"]`
    - Looking at the data values, e.g. in QGIS, or the data documentation,
      we can see that some values are stored in a way which is not intuitive or
      they use German values.
      We want to change that and can use column migrations for that.
      Column migrations receive a Pandas Series and operations can run Pandas operations on them to modify the data.
      The keys must still be the original column names here.
      ```py
      COLUMN_MIGRATIONS = {
          # Source values are either J or None. Change to boolean True and False.
          'AFO': lambda column: column.map({'J': True}).fillna(value=False),
          # Same as above
          'KOND_LE': lambda column: column.map({'J': True}).fillna(value=False),
          # Source values are either Geaendert (changed) or Unveraendert (unchanged) or Neu (new).
          # As we renamed the column to "changed", the following mapping seems natural:
          # Geaendert => True
          # Unveraendert => False
          # Neu => None
          'AENDERUNG': lambda column: column.map({'Geaendert': True, 'Unveraendert': False, 'Neu': None}),
          # The list of ID that were used in the previous years is a string separated by comma, sometimes including spaces.
          # It feels much more natural to store this as an array of strings.
          'FBI_VJ': lambda column: column.str.split(re.compile(r'\s*,\s*'), regex = True),
          # Convert from the German time format (day dot month dot year) to datetime objects
          'GEO_UPDAT': lambda column: pd.to_datetime(column, format = "%d.%m.%Y", utc = True)
      }
      ```
    - The dataset contains not just field boundaries, but also other boundaries, for example forests.
      We'll filter the dataset and only keep the rows where the `LF` column has the value `LF`.
      That's also the reason why we don't include the LF column in the mapping above.
      All values will be `LF` anyway, so it doesn't offer a lot of value.
      The keys must still be the original column names here.
      ```py
      COLUMN_FILTERS = {
        "LF": lambda col: col == "LF"
      }
      ```
    - We have to define schemas for the fields that are not defined in fiboa.
      Here, the keys must be the values from the `COLUMNS` dict, not the keys as before.
      This is used to choose the proper data types for the GeoParquet data.
      Unfortunately, the schemas won't be used for validation later.
      The required fields are non-nullable.
      ```py
      MISSING_SCHEMAS = {
          'required': [
              'valid_year',
              'area_last_year',
              'tk10',
              'bnk'
          ],
          'properties': {
              'valid_year': {
                  # could also be uint16 or string
                  'type': 'int16'
              },
              'flik_last_year': {
                  'type': 'array',
                  'items': {
                      # as define in the flik extension schema
                      'type': 'string',
                      'minLength': 16,
                      'maxLength': 16,
                      'pattern': "^[A-Z]{2}[A-Z0-9]{2}[A-Z0-9]{2}[A-Z0-9]{10}$"
                  }
              },
              'area_last_year': {
                  # as define in the area schema
                  'type': 'float',
                  'exclusiveMinimum': 0,
                  'maximum': 100000
              },
              'tk10': {
                  'type': 'string'
              },
              'afo': {
                  'type': 'boolean'
              },
              'bnk': {
                  'type': 'string'
              },
              'kond_le': {
                  'type': 'boolean'
              },
              'change': {
                  'type': 'boolean'
              }
          }
      }
      ```
    - We don't need `MIGRATIONS` in this example so keep it as is.
      We also don't need to update the `convert` methods itself.
    - Save the file
  
We've finished creating the converter and can run it.

### Run the converter and validate

8. We can run the converter now:
   `fiboa convert de_th -c temp.zip -o de_th.parquet`
   - `-o` defines where to store the GeoParquet file, here in the same folder with the name `de_th.parquet`
   - `-c` is used to cache the file locally to `temp.zip`,
     so if you want to run the conversion again it doesn't download the file again.
     Especially useful while developing your converter.
     If you just want to run the converter once or want the data freshly downloaded, omit this option.
   - The converter has two additional options (`-h` and `--collection`).
     You can check their descriptions by running `fiboa convert --help`
9. We can inspect it:
   `fiboa describe de_th.parquet -j`
10. We can also validate it:
   `fiboa validate de_th.parquet --data`
11. We can also drag&drop it into QGIS for example.

### Publish the converter

We'll not go deeply into this, but for completeness I quickly mention it.

12. If you are happy with the results, you can go the normal GitHub flow to create a PR.
13. Fork the repository (if not done in step 2)
14. Commit the newly created file to a new branch
15. Create a new PR: <https://github.com/fiboa/cli/compare>

### Publish the data

You can now have a deeper look into how to publish the data.
A recommended approach is discussed in the previously mentioned document:
<https://github.com/fiboa/data/blob/main/HOWTO.md>
