# Overview 
Disseminated tumor cells (DTCs) are pivotal in understanding cancer metastasis and advancing liquid biopsy-based diagnostics. Traditional detection approaches, hindered by their dependency on epithelial markers and the requirement for cell fixation, impair the exploration and identification of varied DTC phenotypes within liquid biopsies. We describe DTCFinder, a bench-to-bits, label-free method integrating single-cell RNA sequencing and machine learning, enabling unbiased detection, in-depth molecular characterization, and precise tissue origin tracing of DTCs across various types of liquid biopsies, even with ultra-scarce DTC presence and low sequencing depth. DTCFinder offers a robust and versatile toolkit for liquid biopsy-based cancer diagnostics, enhancing our understanding of the underlying biology of cancer metastasis.

# OS requirements
For smooth operation, DTCFinder requires approximately 350MB of hard drive space for storing essential data and code. It operates within the R software environment, ensuring compatibility and ease of use. We have tested DTCFinder on both Windows 10 and Linux (Ubuntu) platforms

# Dependencies
DTCFinder is designed to be versatile and is not tied to a specific version of R. However, it does require the installation of certain R packages to function optimally. Here is a recommended configuration for setting up DTCFinder:

* R_4.2.1
* Matrix_1.5-3
* rhdf5_2.40.0
* RColorBrewer_1.1-3
* ggplot2_3.4.3
* ggplotify_0.1.0
* mixtools_2.0.0
* pheatmap_1.0.12
* squash_1.0.9
* magick_2.7.3
* grid_4.2.1
* GenomicRanges_1.48.0
* ggbio_1.44.1
* Hmisc_4.7-2
* ggcorrplot_0.1.4
* Epi_2.47

# Installation

    if (!requireNamespace("devtools", quietly = TRUE))
        install.packages("devtools")
    options(timeout=360)
    devtools::install_github("WeiWeiLab/DTCFinder", ref = "master", auth_token = "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")

Installation duration is typically around 10 minutes, varying based on hardware and network conditions. options(timeout=360) is used to increase the timeout setting, in case that you might encounter download issues during installation.


#
# Usage

To run DTCFinder, simply type in

    DTCFinder(PathToData, seed, p_value, bin_size, title)

* **PathToData:** Specify one of the following data inputs: (1) A folder containing sparse data matrices from 10x Genomics Cell Ranger (Version 3 or 2), including 'barcodes.tsv', 'features.tsv' (or 'genes.tsv'), and 'matrix.mtx.gz' (or 'matrix.mtx'); (2) A H5 file of sparse data matrices from the 10x Genomics Cell Ranger pipeline; (3) A raw count matrix in tsv or csv format, with row names (genes) and column names (cell barcodes).
* **seed:** Sets a starting point for random processes, ensuring reproducible results.
* **p_value:** Defines the threshold for outlier identification (malignant cells). A smaller value can offer higher specificity at the cost of sensitivity. The default setting (1e-8) is optimal for blood, pleural effusion, and cerebrospinal fluid samples.
* **bin_size:** Determines the sliding window size (number of genes) for genomic segments across chromosomes (150 genes by default).
* **title:** A customizable field for additional information, such as sample name.

DTCFinder generates several outputs: 
* A summary plot as **a png file**, including (a) the histogram of accumulated outlier bins (AOBs) with baseline immune cells and DTCs identified; (b) a heatmap showing the inferred genome-wide CNA profile of identified DTCs; (c) a heatmap showing the pairwise correlation of inferred CNA profiles across all the DTCs; and (d) a circular heatmap showing the tissue origin and tumor type (TOTT) prediction based on DTCs' transcriptome profile.
* The expression matrix of identified DTCs as **DTC_expr.txt**.
* A table of z-score standardized posterior probabilities (surrogates of inferred CNAs) of DTCs across all the genomic segments/bins as **CNAs.txt**.
* A pairwise correlation matrix of inferred CNA profiles of all the DTCs as **correlation.txt**.
* TOTT prediction scores of DTCs as **prediction.txt** (a positive score denotes a positive prediction of a particular cancer type).  

All of these output files will be saved in the current working directory of R. To check your current working directory, use
    
    getwd()

#
# Demo

**Example: Analyzing DTCs in Cerebrospinal Fluid**

This step-by-step demo uses DTCFinder to identify and analyze DTCs in a cerebrospinal fluid sample from a lung adenocarcinoma patient. The dataset 'LUAD_CSF' is included in DTCFinder for testing. LUAD_CSF is a data frame with over 21000 features (genes) and 5300 variables (cells). This demo should take about an hour, depending on your hardware and network environment.

1. Load DTCFinder Library

       library("DTCFinder")

2. Load a count (UMI) matrix of a DTC-enriched CSF sample from a LUAD patient

       data("LUAD_CSF")

3. Run DTCFinder with default settings. If the cell number is larger than 10000, it will take much longer to load and filter the data.  

       M = DTCFinder("LUAD_CSF", seed = 10003, p_value = 1e-8, title = "LUAD, CSF")

A summary plot will be generated as follows and saved as a png file. 

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/output.png?raw=true)

To create a histogram that showcases the number of DTCs detected, use the following command: 

    DTCPlot(M, title="LUAD, CSF")

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/histogram.png?raw=true)
This plot will illustrate the accumulated outlier bins (AOBs), comparing the baseline immune cells with identified DTCs.

For a deeper insight into the genomic alterations in DTCs, generate a heatmap using:

    DTCHeatmap(M, title="LUAD, CSF")

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/DTCHeatmap.png?raw=true)

The generated CNA heatmap of identified DTCs can be compared with CNAs of primary tumor experimentally measured through low-coverage whole-genome sequencing to reveal the consistant and DTC-specific CNA regions  

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/CNV_DNA_1.png?raw=true)

The **correlation.txt** file produced by DTCFinder contains a matrix showing the pairwise correlation of inferred CNA profiles across all identified DTCs. This matrix is particularly useful for identifying potential subclones of DTCs that exhibit distinct genomic CNA patterns. 

To trace the tissue origin and predict the tumor type based on the transcriptome profiles of the DTCs, use the following command:

    (prediction(M$Data, M$DTCs))

with the following output (for LUAD_CSF data)

    "Prediction: LUAD"
    LUAD  0.59289214
    LUSC -0.75417122
    SCLC -0.70157168
    ACC  -0.89046105
    BLCA -0.85445353
    BRCA -0.55309393
    CESC -0.50433343
    COAD/READ -0.86942346
    GBM  -1.63532972
    HNSC -0.83486803
    KICH -0.73387129
    KIRP -0.65473305
    KIRC -1.05032990
    LGG  -0.56651189
    LIHC/CHOL -0.55257435
    MESO -1.02252912
    OV   -1.21711166
    PAAD -1.74405036
    PCPG -0.85117228
    PRAD -1.09686805
    SARC -0.91576885
    SKCM -0.96411297
    STAD -0.09367624
    TGCT -1.14385551
    THCA -0.12498131
    THYM -1.14346096
    UCEC -1.10233538
    UCS  -1.09680587
    UVM  -0.59709604

The transcriptome profiles of identified DTCs, available in **DTC_expr.txt**, can be used to interrogate various molecular signatures of DTCs or compare against match primary tumor cells.
