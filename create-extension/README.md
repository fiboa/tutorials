# Creating a fiboa extension

**Note:** This tutorial is also covered in a video:
<https://www.youtube.com/watch?v=xQzZ_t5EkFI&list=PLENrKR4uOfvXH-bDf1ornXgO6NdEL25ZS&index=4>

In this tutorial we'll create a new fiboa extension from scratch and 
make use of it in a dataset.

The list of existing extensions:
<https://fiboa.github.io/extensions>

A list of proposed extensions:
<https://github.com/fiboa/extensions/issues>  
Feel free to add your ideas or take over specifying any of them!


## Idea for an extension

We'll create an exemplary extension that will provide information 
about the intended future use of a field.

*Note: This example may not make any sense, it's made up for the sake of this tutorial with properties that cover a wide variety of data types.*

Let's say we've identified the following properties as valuable for our extension that we want to name "Future Extension".

The field prefix is meant to be `ftr`.

| Name           | Constraints                                                  | Description                                                |
| -------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| next_crop_type | Required - text, any of:  sugarcane, maize, rice, wheat, or other | The crop type that is planned to be planted next.          |
| plant_date     | date                                                         | The date when the next crop type is planned to be planted. |
| grow_time      | number of days                                               | The rough time until the crop is harvested, in days.       |

Generally, an extension should consist a small number of properties that are closely related to each other.
Don't mix to many properties into a single extension, better split them into multiple extensions to keep the scope very specific.

## Create a new repository

1. We need to create a new repository.
   There is a detailed instruction here:
   <https://github.com/fiboa/extensions/blob/main/README.md#create-the-repository>

   - This extension shall be created in the `fiboa` organization, i.e. the owner is 'fiboa'.
      You'll probably create it somewhere else.
      In this case remember to replace the `fiboa` organization name with your account or organization name.
   - The repository name shall be `future-extension`
   - The description shall be:
   
      > Provides information about the intended future use of a fields.
   
2. We need to clone the new repository (e.g. `git clone https://github.com/fiboa/future-extension`)
   and navigate to it (e.g. `cd future-extension`) in the CLI.

3. We use the fiboa CLI to [rename the placeholders in the template](https://github.com/fiboa/cli?tab=readme-ov-file#update-an-extension-template-with-new-names):
   `fiboa rename-extension . -t Future -p ftr -s future-extension -o fiboa`
   *You'll need Python 3.9 or later and fiboa CLI 0.3.8 or later installed!*
   You can check the changes and commit the changes back to GitHub.

## Write the extension

We need to update a couple of files.
The most important files and folders are:

- `README.md`: The actual specification text
- `schema/schema.yaml`: The formal schema for the properties we describe in the README
- `examples/`: The examples folder ideally contains real world examples in all fiboa encodings,.
  currently GeoJSON and GeoParquet.

### README.md

The first file we want to update is the `README.md`:

- First, we'll also update the 'Extension Maturity Classification' to 'Proposal'.

- Then, you'll update the owner of the extension to be your GitHub handle, in my case `@m-mohr`

- We'll also replace the extension description. Find:

  > This is the place to add a short introduction.

  and replace it with:

  > This extension will provide information about the intended future use of a field.

- Next, we'll update the properties, select data types and descriptions, select where they can be used, add further documentation, etc.

  The data type selection must follow the [fiboa Schema data types](https://github.com/fiboa/schema/blob/main/datatypes.md).

  - The first property will be named `ftr:next_crop_type` and is of data type `string`

  - The second property will be named `ftr:plant_datetime` and is of data type `date`

  - The third property will be named `ftr:grow_time` and is of data type `uint16`.

    First of all, we are choosing an unsigned integer because the values logically can't be smaller than 0.
    A normal year consists of 365 days and grow times are often probably <= 255 days, so we could have chosen `uint8`.
    But there are likely exceptions to this rule and thus I've chosen `uint16`, which gives us plenty of space (actually nearly 180 years).

  - We could end up with a table such as:

    | Property Name      | Type   | Description                                                  |
    | ------------------ | ------ | ------------------------------------------------------------ |
    | ftr:next_crop_type | string | **REQUIRED.** The crop type that is planned to be planted next. One of: `sugarcane`, `maize`, `rice`, `wheat`, `other` |
    | ftr:plant_date     | date   | The date when the next crop type is planned to be planted.   |
    | ftr:grow_time      | int16  | The rough number of days until the crop is meant to be harvested, must be > 0. |

- We can save the file now.

- The next step is to check whether the README follows the Markdown best-practices:

  We use pre-defined scripts that are define in the PipFile, which needs to be installed once:

  - `pip install pipenv --user`

  Now you can run the script that checks the Markdown file. You can run this as often as you want until you've fixed all issues.

  - `pipenv run test-docs`

  If the command doesn't show anything it means that the documents are valid.

### schema/schema.yaml

The next file to update is the `schema/schema.yaml`, which shall reflect the table above.
The schemas must comply to [fiboa Schema](https://github.com/fiboa/schema).

- We need to update the list of required fields first:
  ```
  required:
    - ftr:next_crop_type
  ```

- Next we'll add the individual schemas for the properties:

  - for `ftr:next_crop_type`:
    ```yaml
    type: string
    enum:
      - sugarcane
      - maize
      - rice
      - wheat
      - other
    ```

  - for `ftr:plant_date`:
    ```yaml
    type: date
    ```
    
  - for `ftr:grow_time`:
    ```yaml
    type: int16
    minimum: 1
    ```

- We can save the file now.

- The next step is to validate the schema: `pipenv run test-schema`

### Examples

All examples reside in the `examples/` folder.

The easiest step is to create GeoJSON example files and then convert them to GeoParquet
with the [fiboa CLI](https://github.com/fiboa/cli?tab=readme-ov-file#create-fiboa-geoparquet-from-geojson).

The GeoJSON example can either be provided as a single GeoJSON FeatureCollection or as multiple GeoJSON Features.
The extension template by default contains a single Feature.

To update the example, open the `examples/geojson/example.json` and replace

```json
    "ftr:field1": 1,
    "ftr:field2": "abc"
```

with

```json
    "ftr:next_crop_type": "maize",
    "ftr:plant_date": "2024-05-10",
    "ftr:grow_time": 123
```

The next step is to validate the schema: `pipenv run test-geojson`

This should validate just fine.
You can try and for example set the grow time to -123 and check whether the tests fail.

The `examples/geojon/collection.json` can contain any collection-level metadata,
which by default only contains the fiboa version and the list of implemented extensions.

We can now create a GeoParquet file based on the GeoJSON, too.

- Run `pipenv run create-geoparquet` to create the GeoParquet file.
- Validate the GeoParquet file: `pipenv run test-geoparquet`
- Inspect the GeoParquet file with fiboa CLI: `fiboa describe examples/geoparquet/example.parquet`

### Next steps

- Update the Changelog file: `CHANGELOG.md`
- Let people discuss your extension, e.g. via chat, email, etc.
- Eventually, release the extension via GitHub Releases. This will automatically publish the schemas.
- Add your extension to the [extension list](https://github.com/fiboa/extensions) if it's not hosted in the fiboa GitHub organization.
  All extensions hosted in the fiboa organization will be added automatically each night.
  You can add external/community extensions to the list above by
  editing the [config file](https://github.com/fiboa/extensions/edit/main/config.yaml) and creating a PR from it.

You can find the created extension code here: [future-extension](./future-extension/)
**I copied the code into this repository for reference, it's not published and is not meant to be used.
It only serves as an example!**
