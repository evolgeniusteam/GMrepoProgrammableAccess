## test test
* work1
* work2
## Install required modules
Please install the following required modules:
```python2
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

## -- simply install all of them without checking:
pip install requests
pip install json
pip install pandas
```
Load required libraries:
```python2
import requests
import json
from pandas.core.frame import DataFrame
```

## Phenotypes
### Get all phenotypes and statistics
`input`: none,

`output`: a `DataFrame`
```python2
### -- get all phenotypes  --
url = 'http://gmrepo.humangut.info/api/get_all_phenotypes'
pheno_01 = requests.post(url, data={})
pheno_01_cont = pheno_01.json().get('phenotypes')

## -- a DateFrame --
phenotypes = DataFrame(pheno_01_cont)
```
The `list` `pheno_01_cont` contains a list of phenotypes and related statistics as shown in http://gmrepo.humangut.info/phenotypes.

The `data.frame` `all_phenotypes` contains the information of phenotypes and related statistics as shown in http://gmrepo.humangut.info/phenotypes.

### Get statistics on a phenotype
Using the corresponding MeSH ID  (e.g. `D006262` for `Health `), uses can first obtain some statistics information of the phenotype, including:
* nr of associated species/genera
* nr of total/processed/qualified runs
* and other information

`input`: a MeSH ID,

`output`: a `DataFrame`
```python2
## -- get summary information by mesh_id
pheno_02_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'http://gmrepo.humangut.info/api/getStatisticsByProjectsByMeshID'
pheno_02 = requests.post(url, data=json.dumps(pheno_02_query))
pheno_02_cont = pheno_02.json()

## --get DataFrame
phenotyp_stats = DataFrame(pheno_02.json())
```
### Get associated species of a phenotype
Again the MeSH ID `D006262` will be needed as the input:

`input`: a MeSH ID,

`output`: a `data.frame`
```python2
pheno_03_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'http://gmrepo.humangut.info/api/getAssociatedSpeciesByMeshID'
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
```python2
pheno_04_query = {'mesh_id':'D006262'}  ## -- to get statistics on MeSH ID D006262
url = 'http://gmrepo.humangut.info/api/getAssociatedGeneraByMeshID'
pheno_04 = requests.post(url, data=json.dumps(pheno_04_query))
pheno_04_cont = pheno_04.json()

## --get DataFrame
phenotyp_assoc_genera = DataFrame(pheno_04.json())

## --show data header of the resulting DataFrame
list(phenotyp_assoc_genera)
```
### Calculate species/genera prevalence
**Prevalence** refers to the percentage of runs in which a species/genus is found out of the total number of valid runs; the latter can be found in `phenotyp_stats$stats$nr_valid_samples``phenotyp_stats.ix['nr_valid_samples']`.
```R
## calculate species/genera prevalence for all species associated with Health (D006262):
phenotyp_assoc_species$species_prevalence <- phenotyp_assoc_species$samples / phenotyp_stats$stats$nr_valid_samples * 100;

## then plot
plot( density( phenotyp_assoc_species$species_prevalence ) );

## calculate species prevalence for all genera associated with Health (D006262):
phenotyp_assoc_genera$genus_prevalence <- phenotyp_assoc_genera$samples / phenotyp_stats$stats$nr_valid_samples * 100;

## then plot
plot( density( phenotyp_assoc_genera$genus_prevalence ) );
```
Users can also use the above data to find, for example, the top 10 most prevalent species/genera:
```R
if( !requireNamespace("dplyr") ){
    install.packages( "dplyr" );
}
require(dplyr);

## -- sort by species_prevalence in decreasing order and select the top 10
phenotyp_assoc_species %>% arrange( desc( species_prevalence ) ) %>% top_n( 10 );
```
