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

Create an environment for:
- qiime
- python

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

```rmarkdown
mkdir illumina_reads_control
cat *.R1.fastq *.R2.fastq > illumina_reads_control/
```

Then, check the quality of all reads with FASTQC

```rmarkdown
fastqc --extract ~/patht/illumina_reads_control/
```

Rename all `.fastq` files to be the same. For example replace all "_" and "-" by "."

```rmarkdown
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
Merge overlapping paired end reads into longer reads.

`in1=` reads 1 files, `in2=`reads 2 files and `out=`final merged file.

Filter fastq sequences based on number of expected error `-fastq_maxee`

See https://jgi.doe.gov/data-and-tools/bbtools/bb-tools-user-guide/bbmerge-guide/

```rmarkdown
while read sample;
do
 bbmerge.sh in1=$sample.R1.fastq in2=$sample.R2.fastq out=$sample.merged.fq
 vsearch --fastq_filter $sample.merged.fq -fastaout $sample.filtered.fa -fastq_maxee 0.5;
done
```

## Step 3: Dereplication
First concatenate all files `*.filtered.fa` to a signle file

```rmarkdown
cat *.filtered.fa > all_sequences.filtered.fa
```

Select representative sequence of several identical sequences and keep only one sequence (supress the others).

`-minseqlength` minimum size of sequence to keep

```rmarkdown
vsearch -derep_fulllength all_sequences.filtered.fa -output all_sequences.filtered.uniques.fa -sizeout -minseqlength 250
```

## Step 4: Size-sorting
`-minsize` minimum abundance of a sequence for sortbysize (suppress singleton)

```rmarkdown
vsearch -sortbysize all_sequences.filtered.uniques.fa -output all_sequences.filtered.uniques.sort.fa -minsize 2
```

## Step 5: Chemira checking
`-minsize` specifies the minimum abundance. The default is 8, but for higher sensitivity, reducing `-minsize` to 4 is acceptable.

All noisy sequences (with low-abundance) are mapped to a ZOTU by `-zotus` option.

```rmarkdown
usearch -unoise3 all_sequences.filtered.uniques.sort.fa -zotus *all_sequences.otus100.fa -minsize 4
```

Then, sort sequences by length

```rmarkdown
usearch -sortbylength all_sequences.otus100.fa -fastaout all_sequences.otus100.sorted.fa
```

## Step 6: OTUs clustering
For Eukaryota is better to use 98% level for clustering with`-id`option. (97% for Bacteria and Archaea).

```rmarkdown
usearch -cluster_smallmem all_sequences.otus100.sorted.fa -id 0.98 -centroids all_sequences.otu.fasta -relabel OTU_
```

## Step 7: Taxonomic affiliation 
Download silva v.132 database to assign taxonomy to OTU clustering. 

Other database can ba use (e.g. [PR2 db](https://github.com/pr2database/pr2database)).

`cutoff=`by default is 80.

`probs=`is set to false for not having bootstrap values next to taxonomy.

```rmarkdown
mothur > classify.seqs(fasta=all_sequences.otu.fasta, reference=/path_to_silva_directory/silva.v.132.fna, taxonomy=path_to_silva_directory/silva.v.132.tax, cutoff=75, processors=24, probs=F)
```

## Step 8: OTUs mapping
`-usearch_gloabl` file name of queries for global alignment search.

`-strand plus` cluster using plus strands.

`-uc` file name for [UCLUST](https://drive5.com/usearch/manual/uclust_algo.html)-like output.

`-maxhits` maximum number of hits to show.

`-maxaccepts`number of hits to accept and show per strand.

```rmarkdown
grep -c ">" all_sequences.otu.fasta # gives a number of hit to accept (e.g. 20)
SAMPLE_FILES=$(ls *.filtered.fa) # all merged files from step 2

for SAMPLE in $SAMPLE_FILES;
do
 vsearch -usearch_global $SAMPLE -db all_sequences.otu.fasta -strand plus -id 0.98 -uc $SAMPLE.uc -maxhits 1 -maxaccepts 20;
done
```

## Step 9: OTUs table construction
First, concatenate all `.uc` file to a single file

```rmarkdown
cat *.uc > all_sequences.uc
```

Download python script [uc2otutab.py](https://drive5.com/python/uc2otutab_py.html) to transform `.uc`to otu table 

```rmarkdown
python uc2otutab.py all_sequences.uc > all_sequences.otumap
```

Activate qiime environment to use qiime implented script.

Convert `.otumap` file to a `.biom` file.

Add atxanomy affiliation to each sequence (taxonomy file is an output file from MOTHUR).

Convert `.biom` file to a `.txt` file, so it will be easier to read on excel or else.

```rmarkdown
source ~/qiime_env/bin/activateeaser

biom convert --table-type="OTU table" -i all_sequences.otumap -o OTU_table_all_seq.biom --to-json

biom add-metadata --sc-separated Taxonomy --observation-header OTU_ID,Taxonomy --observation-metadata-fp *.taxonomy -i OTU_table_all_seq.biom -o OTU_table_all_seq_tax.biom

biom convert -i OTU_table_all_seq_tax.biom -o OTU_table_all_seq_tax.txt --to-tsv --header-key Taxonomy --table-type "OTU table"
```

## Further analysis can be done to the OTU table

In QIIME environment, we can:
- filter OTU table from bigger organism like fungi and metazoan.

```rmarkdown
filter_taxa_from_otu_table.py -i OTU_table_all_seq_tax.biom -o OTU_table_all_seq_tax_filter.biom -n Metazoa
filter_taxa_from_otu_table.py -i OTU_table_all_seq_tax_filter.biom -o OTU_table_all_seq_tax_filter2.biom -n Fungi
```

- summarize OTU table to obtain the total number of sequence per sample. This is usueful if we want to rarefy the OTU table.

```rmarkdown
biom summarize-table -i OTU_table_all_seq_tax_filter2.biom > summarize.OTU_table_all_seq_tax_filter2.txt # check for the lowest abundance number (e.g. 15555)
```

- rarefaction to the lowest abundance in sample. 

```rmarkdown
single_rarefaction.py -i OTU_table_all_seq_tax_filter2.biom -o OTU_table_all_seq_tax_filter_rarefied.biom -d 15555
```

- summarize taxa to obtain relative abondance for each OTU at each taxonomy level (Kingdomm, class, order, phyllum, gender, species).

```rmarkdown
summarize_taxa.py -i OTU_table_all_seq_tax_filter_rarefied.biom -o ./Taxonomy

```

- alpha diversity of rarefied and filtered OTU table

```rmarkdown
multiple_rarefactions_even_depth.py -i OTU_table_all_seq_tax_filter_rarefied.biom -o rarefied_otu_tables_alpha_div/ -d 1000 -n 100

alpha_diversity.py -i rarefied_otu_tables_alpha_div/ -m observed_species,simpson,shannon -o alpha_div_1000
```

- beta diversity of rarefied and filtered OTU table

```rmarkdown
beta_diversity.py -i OTU_table_all_seq_tax_filter_rarefied.biom -m bray_curtis -o BC_beta_div_matrix_filter_rarefied/ #.txt file will be created
```

- [UPGMA](https://link.springer.com/referenceworkentry/10.1007/978-1-4020-6754-9_17806) cluster tree from rarefied and filtered OTU table

```rmarkdown
upgma_cluster.py -i BC_beta_div_matrix_filter_rarefied/BC_beta_div_matrix_filter_rarefied.txt -o BC_beta_div_matrix_filter_rarefied.tre

```

- Number of shared OTU between samples (a Venn diagram can be done with the output file).

```rmarkdown
shared_phylotypes.py -i OTU_table_all_seq_tax_filter_rarefied.biom -o shared_otus.txt

```

- Ordination: PCA (principal components analysis)

```rmarkdown
principal_coordinates.py -i BC_beta_div_matrix_filter_rarefied/BC_beta_div_matrix_filter_rarefied.txt -o BC_beta_div_matrix_filter_rarefied/beta_div_coordinates.txt
```

- Heatmap

```rmarkdown
make_otu_heatmap.py -i OTU_table_all_seq_tax_filter_rarefied.biom -o OTU_table_all_seq_tax_filter_rarefied.heatmap.pdf

```
