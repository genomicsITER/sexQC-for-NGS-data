<!-- ------------------ HEADER ------------------ -->
<!-- Developed and maintained by Genomics Division
<!-- of the Institute of Technology an Renewable Energy (ITER)
<!-- Tenerife, Canary Islands, SPAIN
<!-- See the "Contact us" section to collaborate with us to growth
<!-- this repository. ;=)

<!-- ------------------ SECTION ------------------ -->
<a name="sex-check-qc-code"></a>

<p align="left">
  <a href="https://www.iter.es" title="Instituto Tecnológico y de Energ&iacute;as Renovables (ITER) / Institute of Technology and Renewable Energy (ITER)">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/ITER_logo.png" width="30%" /> 
  </a>
</p>

<a name="code"></a>
#### Detailed code for the Heuristic sex inference quality control
```Bash

#!/bin/bash

## Prepare a BASH script as this and save it to "sex-check-inference-QC.sh"
# Launch as follows in a terminal window: sh /path/to/script/sex-check-inference-QC.sh

## If you use a HPC environment:
## > Uncomment next line to source modules in the actual user profile
#source /etc/profile.d/profile.modules.sh
## > Uncomment this line to load the required modules at HPC #
#module load gcc/10.2.0 xz/5.2.5/gcc htslib/1.12/gcc samtools/1.12/gcc


## Define dirs and files
## Make sure all paths exist
inpath="/path/to/sex-check/classifier"

## List of BAM files with selected gene regions in chromosome X
bam_samples_X="${inpath}/data/BAM_list_X"

## List of BAM files with selected gene regions in chromosome Y
bam_samples_Y="${inpath}/data/BAM_list_Y"

## Outfile name
count_table="${inpath}/data/reads_table.tsv"

## Set minimum MappingQuality value for reads
MQ=50


## Define genes and their genomic regions
##Chromosome X
genes_X=( "gene1" "gene2" ... )
coordinates_X=( "chrX:..-.." "chrX:..-.." ...)

##Chromosome Y
genes_Y=( "gene1" "gene2" ... )
coordinates_Y=( "chrY:..-.." "chrY:..-.." ...)

## chrX
echo ""
echo "###################################"
echo "          Processing chrX          "
echo "###################################"

##Iterate over the BAM files contained in the corresponding list
while IFS= read -r file
do
sample=$( basename ${file} | cut -d"." -f1 )
chr=$( basename ${file} | cut -d"." -f4 )
echo ${file}
echo ${sample}
echo ${chr}

for ((i=0;i<${#genes_X[@]};++i)); do
	gene=${genes_X[i]}
	region=${coordinates_X[i]}
	echo ${gene}
	outfile="${inpath}/data/extracted_regions_bam/${sample}.ready.deduped.${chr}.noPAR.${gene}.bam"
	samtools view -b -h ${file} ${region} > ${outfile}
	count=$( samtools view -q${MQ} ${outfile} | wc -l )
	echo -e "${sample}\t${chr}\t${gene}\t${count}" >> ${count_table}
	done
  
	echo ""
done < "${bam_samples_X}"


# chrY
echo ""
echo "###################################"
echo "          Processing chrY          "
echo "###################################"

##Iterate over the BAM files contained in the corresponding list
while IFS= read -r file
do
sample=$( basename ${file} | cut -d"." -f1 )
chr=$( basename ${file} | cut -d"." -f4 )
echo ${file}
echo ${sample}

for ((i=0;i<${#genes_Y[@]};++i)); do
	gene=${genes_Y[i]}
	region=${coordinates_Y[i]}
	echo ${gene}

	outfile="${inpath}/data/extracted_regions_bam/${sample}.ready.deduped.${chr}.noPAR.${gene}.bam"
	samtools view -b -h ${file} ${region} > ${outfile}
	count=$( samtools view -q${MQ} ${outfile} | wc -l )
	echo -e "${sample}\t${chr}\t${gene}\t${count}" >> ${count_table}
done

	echo ""
done < "${bam_samples_Y}"

## End of script
```

Finally, open the 'reads_table.tsv' file and plot the fractions <i>f<sub>Y</sub></i> <i>vs</i> <i>f<sub>X</sub></i> with your favorite data analysis program.

<p align="right">
  <a href="#sex-check-qc-code" title="Up">
    <img src="https://github.com/genomicsITER/sexQC-for-NGS-data/blob/main/images/home-icon.png" style="float: right; margin: 10px; padding: 2px;" />
  </a>
</p>
