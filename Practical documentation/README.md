# Practical documentation to publish unrecognised files on central NOMAD

This is the practical documentation with the steps to publish your unrecognised files on central NOMAD for different scenarios.
Here are only listed the steps to publish, for explanations about this whole process, please check the `Long documentation` folder.

This documentation is split into different cases to distinguish if the `data` section is used or not in the final entry in NOMAD and if there is an available parser plugin available.
- [No `data` section with available parser](#no-data-parser)
- [No `data` section with no available parser](#no-data-no-parser)
- [`Data` section with available parser](#data-parser)
- [`Data` section with no available parser](#data-no-parser)

## The `data` section is not used
This is the simplest case, where only a JSON file alone can contain all the information necessary to create the entry in central NOMAD.
See for instance the example file `unrecog_data_no_data_section.archive.json`.

<a id="no-data-parser"></a>
### A parser plugin is available
Two options exist to produce the JSON file:
- Install the `nomad-lab` package ([here](https://nomad-lab.eu/prod/rae/docs/client/install.html)) and the parser plugin in your Python environment (`git clone` the repository of the parser and `pip install .` once you are located in the cloned repository). 
Once both are installed, you can used the `parse` command of the `nomad-lab` package to parse locally your file.
See the documentation [here](https://nomad-lab.eu/prod/rae/docs/client/cli_ref.html#parse-cli).\
For instance, you can parse your `unrecog_file.ext` file and produce the JSON entry with this command:
`nomad parse <path>/<to>/unrecog_file.ext --show-archive > <json_name>.archive.json`.

- Go to an Oasis that has the parser plugin and upload the file there. Once the file is parsed, go to the Data section of the entry. 

![Data tab of an entry](../Images/data_tab.png) 

Export the full entry in JSON by using the `<>` button at the top of the 'Entry' column. 

![JSON export of an entry](../Images/json_export_button.png) 

Copy the JSON content. 

![Copy the JSON content](../Images/copy_json_content.png) 

Paste the content into a text editor and save the file with the extension `.archive.json`.
Upload this JSON archive into central NOMAD.

> Note: To be correctly interpreted by NOMAD, the name of the JSON file should end up with `.archive.json`, not only `.json`.

<a id="no-data-no-parser"></a>
### No parser plugin is available
In this case, the JSON archive file has to be written by hand.
The use of the [NOMAD Metainfo Browser](https://nomad-lab.eu/prod/v1/gui/analyze/metainfo/nomad.datamodel.datamodel.EntryArchive) might then be really useful.
By browsing the different sections, one can repeat the same structure within the JSON file.
For instance, if one wants to fill the elemental composition of the sample studied (let's say it is H2O), according to the NOMAD Metainfo Browser, this can be filled in the following quantity `Entry > results > material > elements`.

![Architecture of the elemental composition](../Images/elements_metainfo_tree.png)

The corresponding content in the JSON archive file should then be:
```
{
    "results": {
        "material": {
            "elements": ["H", "O"]
        }
    }
}
```
> Note that there is no `Entry` level in the JSON.

Go on with the rest of the metadata you have to write in the JSON.
However, for big JSON files, the time spent writing the files by hand could better be invested in writing a parser plugin that can do that automatically (see the `Long documentation` for documentation on writing parser plugins).

> Note: To be correctly interpreted by NOMAD, the name of the JSON file should end up with `.archive.json`, not only `.json`.

## The `data` section is used
This case requires the use of a separate schema file to describe the structure of the metadata storage in the `data` section.

<a id="data-parser"></a>
### A parser plugin is available on the Oasis

#### The YAML archive file
The parser plugin should have a Python file describing the schema of the `data` section.
This schema should be translated into a YAML file.
Documentation about YAML schema files can be found [here](https://nomad-lab.eu/prod/v1/staging/docs/howto/customization/basics.html).

For instance, if your Python schema has this structure:
```
class ImportantSubsection(MSection):
    important_date = Quantity(type=str, description="The date of the experiment")

class ImportantSchema(Schema):
    important_information = Quantity(type=int, description="The important information of the experiment)
    important_subsection = SubSection(sub_section=ImportantSubsection.m_def, repeats=False)
```
The translated YAML file should then be:
```
definitions:
  sections:
    ImportantSubsection:
      quantities:
        important_date:
          type: str
          description: The date of the experiment
    ImportantSchema:
      quantities:
        important_information:
          type: int
          description: The important information of the experiment
      sub_sections:
        important_subsection:
          section: ImportantSubsection
          repeats: false
```
> Note: Just like the JSON archive file, the YAML schema file should have `.archive.yaml` as extension in the name of the file.

#### The JSON archive file
The JSON file should be created using the same logic described in the section [above](#no-data-parser).
As an example, the `data` section of the JSON archive could look something like this:
```
{
  "data": {
    "important_information": 42,
    "important_subsection": {
      "important_date": "2025-05-16T12:00:00"
    }
  },
  {... other sections}
}
```

#### Linking the JSON archive to the YAML schema
For NOMAD to understand that the structure of the `data` section in the JSON corresponds to the schema written in the YAML file, one has to specify this in the `m_def` metadata of the `data` section in the JSON file.
This link is called a *reference* and can be done in several ways, described [here](https://nomad-lab.eu/prod/v1/staging/docs/howto/customization/basics.html#different-forms-of-references).
In this case, we will use the references to other files in the same upload or in a different upload, with the references beginning with `../upload`.
For this example, we will assume that the JSON and YAML files will be in the same upload.
In this case, the `data` section of the JSON archive should be like this:
```
{
  "data": {
    "m_def": "../upload/raw/unrecog_schema.archive.yaml#ImportantSchema",
    "important_information": 42,
    "important_subsection": {
      "important_date": "2025-05-16T12:00:00"
    }
  },
  {... other sections}
}
```
> Note: In that case, the name of the schema file is `unrecog_schema.archive.yaml`.

> Note: The YAML schema, being parsed by NOMAD, will create its own entry in the database, only describing the content of the schema. So in this case, by uploading, the JSON and the YAML files, two entries will be produced in the database.

<a id="data-no-parser"></a>
### No parser plugin is available
This is the scenario where both YAML and JSON files have to be written by hand.
In this case, the [documentation](https://nomad-lab.eu/prod/v1/staging/docs/howto/customization/basics.html) to write YAML schema files will be helpful and see the section [above](#no-data-no-parser) to write the content in the JSON file but this time, use the schema file instead of the NOMAD Metainfo Browser to write the architecture of the metadata.
Don't forget to add the `m_def` metadata to link the YAML and JSON files.