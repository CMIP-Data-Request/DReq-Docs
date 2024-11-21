# Data request software

The [data request software](https://github.com/CMIP-Data-Request/CMIP7_DReq_Software) provides python code allowing users to interact programatically with the [CMIP7 data request](https://wcrp-cmip.org/cmip7/cmip7-data-request/).
It will provide an API and scripts that can produce lists of the variables requested for each CMIP7 experiment, information about the requested variables, and in general support different ways of querying and utilizing the information in the data request.


## Overview

The CMIP7 data request **Software** and **Content** are version controlled in separate github repositories.
Official releases of the data request correspond to a tag in each of these repositories (e.g., `v1.0beta`). 
However the Software can interact with different versions of the Content - for example, to examine changes that have occurred when a new version of the data request is issued.

The data request **Content**, which is version controlled [here](https://github.com/CMIP-Data-Request/CMIP7_DReq_Content), refers to the information comprising the data request. 
This includes descriptions of Opportunities and their lists of requested variables, definitions of the variables, etc.
The Content is stored as `json` and read by the data request Software as its input, but users should not interact with this `json` directly and its structure is not designed for readability.
Users do not need to manually download the Content as this is done automatically by the Software (see "Getting Started", below, for further details).

The data request Content is an automatic export from Airtable, a cloud platform used by the Data Request Task Team to facilitate ongoing community engagement in developing the data request.
Airtable provides users with a browseable web interface to explore data request information contained in relational databases that are referred to as "bases".
These Airtable bases contain interlinked tables that constitute the primary source of data request information.

The Content of each official release of the data request can be explored online using the [Airtable interface](https://bit.ly/CMIP7-DReq-v1_0beta).
This provides a browseable web view of the Content, allowing users to follow links between different elements of the data request - for example, to view the variables requested by a given Opportunity, or to view the Opportunities that request a given variable.
This view is complementary to the access to the Content that is provided via the Software, and both access methods (Airtable and Software) are based on the same underlying information about the data request.


Using the data request **Software** provides a way to interact programmatically with the data request Content, such as to:

- Given a list of supported opportunities and their priorities, produce lists of variables to output for each experiment (see Getting Started section to test this functionality),
- Output the CF-compliant metadata characterizing each variable (an example file with some of the metadata for each requested variable is available in v1.0beta),
- Compare the requested output of CMIP7 experiments to a given model's published CMIP6 output (not yet available with v1.0beta, to come in a commit to come soon before v1.0),
- Estimate output volumes (not yet available with v1.0beta, to come in a commit to come soon before v1.0).

The Software should facilitate integration of the data request into modelling workflows.
Suggestions for functionality are welcome in the [github discussion forum](https://github.com/CMIP-Data-Request/CMIP7_DReq_Software/discussions).


During development, the Software and Content repositories reside in the github organisation https://github.com/CMIP-Data-Request.
Stable releases will eventually be migrated into the https://github.com/WCRP-CMIP organisation.


## Getting started

To begin, clone the Software and navigate to the `scripts/` directory:
```
git clone git@github.com:CMIP-Data-Request/CMIP7_DReq_Software.git
cd CMIP7_DReq_Software/scripts
```
The `env.yml` file can be used to create a conda environment in which to run the Software:
```
conda env create -n my_dreq_env --file env.yml
```
replacing `my_dreq_env` with your preferred environment name. 
Activate this environment:
```
conda activate my_dreq_env
```
and run the the example script:
```
python workflow_example.py
```
This script contains a workflow to access the data request Content, specify a list of Opportunities and priority levels of variables, and output the lists of variables requested from each experiment in the specified Opportunities.
Each listed variables is currently identified by a unique "compound name" using CMIP6-era table names and short variable names (`Amon.tas`, `Omon.tos`, etc).
Variable names may change in upcoming releases, but in any case a mapping to CMIP6-era variable names will be retained in the data request so as to allow comparison with CMIP6 output (for those variables that were defined in CMIP6).

To access the data request Content, the example script first needs to identify the version of the data request Content that is being used. 
This is done by specifying a tag in the Content repo and calling the retrieval function.
For example:
```
dc.retrieve('v1.0beta')
```
downloads `v1.0beta` of the Content into local cache, if it is not already there.
The script can then access it by loading it into a python dict variable:
```
content = dc.load('v1.0beta')
```
Currently a single version of the Content `json` file is roughly 20 MB in size.
The size of local cache can be managed by deleting unused versions.
For example, to remove a specific version:
```
dc.delete('v1.0beta')
```
Or to remove all locally cached versions:
```
dc.delete()
```

### Command line tool

The `export_dreq_lists_json.py` script is a simple command line tool
with options to allow the user to control the output by 

1. Constraining the list of opportunities that the user wishes to 
   respond to.
1. Specifying the experiments to be included in the output.
1. Specifying the filename to write the output to.

The command syntax is shown using the help shown below along with a 
couple of usage examples

!!! info "help text for export script"

    ```
    $ export_dreq_lists_json.py --help
    usage: export_dreq_lists_json.py [-h] [--opportunities_file OPPORTUNITIES_FILE] [--all_opportunities] [--experiments EXPERIMENTS [EXPERIMENTS ...]] {v1.0alpha,first_export,v1.0beta,v1.0-alpha} output_file

    positional arguments:
    {v1.0alpha,first_export,v1.0beta,v1.0-alpha}
                            data request version
    output_file           file to write JSON output to

    options:
    -h, --help            show this help message and exit
    --opportunities_file OPPORTUNITIES_FILE
                            path to JSON file listing opportunities to respond to. If it doesn't exist a template will be created
    --all_opportunities   Respond to all opporunities
    --experiments EXPERIMENTS [EXPERIMENTS ...]
                            limit output to the specified experiments
    ```


!!! info "Generate variable list for all opportunities, but for `historical` experiment only"

    ``` 
    $ export_dreq_lists_json.py v1.0beta historical.json --all_opportunities --experiments historical

    For data request version v1.0beta, number of requested variables found by experiment:
    historical : Core=132 ,High=1011 ,Medium=401 ,Low=98, TOTAL=1642

    Wrote requested variables to historical.json
    ```
    The historical.json file will then look similar to 
    ```
    {
        "Header": {
            "Opportunities": [
                "Accurate assessment of land-atmosphere coupling",
                "Advancing Wind-Wave Climate Modelling for Coastal Zone Dynamics, Impacts, and Risk Assessment through International Collaborative Research Projects as Part of the Coordinated Ocean Wave Climate Project (COWCliP)",
                "AerChemMIP",
                ...
                "Water cycle/budget assessment",
                "Wind driven Ocean Surface Waves "
            ],
            "dreq version": "v1.0beta"
        },
        "experiment": {
            "historical": {
                "Core": [
                    "3hr.huss",
                    "3hr.pr",
                    "3hr.tas",
                    "3hr.uas",
                    "3hr.vas",
                    ...
    ```

!!! info "Generate variable list for one opportunity for all experiments"

    Generate opportunity json file (this must not exist when you run this command)
    ```
    $ export_dreq_lists_json.py v1.0beta all_experiments.json --opportunities opportunity_selection.json
    written opportunities dict to opportunity_selection.json. Please edit and re-run
    ```
    The `opportunity_selection.json` file will contain a set of key
    value pairs;
    ```
    {
    "Sea ice changes, drivers and impacts": true,
    "Synoptic systems and impacts": true,
    "Atmospheric dynamics and variability ": true,
    "Role of fire in the Earth system": true,
    "Temperature variability": true,
    ...
    "Causality of Polar Amplification ": true,
    "Changes in marine biogeochemical cycles and ecosystem processes": true
    }
    ```
    Edit this file and change all `true` values to false other than for 
    the `"Temperature variability"` key. Re-running the 
    `export_dreq_lists_json.py` command gives
    ```
    $ export_dreq_lists_json.py v1.0beta all.json --opportunities opportunity_selection.json
    For data request version v1.0beta, number of requested variables found by experiment:
        1pctCO2 : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        1pctCO2-bgc : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        1pctCO2-rad : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        High scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Initialised prediction (2025-2036) : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Low overshoot scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Low scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Medium scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Overshoot scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        Very low scenario : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        abrupt-4xCO2 : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        amip : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        amip-p4k : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        esm-hist : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        esm-piControl : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        hist-GHG : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        hist-aer : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        hist-nat : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        historical : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        newGeoMIP : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32
        piControl : Core=0 ,High=16 ,Medium=16 ,Low=0, TOTAL=32

    Wrote requested variables to all.json
    ```
    The top of the output file `all.json` then contains
    ```
    {
        "Header": {
            "Opportunities": [
                "Temperature variability"
            ],
            "dreq version": "v1.0beta"
        },
        "experiment": {
            "1pctCO2": {
                "Core": [],
                "High": [
                    "day.hfls",
                    "day.hfss",
                    "day.huss",
                    "day.mrsos",
                    "day.rlds",
                    ...
    ```