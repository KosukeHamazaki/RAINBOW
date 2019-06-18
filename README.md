# RAINBOW
###   Reliable Association INference By Optimizing Weights
#### Author : Kosuke Hamazaki (hamazaki@ut-biomet.org)
#### Date : 2019/03/25

In this repository, the `R` package `RAINBOW` is available.
Here, we describe how to install and how to use `RAINBOW`.

----------
## What is `RAINBOW`
`RAINBOW`(Reliable Association INference By Optimizing Weights) is a package to perform several types of `GWAS` as follows.

- Single-SNP GWAS with `RGWAS.normal` function
- SNP-set (or gene set) GWAS with `RGWAS.multisnp` function (which tests multiple SNPs at the same time)
- Check epistatic (SNP-set x SNP-set interaction) effects with `RGWAS.epistasis` (very slow and less reliable)

`RAINBOW` also offers some functions to solve the linear mixed effects model.

- Solve one-kernel linear mixed effects model with `EMM.cpp` function
- Solve multi-kernel linear mixed effects model with `EM3.cpp` function (for the general kernel, not so fast)
- Solve multi-kernel linear mixed effects model with `EM3.linker.cpp` function (for the linear kernel, fast)

By utilizing these functions, you can estimate the genomic heritability and perform genomic prediction (`GP`).

Finally, `RAINBOW` offers other useful functions.

- `qq` and `manhattan` function to draw Q-Q plot and Manhattan plot
- `modify.data` function to match phenotype and marker genotype data
- `CalcThreshold` function to calculate thresholds for GWAS results
- `See` function to see a brief view of data (like `head` function, but more useful)
- `genetrait` function to generate pseudo phenotypic values from marker genotype
- `SS_GWAS` function to summarize GWAS results (only for simulation study)

## Installation
`RAINBOW` is now available on [`GitHub`](https://github.com/KosukeHamazaki/RAINBOW), so please run the following code in the R console.

``` r
### If you have not installed yet, ...
install.packages("devtools")  

### Install RAINBOW from GitHub
devtools::install_github("KosukeHamazaki/RAINBOW")
```

If you get some errors via installation, please check if the following packages are correctly installed.

``` r
Rcpp,
RcppEigen,
rrBLUP,
rgl,
tcltk,
Matrix,
cluster,
MASS
```

In `RAINBOW`,  since part of the code is written in `Rcpp` (`C++` in `R`),  please check if you can use `C++` in `R`.
For `Windows` users,  you should install [`Rtools`](https://cran.r-project.org/bin/windows/Rtools/).

Shortly, we will try to publish `RAINBOW` on `CRAN`.

If you have any questions about the installation, please contact us by e-mail (hamazaki@ut-biomet.org).


##  Usage
First, import `RAINBOW` package and load example datasets. These example datasets consist of marker genotype (scored with {-1, 0, 1}, 1,536 SNP chip (Zhao et al., 2010; PLoS One 5(5): e10780)), map with physical position, and phenotypic data (Zhao et al., 2011; Nature Communications 2:467). Both datasets can be downloaded from `Rice Diversity` homepage (http://www.ricediversity.org/data/). 

``` r
### Import RAINBOW
require(RAINBOW)

### Load example datasets
data("Rice_Zhao_etal")

### View each dataset
See(Rice_geno_score)
See(Rice_geno_map)
See(Rice_pheno)
```
You can check the original data format by `See` function.
Then, select one trait (here, `Flowering.time.at.Arkansas`) for example.

``` r
### Select one trait for example
trait.name <- "Flowering.time.at.Arkansas"
y <- Rice_pheno[, trait.name, drop = FALSE]
```

For  GWAS, first you can remove  SNPs whose MAF <= 0.05 by `MAF.cut` function.

``` r
### Remove SNPs whose MAF <= 0.05
x.0 <- t(Rice_geno_score)
MAF.cut.res <- MAF.cut(x.0 = x.0, map.0 = Rice_geno_map)
x <- MAF.cut.res$x
map <- MAF.cut.res$map
```

Next, we estimate the additive genetic relationship matrix by using the `rrBLUP` package.

``` r
### Estimate genetic relationship matrix 
K.A <- rrBLUP::A.mat(x) ### rrBLUP package can be installed by install.packages("rrBLUP")
```

Next, we modify these data into the GWAS format of `RAINBOW` by `modify.data` function.

``` r
### Modify data
modify.data.res <- modify.data(pheno.mat = y, geno.mat = x, map = map,
                               return.ZETA = TRUE, return.GWAS.format = TRUE)
pheno.GWAS <- modify.data.res$pheno.GWAS
geno.GWAS <- modify.data.res$geno.GWAS
ZETA <- modify.data.res$ZETA

### View each data for RAINBOW
See(pheno.GWAS)
See(geno.GWAS)
str(ZETA)
```
`ZETA` is a list of genetic relationship matrix and its design matrix.

Finally, we can perform `GWAS` using these data.
First, we perform single-SNP GWAS by `RGWAS.normal` function as follows.

``` r
### Perform single-SNP GWAS
normal.res <- RGWAS.normal(pheno = pheno.GWAS, geno = geno.GWAS,
                           ZETA = ZETA, n.PC = 4, P3D = TRUE)
See(normal.res$D)  ### Column 4 contains -log10(p) values for markers
### Automatically draw Q-Q plot and Manhattan by default.
```

Next, we perform SNP-set GWAS by `RGWAS.multisnp` function.

``` r
### Perform SNP-set GWAS (by regarding 11 SNPs as one SNP-set)
SNP_set.res <- RGWAS.multisnp(pheno = pheno.GWAS, geno = geno.GWAS, ZETA = ZETA, 
                              n.PC = 4, test.method = "LR", kernel.method = "linear", gene.set = NULL,
                              test.effect = "additive", window.size.half = 5, window.slide = 11)

See(SNP_set.res$D)  ### Column 4 contains -log10(p) values for markers
```

You can perform SNP-set GWAS with sliding window by setting `window.slide = 1`.
And you can also perform gene-set (or haplotype-based) GWAS by assigning the following data set to `gene.set` argument.

ex.)

|  gene (or haplotype block)   |  marker | 
| :-----: | :------:| 
| gene_1    | id1000556 | 
| gene_1    | id1000673 | 
| gene_2    | id1000830 | 
| gene_2    | id1000955 | 
| gene_2    | id1001516 | 
| ...    | ... | 


### Help
If you have some help before performing `GWAS` with `RAINBOW`, please see the help for each function by `?function_name`.
You can also check how to determine each argument by

``` r
RGWAS.menu()
```
`RGWAS.menu` function asks you some questions, and by answering these questions, the function tells you how to determine which function use and how to set arguments.


### Citation
If you use `RAINBOW`, please cite [article published in `bioRxiv`](https://www.biorxiv.org/content/10.1101/612028v1) as follows.

Hamazaki, K. and Iwata, H. (2019) Haplotype-based genome wide association study using a novel SNP-set method : RAINBOW. bioRxiv. January 2019: 612028.


## References
Kennedy, B.W., Quinton, M. and van Arendonk, J.A. (1992) Estimation of effects of single genes on quantitative traits. J Anim Sci. 70(7): 2000-2012.

Storey, J.D. and Tibshirani, R. (2003) Statistical significance for genomewide studies. Proc Natl Acad Sci. 100(16): 9440-9445.

Yu, J. et al. (2006) A unified mixed-model method for association mapping that accounts for multiple levels of relatedness. Nat Genet. 38(2): 203-208.

Kang, H.M. et al. (2008) Efficient Control of Population Structure in Model Organism Association Mapping. Genetics. 178(3): 1709-1723.

Kang, H.M. et al. (2010) Variance component model to account for sample structure in genome-wide association studies. Nat Genet. 42(4): 348-354.

Zhang, Z. et al. (2010) Mixed linear model approach adapted for genome-wide association studies. Nat Genet. 42(4): 355-360.

Endelman, J.B. (2011) Ridge Regression and Other Kernels for Genomic Selection with R Package rrBLUP. Plant Genome J. 4(3): 250.

Endelman, J.B. and Jannink, J.L. (2012) Shrinkage Estimation of the Realized Relationship Matrix. G3 Genes, Genomes, Genet. 2(11): 1405-1413.

Su, G. et al. (2012) Estimating Additive and Non-Additive Genetic Variances and Predicting Genetic Merits Using Genome-Wide Dense Single Nucleotide Polymorphism Markers. PLoS One. 7(9): 1-7.

Zhou, X. and Stephens, M. (2012) Genome-wide efficient mixed-model analysis for association studies. Nat Genet. 44(7): 821-824.

Listgarten, J. et al. (2013) A powerful and efficient set test for genetic markers that handles confounders. Bioinformatics. 29(12): 1526-1533.

Lippert, C. et al. (2014) Greater power and computational efficiency for kernel-based association testing of sets of genetic variants. Bioinformatics. 30(22): 3206-3214.

Jiang, Y. and Reif, J.C. (2015) Modeling epistasis in genomic selection. Genetics. 201(2): 759-768.

Hamazaki, K. and Iwata, H. (2019) Haplotype-based genome wide association study using a novel SNP-set method : RAINBOW. bioRxiv. January 2019: 612028.
