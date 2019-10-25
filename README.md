# Illumina Miseq amplicon analysis (18S rRNA gene)
This respository contains the pipeline code used to process raw amplicon sequences. 
It contains also several steps to analyse the final OTUs table.

## Pre-requirement and installation
Module load:
- fastqc
- vsearch
- bbmap
- mothur
- usearch

 Download scripts: [uc2otutab.py](https://drive5.com/python/uc2otutab_py.html)
 
 Download eukaryota taxonomy database: [Silva v.132](https://www.arb-silva.de/no_cache/download/archive/release_132/Exports/)


## Pipeline steps
1. Quality reads of `.fastq` files [(FASTQC)](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
2. Paired-end merging [(BBTOOLS)](https://jgi.doe.gov/data-and-tools/bbtools/)
3. Dereplication [(VSEARCH)](https://github.com/torognes/vsearch)
4. Size-sorting [(VSEARCH)](https://github.com/torognes/vsearch)
5. Chimera checking [(USEARCH)](http://www.drive5.com/usearch/)
6. OTUs clustering [(USEARCH)](http://www.drive5.com/usearch/)
7. Taxonomic affiliation [(MOTHUR)](https://www.mothur.org/) with [Silva database](https://www.arb-silva.de/)
8. OTUs mapping [(VSEARCH)](https://github.com/torognes/vsearch)
9. OTUs table construction [(QIIME)](http://qiime.org/)

## Step 1: Quality control of raw reads from Miseq
First, create a directory where all the `.R1.fastq`and `.R2.fastq` files will be together
```markdown
mkdir illumina_reads_control
cat *.R1.fastq *.R2.fastq > illumina_reads_control/
```
Then, check the quality of all reads with FASTQC
```rmarkdown
fastqc --extract ~/patht/illumina_reads_control/
```
Rename all `.fastq` files to be the same. For example replace all "_" and "-" by "."
```markdown
mkdir Fastq_processing
mv *.R1.fastq *.R2.fastq Fastq_processing/
for i in *.fastq; 
  do mv $i $(echo $i | sed 's/\_/\./g'); 
done

for i in *.fastq; 
  do mv $i $(echo $i | sed 's/\-/\./g'); 
done
```

## Step 2: Paired-end merging
Merge overlapping paired end reads into longer reads `in1=` reads 1 files, `in2=`reads 2 files and `out=`final merged file
Filter fastq sequences based on number of expected error `-fastq_maxee`
See https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbmerge-guide/
```rmarkdown
name=
while read sample;
 do
  bbmerge.sh in1=$sample.R1.fastq in2=$sample.R2.fastq out=$sample.merged.fq
  vsearch --fastq_filter $sample.merged.fq -fastaout $sample.filtered.fa -fastq_maxee 0.5
done
```

## Step 3: Dereplication
Select representative sequence of several identical sequences and keep only one sequence (supress the others).
`-minseqlength` minimum size of sequence to keep
```rmarkdown
vsearch -derep_fulllength $name.filtered.fa -output $name.filtered.uniques.fa -sizeout -threads 8 -minseqlength 250
```

## Step 4: Size-sorting

```rmarkdown
vsearch -sortbysize $name.filtered.trim.label.uniques.fasta -output $name.filtered.trim.label.uniques.sort.fasta -minsize 2
````

## Step 5: Chemira checking

## Step 6: OTUs clustering

## Step 7: Taxonomic affiliation 

## Step 8: OTUs mapping

## Step 9: OTUs table construction




### Jekyll Themes
Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/nastasiafd/SaveTheArcticPhytoplankton/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

```markdown
**Bold** and _Italic_ and `Code` text
[Link](url) and ![Image](src)
```
