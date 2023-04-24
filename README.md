---
title: "myExpressAnalystR"
output: html_document
---
- [Description](#description)
- [Installation](#installation)
  * [1. Install package dependencies](#install-package-dependencies)
  * [2. Install the package](#install-the-package)
- [Tips for using the myExpressAnalystR package](#tips-for-using-the-expressanalystr-package)
- [Examples](#examples)
  * [1. Starting from a gene expression matrix](#starting-from-a-gene-expression-matrix)
    + [1.1 Load myExpressAnalystR library and initialize R objects](#load-expressanalystr-library-and-initialize-r-objects)
    + [1.2 Read data table](#read-data-table)
    + [1.3 Annotate gene IDs to Entrez](#annotate-gene-ids-to-entrez)
    + [1.4 Perform data filtering and normalization](#perform-data-filtering-and-normalization)
    + [1.5 Prepare differential expression (DE) analysis](#prepare-differential-expression--de--analysis)
    + [1.6 Perform DE analysis and check DE results](#perform-de-analysis-and-check-de-results)
    + [1.7 Visualize gene expression pattern of individual gene](#visualize-gene-expression-pattern-of-individual-gene)
  * [2. Starting from three datasets for meta-analysis](#starting-from-three-datasets-for-meta-analysis)
    + [2.1 Load myExpressAnalystR library and initialize R objects](#load-expressanalystr-library-and-initialize-r-objects)
    + [2.2 Process each individual dataset](#process-each-individual-dataset)
    + [2.3 Perform data integrity check (compatibility)](#perform-data-integrity-check--compatibility-)
    + [2.4 Check diagnostic plot and perform batch correction](#check-diagnostic-plot-and-perform-batch-correction)
    + [2.5 Perform statistical meta-analysis using combine p-values method](#perform-statistical-meta-analysis-using-combine-p-values-method)
    + [2.6 View result tables](#view-result-tables)

## Description

**_ExpressAnalystR_** is the underlying R package synchronized with ExpressAnalyst web server. It is designed for statistical analysis, enrichment analysis and visual analytics of single and multiple gene expression data, both matrix and gene list. The R
package is composed of R functions necessary for the web-server to perform data annotation, normalization, differential expression and meta-analysis.

Following installation and loading of myExpressAnalystR, users will be able to reproduce web server results from their local computers using the R command history downloaded from myExpressAnalystR. Running the R functions will allow more flexibility and reproducibility.

Note - myExpressAnalystR is still under development - we cannot guarantee full functionality

## Installation

### 1. Install package dependencies

To use myExpressAnalystR, make sure your R version is >4.0.3 and install all package dependencies. Ensure that you are able to download packages from Bioconductor. To install package dependencies, use the pacman R package. Note that some of these packages may require additional library dependencies that need to be installed prior to their own successful installation.

```
install.packages("pacman")

library(pacman)

pacman::p_load(igraph, RColorBrewer, qs, rjson, RSQLite)
```

### 2. Install the package

myExpressAnalystR is freely available from GitHub. The package documentation, including the vignettes for each module and user manual is available within the downloaded R package file. If all package dependencies were installed, you will be able to install the myExpressAnalystR. 

Install the package directly from github using the _devtools_ package. Open R and enter:

```
# Step 1: Install devtools
install.packages('devtools')
library(devtools)

# Step 2: Install myExpressAnalystR WITHOUT documentation
devtools::install_github("xia-lab/myExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual", "--no-build-vignettes"))

# Step 2: Install myExpressAnalystR WITH documentation
devtools::install_github("xia-lab/myExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual"), build_vignettes = TRUE)
```

## Tips for using the myExpressAnalystR package

1. The first function that you will use in every module is the `Init.Data` function, which initiates R objects that stores user's data, parameters for further processing and analysis.
2. The myExpressAnalystR package will output data files/tables/analysis/networks outputs in your current working directory.
3. Every function must be executed in sequence as it is shown on the R Command history, please do not skip any commands as this can result in errors downstream.
4. Main functions in myExpressAnalystR are documented. Use the _?Function_ format to open its documentation. For instance, use `?myExpressAnalystR::ReadTabExpression` to find out more about this function.
5. It is recommended to set the working folder to an empty folder because numerous files will be generated through the process.
6. R package is not useful for visual analytics as they are hosted on the website. It's mainly useful for statistical analysis (differential expression and meta-analysis).
7. R package is derived from R scripts used for powering web server. The values returned are often not useful in the context of local usage. The results from R functions are saved in a format qs file named as the file name of original data table. For gene list, the format qs file is named "datalist1". use this function to access: 
```dataSet <- readDataset(fileName)```

## Examples

### 1. Starting from a gene expression matrix
Before you start, please download the [example](https://www.expressanalyst.ca/ExpressAnalyst/resources/data/test/estrogen.txt) dataset. It is a microarray gene expression data of a human breast-cancer cell line. <br />
#### 1.1 Load myExpressAnalystR library and initialize R objects
```
library(myExpressAnalystR)

#boolean FALSE indicates local mode (vs web mode);
Init.Data(FALSE);

# Set analysis type to single gene expression matrix
SetAnalType("onedata");
```
#### 1.2 Read data table
```
dataSets <- ReadTabExpressData("estrogen.txt");
```
#### 1.3 Annotate gene IDs to Entrez
For this step it is imortant to please select correct organism, data type (array or RNA-seq), id type and gene-level summary (mean, median, sum).
For gene-level summary, microarray can use mean or median while RNA-seq needs to be sum.
```
dataSets <- PerformDataAnnot("estrogen.txt", "hsa", "array", "hgu95av2", "mean");
```
Take a look at the mapped dataset by reading the dataset's qs file using ```readDataset(fileName)``` function.
```
dataSet <- readDataset("estrogen.txt")
print(head(dataSet$data.anot[c(1:5),]))
```
Check diagnostics plots to look at overall data distribution, sample separation.
```
PlotDataBox("estrogen.txt", "qc_boxplot_", 72, "png");
PlotDataPCA("estrogen.txt", "qc_pca_", 72, "png");
```
Check your working directory for png images named ``qc_boxplot_dpi72.png`` and ``qc_pca_dpi72.png``, open them. <br />
![Box Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_boxplot.png) <br />
Box plot shows that the expression distribution of samples are between around -4 to 12.5. This shows that the data has already been normalized. <br />
![PCA Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_pca.png)
PCA plot shows sample separation both between absent and present, and also, low and high. Depending of your experimental design, try to see if the samples are separated by the metadata of interest, it can also be used to see whether there are potentially mislabed sample.

#### 1.4 Perform data filtering and normalization
No normalization need to be performed, PCA plot from previous step shows that the dataset is already normalized.
Filter by variance (lower 15% removed)
Filter by relative abundance (lower 4 percentile of average expression signal)
Filter by count not applied (only for RNASeq data)
Filter unannotated genes TRUE
```
dataSets <- PerformExpressNormalization("estrogen.txt", "none", 15, 4, 0);
```
#### 1.5 Prepare differential expression (DE) analysis
Selected metadata of interest, in this case we are interested in investigating the effect of presence of Estrogen Receptor (ER) vs absence. We are not setting secondary factor and blocking factor. After selecting metadata, compute design matrix and select DE analysis algorithm by running ``SetupDesignMatrix`` function. For microarray data, only ``limma`` can be used.
```
dataSets <- SetSelectedMetaInfo("estrogen.txt","ER", "NA", F);
dataSets <- SetupDesignMatrix("estrogen.txt", "limma");
```
#### 1.6 Perform DE analysis and check DE results
Fold change is log2 transformed.
Adjusted P-value using False Discovery Rate (FDR) method.
```
dataSets <- PerformDEAnal("estrogen.txt", "custom", "absent vs. present", "NA", "intonly");
dataSet <- readDataset("estrogen.txt");
print(head(dataSet$comp.res));
```

#### 1.7 Visualize gene expression pattern of individual gene
```
PlotSelectedGene("estrogen.txt","5111");
```
Check the resulting png image (Gene_5111.png) in your working directory. <br />
![Violin Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/Gene_5111.png)

### 2. Starting from three datasets for meta-analysis
Before you start, please download the example datasets into your working directory [E-GEOD-25713](https://dev.expressanalyst.ca/resources/data/test/E-GEOD-25713.txt), [E-GEOD-59276.txt](https://dev.expressanalyst.ca/resources/data/test/E-GEOD-59276.txt),
[GSE69588.txt](https://dev.expressanalyst.ca/resources/data/test/GSE69588.txt). These three testing datasets (containing subset of 5000 genes) are from a meta-analysis of helminth infections in mouse liver.

#### 2.1 Load myExpressAnalystR library and initialize R objects
```
library(myExpressAnalystR)

#boolean FALSE indicates local mode (vs web mode);
Init.Data(FALSE);

# Set analysis type to meta-analysis
SetAnalType("metadata");
```

#### 2.2 Process each individual dataset
```
#Read dataset text file
dataSets <- ReadOmicsData("E-GEOD-25713.txt");
dataSets <- SanityCheckData("E-GEOD-25713.txt");

#Map gene id to entrez id
dataSets <- AnnotateGeneData("E-GEOD-25713.txt", "mmu", "entrez");
```

Visually inspect dataset using box plot (``qc_boxplot_0_dpi72.png``) and pca plot (``qc_pca_0_dpi72.png``).
```
PlotDataProfile("E-GEOD-25713.txt", "raw", "qc_boxplot_0_", "qc_pca_0_");
```
![Box Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_boxplot_meta.png)
![PCA Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_pca_meta.png)

```
#Remove variables with more than 50% missing data
dataSets <- RemoveMissingPercent("E-GEOD-25713.txt", 0.5);

#Replace missing value with minimum values across dataset
dataSets <- ImputeMissingVar("E-GEOD-25713.txt", "min");

#Replace missing value with minimum values across dataset
dataSets <- NormalizingDataMeta("E-GEOD-25713.txt", "NA");
dataSets <- PerformDEAnalMeta("E-GEOD-25713.txt", "limma", "CLASS", 0.05, 0.0);

#read and process the other two datasets
dataSets <- ReadOmicsData("E-GEOD-59276.txt");
dataSets <- SanityCheckData("E-GEOD-59276.txt");
dataSets <- AnnotateGeneData("E-GEOD-59276.txt", "mmu", "entrez");
dataSets <- RemoveMissingPercent("E-GEOD-59276.txt", 0.5)
dataSets <- ImputeMissingVar("E-GEOD-59276.txt", "min")
dataSets <- NormalizingDataMeta("E-GEOD-59276.txt", "NA");
dataSets <- PerformDEAnalMeta("E-GEOD-59276.txt", "limma", "CLASS", 0.05, 0.0);

dataSets <- ReadOmicsData("GSE69588.txt");
dataSets <- SanityCheckData("GSE69588.txt");
dataSets <- AnnotateGeneData("GSE69588.txt", "mmu", "entrez");
dataSets <- RemoveMissingPercent("GSE69588.txt", 0.5)
dataSets <- ImputeMissingVar("GSE69588.txt", "min")
dataSets <- NormalizingDataMeta("GSE69588.txt", "NA");
dataSets <- PerformDEAnalMeta("GSE69588.txt", "limma", "CLASS", 0.05, 0.0);
```
#### 2.3 Perform data integrity check (compatibility)
```
CheckMetaDataIntegrity();
```
#### 2.4 Check diagnostic plot and perform batch correction
```
PlotMetaPCA("qc_meta_pca_","72", "png", "");
```
![PCA Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_meta_pca_beforeBatch.png) <br />
There is clear signs of batch effect. The samples from same dataset are largely clustered together. To remove the batch effect, we need to run comBat batch correction algorithm
```
#Apply batch effect correction
PerformBatchCorrection();

#Check the result 
PlotMetaPCA("qc_meta_pca_afterBatch_","72", "png", "");
```
Here is the result after batch correction. <br />
![PCA Plot](https://dev.expressanalyst.ca/ExpressAnalyst/resources/images/RTutorial/qc_meta_pca_afterBatch.png) <br />

#### 2.5 Perform statistical meta-analysis using combine p-values method
```
analSet <- PerformPvalCombination("fisher", 0.05)
```
#### 2.6 View result tables
```
analSet <- readSet(analSet, "analSet");
print(head(analSet$meta.mat));
       CombinedTstat CombinedPval
16854         89.093   2.7728e-14
246256        99.964   2.7728e-14
105855        94.751   2.7728e-14
19241        105.030   2.7728e-14
319169        94.339   2.7728e-14
16819        100.880   2.7728e-14
```
For a more detailed table containing additionally log fold change and p-values of features for individual dataset, please check this csv file [meta_sig_genes_metap.csv](https://dev.expressanalyst.ca/ExpressAnalyst/resources/data/RTutorial/meta_sig_genes_metap.csv), it is also generated in your working directory.
