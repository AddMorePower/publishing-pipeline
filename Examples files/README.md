# Test files to upload on NOMAD
This folder contains some basic examples files to apprehend the behaviour of NOMAD when it is confronted to certain files.
The idea is that anyone can drop these files in an upload and see what happens.
Each file will be presented below with a small explanation of what happens when they are dropped and why.

- [unrecog.txt](#unrecogtxt)
- [unrecog_data.json](#unrecog_datajson)
- [unrecog_data.archive.json](#unrecog_dataarchivejson)
- [unrecog_schema.archive.yaml](#unrecog_schemaarchiveyaml)
- [unrecog_data_no_data_section.archive.json](#unrecog_data_no_data_sectionarchivejson)
- [unrecog_data_no_schema_link.archive.json](#unrecog_data_no_schema_linkarchivejson)

> **Note**: Dropping several of these files at once in an upload might require a reprocessing of the entries if one of them fails (especially for the YAML schema file)

## unrecog.txt
This file is there to represent a scientific file that is not recognised by any of the parsers of NOMAD.
This situation is the whole purpose of the documentation.
As a consequence, if you drop this file in an upload, then you will see nothing happen because NOMAD doesn't know how to read its content.

## unrecog_data.json
This file is here to poit out the importance of a good naming for a JSON file.
As pointed out in the [documentation](../Long%20documentation/README.md#important-point-the-archivejson-extension), any JSON file (or YAML file) dropped in NOMAD to be interpreted as a JSON containing data (YAML containing schema respectively) *must* have the `.archive.json` extenstion.

## unrecog_data.archive.json
This is the type of JSON file one wants to upload alongside the unrecognised files in the upload.
This file contains a `results` section as a short example of how this will be displayed later on in the entry and most importantly for our case, a `data` section.
This `data` section is assumed to have been either written by hand or exported from a parsed entry.
The structure used is the one from the `ImportantSchema` section from the `unrecog_schema.archive.yaml` file, in this case, from the same upload.

## unrecog_schema.archive.yaml
This is the type of YAML file one wants to upload alongside the JSON file containing information.
The presence of this file is only necessary if the `data` section is used in the JSON file.
Don't forget to link this file to your JSON file in the `m_def` metadata of the `data` section.

## unrecog_data_no_data_section.archive.json
This JSON file is here to illustrate that an entry can be created based only on a JSON.
This is because the content of the file doesn't feature the `data` section or, at least, it features only sections that NOMAD already understands.
In this case, only the `results` is filled.

## unrecog_data_no_schema_link.archive.json
This JSON file is here to illustrate that for the `data` section to be understood by NOMAD, it needs to be linked to a schema section.
In this case, the `data` and `results` are filled but the `data` section is missing the `m_def` metadata.
First thing to see is that the missing `m_def` is not something that is blocking the parsing of the file, as it should parsed with `SUCCESS`.
Then, once one goes to the "Data" tab, they can see that the `results` section is filled with the elements in the `material` section but that the `data` section is empty, even though some information is stored in the JSON file.

As NOMAD doesn't have a clue about the structure of the information in the `data` section, then it won't store it in the entry.
This is solved by adding the reference to the schema section in the JSON file through the `m_def` metadata.
