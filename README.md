# Illumina Miseq amplicon analysis (18S rRNA gene)
This respository contains the pipeline code used to process raw amplicon sequences. 
It contains also several steps to analyse the final OTUs table.

## Pre-requirement and installation
Module load:
            fastqc
            vsearch
            bbmap
            mothur
            usearch

 Download scripts: uc2otutab.py
 
 Download eukaryota taxonomy database
 ref: 
 tax: 


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






### Jekyll Themes
Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/nastasiafd/SaveTheArcticPhytoplankton/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```
