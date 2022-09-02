<!-- ------------------ SECTION ------------------ -->
<a name="sex-check-qc"></a>

<p align="left">
  <a href="https://www.iter.es" title="Instituto Tecnológico y de Energ&iacute;as Renovables (ITER) / Institute of Technology and Renewable Energy (ITER)">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/ITER_logo.png" width="30%" /> 
  </a>
</p>

<!-- ------------------ SECTION ------------------ -->
## Sex Inference as a Quality Control for NGS data ##

The inference of the genetic sex of a sample from its sequence obtained in a Next Generation Sequencing (NGS) experiment is a mandatory quality control step for the discovery of errors in the metadata provided and to contribute to sample traceability. Here, we explain how to use the self-reported sex of an individual and two different bioinformatic approaches, `Somalier` and `Heuristic` based analysis, to infere the genetic sex of a sample and perform a quality control analysis.

---

<!-- ------------------ SECTION ------------------ -->
## Approach 1: sex inference with Somalier
A first approach, performed by means of the `Somalier` v0.2.15. tool ([Pedersen et al. 2019](https://genomemedicine.biomedcentral.com/articles/10.1186/s13073-020-00761-2)), let us to infer the sex of the sample from the depth of the X and Y chromosome reads at some selected positions in the genome. This tool uses a total of 17,766 positions in coding regions to be able to work with data from different type of experiments (whole-exome and -genome sequencing, RNA-Seq, etc.). These positions meet several requisites such as: (i) frequently sequenced with high quality, (ii) population allele frequency around 0.5, and (iii) exclusion of segmental duplications regions, low complexity regions and regions nearby insertion and deletions. Relatedness among samples is calculated by allelic concordance from single nucleotide variants within these positions (classified as homozygous, heterozygous, and alternative homozygous).

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
## Approach 2: sex inference with a Heuristic
A second approach uses an in-house heuristic algorithm coded in BASH. The script let us to analyze the coverage or vertical-depth of 11 selected genes located in the non-pseudoautosomal regions (NPAR) of the X and Y chromosomes (Table 1). We then assess the depth distribution accross chromosomes X and Y in order to identify high covered genes suitable for sex classification based on read depth.

Briefly, the heuristic follows this algorithm:
<ol>
  <li>Compute the coverage distribution in NPAR of chromosomes X and Y.</li>
  <li>Identify genes with high coverage.</li>
  <li>Extract selected gene regions from the corresponding BAM files.</li>
  <li>Filter reads with a mapping quality lower than 50 (MQ<50).</li>
  <li>Obtain the number of reads per-chromosome.</li>
  <li>Obtain the number of reads per-gene.</li>
  <li>Compute the fraction of reads in chromosomes X and Y according to [Equation 1].</li>
  <li>Plot results.</li>
</ol>

<br />

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
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=RAB39B">RAB39B</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=ACTRT1">ACTRT1</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=SSX1">SSX1</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=F8">F8</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=UBE2E4P">UBE2E4P</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=SSX9P">SSX9P</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=CMC4">CMC4</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=FAM47B">FAM47B</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=SSX3">SSX3</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=TEX13A">TEX13A</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=PPP1R2P9">PPP1R2P9</a></i></td>
      <td align="center"></td>
    </tr>
    <tr>
      <td align="center" rowspan=4>ChrY</td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=PRORY">PRORY</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=TBL1Y">TBL1Y</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=EIF1AY">EIF1AY</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=KDM5D">KDM5D</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=FAM41AY2">FAM41AY2</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=RPS4Y2">RPS4Y2</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=AMELY">AMELY</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=XKRY">XKRY</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=DAZ4">DAZ4</a></i></td>
    </tr>
    <tr>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=TSPY2">TSPY2</a></i></td>
      <td align="center"><i><a href="https://www.genecards.org/cgi-bin/carddisp.pl?gene=TXLNGY">TXLNGY</a></i></td>
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

Theoretically, female samples will have all reads mapped to chromosome X and none in chromosome Y. Accordingly, <i>f<sub>X</sub></i>=1 and <i>f<sub>Y</sub></i>=0. In practice, female samples show nearly all reads mapped to chromosome X and nearly none in chromosome Y, thus resulting in fractions with values as <i>f<sub>X</sub></i>&#8771;1 and <i>f<sub>Y</sub></i>&#8771;0. 

In our testing and validation datasets we have observed that the fraction of reads in chromosome X, <i>f<sub>X</sub></i>, shows values within the interval [0.988, 1], and the fraction of reads mapped to chromosome Y, <i>f<sub>Y</sub></i>, ranges in the interval [0, 0.012].

Male samples will have mapped reads splitted between genes on the X and Y chromosomes. As a result, we have observed a fraction of mapped reads in chromosome X in the interval [0.176, 0.593] and the fraction of reads mapped to chromosome Y ranges in the interval [0.407, 0.824].

Male samples use to have a higher dispersion than females in the calculated fractions as a consequence of the observed variation in read depth between X and Y chromosomes.

A scatter plot representing the fraction of reads in X-chromosome in the x-axis, and the fraction of reads in Y-chromosome in the y-axis for each sample is shown (Figure 1). This plots shows three different clusters: one for female samples, one for male samples, and a third cluster representing samples with uncertain sex assignation. Uncertainties in sex inference may arise as a consequence of sample contamination (i.e. contamination of a sample with a different sex), sample swapping, error in the sample labeling, etc.


<div align="center">

<p align="left">
  <a href="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/classification_figure-heuristic_analysis.png" title="Figure 1">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/classification_figure-heuristic_analysis.png" width="auto" /> 
  </a>
</p>

<b>Figure 1</b>. Sex inference of multiple samples based on our heuristic analysis.

</div>

<p align="right">
  <a href="#sex-check-qc" title="Up">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/home-icon.png" style="float: right; margin: 10px; padding: 2px;" />
  </a>
</p>
