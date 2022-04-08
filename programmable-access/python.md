Access to GMrepo using python through RESTful APIs

## Table of contents
<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Table of contents](#table-of-contents)   
   - [Install and load required libraries](#install-and-load-required-libraries)   
   - [Phenotypes](#phenotypes)   
      - [Get all phenotypes and statistics](#get-all-phenotypes-and-statistics)   
      - [Get statistics on a phenotype](#get-statistics-on-a-phenotype)   
      - [Get associated species of a phenotype](#get-associated-species-of-a-phenotype)   
      - [Get associated genera of a phenotype](#get-associated-genera-of-a-phenotype)   
      - [Calculate species/genera prevalence](#calculate-speciesgenera-prevalence)   
      - [Get associated projects](#get-associated-projects)   
      - [Get associated runs](#get-associated-runs)   
      - [Get relative species/genus abundances in samples/runs associated with a phenotype](#get-relative-speciesgenus-abundances-in-samplesruns-associated-with-a-phenotype)   
   - [Species/genera](#speciesgenera)   
      - [Get an overview of the species/genera](#get-an-overview-of-the-speciesgenera)   
      - [Get summary information of the prevalence and relative abundance of a species/genus in all associated phenotypes](#get-summary-information-of-the-prevalence-and-relative-abundance-of-a-speciesgenus-in-all-associated-phenotypes)   
      - [Get detailed information of the prevalence and relative abundance of a species/genus in all associated phenotypes](#get-detailed-information-of-the-prevalence-and-relative-abundance-of-a-speciesgenus-in-all-associated-phenotypes)   
      - [Get relative species/genus abundances for a sample/run](#get-relative-speciesgenus-abundances-for-a-samplerun)   
      - [Get relative species/genus abundances for a project](#get-relative-speciesgenus-abundances-for-a-project) 
   - [Projects and runs](#projects-and-runs)   

<!-- /MDTOC -->

=======

## Install required modules
Please install the following required modules:
```python
## --install requests
import os
count = 2
while count:
    try:
        import requests
        print('importing module requests...done!')
        break
    except:
        print('Cannot find module named requests, installing...')
        os.system('pip install requests')
        count -= 1
        continue

## --install json
import os
count = 2
while count:
    try:
        import json
        print('importing module json...done!')
        break
    except:
        print('Cannot find module named json, installing...')
        os.system('pip install json')
        count -= 1
        continue

## --install pandas
import os
count = 2
while count:
    try:
        import pandas
        print('importing module pandas...done!')
        break
    except:
        print('Cannot find module named pandas, installing...')
        os.system('pip install pandas')
        count -= 1
        continue

```

```bash
## -- simply install all of them without checking:
## run in your terminal --
pip install requests
pip install json
pip install pandas
```

Load required modules:
```python
import requests
import json
from pandas.core.frame import DataFrame
```

## Phenotypes
### Get all phenotypes and statistics
`input`: none,

`output`: a `DataFrame`
```python
### -- get all phenotypes  --
url = 'https://gmrepo.humangut.info/api/get_all_phenotypes'
pheno_01 = requests.post(url, data={})
pheno_01_cont = pheno_01.json().get('phenotypes')

## -- a DateFrame --
phenotypes = DataFrame(pheno_01_cont)
```
The `list` `pheno_01_cont` contains a list of phenotypes and related statistics as shown in https://gmrepo.humangut.info/phenotypes.

The `data.frame` `all_phenotypes` contains the information of phenotypes and related statistics as shown in https://gmrepo.humangut.info/phenotypes.

### Get statistics on a phenotype
Using the corresponding MeSH ID  (e.g. `D006262` for `Health `), uses can first obtain some statistics information of the phenotype, including:
* nr of associated species/genera
* nr of total/processed/qualified runs
* and other information

`input`: a MeSH ID,

`output`: a `DataFrame`
```python
## -- get summary information by mesh_id
pheno_02_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'https://gmrepo.humangut.info/api/getStatisticsByProjectsByMeshID'
pheno_02 = requests.post(url, data=json.dumps(pheno_02_query))
pheno_02_cont = pheno_02.json()

## --get DataFrame
phenotyp_stats = DataFrame(pheno_02.json())
```
### Get associated species of a phenotype
Again the MeSH ID `D006262` will be needed as the input:

`input`: a MeSH ID,

`output`: a `data.frame`
```python
pheno_03_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'https://gmrepo.humangut.info/api/getAssociatedSpeciesByMeshID'
pheno_03 = requests.post(url, data=json.dumps(pheno_03_query))
pheno_03_cont = pheno_03.json()

## --get DataFrame
phenotyp_assoc_species = DataFrame(pheno_03.json())

## --show data header of the resulting DataFrame
list(phenotyp_assoc_species)
```

### Get associated genera of a phenotype
`input`: a MeSH ID,

`output`: a `data.frame`
```python
pheno_04_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'https://gmrepo.humangut.info/api/getAssociatedGeneraByMeshID'
pheno_04 = requests.post(url, data=json.dumps(pheno_04_query))
pheno_04_cont = pheno_04.json()

## --get DataFrame
phenotyp_assoc_genera = DataFrame(pheno_04.json())

## --show data header of the resulting DataFrame
list(phenotyp_assoc_genera)
```
### Calculate species/genera prevalence
**Prevalence** refers to the percentage of runs in which a species/genus is found out of the total number of valid runs the latter can be found in `phenotyp_stats.stats.ix['nr_valid_samples']`.
```python
## calculate species/genera prevalence for all species associated with Health (D006262):
species_prevalence = phenotyp_assoc_species.samples / phenotyp_stats.stats.ix['nr_valid_samples'] * 100

## then plot (if you have GUI)
species_prevalence.plot(kind='density')

## calculate species prevalence for all genera associated with Health (D006262):
genus_prevalence <- phenotyp_assoc_genera.samples / phenotyp_stats.stats.ix['nr_valid_samples'] * 100

## then plot (if you have GUI)
genus_prevalence.plot(kind='density')

## if you don't have GUI
import matplotlib.pyplot as plt
plt.switch_backend('agg')
fig = species_prevalence.plot(kind='density').get_figure()
fig.savefig('species_prevalence.png')
```
### Get associated projects

`input`: a MeSH ID,

`output`: a `data.frame`
```python
## -- all associted projects --
pheno_05_query = {'mesh_id':'D006262'}
url = 'https://gmrepo.humangut.info/api/getAssociatedProjectsByMeshID'
pheno_05 = requests.post(url, data=json.dumps(pheno_05_query))
pheno_05_cont = pheno_05.json()

## --get DataFrame
phenotyp_assoc_pros = DataFrame(pheno_05.json())

## --show data header of the resulting DataFrame
list(phenotyp_assoc_pros)
```
Please note very often a project may contain samples/runs of multiple phenotypes.

### Get associated runs
Some phenotypes are associated with tens of thousands of runs (e.g. `Health (D006262)`) that are too many to be retrieved with one call. Therefore it may take a two-step procedure to retrieve all runs associated with phenotype.

First, count the number of runs associated with a phenotype:

`input`: a MeSH ID,

`output`: a `vector`
```python
## -- count associated runs --
pheno_06_query = {'mesh_id':'D006262'}  
url = 'https://gmrepo.humangut.info/api/countAssociatedRunsByPhenotypeMeshID'
pheno_06 = requests.post(url, data=json.dumps(pheno_06_query))
pheno_06_cont = pheno_06.json()

## -- the resulting variable is a vector --
phenotyp_nr_assoc_runs = DataFrame(pheno_06.json())
print(phenotyp_nr_assoc_runs)
```

Then users can use a loop retrieve the associated runs, 100 runs at a time:

`input`: a MeSH ID, the number of records to skip, the number of records to retrieve; see below.

`output`: a `data.frame`
```python
## -- get all associted runs --
## use skip = 0, limit = 100 to retrieve the first 100 runs, then
##     skip = 100, limit = 100 to retrieve the next 100 runs ....

pheno_07_query = {'mesh_id':'D006262',"skip":0, "limit":100}  
url = 'https://gmrepo.humangut.info/api/getAssociatedRunsByPhenotypeMeshIDLimit'
pheno_07 = requests.post(url, data=json.dumps(pheno_07_query))
pheno_07_cont = pheno_07.json()

## -- the resulting variable is a vector --
phenotyp_a_page_of_assoc_runs = DataFrame(pheno_07.json())
print(phenotyp_a_page_of_assoc_runs)
```
### Get relative species/genus abundances in samples/runs associated with a phenotype

To get the related information, two input parameters are required:
* MeSH ID of interests, e.g. `D003093` for `Colitis, Ulcerative`
* NCBI taxonomy ID of the species/genus of interests, e.g. `40520` for `Blautia obeum (species)`.

```python
data_query = {'mesh_id':'D003093',"ncbi_taxon_id" : "40520"}  ## -- to get statistics on MeSH ID D006262
url = 'https://gmrepo.humangut.info/api/getMicrobeAbundancesByPhenotypeMeshIDAndNCBITaxonID'
data = requests.post(url, data=json.dumps(data_query))

## --get DataFrames
hist_data_for_phenotype = DataFrame(data.json().get('hist_data_for_phenotype'))
list(hist_data_for_phenotype)
```
The resulting `data` is a list containing:
* `hist_data_for_phenotype`: a data frame contains the distribution of the relative abundances of the species/genus of interests in all samples of current phenotype,
* `hist_data_for_health`: if current phenotype is not `Health`, the  distribution of the relative abundances of the species/genus of interests in all samples of `Health` will also be retrieved,
* `abundant_data_for_disease`: a numeric vector contains the relative abundance data of the species/genus of interests in all samples of current phenotype,
* `abundant_data_for_health`: if current phenotype is not `Health`, the relative abundances of the species/genus of interests in all samples of `Health` will also be retrieved,
* `taxon`: NCBI taxonomy information for current taxonomy ID,
* `disease`: details of current phenotype,
* `abundance_and_meta_data`: runs in which current taxon is found and related meta data,
* `co_occurred_taxa`: cooccurred taxa of the taxon of interests in current phenotype

See https://gmrepo.humangut.info/phenotypes/D003093/40520 for more details.

## Species/genera
### Get an overview of the species/genera

`input`: none,

`output`: a `list`.
```python
### --- get all species and genera that presented in >= 2 runs with median relative abundance >= 0.01%
url = 'https://gmrepo.humangut.info/api/get_all_gut_microbes'
data = requests.post(url, data={}).json()

## --get DataFrames
all_species = DataFrame(data.get('all_species'))
list(all_species)
```
The retrieved `data` is a list containing:
* `all_species`: a `data.frame` that contains all species that presented in >= 2 runs with median relative abundance >= 0.01%,
* `all_genus`: a `data.frame` that contains all genera that presented in >= 2 runs with median relative abundance >= 0.01%,
* `metadata`: a list contains additional statistics:
  * `loaded_samples`: nr. qualified runs for which the relative abundance data are available,
  * `all_species_count`: nr. all species
  * `retrieved_species_count`: nr. species in the `data.frame`: `all_species`,
  * `all_genus_count`: nr. all genera,
  * `retrieved_genus_count`: nr. genera in the `data.frame`: `all_genus`.

With the retrieved data, users can plot the **species prevalence in phenotypes** and **species prevalence in samples**, as shown below:
```python
## IF YOU HAVE GUI!!
## -- species prevalence in samples
DataFrame(data.get('all_species')).pct_of_all_samples.plot(kind='density')

## -- genus prevalence in samples
DataFrame(data.get('all_genus')).pct_of_all_samples.plot(kind='density')

## -- species prevalence in phenotypes --
DataFrame(data.get('all_species')).nr_phenotypes.plot(kind='density')

## -- genus prevalence in phenotypes --
DataFrame(data.get('all_genu')).nr_phenotypes.plot(kind='density')

## if you don't have GUI
import matplotlib.pyplot as plt
plt.switch_backend('agg')
fig = DataFrame(data.get('all_species')).pct_of_all_samples.plot(kind='density').get_figure()
fig.savefig('species_pct_of_all_samples.png')
```

See https://gmrepo.humangut.info/species for more details.

### Get summary information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a `data.frame`
```python
query = {"ncbi_taxon_id":40520}  
url = 'https://gmrepo.humangut.info/api/getPhenotypesAndAbundanceSummaryOfAAssociatedTaxon'
data = DataFrame(requests.post(url, data=json.dumps(query)).json().get('phenotypes_associated_with_taxon'))
```

See the first table at https://gmrepo.humangut.info/species/40520 for details.

### Get detailed information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a bunch of `DataFrames`
```python
query = {"ncbi_taxon_id":40520}  
url = 'https://gmrepo.humangut.info/api/getAssociatedPhenotypesAndAbundancesOfATaxon'
data = requests.post(url, data=json.dumps(query)).json()

## --get DataFrames
phenotypes_associated_with_taxon = DataFrame(data.get("phenotypes_associated_with_taxon"))
taxon = DataFrame(data.get("taxon"))
density_data_groupped = DataFrame(data.get("density_data_groupped"))
```
The retrieved `data` is a list containing:
* `phenotypes_associated_with_taxon`: a `data.frame` contains summary information on associated phenotypes,
* `taxon`: a list contains detailed information about this taxon, such as scientific name and taxonomic level,
* `density_data_groupped`: a list of `data.frame`, each contains abundance information of the current taxon in an associated phenotype; the number of `data.frame` corresponds to the number of phenotypes the current taxon is associated with.

The retrieved data can be used to generate the plots at https://gmrepo.humangut.info/species/40520.

### Get relative species/genus abundances for a sample/run

Two APIs are available here, namely `getRunDetailsByRunID` and `getFullTaxonomicProfileByRunID`. The usages are the same (see below). However, `getFullTaxonomicProfileByRunID` will produce the full taxonomic profiles at species and genus levels, while `getRunDetailsByRunID` only produces the top ten most abundant ones, and merge the others into a 'Others' category.

`input`: run ID, e.g. `ERR475468`,

`output`: a list, see below:
```python
query = {"run_id":"ERR475468"}  
url = 'https://gmrepo.humangut.info/api/getFullTaxonomicProfileByRunID'
data = requests.post(url, data=json.dumps(query)).json()

## --get run List
run = data.get("run")

## --get DataFrames
species = DataFrame(data.get("species"))
genus = DataFrame(data.get("genus"))
```
The retrieved `data` is a `list` containing:
* `run`: a `list` contains run metadata,
* `species`: a `data.frame` contains relative abundances of all species,
* `genus`: a `data.frame` contains relative abundances of all genera.

See https://gmrepo.humangut.info/data/run/ERR475468 for details.

### Get relative species/genus abundances for a project
`data input`: project id, e.g. `PRJNA489760`, and a MeSH ID

`data output`: a `list`
```python
# Get relative species/genus abundances for all phenotypes
query = {"project_id":"PRJNA489760","mesh_id":""}

# Get relative species/genus abundances for one of the phenotype in the project
query = {"project_id":"PRJNA489760","mesh_id":"D006262"}

# Query data
url = 'https://gmrepo.humangut.info/api/getMicrobeAbundancesByPhenotypeMeshIDAndProjectID'
data = requests.post(url, data=json.dumps(query)).json()

# Get project and disease information
project = data.get("project_info")
disease = data.get("disease_info")

# Get abundance and meta data
abundance_and_meta = DataFrame(data.get("abundance_and_meta_data"))
print(abundance_and_meta)

```
The retrieved `data` is a `list` containing:
* `project_info`: a `list` contains project information,
* `disease_info`: a `list` contains disease infromation,
* `abundance_and_meta_data`: a `data.frame` contains relative abundances of the project.

## Projects and runs
Although it is possible to download projects and runs through our RESTful API, it is highly recommended to download them from our website, or use the following URLs:
* download all projects: https://gmrepo.humangut.info/Downloads/AllSummaryData/all_projects_metadata.tsv.gz,
* download all runs associated with a project: https://gmrepo.humangut.info/Downloads/RunsByProjectID/all_runs_in_project_PRJEB6070.tsv.gz; please replace `PRJEB6070` with any other project ID of interests,
* other downloads please consult the `Data downloads` section of the Help page: https://gmrepo.humangut.info/help.
