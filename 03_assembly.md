## Assembly tutorial 

Here we will assemble our trimmed reads using three assembly tools [hifiasm](https://github.com/chhylp123/hifiasm), [canu](https://canu.readthedocs.io/en/latest/tutorial.html) and [flye](https://github.com/mikolmogorov/Flye)  


**Running hifiasm**

`hifiasm -o star -t 2 cel1_50K_hifi_trimmed.fastq --primary`
Convert gfa to fasta
`awk '/^S/{print ">"$2;print $3}' star.p_ctg.gfa > star.p.fa `

Discuss the output files

**Running canu**

`canu -d canu/ -p starCanu -pacbio-hifi cel1_50K_hifi_trimmed.fastq genomeSize=3m -useGrid=false -merylThreads=4 -merylMemory=8 corOverlapper=ovl`

Discuss the output files

**Running flye**

`flye --pacbio-hifi ../../QC/trim/cel1_50K_hifi_trimmed.fastq -o flye/ --threads 2`

Discuss the output files


## Assembly statistics 

Compare the three assemblies using QUality ASsessment Tool [QUAST](https://github.com/ablab/quast)


`quast.py <assembly1> <assembly2> <assembly3> -o outDir`

Download and discuss the report 

## Assembly completness 

We will now assess the completness of the best assembly from our previous comparison using - Benchmarking Universal Single-Copy Orthologs [BUSCO](https://busco.ezlab.org/)

We will first install busco locally on our accounts

`git clone https://gitlab.com/ezlab/busco.git`

`cd busco/`

`python -m pip install .`

Run busco

`busco -m genome -i star.p.fasta -o  starBusco --metaeuk -l eudicots_odb10 -c 2`

Explore the summary report

|[Previous]()|[Next]()|
|---|---|
