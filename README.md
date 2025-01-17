# Global Sensitivity Analysis of an Energy System Optimisation Model

The snakemake workflow to conduct a global sensitivity analysis for an OSeMOSYS Model

## Paper

This repository is used in [the paper](https://doi.org/10.12688/openreseurope.15461.1) currently under review at Open Research Europe.

    @article{10.12688/openreseurope.15461.1,
        author = {Usher, W and Barnes, T and Moksnes, N and Niet, T},
        doi = {10.12688/openreseurope.15461.1},
        journal = {Open Research Europe},
        number = {30},
        title = {Global sensitivity analysis to enhance the transparency and rigour of energy system optimisation modelling [version 1; peer review: awaiting peer review]},
        volume = {3},
        year = {2023},
        Bdsk-Url-1 = {https://doi.org/10.12688/openreseurope.15461.1}}


## Getting Started

### Follow the tutorial

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/KTH-dESA/esom_gsa/envs?labpath=Tutorial.ipynb)

    jupyter notebook Tutorial.ipynb

### Create Slides

    jupyter nbconvert Tutorial.ipynb --to slides --post serve

## Installation

Install snakemake using conda into a new environment called `snakemake`:

```bash
conda install -c conda-forge mamba
mamba create -c bioconda -c conda-forge -n snakemake snakemake-minimal pandas
```

Then, activate the environment using `conda activate snakemake` on Mac and Linux, or `activate snakemake` on Windows.

Now install the other dependencies using pip:

```python
pip install -r requirements.txt
```

## Configure the workflow

### Configuration File: `config.yaml`

The file `config/config.yaml` holds workflow configuration options. These options include the location of the model and accompanying data, solver, and sensitivity analysis parameters. Details on how to modify all configuration parameters are located in the `config.yaml` file.

Datapackage and model file paths can  point to the repositories outside of the gui_workflow. So deployment to an HPC will involve:

1. Copying a release package of OSeMOSYS to the server and unzipping
2. Cloning the gui_osemosys repository to the server
3. Cloning this repository to the server
4. Updating the config so that the paths point to the correct locations

### Parameters file: `parameters.csv`

The file `config/parameters.csv` holds information on what parameters to iterate when performing the sensitivity analysis. The information required is summarized below

column_name | description
:-- | :--
name | the name of the OSeMOSYS parameter file into which the values should be written
group | the group to which the parameter belongs (groups of like names are moved together)
indexes | a string of comma-separated entries matching the OSeMOSYS set elements for the parameter
min_value_base_year | the minimum value that the parameter will be sampled in the base year
max_value_base_year | the maximum value that the parameter will be sampled in the base year
min_value_end_year | the minimum value that the parameter will be sampled in the end year
max_value_end_year | the maximum value that the parameter will be sampled in the end year
dist | the probability distribution - currently, only 'unif' for uniform is supported
interpolation_index | the index name over which the values will be interpolated. Only needed for 'interpolate' actions
action | 'interpolate' will interpolate with a straight-line between the start and end years, where the sample value replaces the end year value; 'fixed' will replace all values in the interpolation index with the sampled value

An example of a correctly formatted CSV file is shown below:

```csv
name,group,indexes,min_value_base_year,max_value_base_year,min_value_end_year,max_value_end_year,dist,interpolation_index,action
CapitalCost,capex,"SIMPLICITY,NGCC",500,600,1000,1100,unif,YEAR,interpolate
CapitalCost,capex,"SIMPLICITY,NGOC",400,500,900,1000,unif,YEAR,interpolate
DiscountRate,discountrate,"GLOBAL",0.04,0.05,0.15,0.20,unif,None,fixed
```

### Results file `results.csv`

The file `config/results.csv` holds information on what variables to run the sensitivity analysis over. The information required is summarized below. If a custom OSeMOSYS model is provided with new sets, the additional set can be added as a new column.

column_name | description
:-- | :--
resultfile | Name of OSeMOSYS Result file
filename | Name of file to write results to
REGION | Model region
TECHNOLOGY | Model technology
FUEL | Model fuel
EMISSION | Model emission
YEAR | Model year

An example of a correctly formatted CSV file is shown below. Note that if no index value is provided, the results will sum over all items in that set. For example, for the `AnnualEmissions` results, the sensitivity analysis will be run on the sum of all annual CO2 emissions in the region `SIMPLICITY` over the model horizon.

```csv
resultfile,filename,REGION,TECHNOLOGY,FUEL,EMISSION,YEAR
TotalDiscountedCost,DiscountedCost,"SIMPLICITY",,,,
NewCapacity,NewCapacityHydro,"SIMPLICITY","HYDRO",,,
NewCapacity,NewCapacityCCNG,"SIMPLICITY","CCNG",,,
AnnualEmissions,Emissions,"SIMPLICITY",,,"CO2",
TotalCapacityAnnual,TotalCapacityAnnualCCNGLastYear,"R1","CCNG",,,2070
```

### Scenarios file `scenarios.csv`

The file `config/scenarios.csv` is used to point the workflow to master models. Use each master models to define macro scenarios - e.g. forcing in and out a key technology.

Importantly, each of the master models is used as a base for the N replicates defined in `config.yaml. If you define 3 master models in this file, and N=100, then 300 model runs will be scheduled, but with the same 100 parameter values.

column_name | description
:-- | :--
name | Index of the scenarion
description | Description of the scenario
datapackage | path to otool datapackage
config | path to otool config file

An example of a correctly formatted CSV file is shown below.

```csv
name,description,datapackage, config
0,"Interconnector Optimised",config/scenarios/scenario_1/datapackage.json, config/scenarios/scenario_1/config.yaml
```

## Running the workflow

To run the workflow, using the command `snakemake --use-conda --cores 4 --resources mem_mb=16000 disk_mb=30000`

You can also change parts of the configuration by adding the `--config` flag, followed by the names of one
or more of the config items. E.g.

    snakemake --use-conda --cores 4 --config filetype=parquet replicates=100

## Plotting the workflow

To visualise the workflow, run the following rule: `snakemake plot_dag --use-conda  --cores 2`

## Folder structure

This repository follows the snakemake guidelines for reproducibility:

    ├── .gitignore
    ├── README.md
    ├── LICENSE.md
    ├── modelrun
    ├── workflow
    │   ├── rules
    |   │   ├── module1.smk
    |   │   └── module2.smk
    │   ├── envs
    |   │   ├── tool1.yaml
    |   │   └── tool2.yaml
    │   ├── scripts
    |   │   ├── script1.py
    |   │   └── script2.R
    │   ├── notebooks
    |   │   ├── notebook1.py.ipynb
    |   │   └── notebook2.r.ipynb
    │   ├── report
    |   │   ├── plot1.rst
    |   │   └── plot2.rst
    |   └── Snakefile
    ├── config
    │   ├── config.yaml
    │   └── some-sheet.csv
    ├── results
    └── resources

## Acknowledgements

This research was financially supported by the European Union’s Horizon 2020 research and innovation programme under the grant agreement No [101022622](https://cordis.europa.eu/project/id/101022622) (European Climate and Energy Modelling Forum [ECEMF](https://ecemf.eu)).

The original version of the computational workflow that was extended for this work was developed by Will Usher under the [Climate Compatible Growth programme](https://climatecompatiblegrowth.com/), which is funded by UK aid from the UK government. The views expressed herein do not necessarily reflect the UK government’s official policies.

Trevor Barnes contribution to this paper was funded via a [Mitacs Globalink Research Award IT2569](https://www.mitacs.ca/en/programs/globalink).
