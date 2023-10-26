# Overview 
Disseminated tumor cells (DTCs) are pivotal in understanding cancer metastasis and advancing liquid biopsy-based diagnostics. Traditional detection approaches, hindered by their dependency on epithelial markers and the requirement for cell fixation, impair the exploration and identification of varied DTC phenotypes within liquid biopsies. We describe DTCFinder, a bench-to-bits, label-free method integrating single-cell RNA sequencing and machine learning, enabling unbiased detection, in-depth molecular characterization, and precise tissue origin tracing of DTCs across various types of liquid biopsies, even with ultra-scarce DTC presence and low sequencing depth. DTCFinder offers a robust and versatile toolkit for liquid biopsy-based cancer diagnostics, enhancing our understanding of the underlying biology of cancer metastasis.

# OS requirements
DTCFinder itself requires about 350MB hard drive space to store data and codes, and the software environment R. It has been tested under both Windows and Uinux platforms.

# Dependency
DTCFinder was not developed for a specific version of R, but needs some R packages. Following is an example of R configuratioen for DTCFinder:
R_4.2.1, Matrix_1.5-3, rhdf5_2.40.0, RColorBrewer_1.1-3, ggplot2_3.4.3, ggplotify_0.1.0, mixtools_2.0.0, pheatmap_1.0.12, squash_1.0.9, magick_2.7.3, grid_4.2.1, GenomicRanges_1.48.0, ggbio_1.44.1, Hmisc_4.7-2, ggcorrplot_0.1.4, Epi_2.47.

# Installation

    if (!requireNamespace("devtools", quietly = TRUE))
        install.packages("devtools")
    devtools::install_github("WeiWeiLab/DTCFinder", dependencies = TRUE)

The installation can take about 10 minutes, dependent on the hardware and network environment. If the download fails during installation, try to set a longer timeout value. For example
    options(timeout=360)

#
# Usage

To run DTCFinder, simply type in

    DTCFinder(PathToData, seed, p_value, bin_size, title)

where PathToData can be one of these inputs: a folder of sparse data matrices from the 10x Genomics Cell Ranger either Version 3 or 2, with barcodes.tsv, features.tsv (or genes.tsv), and matrix.mtx.gz (or matrix.mtx);
a H5 file of sparse data matrices from the 10x Genomics Cell Ranger pipeline;
a raw count matrix through a text file with both row names and column names, in tsv or csv format, where rows are genes and columns are cell-barcodes;
seed is used to specify a start-point for randomization and get reproducible results; 
p_value is the threshold to define outliers (malignant cells) relative to baseline (CD45-positive cells), the smaller the more confident. 
For example, 1e-8 by default should be good for a PBMC sample, while 1e-3 would be recommended for a tissue sample. 
bin_size is the genes number used as the bin size for segmentation (150 genes by default).
title can be the sample name or something else for information.

DTCFinder will return a list, including Data (a genes-by-cells normalized expression table), 
CNVs (a matrix of Gaussian mixture model posterior probabilities (MMPP) as the CNV profile), 
DTCs (the row numbers of predicted malignant cells in MMPP), 
baseline (the row numbers, namely the number of baseline cells in MMPP), 
and model_parameters (model parameters for baseline histogram distribution, assuming that it follows a normal distribution).

#
# Demo

An example of dataset LUAD_PBMC is included in DTCFinder package for testing. LUAD_PBMC is a data frame with over 30000 features (genes) and 300 variables (cells). Running the following testing can take about an hour, dependent on the hardware and network environment.

    library("DTCFinder")

load an UMI matrix of a PBMC sample of a LUAD patient

    data("LUAD_PBMC")

Run DTCFinder. If the cell number is larger than 10000, it will take much longer to load and filter the data.  

    M = DTCFinder("LUAD_PBMC", seed = 10003, p_value = 1e-8, title = "LUAD, PBMC")

A summary plot will be generated as follows, and output to a png file. An expression matrix of putative DTCs (DTC_expr.txt), an inferred CNA table for CDTs over bins (CNAs.txt), a CNA correlation matrix of DTCs (correlation.txt), and a prediction list of DTCs origin (prediction.txt) are also generated.

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/output.png?raw=true)

To see how many DTCs have been found, you can check a histogram by using

    DTCPlot(M, title="LUAD, PBMC")

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/histogram.png?raw=true)

To validate those putative DTCs, you can check their CNAs in a heatmap by using

    DTCHeatmap(M, title="Test")

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/histogram.png?raw=true)

and compare to the CNA plot derived by using DNA-seqencing profile 

![alt text](https://github.com/WeiWeiLab/DTCFinder-demo/blob/main/plot/CNV_DNA.png.png?raw=true)

The origin of DTCs can be calculated by using

    (prediction(M$Data, M$DTCs))

The following list is output

    "Prediction: LUAD"
    LUAD  0.59289214
    LUSC -0.75417122
    SCLC -0.70157168
    ACC  -0.89046105
    BLCA -0.85445353
    BRCA -0.55309393
    CESC -0.50433343
    CHOL -0.55257435
    COAD -0.86942346
    GBM  -1.63532972
    HNSC -0.83486803
    KICH -0.73387129
    KIRP -0.65473305
    KIRC -1.05032990
    LGG  -0.56651189
    LIHC -0.72998782
    MESO -1.02252912
    OV   -1.21711166
    PAAD -1.74405036
    PCPG -0.85117228
    PRAD -1.09686805
    READ -0.90917633
    SARC -0.91576885
    SKCM -0.96411297
    STAD -0.09367624
    TGCT -1.14385551
    THCA -0.12498131
    THYM -1.14346096
    UCEC -1.10233538
    UCS  -1.09680587
    UVM  -0.59709604
