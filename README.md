# Sex Quality Control for NGS data

To identify the sex of the samples in Next Generation Sequencing data, a quality control is performed based on both the self-reported sex of the individual and two bioinformatics approaches.

## Somalier analysis
The first approach, performed by the Somalier v0.2.15. tool (Pedersen et al. 2019), identifies the sex of the sample from the depth of the X and Y chromosome reads. This software utilizes a total of 17,766 positions of coding regions to be able to work with different experiments (whole-exome and -genome sequencing, RNA-Seq, etc.). These positions accomplish several/various requisites such as, a high quality, population allele frequency around 0.5, exclusion of segmental duplications regions, low complexity regions, and regions nearby insertion and deletions. Relatedness is calculated allelic concordance from single nucleotide variants within these positions (homozygous, heterozygous, and alternative homozygous).

## Heuristic analysis
For the second approach, an in-house heuristic script was used that analyzes the depth of 11 genes in the non-pseudoautosomal regions of the X and Y chromosomes (Table 1). We assessed depth distribution along both chromosomes X and Y to set high covered genes by capture sequencing suitable for sex classification based on read depth. In brief, we extract selected gene regions from BAM files, and then, reads on these genes are counted firstly by gene, and later, the total number of reads per chromosome. Reads with a mapping quality below 50 (MappingQuality>50) are filtered out for the analysis. Once we calculate the read count, we compare the number of reads between both chromosomes. The fraction of reads in chromosome X compared to chromosome Y is calculated, and vice versa (Equation 1).

