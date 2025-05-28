## Repeat and Gene annotation

**Repeat identification, annotation, and masking**

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
