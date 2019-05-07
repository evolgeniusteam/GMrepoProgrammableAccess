Access to GMrepo using Perl through RESTful APIs

## Table of contents
<!-- MDTOC maxdepth:6 firsth1:1 numbering:0 flatten:0 bullets:1 updateOnSave:1 -->

   - [Table of contents](#table-of-contents)   
   - [Install and load required module](#install-and-load-required-module)   
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

## Install and load required module
Please install the following required module:
```Shell
## -- if you are using Debian Linux, following is the recommended way.
## -- start CPAN shell
perl -MCPAN -e shell

## -- install HTTP::Request
cpan> install HTTP::Request

## -- install LWP::UserAgent
cpan> install LWP::UserAgent

## -- install Encode
cpan> install Encode

## -- install JSON
cpan> install JSON

## -- install Data::Dumper
cpan> install Data::Dumper

## -- quit CPAN shell
cpan> q
```

Load required module:
```Perl
#!usr/bin/perl      -w
use HTTP::Request;
use LWP::UserAgent;
use Encode;
use JSON;
use Data::Dumper;
```

Declare a variable:
```Perl
my $ua = LWP::UserAgent->new;
```

## Phenotypes
### Get all phenotypes and statistics

`input`: none,

`output`: an `array`
```Perl
### -- get all phenotypes  --
my $url1 = 'http://gmrepo.humangut.info/api/get_all_phenotypes';
my $req1 = HTTP::Request->new('POST' => $url1);
my $all_phenotypes = decode_json($ua->request($req1)->content())->{'phenotypes'};

### -- select the valid key-value pairs
my @col = ('disease','term','note', 'all_samples', 'processed_runs','valid_runs','failed_runs','nr_species','nr_genus');
my $row = @$all_phenotypes;
my @phenotypes = map{my %h;@h{@col}=@$_{@col};\%h}@$all_phenotypes[0..($row-1)];

### -- the resulting variable is an array of hashes
print "print the top 5 elements:\n";
print Dumper(@phenotypes[0..4]);
```
The `array` `phenotypes` contains phenotypes and related statistics as shown in http://gmrepo.humangut.info/phenotypes, the elements of '@phenotypes' are anonymous hashes.

### Get statistics on a phenotype
Using the corresponding MeSH ID  (e.g. `D006262` for `Health `), uses can first obtain some statistics information of the phenotype, including:
* nr of associated species/genera
* nr of total/processed/qualified runs
* and other information

`input`: a MeSH ID,

`output`: a `scalar`
```Perl
## -- get summary information by mesh_id
my $url2 = 'http://gmrepo.humangut.info/api/getStatisticsByProjectsByMeshID';
my $res2 = $ua->post($url2,Content => '{"mesh_id": "D006262"}');## -- to get statistics on MeSH ID D006262
my $phenotype_stats = decode_json($res2->content());

### -- the resulting variable is a reference to a hash of hashes
print "\$phenotype_stats:","\n";
print Dumper($phenotype_stats);
```

### Get associated species of a phenotype
Again the MeSH ID `D006262` will be needed as the input:

`input`: a MeSH ID,

`output`: a `scalar`
```Perl
## -- all associted species --
## -- please note only species that are found in >= 2 runs & with median relative abundance >= 0.01% will be retrieved
my $url3 = 'http://gmrepo.humangut.info/api/getAssociatedSpeciesByMeshID';
my $res3 = $ua->post($url3,Content => '{"mesh_id": "D006262"}');
my $phenotype_assoc_species = decode_json($res3->content());

### -- the resulting variable is a reference to an array of hashes
print "print the top 5 elements:\n";
print Dumper(@$phenotype_assoc_species[0..4]);
```

### Get associated genera of a phenotype

`input`: a MeSH ID,

`output`: a `scalar`
```Perl
## -- all associted genera --
## -- please note only genera that are found in >= 2 runs & with median relative abundance >= 0.01% will be retrieved
my $url4 = 'http://gmrepo.humangut.info/api/getAssociatedGeneraByMeshID';
my $res4 = $ua->post($url4,Content => '{"mesh_id": "D006262"}');
my $phenotype_assoc_genera = decode_json($res4->content());

### -- the resulting variable is a reference to an array of hashes
print "print the top 5 elements:\n";
print Dumper(@$phenotype_assoc_genera[0..4]);
```

### Calculate species/genera prevalence
**Prevalence** refers to the percentage of runs in which a species/genus is found out of the total number of valid runs; the latter can be found in `$phenotype_stats->{stats}->{nr_valid_samples}`.
```Perl
## calculate species/genera prevalence for all species associated with Health (D006262):
my @species_prevalence = map {
        $_->{samples} / $phenotype_stats->{stats}->{nr_valid_samples} * 100
}@$phenotype_assoc_species;

## calculate species prevalence for all genera associated with Health (D006262):
my @genera_prevalence = map {
        $_->{samples} / $phenotype_stats->{stats}->{nr_valid_samples} * 100
}@$phenotype_assoc_species;
```

### Get associated projects

`input`: a MeSH ID,

`output`: a `scalar`
```Perl
## -- all associted projects --
my $url6 = 'http://gmrepo.humangut.info/api/getAssociatedProjectsByMeshID';
my $res6 = $ua->post($url6,Content => '{"mesh_id": "D006262"}');
my $phenotype_assoc_pros = decode_json($res6->content());

### -- the resulting variable is a reference to an array of hashes
print "print the top 5 elements:\n";
print Dumper(@$phenotype_assoc_pros[0..4]);
```
Please note very often a project may contain samples/runs of multiple phenotypes.

### Get associated runs
Some phenotypes are associated with tens of thousands of runs (e.g. `Health (D006262)`) that are too many to be retrieved with one call. Therefore it may take a two-step procedure to retrieve all runs associated with phenotype.

First, count the number of runs associated with a phenotype:

`input`: a MeSH ID,

`output`: a `scalar`
```Perl
## -- count associated runs --
my $url7 = 'http://gmrepo.humangut.info/api/countAssociatedRunsByPhenotypeMeshID';
my $res7 = $ua->post($url7,Content => '{"mesh_id": "D006262"}');
my $phenotyp_nr_assoc_runs = decode_json($res7->content());

### -- the resulting variable is a reference to an array of hash
print (keys %{@$phenotyp_nr_assoc_runs[0]},":",values %{@$phenotyp_nr_assoc_runs[0]},"\n");
```

Then users can use a loop retrieve the associated runs, 100 runs at a time:

`input`: a MeSH ID, the number of records to skip, the number of records to retrieve; see below.

`output`: a `scalar`
```Perl
## -- get all associted runs --
## use skip = 0, limit = 100 to retrieve the first 100 runs, then
##     skip = 100, limit = 100 to retrieve the next 100 runs ....
my $url8 = 'http://gmrepo.humangut.info/api/getAssociatedRunsByPhenotypeMeshIDLimit';
my $res8 = $ua->post($url8,Content => '{"mesh_id": "D006262", "skip": 0, "limit": 100}');
my $phenotyp_a_page_of_assoc_runs = decode_json($res8->content());

### -- the resulting variable is a reference to an array of hashes
print "print the top 5 elements:\n";
print Dumper(@$phenotyp_a_page_of_assoc_runs[0..4]);
```

### Get relative species/genus abundances in samples/runs associated with a phenotype
To get the related information, two input parameters are required:
* MeSH ID of interests, e.g. `D003093` for `Colitis, Ulcerative`
* NCBI taxonomy ID of the species/genus of interests, e.g. `40520` for `Blautia obeum (species)`.

```Perl
my $url9 = 'http://gmrepo.humangut.info/api/getMicrobeAbundancesByPhenotypeMeshIDAndNCBITaxonID';
my $res9 = $ua->post($url9,Content => '{"mesh_id": "D003093","ncbi_taxon_id": "40520"}');
my $data = decode_json($res9->content());

### -- the resulting variable is a reference to a hash of hashes
print "This data set contains the following information:\n";
print join("," ,keys %{$data},"\n");
```
The resulting `$data` is a reference to a hash of hashes containing:
* `hist_data_for_phenotype`: the value corresponding to this key contains the distribution of the relative abundances of the species/genus of interests in all samples of current phenotype,
* `hist_data_for_health`: if current phenotype is not `Health`, the  distribution of the relative abundances of the species/genus of interests in all samples of `Health` will also be retrieved,
* `abundant_data_for_disease`: the value corresponding to this key contains the relative abundance data of the species/genus of interests in all samples of current phenotype,
* `abundant_data_for_health`: if current phenotype is not `Health`, the relative abundances of the species/genus of interests in all samples of `Health` will also be retrieved,
* `taxon`: NCBI taxonomy information for current taxonomy ID,
* `disease`: details of current phenotype,
* `abundance_and_meta_data`: runs in which current taxon is found and related meta data,
* `co_occurred_taxa`: cooccurred taxa of the taxon of interests in current phenotype

See http://gmrepo.humangut.info/phenotypes/D003093/40520 for more details.

## Species/genera
### Get an overview of the species/genera

`input`: none,

`output`: a `scalar`.
```Perl
### --- get all species and genera that presented in >= 2 runs with median relative abundance >= 0.01%
my $url10 = 'http://gmrepo.humangut.info/api/get_all_gut_microbes';
my $req10 = HTTP::Request->new('POST' => $url10);
my $data10 = decode_json($ua->request($req10)->content());

### -- the resulting variable is a reference to a hash of hashes
print "This data set contains the following information:\n";
print join("," ,keys %{$data10},"\n");
```
The retrieved `data` is a list containing:
* `all_species`:the value corresponding to this key contains all species that presented in >= 2 runs with median relative abundance >= 0.01%,
* `all_genus`: the value corresponding to this key contains all genera that presented in >= 2 runs with median relative abundance >= 0.01%,
* `metadata`: the value corresponding to this key contains additional statistics:
  * `loaded_samples`: nr. qualified runs for which the relative abundance data are available,
  * `all_species_count`: nr. all species
  * `retrieved_species_count`: nr. species in the `array`: `all_species`,
  * `all_genus_count`: nr. all genera,
  * `retrieved_genus_count`: nr. genera in the `array`: `all_genus`.

With the retrieved data, users can plot the **species prevalence in phenotypes** and **species prevalence in samples**, as shown at http://gmrepo.humangut.info/species.

### Get summary information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a `scalar`
```Perl
my $url12 = 'http://gmrepo.humangut.info/api/getPhenotypesAndAbundanceSummaryOfAAssociatedTaxon';
my $res12 = $ua->post($url12,Content => '{"ncbi_taxon_id": "40520"}');
my $data12 = decode_json($res12->content())->{'phenotypes_associated_with_taxon'};

### -- the resulting variable is a reference to an array of hashes
print "print the top 5 elements:\n";
print Dumper(@$data12[0..4]);
```

See the first table at http://gmrepo.humangut.info/species/40520 for details.

### Get detailed information of the prevalence and relative abundance of a species/genus in all associated phenotypes

`data input`: ncbi taxonomy id of a species/genus,

`data output`: a `scalar`
```Perl
my $url13 = 'http://gmrepo.humangut.info/api/getAssociatedPhenotypesAndAbundancesOfATaxon';
my $res13 = $ua->post($url13,Content => '{"ncbi_taxon_id": "40520"}');
my $data13 = decode_json($res13->content());

## --- the resulting variable is a reference to a hash of hashes
print "This data set contains the following information:\n";
print join(",",keys %{$data13},"\n");
```
The retrieved `$data` is a reference to a hash of hashes containing:
* `phenotypes_associated_with_taxon`: the value corresponding to this key contains summary information on associated phenotypes,
* `taxon`: the value corresponding to this key contains detailed information about this taxon, such as scientific name and taxonomic level,
* `density_data_groupped`: the value corresponding to this key is a reference to a hash, contains abundance information of the current taxon in an associated phenotype; the number of key-value pairs corresponds to the number of phenotypes the current taxon is associated with.

The retrieved data can be used to generate the plots at http://gmrepo.humangut.info/species/40520.

### Get relative species/genus abundances for a sample/run

`input`: run ID, e.g. `ERR475468`,

`output`: a `scalar`, see below:
```Perl
my $url14 = 'http://gmrepo.humangut.info/api/getRunDetailsByRunID';
my $res14 = $ua->post($url14,Content => '{"run_id":"ERR475468"}');
my $data14 = decode_json($res14->content());

### -- the resulting variable is a reference to a hash of hashes
print "This data set contains the following information:\n";
print join(",",keys %{$data14},"\n");
```
The retrieved `$data` is a reference to a hash of hashes containing:
* `run`: the value corresponding to this key is a reference to a hash, contains run metadata,
* `species`: the value corresponding to this key is a reference to an array, contains relative abundances of all species,
* `genus`: the value corresponding to this key is a reference to an array, contains relative abundances of all genera.

See http://gmrepo.humangut.info/data/run/ERR475468 for details.

## Projects and runs
Although it is possible to download projects and runs through our RESTful API, it is highly recommended to download them from our website, or use the following URLs:
* download all projects: http://gmrepo.humangut.info/Downloads/AllSummaryData/all_projects_metadata.tsv.gz,
* download all runs associated with a project: http://gmrepo.humangut.info/Downloads/RunsByProjectID/all_runs_in_project_PRJEB6070.tsv.gz; please replace `PRJEB6070` with any other project ID of interests,
* other downloads please consult the `Data downloads` section of the Help page: http://gmrepo.humangut.info/help.
