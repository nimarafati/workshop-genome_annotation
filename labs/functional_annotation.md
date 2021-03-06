---
layout: default-overview
title: Functional annotation
exercises: 1h30
questions:
  - What is needed for functional annotation?
  - How to do functional annotation?
objectives:
  - understand the different tools/processes to do a functional annotation
  - be able to run a functional annotation
---

# Prerequisites
For this exercise you need to be logged in to Uppmax.

Setup the folder structure:

```bash
conda activate bioinfo
export data=/home/data/data_annotation/
export functional_annotation_path=/home/$USER/functional_annotation
export structural_annotation_path=/home/$USER/structural_annotation
mkdir -p $functional_annotation_path
chmod +w $data/blastdb/uniprot_dmel/
```

## Introduction

Functional annotation is the process during which we try to put names to faces - what do genes that we have annotated and curated? Basically all existing approaches accomplish this by means of similarity. If a translation product has strong similarity to a protein that has previously been assigned a function, the function in this newly annotated transcript is probably the same. Of course, this thinking is a bit problematic (where do other functional annotations come from...?) and the method will break down the more distant a newly annotated genome is to existing reference data. A complementary strategy is to scan for more limited similarity - specifically to look for the motifs of functionally characterized protein domains. It doesn't directly tell you what the protein is doing exactly, but it can provide some first indication.

In this exercise we will use an approach that combines the search for full-sequence similarity by means of 'Blast' against large public databases with more targeted characterization of functional elements through the InterproScan pipeline. Interproscan is a meta-search engine that can compare protein queries against numerous databases. The output from Blast and Interproscan can then be used to add some information to our annotation.

## Prepare the input data

Since we do not wish to spend too much time on this, we will again limit our analysis to chromosome 4. It is also probably best to choose the analysis with ab-initio predictions enabled (unless you found the other build to be more convincing). Maker produces a protein fasta file (called "annotations.proteins.fa") together with the annotation and this file should be located in your maker directory.

Move in the proper folder:  
```
cd $functional_annotation_path
```
Now link the annotation you choose to work with. The command will looks like:
```
ln -s $structural_annotation_path/maker/complement/maker_abinitio_cplt_by_evidence.gff maker_final.gff  
ln -s $structural_annotation_path/maker/complement/maker_abinitio_cplt_by_evidence.fa maker_final.faa
ln -s $data/genome/genome.fa
```

## Interproscan approach
 Interproscan combines a number of searches for conserved motifs and curated data sets of protein clusters etc. This step may take fairly long time. It is recommended to paralellize it for huge amount of data by doing analysis of chunks of tens or hundreds proteins.

### Perform [InterproScan](https://github.com/ebi-pf-team/interproscan/wiki) analysis
InterproScan can be run through a website or from the command line on a linux server. Here we are interested in the command line approach.
<u>Interproscan allows to look up pathways, families, domains, sites, repeats, structural domains and other sequence features.</u>  

Launch Interproscan with the option -h if you want have a look about all the parameters.

<br> - The '-app' option allows defining the database used. Here we will use the PfamA,ProDom and SuperFamily databases.
<br> - Interproscan uses an internal database that related entries in public databases to established GO terms. By running the '-goterms' option, we can add this information to our data set.
<br> - If you enable the InterPro lookup ('-iprlookup'), you can also get the InterPro identifier corresponding to each motif retrieved: for example, the same motif is known as PF01623 in Pfam and as IPR002568 in InterPro.
<br> - The option '-pa' provides mappings from matches to pathway information (MetaCyc,UniPathway,KEGG,Reactome).
```
interproscan.sh -i maker_final.faa -t p -dp -pa -appl Pfam,ProDom-2006.1,SuperFamily-1.75 --goterms --iprlookup
```
This analysis will fail.  

:question:Why? What is the error message displaying?  

If you did not have a look at the maker_final.faa, please have a look and find a solution to make interproscan run.  

:bulb:***TIP*** : ```agat_sp_extract_sequences.pl  --help```  

<details>
<summary> :key: **Interproscan problem** - Click to expand the solution </summary>

Interproscan is really selective on the fasta input data, there should not be any stop codon * or any character other than ATCG (except in the header of course)  

You need to rerun the first script with the parameters --cfs and --cis:  
<br>
<code>
conda activate agat
agat_sp_extract_sequences.pl -g maker_abinitio_cplt_by_evidence.gff -f genome.fa -p -o maker_final_fixed.faa --cfs --cis
</code>  
<br>
or you can do  
<code>sed -e 's/*//g' maker_final.faa > maker_final_fixed.faa</code>
</details>

Rerun the previous interproscan command.

The analysis should take 2-3 secs per protein request - depending on how many sequences you have submitted, you can make a fairly deducted guess regarding the running time.  
You will obtain 3 result files with the following extension '.gff3', '.tsv' and '.xml'. Explanation of these output are available [>>here<<](https://github.com/ebi-pf-team/interproscan/wiki/OutputFormats).


### load the retrieved functional information in your annotation file:
Next, you could write scripts of your own to merge interproscan output into your annotation. Incidentally, Maker comes with utility scripts that can take InterProscan output and add it to a Maker annotation file (you need to load maker).  

<br> - ipr\_update\_gff: adds searchable tags to the gene and mRNA features in the GFF3 files.
<br> - iprscan2gff3: adds physical viewable features for domains that can be displayed in JBrowse, Gbrowse, and Web Apollo.

We also created a script that can do the merging between the structural annotation and the interpro results :

```
agat_sp_manage_functional_annotation.pl --gff maker_final.gff -i maker_final.faa.tsv -o  maker_final.interpro
```
Where a match is found, the new file will now include features called Dbxref and/or Ontology_term in the gene and transcript feature field (9th column).
The improved annotation is the gff file inside the maker_final.interpro folder.

## BLAST approach
Blast searches provide an indication about potential homology to known proteins. This approach can be used to give a name to the genes and a function to the transcripts. A 'full' Blast analysis can take several days and consume several GB of Ram. Consequently, for big data‚ it is recommended to parallelize this step by doing in chunks of tens or hundreds proteins. 

### Perform Blast searches from the command line on Uppmax:

To run Blast on your data, use the Ncbi Blast+ package against a Drosophila-specific database (included in the folder we have provided for you, under **$data/blastdb/uniprot\_dmel/uniprot\_dmel.fa**) - of course, any other NCBI database would also work:
```
conda activate blast
cd $functional_annotation_path
blastp -db $data/blastdb/uniprot_dmel/uniprot_dmel.fa -query maker_final.faa -outfmt 6 -out blast.out -num_threads 8
```
if you received the following error:  
_BLAST Database error: No alias or index file found for protein database [/blastdb/uniprot_dmel/uniprot_dmel.fa] in search path [/home/nima_rafati/functional_annotation::]_

then create the database again, it is due to new formating implemented in blast.  
```
cd $data/blastb/uniprot_dmel
makeblastdb -in uniprot_dmel.fa -dbtype prot -out uniprot_dmel.fa
```

Against the Drosophila-specific database, the blast search takes about 2 secs per protein request - depending on how many sequences you have submitted, you can make a fairly educated guess regarding the running time.

### load the retrieved information in your annotation file:  

Now you should be able to use the following script:
```
conda activate agat
agat_sp_manage_functional_annotation.pl -f maker_final.interpro/maker_final.gff -b blast.out --db $data/blastdb/uniprot_dmel/uniprot_dmel.fa -o maker_final.interpro.blast  
```
That will add the name attribute to the "gene" feature and the description attribute (corresponding to the product information) to the "mRNA" feature into your annotation file.
The improved annotation is the gff file inside the maker_final.interpro.blast folder.

:question:How many genes do not have any names?

<details>
<summary>:key: Click to see the solution .</summary>  
For instance, you can do this to count genes with names :  
<code> grep -P "\tgene" maker_final.interpro.blast/maker_final.gff | grep -v "Name" | wc -l</code>
</details>

:mortar_board: Choose one gene in your gff file with annotation with at least one GO terms (they look like GO:XXXX) and some domains annotated and have a look at them in the [Gene Ontology webpage](http://www.geneontology.org/) and [Interproscan webpage](http://www.ebi.ac.uk/interpro/).


### Set nice IDs

The purpose is to modify the ID value by something more convenient (i.e FLYG00000001 instead of maker-4-exonerate_protein2genome-gene-8.41).  
```
agat_sp_manage_functional_annotation.pl -f maker_final.interpro.blast/maker_final.gff -id FLY -o maker_final.interpro.blast.ID  
```
The improved annotation is the gff file inside the maker_final.interpro.blast.ID folder.

### Polish your file for a nice display within Webapollo

For a nice display of a gff file within Webapollo some modification might be needed.
As example the attribute ***product*** is not displayed in Webapollo, whereas renaming it to ***description*** will work out.
```
agat_sp_webApollo_compliant.pl -gff maker_final.interpro.blast.ID/maker_final.gff -o final_annotation.gff
```

## Visualise the final annotation

Transfer the final_annotation.gff file to your computer using scp in a new terminal:
```
scp -P 65022 __USER__@hpc.mf.uni-lj.si:/home/USER/functional_annotation/final_annotation.gff .
```

Load the file into the genome portal called drosophila_melanogaster_chr4 in the Webapollo genome browser available at the address [https://webapollo.nbis.se/elixirannotation2021/annotator/index](https://webapollo.nbis.se/elixirannotation2021/annotator/index). [Here find the WebApollo instruction](https://nbisweden.github.io/workshop-genome_annotation_elixir/labs/webapollo_usage)

Wonderfull ! isn't it ?

## What's next?

Because of Makers' compatibility with GMOD standards, a functional annotation created in one or both of this way can be loaded into e.g. WebApollo and will save annotators a lot of work when e.g. adding meta data to transcript models.
