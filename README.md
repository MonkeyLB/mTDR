# mTDR
A model-based method to comprehensively evaluate the reproducibility of chromatin profiling sequencing data

## Background

Enrichment-based chromatin profiling sequencing experiments have become essential tools to investigate the functional roles of genomic regions. ChIP-seq, ATAC-seq and DNAse-seq are among the most popular experiments. Measuring reproducibility is central to the data quality control, and critical to ensure the credibility of scientific discoveries. Evaluating the reproducibility of enrichment-based sequencing data is complicated by the variation of enrichment characteristics and the heterogeneous correlation structure between replicated samples. We present a model-based method to comprehensively assess the reproducibility between replicated samples. The method only requires minimum preprocessing of raw data and does not rely on peak calling. Thus it involves less information loss than the peak level reproducibility measure. The model is designed to assess three aspects of the data reproducibility – the dependence between the enriched signals, the bulk correlation across whole range of signal values, and the degree of lack of enrichment. By the combination of the three quantities, our model is flexible to assess the reproducibility of data with different signal types (i.e., narrow peak, broad peak) and enrichment levels. We demonstrate that our method is also more accurate than the other existing measures.

## Citation

Cite the following paper: to be added.

## Installation

Download the source package [mTDR.0.99.1.gz](https://github.com/MonkeyLB/mTDR/blob/master/mTDR_0.99.1.tar.gz) from Github. Or install it from Bioconductor.

## Method overview

A crucial step of ChIP-seq experiment is to enrich the targeted regions. The enriched signals are usually of the primary interest to biologists. Analyses of ChIP-seq data typically rely on peak calling. Our method directly models the binned reads rather than peaks, so it is independent of peak calling and is workable for broad peak data (i.e., most HM data). Our strategy is to handle heterogeneity by modeling the behavior of enriched signal and the background separately through mixture of two copulas. Unlike traditional probability distribution functions, copula models the structure of dependence between the cumulative densities of random variables (equivalent to ranks) rather than the variables’ original values.

We quantify the correlation of the enriched signals by using upper tail dependence. Intuitively, upper tail dependence of two random variables is defined as the probability that one exceeds a given threshold of cumulative density t, conditioning on that the other has exceeded the same t as t → 1 from left. Upper tail dependence is suitable to quantify correlation between the enriched signals because it measures comovement of random variables toward the high end of data distribution. Since it’s defined based on cumulative density, it is scale-free and not affected by dynamic range of data values and outliers. Some copula models innately possess upper tail dependence. We thus can adopt a suitable copula to fit the data and estimate the upper tail dependence. We observe that the cumulative density of a pair of sufficiently enriched ChIP-seq replicates is best approximated by survival Clayton (s-Clayton) copula (Figure 1).

The upper tail dependence alone is not sufficient to depict the whole picture of the data reproducibility as it only describes the local correlation at the tail. We introduce a Gaussian copula to capture the correlation of the bulk of the data. The Gaussian copula does not have upper tail dependence, but it has a parameter that measures the correlation in general. In fact, because typically the data points at the tail only takes up a small proportion, the bulk correlation parameter greatly depends on the relatively weak signals that are not at the tail. Therefore, we can use the correlation parameter from the Gaussian copula to complementarily capture the correlation of data that is not described by tail dependence of the s-Clayton copula. Combining the two copulas, we constructed a flexible mixture model that can fit a wide range of ChIP-seq data with different degrees of enrichment and signal characteristics.


![Figure1. Mixture copula model to evaluate the reproducibility of ChIP-seq data](https://github.com/MonkeyLB/mTDR/blob/master/vignettes/Figure1.JPG)

## Preprocessing your data

We suggest to divide the genome into non-overlapping bins of 200 base pairs (bp), and count the number of reads within each bin. These procedures can be done by using bedtools. Following ENCODE procedure, it’s better to remove the bins falling in the blacklisted regions. To expedite the estimation, it is also suggested to remove the bins with low read counts (combined counts across replicates < 5). Bins with such low read count are obvious noise, but they could greatly increase the computation load. A script to perform the preprocessing can be found here.

## The format of input

We include an example of preprocessed ChIP-seq data. The data has two columns. Each column is a set of signals(i.e., number of reads in each 200 bp genomic bin).


## Estimation

The function est.TDR would run the EM algorithm to estimate the paramters of the mixtue copula model, and calculate the desired quantities (i.e., tail dependence). The function takes the two vectors (i.e., replicated samples) as the first two auguments. In most of cases, the default values for the other auguments are sufficient for estimation. Please refer to the documentation for a detailed explaination of the augments.

The estimated quantities are stored in the list element named para. In case you are interested in studying the convergence behavior along the iterations, the parameter and likelihood traces are stored in the element names para.trace and likelihood.trace respectively. Above is an example to show the likelihood trace. It should be monotonically increasing after first few iterations.

## Interpretation

Our mixture copula model estimates three parameters – the shape parameter β for Clayton copula, the bulk correlation parameter ρ for Gaussian copula, and the proportion parameter π. The upper tail dependence, denoted as λ, can be deterministically computed from βThe three quantities λ, ρ, π all range from 0 to 1. A higher value of λ means stronger correlation between the enriched signals of two replicates; the proportion parameter π reflects the contribution of Gaussian copula. A high value of π implies substantial signal are not sufficiently enriched.

To help end users identify suboptimal samples, we provided an empirical rule to classify samples, based on our extensive analyses on ENCODE2 data. For TF data, we recommend to classify samples with λ > 0.5 as highly reproducible, those with 0.2 < λ < 0.5 as moderately reproducible, and those with λ < 0.2 as poor agreement between replicates. While the value of π is not critical to the classification, it reflects the degree of lack of enrichment. A value of π > 0.2 typically indicates a substantial proportion of signals are not sufficiently enriched. For HM data, we recommend to use the bulk correlation ρ in addition to λ. Still, λ > 0.5 indicates high reproducibility. For the broad peak type data, it is sometimes hard to achieve λ > 0.5. If λ < 0.5, a high value of ρ (> 0.6) would also indicate high reproducibility. Both λ < 0.2 and ρ < 0.2 imply poor reproducibility.

## Computation efficiency

Given a pair of replicated samples that has 450K matched genomic bins (typical size after preprocess for ChIP-seq data), it takes 2.5 min on a laptop with 2.6GHz Intel Core i7-6600U and 8Gb of RAM.
