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
   - [Projects and runs](#projects-and-runs)   

<!-- /MDTOC -->

## Install and load required libraries
Please install the following required libraries:
```R
## -- install httr
if( !requireNamespace("httr") ){
  install.packages( "httr" );
}

## -- install jsonlite
if( !requireNamespace("jsonlite") ){
  install.packages( "jsonlite" );
}

## -- install xml2
if( !requireNamespace("xml2") ){
  install.packages( "xml2" );
}

## -- simply install all of them at once:
install.packages( c( "httr", "jsonlite", "xml2" ) );
```

Load required libraries:
```R
require("httr");
require("jsonlite");
require("xml2");
```

## Phenotypes
### Get all phenotypes and statistics
`input`: none,

`output`: a `list`
```R
### -- get all phenotypes  --
pheno_01 <- POST("http://gmrepo.humangut.info/api/get_all_phenotypes", body = list());
pheno_01_cont <- content(pheno_01);
all_phenotypes <- fromJSON( xml_text( pheno_01_cont ));

## -- a list --
str(all_phenotypes);
head(all_phenotypes$phenotypes);
```
The `data.frame` `phenotypes` in the retrieved list `all_phenotypes` (can be accessed using `all_phenotypes$phenotypes`) contains a list of phenotypes and related statistics as shown in http://gmrepo.humangut.info/phenotypes.

### Get statistics on a phenotype
Using the corresponding MeSH ID  (e.g. `D006262` for `Health `), uses can first obtain some statistics information of the phenotype, including:
* nr of associated species/genera
* nr of total/processed/qualified runs
* and other information

`input`: a MeSH ID,

`output`: a `list`
```R
## -- get summary information by mesh_id
pheno_02_query <- list( "mesh_id" = "D006262");  ## -- to get statistics on MeSH ID D006262
pheno_02 <- POST("http://gmrepo.humangut.info/api/getStatisticsByProjectsByMeshID", body = pheno_02_query, encode = "json");
pheno_02_cont <- content( pheno_02 );
phenotyp_stats <- fromJSON( xml_text( pheno_02_cont ));

## show data structure of the resulting list
str(phenotyp_stats);
```
### Get associated species of a phenotype
Again the MeSH ID `D006262` will be needed as the input:

`input`: a MeSH ID,

`output`: a `data.frame`
```R
## -- all associted species --
## -- please note only species that are found in >= 2 runs & with median relative abundance >= 0.01% will be retrieved
pheno_03 <- POST("http://gmrepo.humangut.info/api/getAssociatedSpeciesByMeshID", body = list( "mesh_id" = "D006262"), encode = "json");
pheno_03_cont <- content( pheno_03 );
phenotyp_assoc_species <- fromJSON( xml_text( pheno_03_cont ));

## -- the resulting variable is a data.frame --
head( phenotyp_assoc_species );
```

### Get associated genera of a phenotype
`input`: a MeSH ID,

`output`: a `data.frame`
```R
## -- all associted genera --
## -- please note only genera that are found in >= 2 runs & with median relative abundance >= 0.01% will be retrieved
pheno_04 <- POST("http://gmrepo.humangut.info/api/getAssociatedGeneraByMeshID", body = list( "mesh_id" = "D006262"), encode = "json");
pheno_04_cont <- content( pheno_04 );
phenotyp_assoc_genera <- fromJSON( xml_text( pheno_04_cont ));

## -- the resulting variable is a data.frame --
head( phenotyp_assoc_genera );
```

### Calculate species/genera prevalence
**Prevalence** refers to the percentage of runs in which a species/genus is found out of the total number of valid runs; the latter can be found in `phenotyp_stats$stats$nr_valid_samples`.
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

### Get associated projects

`input`: a MeSH ID,

`output`: a `data.frame`
```R
## -- all associted projects --
pheno_05 <- POST("http://gmrepo.humangut.info/api/getAssociatedProjectsByMeshID", body = list( "mesh_id" = "D006262"), encode = "json");
pheno_05_cont <- content( pheno_05 );
phenotyp_assoc_pros <- fromJSON( xml_text( pheno_05_cont ));

## -- the resulting variable is a data.frame --
head( phenotyp_assoc_pros );
```
Please note very often a project may contain samples/runs of multiple phenotypes.

### Get associated runs
Some phenotypes are associated with tens of thousands of runs (e.g. `Health (D006262)`) that are too many to be retrieved with one call. Therefore it may take a two-step procedure to retrieve all runs associated with phenotype.

First, count the number of runs associated with a phenotype:

`input`: a MeSH ID,

`output`: a `vector`
```R
## -- count associated runs --
pheno_07 <- POST("http://gmrepo.humangut.info/api/countAssociatedRunsByPhenotypeMeshID", body = list( "mesh_id" = "D006262"), encode = "json");
pheno_07_cont <- content( pheno_07 );
phenotyp_nr_assoc_runs <- fromJSON( xml_text( pheno_07_cont ));

## -- the resulting variable is a vector --
head( phenotyp_nr_assoc_runs );
```

Then users can use a loop retrieve the associated runs, 100 runs at a time:

`input`: a MeSH ID, the number of records to skip, the number of records to retrieve; see below.

`output`: a `data.frame`
```R
## -- get all associted runs --
## use skip = 0, limit = 100 to retrieve the first 100 runs, then
##     skip = 100, limit = 100 to retrieve the next 100 runs ....
params <- list( "mesh_id" = "D006262", "skip" = 0, "limit" = 100 );
pheno_08 <- POST("http://gmrepo.humangut.info/api/getAssociatedRunsByPhenotypeMeshIDLimit", body = params, encode = "json");
pheno_08_cont <- content( pheno_08 );
phenotyp_a_page_of_assoc_runs <- fromJSON( xml_text( pheno_08_cont ));

## -- the resulting variable is a data.frame --
head( phenotyp_a_page_of_assoc_runs );
```

### Get relative species/genus abundances in samples/runs associated with a phenotype
To get the related information, two input parameters are required:
* MeSH ID of interests, e.g. `D003093` for `Colitis, Ulcerative`
* NCBI taxonomy ID of the species/genus of interests, e.g. `40520` for `Blautia obeum (species)`.

```R
params <- list( "mesh_id" = "D003093", "ncbi_taxon_id" = "40520" );
query <- POST("http://gmrepo.humangut.info/api/getMicrobeAbundancesByPhenotypeMeshIDAndNCBITaxonID", body = params, encode = "json");
retrieved_contents <- content( query );
data <- fromJSON( xml_text( retrieved_contents ));

## -- the resulting variable is a list --
str( data );
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

See http://gmrepo.humangut.info/phenotypes/D003093/40520 for more details.

## Species/genera
### Get an overview of the species/genera

`input`: none,

`output`: a `list`.
```R
### --- get all species and genera that presented in >= 2 runs with median relative abundance >= 0.01%
query <- POST("http://gmrepo.humangut.info/api/get_all_gut_microbes", body = NULL, encode = "json");
retrieved_contents <- content( query );
data <- fromJSON( xml_text( retrieved_contents ));

## --- a list --
str(data);
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
```R
## -- species prevalence in samples
plot(density( data$all_species$pct_of_all_samples ));

## -- genus prevalence in samples
plot(density( data$all_genus$pct_of_all_samples ));

## -- species prevalence in phenotypes --
plot(density( data$all_species$nr_phenotypes ));

## -- genus prevalence in phenotypes --
plot(density( data$all_genus$nr_phenotypes ));
```

See http://gmrepo.humangut.info/species for more details.

### Get summary information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a `data.frame`
```R
query <- POST("http://gmrepo.humangut.info/api/getPhenotypesAndAbundanceSummaryOfAAssociatedTaxon", body = list( "ncbi_taxon_id" = 40520 ), encode = "json");
retrieved_contents <- content( query );
data <- fromJSON( xml_text( retrieved_contents ));

## --- a data.frame --
head(data);
```

See the first table at http://gmrepo.humangut.info/species/40520 for details.

### Get detailed information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a `list`
```R
query <- POST("http://gmrepo.humangut.info/api/getAssociatedPhenotypesAndAbundancesOfATaxon", body = list( "ncbi_taxon_id" = 40520 ), encode = "json");
retrieved_contents <- content( query );
data <- fromJSON( xml_text( retrieved_contents ));

## --- a list --
str(data);
```
The retrieved `data` is a list containing:
* `phenotypes_associated_with_taxon`: a `data.frame` contains summary information on associated phenotypes,
* `taxon`: a list contains detailed information about this taxon, such as scientific name and taxonomic level,
* `density_data_groupped`: a list of `data.frame`, each contains abundance information of the current taxon in an associated phenotype; the number of `data.frame` corresponds to the number of phenotypes the current taxon is associated with.

The retrieved data can be used to generate the plots at http://gmrepo.humangut.info/species/40520.

### Get relative species/genus abundances for a sample/run

`input`: run ID, e.g. `ERR475468`,

`output`: a list, see below:
```R
query <- POST("http://gmrepo.humangut.info/api/getRunDetailsByRunID", body = list( "run_id" = "ERR475468" ), encode = "json");
retrieved_contents <- content( query );
data <- fromJSON( xml_text( retrieved_contents ));

## --- a list --
str(data);
```
The retrieved `data` is a `list` containing:
* `run`: a `list` contains run metadata,
* `species`: a `data.frame` contains relative abundances of all species,
* `genus`: a `data.frame` contains relative abundances of all genera.

See http://gmrepo.humangut.info/data/run/ERR475468 for details.

## Projects and runs
Although it is possible to download projects and runs through our RESTful API, it is highly recommended to download them from our website, or use the following URLs:
* download all projects: http://gmrepo.humangut.info/Downloads/AllSummaryData/all_projects_metadata.tsv.gz,
* download all runs associated with a project: http://gmrepo.humangut.info/Downloads/RunsByProjectID/all_runs_in_project_PRJEB6070.tsv.gz; please replace `PRJEB6070` with any other project ID of interests,
* other downloads please consult the `Data downloads` section of the Help page: http://gmrepo.humangut.info/help.
