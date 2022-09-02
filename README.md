<!-- ------------------ SECTION ------------------ -->
<p align="left">
  <a href="https://www.iter.es" title="Instituto Tecnológico y de Energ&iacute;as Renovables (ITER) / Institute of Technology and Renewable Energy (ITER)">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/ITER_logo.png" width="30%" /> 
  </a>
</p>

<!-- ------------------ SECTION ------------------ -->
## Sex Quality Control for NGS data ##

Identifying the genetic sex of a sample from the sequence obtained in a Next Generation Sequencing (NGS) experiment is a mandatory quality control step for the discovery of errors in the metadata provided and to contribute to sample traceability. Here, we explain how to use the self-reported sex of an individual and two different bioinformatic approaches, `Somalier` and `Heuristic` based analysis, for quality control analysis of sex.

---

<!-- ------------------ SECTION ------------------ -->
## Approach 1: sex check analysis with Somalier
A first approach, performed by means of the `Somalier` v0.2.15. tool ([Pedersen et al. 2019](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-020-00761-2)), let us to infer the sex of the sample from the depth of the X and Y chromosome reads. This tool uses a total of 17,766 positions in coding regions to be able to work with data from different type of experiments (whole-exome and -genome sequencing, RNA-Seq, etc.). These positions meet several requisites such as: (i) frequently sequenced with high quality, (ii) population allele frequency around 0.5, and (iii) exclusion of segmental duplications regions, low complexity regions and regions nearby insertion and deletions. Relatedness among samples is calculated by allelic concordance from single nucleotide variants within these positions (classified as homozygous, heterozygous, and alternative homozygous).

`Somalier` uses a genomic VCF file (gVCF) to extract variant and non-variant information from these positions. With `somalier extract` command we extract position data to a binary file. In a second step, `somalier relate` calculate and create an [HTML file](https://brentp.github.io/somalier/ex.html) for results visualization. An example code is shown below:

```
# Path to Somalier binary
SOMALIER="/path/to/somalier_bin"

# Path to Sites file
sites="/path/to/sites.hg19.vcf.gz"

# Path to reference genome
ref="/path/to/ucsc.hg19.fasta"

# Path to your input VCF file
infile=”/path/to/VCF_file”

# Run these commands
${SOMALIER} extract -d ${outdir} --sites ${sites} -f ${ref} ${infile}
${SOMALIER} relate -o ${outname} ./*.somalier
```

[Visit the repository of `Somalier` at GitHub](https://github.com/brentp/somalier)

---

<!-- ------------------ SECTION ------------------ -->
## Approach 2: sex check analysis with a Heuristic
A second approach used an in-house heuristic algorithm coded in BASH. The script let us to analyze the coverage or vertical-depth of 11 selected genes in the non-pseudoautosomal regions of the X and Y chromosomes ([Table 1](table-1-list-of-genes-assessed-in-sex-classification-in-both-x-and-y-chromosomes)). We then assess the depth distribution accross chromosomes X and Y in order to identify high covered genes suitable for sex classification based on read depth. In brief, we identify and extract selected gene regions from BAM files. Then the number of total reads per-chromosome and per-gene are obtained. Reads with a mapping quality lower than 50 (MQ<50) are filtered out for the analysis. Once we calculate the read counts, we compare the number of reads between both chromosomes. The fraction of reads in chromosome X compared to that of chromosome Y is calculated, and viceversa ([Equation 1](equation-1-fraction-of-reads-comparing-both-chromosomes)).

<div align="center">
  <p></p>
<b>Table 1</b>. List of genes assessed in sex classification in both X and Y chromosomes.
  <p></p>
<table>
  <thead>
    <tr>
      <th align="center">Chromosomes</th>
      <th align="center" colspan=3>Genes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center" rowspan=4>ChrX</td>
      <td align="center"><i>RAB39B</i></td>
      <td align="center"><i>ACTRT1</i></td>
      <td align="center"><i>SSX1</i></td>
    </tr>
    <tr>
      <td align="center"><i>F8</i></td>
      <td align="center"><i>UBE2E4P</i></td>
      <td align="center"><i>SSX9P</i></td>
    </tr>
    <tr>
      <td align="center"><i>CMC4</i></td>
      <td align="center"><i>FAM47B</i></td>
      <td align="center"><i>SSX3</i></td>
    </tr>
    <tr>
      <td align="center"><i>TEX13A</i></td>
      <td align="center"><i>PPP1R2P9</i></td>
      <td align="center"></td>
    </tr>
    <tr>
      <td align="center" rowspan=4>ChrY</td>
      <td align="center"><i>PRORY</i></td>
      <td align="center"><i>TBL1Y</i></td>
      <td><i>EIF1AY</i></td>
    </tr>
    <tr>
      <td align="center"><i>KDM5Dv</td>
      <td align="center"><i>FAM41AY2</i></td>
      <td align="center"><i>RPS4Y2</i></td>
    </tr>
    <tr>
      <td align="center"><i>AMELY</i></td>
      <td align="center"><i>XKRY</i></td>
      <td align="center"><i>DAZ4</i></td>
    </tr>
    <tr>
      <td align="center"><i>TSPY2</i></td>
      <td align="center"><i>TXLNGY</i></td>
      <td align="center"></td>
    </tr>
  </tbody>
</table>
</div>

<br />

<div align="center">

<b>Equation 1</b>. Fraction of reads per chromosome.
  <p></p>
  
$$f_X = {Nreads_{chrX} \over Nreads_{chrX} + Nreads_{chrY}}$$

$$f_Y = {Nreads_{chrY} \over Nreads_{chrY} + Nreads_{chrX}}$$

</div>

<br />

Female samples will have nearly all reads mapped to chromosome X and none in chromosome Y. 

We have observed in our testing and validation datasets that the fraction of reads in chromosome X, <i>f<sub>X</sub></i>, is within the interval [0.988, 1], and the fraction of reads mapped to chromosome Y, <i>f<sub>Y</sub></i>, ranges in the interval [0, 0.012]. Conversely, male samples will have mapped reads splitted between genes on the X and Y chromosomes. In fact, we have observed a fraction of mapped reads in chromosome X in the interval [0.176, 0.593] and a fraction of reads mapped to chromosome Y in the interval [0.407, 0.824]. Male samples use to have a higher dispersion than females because of the observed variation in read depth between X and Y chromosomes.

Finally, we create a scatter plot representing the fraction of reads in X-chromosome in the x-axis, and the fraction of reads in Y-chromosome in the y-axis for each sample ([Figure 1](figure-1-sex-classification-of-samples-based-on-heuristic-analysis)). As can be seen, the figure shows three different clusters: one for female samples, one for male samples, and a third cluster representing samples with uncertain sex assignation. The reasons for an unassigned sex could be related to a contamination of another sample with the opposite sex, sample swapping, or an error in the sample name.

##### Figure 1. Sex classification of multiple samples based on heuristic analysis.

![](https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/classification_table-heuristic_analysis.png)
