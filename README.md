# DTCFinder-demo
DTCFinder is an R package tool to identify disseminated tumor cells. A cell can be predicted as a DTC in light of three aspects: CNA outlier assessment, CNA pattern recognition, and cell origin classification.

Installation:  
    devtools::install_github("WeiWeiLan/DTCFinder",
                             ref = "master",
                             auth_token = "......")                          
                            )

Run the package:
    library("DTCFinder")
    DTCFinder("SCLC_BPMC", pvalue = 1e-8, seed = 123, titl e= "Test Sample")

Please contact Wei Wei for an authentication token at wwei@systemsbiology.org
