
# MaAsLin2 User Manual #

MaAsLin2 is the next generation of MaAsLin.

[MaAsLin](http://huttenhower.sph.harvard.edu/maaslin) is comprehensive R package for efficiently determining multivariable association between clinical metadata and microbial meta'omic features. MaAsLin2 relies on general linear models to accommodate most modern epidemiological study designs, including cross-sectional and longitudinal, and offers a variety of data exploration, normalization, and transformation methods.

If you use the MaAsLin2 software, please cite our manuscript: 
Himel Mallick, Timothy L. Tickle, Lauren J. McIver, 
Gholamali Rahnavard, George Weingart, Joseph N. Paulson, 
Siyuan Ma, Boyu Ren, Emma Schwager, Ayshwarya Subramanian, 
Eric A. Franzosa, Hector Corrada Bravo, Curtis Huttenhower. 
"Multivariable Association in Population-scale Meta'omic Surveys" 
(In Preparation).

If you have questions, please email the google group
[MaAsLin Users](https://groups.google.com/forum/#!forum/maaslin-users).

--------------------------------------------

## Contents ##
* [Description](#markdown-header-description)
* [Requirements](#markdown-header-requirements)
* [Installation](#markdown-header-installation)
* [How to Run](#markdown-header-how-to-run)
    * [Input Files](#markdown-header-input-files)
    * [Output Files](#markdown-header-output-files)
    * [Run a Demo](#markdown-header-run-a-demo)
    * [Options](#markdown-header-options)
* [Visualization](#markdown-header-visualization)
* [Troubleshooting](#markdown-header-troubleshooting)

## Description ##

MaAsLin2 was developed to find associations between microbiome
multi'omics features and complex metadata in population-scale 
epidemiological studies. The software includes multiple analysis 
methods, normalization, and transform options to customize 
analysis for your specific study. 

## Requirements ##

MaAsLin2 is an R package that can be run on the command line or 
as an R function.

## Installation ##

MaAsLin2 can be run from the command line or as an R function. 
If only running from the command line, you do not need to 
install the MaAsLin2 package but you will need to install 
the MaAsLin2 dependencies.

### From command line ###

1. Download the source: [MaAsLin2.tar.gz](https://bitbucket.org/biobakery/maaslin2/get/0.2.tar.gz)
2. Decompress the download: 
    * ``$ tar xzvf maaslin2.tar.gz``
3. Install the Bioconductor dependencies edgeR and metagenomeSeq. 
4. Install the CRAN dependencies:
    * ``$ R -q -e "install.packages(c('lmerTest','pscl','pbapply','car','dplyr','vegan','chemometrics','ggplot2','pheatmap','cplm','hash','logging','data.table','MASS','MuMIn'), repos='http://cran.r-project.org')"``
5. Install the MaAsLin2 package (only r,equired if running as an R function): 
    * ``$ R CMD INSTALL maaslin2``

### From R ###

Install Bioconductor and then install Maaslin2

```{r, eval=FALSE}
if(!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("Maaslin2")
```

## How to Run ##

MaAsLin2 can be run from the command line or as an R function. Both 
methods require the same arguments, have the same options, 
and use the same default settings.

### Input Files ###

MaAsLin2 requires two input files.

1. Data (or features) file
    * This file is tab-delimited.
    * Formatted with features as columns and samples as rows.
    * The transpose of this format is also okay.
    * Possible features in this file include taxonomy or genes.
2. Metadata file
    * This file is tab-delimited.
    * Formatted with features as columns and samples as rows.
    * The transpose of this format is also okay.
    * Possible metadata in this file include gender or age.

The data file can contain samples not included in the metadata file
(along with the reverse case). For both cases, those samples not 
included in both files will be removed from the analysis. 
Also the samples do not need to be in the same order in the two files.

NOTE: If running MaAsLin2 as a function, the data and metadata 
inputs can be of type ``data.frame`` instead of a path to a file.

### Output Files ###

MaAsLin2 generates two types of output files: data and visualization.

1. Data output files
    * ``all_results.tsv``
        * This includes the same data as the data.frame returned.
        * This file contains all results ordered by increasing q-value.
        * The first columns are the metadata and feature names.
        * The next two columns are the value and coefficient from the model.
        * The next column is the standard deviation from the model.
        * The ``N`` column is the total number of data points.
        * The ``N.not.zero`` column is the total of non-zero data points.
        * The pvalue from the calculation is the second to last column.
        * The qvalue is computed with `p.adjust` with the correction method.
    * ``significant_results.tsv``
        * This file is a subset of the results in the first file.
        * It only includes associations with q-values <= to the threshold.
    * ``residuals.rds``
        * This file contains a data frame with residuals for each feature.
    * ``maaslin2.log``
        * This file contains all log information for the run.
        * It includes all settings, warnings, errors, and steps run.
2. Visualization output files
    * ``heatmap.pdf``
        * This file contains a heatmap of the significant associations.
    * ``[a-z/0-9]+.pdf``
        * A plot is generated for each significant association.
        * Scatter plots are used for continuous metadata.
        * Box plots are for categorical data.
        * Data points plotted are after normalization, filtering, and transform.

### Run a Demo ###

Example input files can be found in the ``inst/extdata`` folder 
of the MaAsLin2 source. The files provided were generated from
the HMP2 data which can be downloaded from https://ibdmdb.org/ .

``HMP2_taxonomy.tsv``: is a tab-demilited file with species as columns and samples as rows. It is a subset of the taxonomy file so it just includes the species abundances for all samples.

``HMP2_metadata.tsv``: is a tab-delimited file with samples as rows and metadata as columns. It is a subset of the metadata file so that it just includes some of the fields.


#### Command line ####

``$ Maaslin2.R --transform=AST --fixed_effects="diagnosis,dysbiosisnonIBD,dysbiosisUC,dysbiosisCD,antibiotics,age" --random_effects="site,subject" --standardize=FALSE inst/extdata/HMP2_taxonomy.tsv inst/extdata/HMP2_metadata.tsv demo_output``

* Make sure to provide the full path to the MaAsLin2 executable (ie ./R/Maaslin2.R).
* In the demo command:
    * ``HMP2_taxonomy.tsv`` is the path to your data (or features) file
    * ``HMP2_metadata.tsv`` is the path to your metadata file
    * ``demo_output`` is the path to the folder to write the output


#### In R ####

```{r}
library(Maaslin2)
input_data <- system.file(
    'extdata','HMP2_taxonomy.tsv', package="Maaslin2")
input_metadata <-system.file(
    'extdata','HMP2_metadata.tsv', package="Maaslin2")
fit_data <- Maaslin2(
    input_data, input_metadata, 'demo_output', transform = "AST",
    fixed_effects = c('diagnosis', 'dysbiosisnonIBD','dysbiosisUC','dysbiosisCD', 'antibiotics', 'age'),
    random_effects = c('site', 'subject'),
    standardize = FALSE)
```

##### Session Info #####

Session info from running the demo in R can be displayed with the following command.

```{r}
sessionInfo()
```

### Options ###

Run MaAsLin2 help to print a list of the options and the default settings.


$ Maaslin2.R --help
Usage: ./R/Maaslin2.R [options] <data.tsv> <metadata.tsv> <output_folder>


Options:
    -h, --help
        Show this help message and exit

    -a MIN_ABUNDANCE, --min_abundance=MIN_ABUNDANCE
        The minimum abundance for each feature [ Default: 0 ]

    -p MIN_PREVALENCE, --min_prevalence=MIN_PREVALENCE
        The minimum percent of samples for which a feature 
        is detected at minimum abundance [ Default: 0.1 ]

    -s MAX_SIGNIFICANCE, --max_significance=MAX_SIGNIFICANCE
        The q-value threshold for significance [ Default: 0.25 ]

    -n NORMALIZATION, --normalization=NORMALIZATION
        The normalization method to apply [ Default: TSS ]
        [ Choices: TSS, CLR, CSS, NONE, TMM ]

    -t TRANSFORM, --transform=TRANSFORM
        The transform to apply [ Default: LOG ]
        [ Choices: LOG, LOGIT, AST, NONE ]

    -m ANALYSIS_METHOD, --analysis_method=ANALYSIS_METHOD
        The analysis method to apply [ Default: LM ]
        [ Choices: LM, CPLM, ZICP, NEGBIN, ZINB ]

    -r RANDOM_EFFECTS, --random_effects=RANDOM_EFFECTS
        The random effects for the model, comma-delimited
        for multiple effects [ Default: none ]

    -f FIXED_EFFECTS, --fixed_effects=FIXED_EFFECTS
        The fixed effects for the model, comma-delimited
        for multiple effects [ Default: all ]

    -c CORRECTION, --correction=CORRECTION
        The correction method for computing the 
        q-value [ Default: BH ]

    -z STANDARDIZE, --standardize=STANDARDIZE
        Apply z-score so continuous metadata are 
        on the same scale [ Default: TRUE ]

    -l PLOT_HEATMAP, --plot_heatmap=PLOT_HEATMAP
        Generate a heatmap for the significant 
        associations [ Default: TRUE ]

    -i HEATMAP_FIRST_N, --heatmap_first_n=HEATMAP_FIRST_N
        In heatmap, plot top N features with significant 
        associations [ Default: TRUE ]

    -o PLOT_SCATTER, --plot_scatter=PLOT_SCATTER
        Generate scatter plots for the significant
        associations [ Default: TRUE ]

    -e CORES, --cores=CORES
        The number of R processes to run in parallel
        [ Default: 1 ]

## Troubleshooting ##

1. Question: When I run from the command line I see the error ``Maaslin2.R: command not found``. How do I fix this? 
    * Answer: Provide the full path to the executable when running Maaslin2.R.
2. Question: When I run as a function I see the error ``Error in library(Maaslin2): there is no package called 'Maaslin2'``. How do I fix this? 
    * Answer: Install the R package and then try loading the library again.
3. Question: When I try to install the R package I see errors about dependencies not being installed. Why is this?
    * Answer: Installing the R package will not automatically install the packages MaAsLin2 requires. Please install the dependencies and then install the MaAsLin2 R package.

