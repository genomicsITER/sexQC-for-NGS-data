# Sex Quality Control for NGS data
Identify the sex of samples in a sequencing experiment is an important quality control step for the discovering of errors both as metadata information and sample trazability. Here, we explain how to use the self-reported sex of the individual and two bioinformatic approaches for sex quality control analysis.

---

## Somalier analysis
The first approach, performed by the `somalier` v0.2.15. tool ([Pedersen et al. 2019](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-020-00761-2)), identifies the sex of the sample from the depth of the X and Y chromosome reads. This software utilizes a total of 17,766 positions in coding regions to be able to work with different types of experiments (whole-exome and -genome sequencing, RNA-Seq, etc.). These positions accomplish several requisites such as, a high quality, population allele frequency around 0.5 and, exclusion of segmental duplications regions, low complexity regions, and regions nearby insertion and deletions. Relatedness is calculated by allelic concordance from single nucleotide variants within these positions (homozygous, heterozygous, and alternative homozygous).

The software uses a genomic VCF file (gVCF) to extract variant and non-variant information in these positions. `somalier extract` extracts position data to a binary file. And then, `somalier relate` calculate and create an [HTML file](https://brentp.github.io/somalier/ex.html) for results visualization. An example code is shown below:

```
# Somalier binary
SOMALIER="/path/to/somalier_bin"
# Sites file
sites="/path/to/sites.hg19.vcf.gz"
# Reference
ref="/path/to/ucsc.hg19.fasta"
# Input VCF file
infile=”/path/to/VCF_file”

$SOMALIER extract -d ${outdir} --sites ${sites} -f ${ref} ${infile}
$SOMALIER relate -o ${outname} ./*.somalier
```

---

## Heuristic analysis
For the second approach, an in-house heuristic script is used analyzying the depth of 11 genes in the non-pseudoautosomal regions of the X and Y chromosomes ([Table 1](table-1-list-of-genes-assessed-in-sex-classification-in-both-chromosomes-X-and-Y)). We assess depth distribution along both chromosomes X and Y to set high covered genes by capture sequencing suitable for sex classification based on read depth. In brief, we extract selected gene regions from BAM files, and then, reads on these genes are counted firstly by gene, and later, the total number of reads per chromosome. Reads with a mapping quality below 50 (MappingQuality>50) are filtered out for the analysis. Once we calculate the read count, we compare the number of reads between both chromosomes. The fraction of reads in chromosome X compared to chromosome Y is calculated, and vice versa ([Equation 1](equation-1-fraction-of-reads-comparing-both-chromosomes)).

##### Table 1. List of genes assessed in sex classification in both chromosomes X and Y.
<table>
  <thead>
    <tr>
      <th>Chromosomes</th>
      <th colspan=3>Genes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan=4>ChrX</td>
      <td>RAB39B</td>
      <td>ACTRT1</td>
      <td>SSX1</td>
    </tr>
    <tr>
      <td>F8</td>
      <td>UBE2E4P</td>
      <td>SSX9P</td>
    </tr>
    <tr>
      <td>CMC4</td>
      <td>FAM47B</td>
      <td>SSX3</td>
    </tr>
    <tr>
      <td>TEX13A</td>
      <td>PPP1R2P9</td>
      <td></td>
    </tr>
    <tr>
      <td rowspan=4>ChrY</td>
      <td>PRORY</td>
      <td>TBL1Y</td>
      <td>EIF1AY</td>
    </tr>
    <tr>
      <td>KDM5D</td>
      <td>FAM41AY2</td>
      <td>RPS4Y2</td>
    </tr>
    <tr>
      <td>AMELY</td>
      <td>XKRY</td>
      <td>DAZ4</td>
    </tr>
    <tr>
      <td>TSPY2</td>
      <td>TXLNGY</td>
      <td></td>
    </tr>
  </tbody>
</table>


##### Equation 1. Fraction of reads comparing both chromosomes.
$$f_X = {Nreads_{chrX} \over Nreads_{chrX} + Nreads_{chrY}}$$

$$f_Y = {Nreads_{chrY} \over Nreads_{chrY} + Nreads_{chrX}}$$

Female samples will have nearly all reads in chromosome X and no one in chromosome Y. We have observed in our dataset that the fraction of reads in chromosome X is within the interval 0.988-1 and the fraction of reads in chromosome Y within the interval 0-0.012. Conversely, male samples will have reads divided between chromosome X and Y genes. In reality, we have observed a fraction of reads in chromosome X within the interval 0.176-0.593 and a fraction of reads in chromosome Y within the interval 0.407-0.824. Male samples have a higher dispersion than females because of a variation in read depth between chromosomes X and Y.

Finally, we create a scatter plot representing the fraction of reads in chromosome X in the x-axis, and the fraction of reads in chromosome Y in the y-axis for each sample ([Figure 1](figure-1-sex-classification-of-samples-based-on-heuristic-analysis)). As can be seen, figure shows three different clusters: one for female samples, one for male samples, and a third cluster representing wrong sex assignation samples. Reasons for a not-assigned sample could be related to a contamination of another sample with the opposite sex, sample swapping, or an error in the sample name.

##### Figure 1. Sex classification of multiple samples based on heuristic analysis.

![](https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/classification_table-heuristic_analysis.png)
