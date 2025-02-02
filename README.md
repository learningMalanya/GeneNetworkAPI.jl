[![Dev](https://img.shields.io/badge/docs-dev-blue.svg)](https://senresearch.github.io/GeneNetworkAPI.jl/dev)
[![CI](https://github.com/senresearch/GeneNetworkAPI.jl/actions/workflows/ci.yml/badge.svg)](https://github.com/senresearch/GeneNetworkAPI.jl/actions/workflows/ci.yml)

# GeneNetworkAPI

Provides access to the [GeneNetwork](http://genenetwork.org) database
and analysis functions using the [GeneNetwork REST
API](https://github.com/genenetwork/gn-docs/blob/master/api/GN2-REST-API.md).

Karl Broman wrote the
[GNapi](https://github.com/kbroman/GNapi/blob/main/README.md) R
package for providing access to GeneNetwork from R.  This package
follows the structure and function of that package closely.

## Note on terminology

GeneNetwork collects data on genetically segregating populations
(called _groups_) in a number of _species_ including humans.  Most of
the phenotype data is "omic" data which are organized as _datasets_. 

## Check connection

To check if the website is responding properly:
```
julia> check_gn()
GeneNetwork is alive.
200
```

## Get species list

Which species have data on them?

```
julia> list_species()
11×4 DataFrame
 Row │ FullName                           Id     Name             TaxonomyId 
     │ String                             Int64  String           Int64      
─────┼───────────────────────────────────────────────────────────────────────
   1 │ Mus musculus                           1  mouse                 10090
   2 │ Rattus norvegicus                      2  rat                   10116
   3 │ Arabidopsis thaliana                   3  arabidopsis            3702
   4 │ Homo sapiens                           4  human                  9606
   5 │ Hordeum vulgare                        5  barley                 4513
   6 │ Drosophila melanogaster                6  drosophila             7227
   7 │ Macaca mulatta                         7  macaque monkey         9544
   8 │ Glycine max                            8  soybean                3847
   9 │ Solanum lycopersicum                   9  tomato                 4081
  10 │ Populus trichocarpa                   10  poplar                 3689
  11 │ Oryzias latipes (Japanese medaka)     11  Oryzias latipes        8090
```

To get information on a single species:

```
julia> list_species("rat")
1×4 DataFrame
 Row │ FullName           Id     Name    TaxonomyId 
     │ String             Int64  String  Int64      
─────┼──────────────────────────────────────────────
   1 │ Rattus norvegicus      2  rat          10116
```

You could also subset (safer):
```
julia> GeneNetworkAPI.subset(list_species(), :Name => x->x.=="rat")
1×4 DataFrame
 Row │ FullName           Id     Name    TaxonomyId 
     │ String             Int64  String  Int64      
─────┼──────────────────────────────────────────────
   1 │ Rattus norvegicus      2  rat          10116
```

## List groups for a species

Since the information is organized by segregating population
("group"), it is useful to get a list for a preticular species you
might be interested in.

```
julia> list_groups("rat")
7×8 DataFrame
 Row │ DisplayName                        FullName                           GeneticType  Id     MappingMethodId  Name             SpeciesId  public 
     │ String                             String                             String       Int64  String           String           Int64      Int64  
─────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │ Hybrid Rat Diversity Panel (Incl…  Hybrid Rat Diversity Panel (Incl…  None            10  1                HXBBXH                   2       2
   2 │ UIOWA SRxSHRSP F2                  UIOWA SRxSHRSP F2                  intercross      24  1                SRxSHRSPF2               2       2
   3 │ NIH Heterogeneous Stock (RGSMC 2…  NIH Heterogeneous Stock (RGSMC 2…  None            42  1                HSNIH-RGSMC              2       2
   4 │ NIH Heterogeneous Stock (Palmer)   NIH Heterogeneous Stock (Palmer)   None            55  1                HSNIH-Palmer             2       2
   5 │ NWU WKYxF344 F2 Behavior           NWU WKYxF344 F2 Behavior           intercross      82  3                NWU_WKYxF344_F2          2       2
   6 │ HIV-1Tg and Control                HIV-1Tg and Control                None            83  1                HIV-1Tg                  2       2
   7 │ HRDP-HXB/BXH Brain Proteome        HRDP-HXB/BXH Brain Proteome        None            87  1                HRDP_HXB-BXH-BP          2       2
```

You can see the type of population it is.  Note the short name
(`Name`) as that will be used in queries involving that population
(group).


## Get genotypes for a group

To get the genotypes of a group:


```
julia> get_geno("BXD") |> (x->first(x,10))
10×240 DataFrame
 Row │ Chr      Locus        cM       Mb       BXD1     BXD2     BXD5     BXD6     BXD8     BXD9     BXD11    BXD12    BXD13    BXD14    BXD15    BXD16    BXD18 ⋯
     │ String3  String31     Float64  Float64  String1  String1  String1  String1  String1  String1  String1  String1  String1  String1  String1  String1  Strin ⋯
─────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │ 1        rs31443144      1.5   3.01027  B        B        D        D        D        B        B        D        B        B        D        D        B     ⋯
   2 │ 1        rs6269442       1.5   3.4922   B        B        D        D        D        B        B        D        B        B        D        D        B
   3 │ 1        rs32285189      1.63  3.5112   B        B        D        D        D        B        B        D        B        B        D        D        B
   4 │ 1        rs258367496     1.63  3.6598   B        B        D        D        D        B        B        D        B        B        D        D        B
   5 │ 1        rs32430919      1.75  3.77702  B        B        D        D        D        B        B        D        B        B        D        D        B     ⋯
   6 │ 1        rs36251697      1.88  3.81227  B        B        D        D        D        B        B        D        B        B        D        D        B
   7 │ 1        rs30658298      2.01  4.43062  B        B        D        D        D        B        B        D        B        B        D        D        B
   8 │ 1        rs51852623      2.01  4.44674  B        B        D        D        D        B        B        D        B        B        D        D        B
   9 │ 1        rs31879829      2.14  4.51871  B        B        D        D        D        B        B        D        B        B        D        D        B     ⋯
  10 │ 1        rs36742481      2.14  4.77632  B        B        D        D        D        B        B        D        B        B        D        D        B
                                                                                                                                               224 columns omitted
```

Currently, we only support the `.geno` format which returns a data
frame of genotypes with rows as marker and columns as individuals.

## List datasets for a group

To list the (omic) datasets available for a group, you have to use the
name as listed in the group list for a species:

```
julia> list_datasets("HSNIH-Palmer")
10×11 DataFrame
 Row │ AvgID  CreateTime                     DataScale  FullName                           Id     Long_Abbreviation              ProbeFreezeId  ShortName        ⋯
     │ Int64  String                         String     String                             Int64  String                         Int64          String           ⋯
─────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │    24  Mon, 27 Aug 2018 00:00:00 GMT  log2       HSNIH-Palmer Nucleus Accumbens C…    860  HSNIH-Rat-Acbc-RSeq-Aug18                347  HSNIH-Palmer Nuc ⋯
   2 │    24  Sun, 26 Aug 2018 00:00:00 GMT  log2       HSNIH-Palmer Infralimbic Cortex …    861  HSNIH-Rat-IL-RSeq-Aug18                  348  HSNIH-Palmer Inf
   3 │    24  Sat, 25 Aug 2018 00:00:00 GMT  log2       HSNIH-Palmer Lateral Habenula RN…    862  HSNIH-Rat-LHB-RSeq-Aug18                 349  HSNIH-Palmer Lat
   4 │    24  Fri, 24 Aug 2018 00:00:00 GMT  log2       HSNIH-Palmer Prelimbic Cortex RN…    863  HSNIH-Rat-PL-RSeq-Aug18                  350  HSNIH-Palmer Pre
   5 │    24  Thu, 23 Aug 2018 00:00:00 GMT  log2       HSNIH-Palmer Orbitofrontal Corte…    864  HSNIH-Rat-VoLo-RSeq-Aug18                351  HSNIH-Palmer Orb ⋯
   6 │    24  Fri, 14 Sep 2018 00:00:00 GMT  log2       HSNIH-Palmer Nucleus Accumbens C…    868  HSNIH-Rat-Acbc-RSeqlog2-Aug18            347  HSNIH-Palmer Nuc
   7 │    24  Fri, 14 Sep 2018 00:00:00 GMT  log2       HSNIH-Palmer Infralimbic Cortex …    869  HSNIH-Rat-IL-RSeqlog2-Aug18              348  HSNIH-Palmer Inf
   8 │    24  Fri, 14 Sep 2018 00:00:00 GMT  log2       HSNIH-Palmer Lateral Habenula RN…    870  HSNIH-Rat-LHB-RSeqlog2-Aug18             349  HSNIH-Palmer Lat
   9 │    24  Fri, 14 Sep 2018 00:00:00 GMT  log2       HSNIH-Palmer Prelimbic Cortex RN…    871  HSNIH-Rat-PL-RSeqlog2-Aug18              350  HSNIH-Palmer Pre ⋯
  10 │    24  Fri, 14 Sep 2018 00:00:00 GMT  log2       HSNIH-Palmer Orbitofrontal Corte…    872  HSNIH-Rat-VoLo-RSeqlog2-Aug18            351  HSNIH-Palmer Orb
                                                                                                                                                 4 columns omitted
```

## Get sample data for a group

This gives you a matrix with rows as individuals/samples/strains and
columns as "clinical" (non-omic) phenotypes.  The number after the
underscore is the phenotype number (to be used later).  Some data may
be missing.

```
julia> get_pheno("HSNIH-Palmer") |> (x->x[81:100,:]) |> show
20×509 DataFrame
 Row │ id          HSR_10308  HSR_10309  HSR_10310  HSR_10311  HSR_10312  HSR_10313  HSR_10314  HSR_10315       HSR_10316       HSR_10317      ⋯
     │ String15    Float64?   Float64?   Float64?   Float64?   Float64?   Float64?   Float64?   Float64?        Float64?        Float64?       ⋯
─────┼──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │ 000721E489  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing        ⋯
   2 │ 00072AAC0D  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
   3 │ 00072AC972  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
   4 │ 00077E61DC  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
   5 │ 00077E61EC  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing        ⋯
   6 │ 00077E61F3       18.0       43.0       25.0       42.0       36.0        8.0       43.0       -0.514286        1.14667         1.125
   7 │ 00077E61F5  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
   8 │ 00077E6204  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
   9 │ 00077E6207       22.0       63.0       54.0       77.0       54.0       42.0       77.0        0.914286        1.07959         1.0      ⋯
  10 │ 00077E6299  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  11 │ 00077E62CD  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  12 │ 00077E62D2       55.0       54.0       31.0       16.0       25.0       18.0       55.0       -2.73333         0.780392        1.22222
  13 │ 00077E633D       25.0       47.0       58.0       35.0       27.0       35.0       58.0       -0.314286        1.19474         0.925926 ⋯
  14 │ 00077E634B  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  15 │ 00077E63D9  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  16 │ 00077E641E  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  17 │ 00077E6433      112.0      131.0      117.0       60.0       82.0       70.0      131.0       -3.94286         1.95222         2.54546  ⋯
  18 │ 00077E64B3  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
  19 │ 00077E64BA      135.0      154.0      188.0      267.0       98.0       76.0      267.0       -3.65714         4.19178         4.35484
  20 │ 00077E64C1  missing    missing    missing    missing    missing    missing    missing    missing         missing         missing
                                                                                                                             498 columns omitted
```

## Get information about traits

To get information on a particular (non-omic) trait use the group name
and the trait number:

```
julia> info_dataset("HSNIH-Palmer","10308")
1×4 DataFrame
 Row │ dataset_type  description                        id     name                  
     │ String        String                             Int64  String                
─────┼───────────────────────────────────────────────────────────────────────────────
   1 │ phenotype     Central nervous system, behavior…  10308  reaction_time_pint1_5
```

To get information on a dataset (of omic traits) for a group, use:

```
julia> info_dataset("HSNIH-Rat-Acbc-RSeq-Aug18")
1×10 DataFrame
 Row │ confidential  data_scale  dataset_type     full_name                          id     name                      public  short_name                         ti ⋯
     │ Int64         String      String           String                             Int64  String                    Int64   String                             St ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │            0  log2        mRNA expression  HSNIH-Palmer Nucleus Accumbens C…    860  HSNIH-Rat-Acbc-RSeq-0818       1  HSNIH-Palmer Nucleus Accumbens C…  Nu ⋯
                                                                                                                                                    2 columns omitted
```

## Summary information on traits

Get a list of the maximum LRS for each trait and position.

```
julia> info_pheno("HXBBXH") |> (x->first(x,10))
10×7 DataFrame
 Row │ Additive     Id     LRS       Locus        PhenotypeId  PublicationId  Sequence 
     │ Float64?     Int64  Float64?  String?      Int64        Int64          Int64    
─────┼─────────────────────────────────────────────────────────────────────────────────
   1 │  0.0499968   10001  16.2831   rs106114574         1449            319         1
   2 │ -0.0926364   10002  10.9777   rs63915446          1450            319         1
   3 │  0.60189     10003  13.6515   rs107486115         1451            319         1
   4 │ -0.543799    10004   8.43965  D5Rat147            1452            319         1
   5 │  0.00854221  10005  18.5895   rs106114574         1453            319         1
   6 │ -0.0142273   10006  11.9965   rs63915446          1454            319         1
   7 │  0.427167    10007  10.541    rs13452609          1455            319         1
   8 │ -0.936806    10008  13.2494   rs8143630           1456            319         1
   9 │ -0.635833    10009   9.97609  rs107549352         1457            319         1
  10 │ -0.681451    10010   9.59226  D7Mit13             1458            319         1
```

You could also specify a group and a trait number or a dataset and a probename.

```
julia> info_pheno("BXD","10001")
1×4 DataFrame
 Row │ additive  id     locus       lrs     
     │ Float64   Int64  String      Float64 
─────┼──────────────────────────────────────
   1 │  2.39444      4  rs48756159  13.4975
```

```
julia> info_pheno("HC_M2_0606_P","1436869_at")
1×13 DataFrame
 Row │ additive   alias                              chr     description                id     locus      lrs      mb       mean     name        p_value  se        ⋯
     │ Float64    String                             String  String                     Int64  String     Float64  Float64  Float64  String      Float64  Nothing   ⋯
─────┼───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1 │ -0.214088  HHG1; HLP3; HPE3; SMMCI; Dsh; Hh…  5       sonic hedgehog (hedgehog)  99602  rs8253327  12.7711  28.4572  9.27909  1436869_at    0.306            ⋯
                                                                                                                                                     1 column omitted
```

## Analysis commands


### GEMMA

```
julia> run_gemma("BXDPublish","10015",use_loco=true) |> (x->first(x,10))
10×6 DataFrame
 Row │ Mb       additive   chr  lod_score  name         p_value  
     │ Float64  Float64    Any  Float64    String       Float64  
─────┼───────────────────────────────────────────────────────────
   1 │ 3.01027  -0.906398  1     0.448914  rs31443144   0.355702
   2 │ 3.4922   -0.906398  1     0.448914  rs6269442    0.355702
   3 │ 3.5112   -0.906398  1     0.448914  rs32285189   0.355702
   4 │ 3.6598   -0.906398  1     0.448914  rs258367496  0.355702
   5 │ 3.77702  -0.906398  1     0.448914  rs32430919   0.355702
   6 │ 3.81227  -0.906398  1     0.448914  rs36251697   0.355702
   7 │ 4.43062  -0.906398  1     0.448914  rs30658298   0.355702
   8 │ 4.44674  -0.906398  1     0.448914  rs51852623   0.355702
   9 │ 4.51871  -0.906398  1     0.448914  rs31879829   0.355702
  10 │ 4.77632  -0.906398  1     0.448914  rs36742481   0.355702
```

### R/qtl

This function performs a one-dimensional genome scan.  The arguments
are

- db (required) - DB name for trait above (Short_Abbreviation listed
  when you query for datasets)
- trait (required) - ID for trait being mapped
- method - hk (default) | ehk | em | imp | mr | mr-imp |
  mr-argmax ; Corresponds to the "method" option for the R/qtl scanone
  function.
- model - normal (default) | binary | 2-part | np ; corresponds
  to the "model" option for the R/qtl scanone function
- n_perm - number of permutations; 0 by default
- control_marker - Name of marker to use as control; this relies on
  the user knowing the name of the marker they want to use as a
  covariate
- interval_mapping - Whether to use interval mapping; "false" by default

```
julia> run_rqtl("BXDPublish", "10015") |> (x->first(x,10))
10×5 DataFrame
 Row │ Mb       cM       chr  lod_score  name        
     │ Float64  Float64  Any  Float64    String      
─────┼───────────────────────────────────────────────
   1 │ 3.01027  3.01027  1     0.116927  rs31443144
   2 │ 3.4922   3.4922   1     0.117404  rs6269442
   3 │ 3.5112   3.5112   1     0.117424  rs32285189
   4 │ 3.6598   3.6598   1     0.117573  rs258367496
   5 │ 3.77702  3.77702  1     0.117691  rs32430919
   6 │ 3.81227  3.81227  1     0.117727  rs36251697
   7 │ 4.43062  4.43062  1     0.118356  rs30658298
   8 │ 4.44674  4.44674  1     0.118372  rs51852623
   9 │ 4.51871  4.51871  1     0.118447  rs31879829
  10 │ 4.77632  4.77632  1     0.118714  rs36742481
```

### Correlation

This function correlates a trait in a dataset against all traits in a
target database.


- trait_id (required) - ID for trait used for correlation
- db (required) - DB name for the trait above (this is the Short_Abbreviation listed when you query for datasets)
- target_db (required) - Target DB name to be correlated against
- type - sample (default) | tissue
- method - pearson (default) | spearman
- return - Number of results to return (default = 500)


```
julia> run_correlation("1427571_at","HC_M2_0606_P","BXDPublish") |> (x->first(x,10))
10×4 DataFrame
 Row │ #_strains  p_value      sample_r   trait  
     │ Int64      Float64      Float64    String 
─────┼───────────────────────────────────────────
   1 │         6  0.00480466   -0.942857  20511
   2 │         6  0.00480466   -0.942857  20724
   3 │        12  1.82889e-5   -0.923362  13536
   4 │         7  0.00680719    0.892857  10157
   5 │         7  0.00680719   -0.892857  20392
   6 │         6  0.0188455     0.885714  20479
   7 │        12  0.000189298  -0.875658  12762
   8 │        12  0.000245942   0.868653  12760
   9 │         7  0.0136973    -0.857143  20559
  10 │        10  0.00222003   -0.842424  10925
```
