## Repeat and Gene annotation

### Repeat identification, annotation, and masking

After evaluating your assemblies and ascertaining that they are of suitable quality to proceed to annotation, the next logical step is to identify and annotate repeat elements in the assembly. Repeats are annotated before genes because there is a crucial step to mask them before the assembly can be parsed to gene annotation tools/pipelines.


In this tutorial, we will exemplify how to use two tools for the purposes of repeat identification, annotation, and masking:
- EDTA
- Repeatmasker

**EDTA**

The Extensive de novo TE Annotator (EDTA) is a fully automated whole-genome de novo TE annotator. It integrates eight TE identification programs to create a powerful and complete TE annotation pipeline. These programs include LTRharvest. LTRfinder and LTR retriever to find LTRS, Generic Repeat Finder and TIR learner to find TIR transposons, Helitron scanner to identify helitron transposons, Repeat modeller to identify TEs (including SINES and LINEs) missed by other structure-based programs, and RepeatMasker to annotate fragmented TEs.


To start, we will make a directory to store our EDTA results and intermediate files

`mkdir EDTA`

Now we will enter the EDTA directory and compute EDTA within this directory.

`cd EDTA`

`perl EDTA.pl --genome genome.fa --overwrite 1 --sensitive 1 --anno 1 --threads 10`

The options used do the following. The genome file [FASTA] is required and should be replaced with the respective genome that is being used. Please make sure sequence names are short (<=13 characters) and simple (i.e, letters, numbers, and underscore)

  	--genome [File]    The genome FASTA file. Required. <br>
	--sensitive [0|1]  Use RepeatModeler to identify remaining TEs (1) or not (0, default). This step may help to recover some TEs.<br>
	--anno [0|1]    Perform (1) or not perform (0, default) whole-genome TE annotation after TE library construction.<br>
	--overwrite [0|1]   If previous raw TE results are found, decide to overwrite (1, rerun) or not (0, default)<br>

When the --anno 1 parameter is specified, you will get:

 - Whole-genome TE annotation: genome.mod.EDTA.TEanno.gff3. This file contains both structurally intact and fragmented TE annotations.
 - Summary of whole-genome TE annotation: genome.mod.EDTA.TEanno.sum.
 - Low-threshold TE masking: genome.mod.MAKER.masked. This is a genome file with only long TEs (>=1 kb) being masked. You may use this for de novo gene annotations. In practice, this approach will reduce overmasking for genic regions, which can improve gene prediction quality. However, initial gene models should contain TEs and need further filtering.
 - Annotation inconsistency for simple TE genome.mod.EDTA.TE.fa.stat.redun.sum.
 - Annotation inconsistency for nested TEs genome.mod.EDTA.TE.fa.stat.nested.sum.
 - Overall annotation inconsistency: genome.mod.EDTA.TE.fa.stat.all.sum.

NB: You can look at genome.mod.EDTA.TEanno.sum to see the composition of transposable elements in your genome. Something like what is shown below.
![TEout](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/TE.png)

After TE identification and annotation, it is important to mask these repeats before gene annotation. This is because repeats will cause non-specific gene hits, leading to interference with the accuracy of gene annotation. Gene prediction tools often struggle to identify gene boundaries and structures in regions with high repeat content, potentially leading to overestimation of genes or misinterpretation of gene structures. Masking repeats helps to reduce noise and bias, allowing for more accurate gene prediction. 

To mask repeats, we will use make_masked.pl script from EDTA.

`perl (path-to-EDTA-folder)/bin/make_masked.pl -genome genome.fa -rmout genome.mod.EDTA.anno/genome.mod.EDTA.TEanno.out -hardmask 0 -t 2 > ../data/genome.masked.fa`

	-rmout  [file]  Required. The repeatmasker.out file
	-hardmask [0|1] Output softmask (0) or hardmask (1, default) genome.
	-threads|-t  [int]   Number of threads to run this program. Default: 4

We will softmask the genome i.e., replacing the repetitive sequences with lowercase letters (a, c, g, t). Soft masking allows the genome prediction tool to still have sequence information at its disposal. If, for instance, the masking tool has some false positive maskings, those might still get recovered by the gene predictor as they might have some transcript data aligned to it and might be part of a valid gene.

**Repeatmasker**

RepeatMasker is a program that screens DNA sequences for interspersed repeats and low-complexity DNA sequences. The output of the program is a detailed annotation of the repeats that are present in the query sequence, as well as a modified version of the query sequence in which all the annotated repeats have been masked (default: replaced by Ns). 

`mkdir RepeatMasker`

`cd RepeatMasker`

`RepeatMasker -e ncbi -pa 36 -q -no_is -norna -nolow -div 40 genome.fa`

This is what these options used mean:

	-e(ngine) [crossmatch|wublast|abblast|ncbi|rmblast|hmmer]
        Use an alternate search engine to the default. Note: 'ncbi' and 'rmblast' are both aliases for the rmblastn search engine. The generic NCBI blastn program is not sensitive enough for use with RepeatMasker at this time.
	-pa(rallel) [number]
        The number of sequence batch jobs [50kb minimum] to run in parallel. RepeatMasker will fork off this number of parallel jobs, each running the search engine specified. For each search engine invocation (where applicable), a fixed number of cores/ threads

	-q  Quick search; 5-10% less sensitive, 2-5 times faster than default

	-nolow  Does not mask low-complexity DNA or simple repeats

	-norna Does not mask small RNA (pseudo) genes

	-div [number]
    Masks only those repeats < x percent diverged from consensus seq

	-no_is  Skips bacterial insertion element check

Once RepeatMakser is run, the program returns three or four output files for each query. 
1. One contains the submitted sequence(s) in which all recognized interspersed or simple repeats have been masked. In the masked areas, each base is replaced with an N, so that the returned sequence is of the same length as the original.
2. A table annotating the masked sequences, as well as a table summarizing the repeat content of the query sequence, will be returned to your screen.
3. Optionally, a file with alignments of the query with the matching repeats will be returned as well.

In the "html" return format (default when the browser runs on a Mac or PC), all output is returned to your screen in one file. In the "tar file" return format, the masked sequence(s) and alignments can be saved as compressed files. The "links" return format returns links to these output files in a text format (they look bad on the browser, but are fine when saved to your computer).

### Gene prediction and annotation

Once repeats have been identified, annotated, and masked, it is time to predict and annotate genes. For this part of the tutorial, we will use BRAKER2 and Funannotate.

**Gene prediction**

BRAKER is a fully automated pipeline to train and predict genes with GeneMark and AUGUSTUS. 
BRAKER1, a combination of GeneMark-ET and AUGUSTUS, that uses genomic and RNA-Seq data to automatically generate full gene structure annotations in a novel genome.
However, the quality of RNA-Seq data that is available for annotating a novel genome is variable, and in some cases, RNA-Seq data is not available at all.
BRAKER2 is an extension of BRAKER1, which allows for fully automated training of the gene prediction tools GeneMark-ES/ET/EP/ETP and AUGUSTUS from RNA-Seq and/or protein homology information, and integrates the extrinsic evidence from RNA-Seq and protein homology information into the prediction.
BRAKER2 reaches high gene prediction accuracy even in the absence of the annotation of very closely related species and in the absence of RNA-Seq data.

![brakerWorkflow](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/brakerWorkflow.png)

To run BRAKER2, a database of proteins that may be of unknown evolutionary distance to the target species is required, particularly suitable if no RNA-Seq data is available. 
We will use pre-partitioned Viridiplantae OrthoDB v.11 clades downloaded from [here](https://bioinf.uni-greifswald.de/bioinf/partitioned_odb11/). Additional protein sequence data can be obtained from Uniprot.

We will run BRAKER2.

`mkdir braker_prediction`

`cd braker_prediction`

`singularity exec -B ${PWD}:${PWD} ~/software/braker/braker3.sif braker.pl --genome=genome.masked.fa --prot_seq=Viridiplantae.fa`

The input required is as follows:
	genome.masked.fa - genome file in fasta format
	proteins.fa - protein sequences in fasta format
 
BRAKER produces several important output files in the working directory.

	braker.gtf: Final gene set of BRAKER. This file may contain different contents depending on how you called BRAKER. Since BRAKER2 was run, this file will contain:
	- Union of augustus.hints.gtf and reliable GeneMark-ES/ET/EP predictions (genes fully supported by external evidence). In --esmode, this is the union of augustus.ab_initio.gtf and all GeneMark-ES 	genes. Thus, this set is generally more sensitive (more genes correctly predicted) and can be less specific (more false-positive predictions can be present). This output is not necessarily better 	than augustus.hints.gtf, and it is not recommended to use it if BRAKER was run in ES mode.
	braker.codingseq: Final gene set with coding sequences in FASTA format
	braker.aa: Final gene set with protein sequences in FASTA format
	braker.gff3: Final gene set in gff3 format (only produced if the flag --gff3 was specified to BRAKER.
	Augustus/*: Augustus gene set(s) in as gtf/conding/aa files
	GeneMark-E*/genemark.gtf: Genes predicted by GeneMark-ES/ET/EP/EP+/ETP in GTF-format.
	hintsfile.gff: The extrinsic evidence data extracted from RNAseq.bam and/or protein data.
	braker_original/*: Genes predicted by BRAKER (TSEBRA merge) before compleasm was used to improve BUSCO completeness
	bbc/*: output folder of best_by_compleasm.py script from TSEBRA that is used to improve BUSCO completeness in the final output of BRAKER

 
 **InterProScan**

InterPro is a database that integrates predictive information about proteins’ function from a number of partner resources, giving an overview of the families that a protein belongs to and the domains and sites it contains. 


Before running InterProScan, it is imperative to know that there are asterisks (*) at the end of each record of protein sequences outputted from BRAKER2. These should be removed for the input to be compatible with Interproscan. Because InterProScan does not allow gaps (‘-‘), periods (‘.’), asterisks (‘*’), or underscores (_), symbols are not allowed and will produce warnings, and InterProScan will exit immediately. The file requires modification.

Modifying the braker.aa file

`cd braker_prediction`

`sed '/^[^>]/ s/\*$//' braker.aa > braker.modified.aa`

Run interproscan

`mkdir interproscan`

`cd interproscan`

`bash ~/software/interproscan/interproscan-5.65-97.0/interproscan.sh -i results/braker_prediction/braker.modified.aa -d results/interproscan/ --goterms`

	-i,--input <INPUT-FILE-PATH>
	-d,--output-dir <OUTPUT-DIR>
	-goterms,--goterms  Optional, switch on lookup of corresponding Gene Ontology annotation (IMPLIES -iprlookup option)

 **Eggnog**

Eggnog infers gene function via orthology rather than homology, which is considered more reliable for transferring functional information between molecular sequences, as orthologs are expected to retain function more often than paralogs.

Running eggnog

`mkdir eggnog`

`cd eggnog`

`/data01/nyasita/miniconda3/bin/emapper.py -i results/braker_prediction/braker.aa --output eggnog.emapper -m diamond --cpu 4 --tax_scope Viridiplantae --data_dir ../data/EggNog_Data/`

	-m diamond,mmseqs,hmmer,no_search, cache,novel_fams 

	diamond: search seed orthologs using diamond (-i is required). mmseqs: search seed orthologs using MMseqs2 (-i is required). hmmer: search seed orthologs using HMMER. (-i is required). no_search: 	skip seed orthologs search (--annotate_hits_table is required, unless --no_annot). cache: skip seed orthologs search and annotate based on cached results (-i and -c are required).novel_fams: search 	against the novel families database (-i is required). (default: diamond)

	--tax_scope TAX_SCOPE
	Fix the taxonomic scope used for annotation, so only speciation events from a particular clade are used for functional transfer. More specifically, the --tax_scope list is intersected with the seed 	orthologs clades, and the resulting clades are used for annotation based on --tax_scope_mode. Note that those seed orthologs without clades intersecting with --tax_scope will be filtered out, and 	won't annotated. Possible arguments for --tax_scope are: 
	1) A path to a file defined by the user, which contains a list of tax IDs and/or tax names. 
	2) The name of a pre-configured tax scope, whose source is a file stored within the 'eggnogmapper/annotation/tax_scopes/' directory By default, available ones are: 'auto' ('all'), 'auto_broad' 	('all_broad'), 'all_narrow', 'archaea', 'bacteria', 'bacteria_broad', 'eukaryota', 'eukaryota_broad' and 'prokaryota_broad'.
	3) A comma-separated list of taxonomic names and/or taxonomic
	IDs, sorted by preference. An example of list of tax IDs would be 2759,2157,2,1 for Eukaryota, Archaea, Bacteria and root, in that order of preference. 
	4) 'none': do not filter out annotations based on taxonomic scope. (default: auto)


**Annotation evaluation with BUSCO**

Once annotation is complete, it is smart to evaluate your annotation. One such way is to use BUSCO in the protein mode. The input file should be a protein fasta file. The input file is passed directly to HMMER, which scores the proteins against the HMM profiles for the BUSCO marker genes in a given dataset.

Running busco

`mkdir busco_results`

`cd busco_results`

`busco -m prot -i results/funannotate_annotate/annotate_results/Arabidopsis_thaliana.proteins.fa -o . --auto-lineage`

_outputs_

![busco](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/buscoScreenShot.png)

|[Previous](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/04_YAHS.md)|
|---|














