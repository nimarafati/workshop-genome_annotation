# 1. Assembly Check

Before starting an annotation project, we need to carefully inspect the assembly to identify potential problems before running expensive computes.
You can look at i) the Fragmentation (N50, N90, how many short contigs); ii) the Sanity of the fasta file (Presence of Ns, presence of ambiguous nucleotides, presence of lowercase nucleotides, single line sequences vs multiline sequences); iii) completeness using BUSCO; iv) presence of organelles; v) Others (GC content, How distant the investigated species is from the others annotated species available).
The two next exercices will perform some of these checks.

## 1.1 Checking the gene space of your assembly

BUSCO provides measures for quantitative assessment of genome assembly, gene set, and transcriptome completeness. Genes that make up the BUSCO sets for each major lineage are selected from orthologous groups with genes present as single-copy orthologs in at least 90% of the species.

***Note:*** In a real-world scenario, this step should come first and foremost. Indeed, if the result is under your expectation you might be required to enhance your assembly before to go further.

**_Exercise 1_ - BUSCO -:**

You will run BUSCO on the genome assembly.

First create a busco folder where you work:
```
mkdir busco
cd busco
```

The [BUSCO website](http://busco.ezlab.org) provides a list of datasets containing the cores genes expected in the different branches of the tree of life. To know in which part/branch of the tree of life is originated your species you can have a look at the [NCBI taxonomy website](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?id=7227) (Lineage line).
Then select the proper BUSCO Dataset on the [busco website](http://busco.ezlab.org) to check the completness of your assembly. To download the dataset to the cluster, you need the URL (right click on it, Copy Link). Then download the dataset.
/!\ In the example below the link copied is **http://busco.ezlab.org/datasets/metazoa_odb9.tar.gz**, so replace it by something else if you decided to take another dataset.
```
wget http://busco.ezlab.org/datasets/metazoa_odb9.tar.gz
tar xzvf metazoa_odb9.tar.gz
```

Now you are ready to launch BUSCO on our genome (genome.fa).
```
BUSCO.py -i ~/annotation_course/data/genome/genome.fa -o genome_dmel_busco -m geno -c 8 -l metazoa_odb9
```

While BUSCO is running, start the exercise 2.
When done, check the short\_summary\_genome\_dmel\_busco file in the output folder. How many core genes have been searched in you assembly ? How many are reported as complete? Does this sound reasonable?
**Tips**: the "genome" is here in fact only the chromosome 4 that corresponds to less than 1% of the real size of the genome.

## 1.2 Various Check of your Assembly

**_Exercise 2_ :**
Launching the following script will provide you some useful information.

```
cd ~/annotation_course/practical1
fasta_statisticsAndPlot.pl -f ~/annotation_course/data/genome/genome.fa -o fasta_check
```

Is your genome very fragmented (number of sequences)? Do you have high GC content ? Do you have lowercase nucleotides ? Do you have N at sequence extremities? 
If you don't see any peculiarities, you can then decide to go forward and start to perform your first wonderful annotation.