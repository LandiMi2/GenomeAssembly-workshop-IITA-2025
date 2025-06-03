## Scaffolding tutorial

We now have a draft assembly that we will now scaffold using the OmniC reads. First, we will map the OmniC reads to the reference genome, then scaffold the genome using [YAHS](https://github.com/c-zhou/yahs) “yet another Hi-C scaffolding tool”

Before mapping the OmniC reads follow [QC tutorial](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/01_QC.md) - check for the quality and trim the reads based on the QC report

The data is avalable: `/data01/dataRepository/StarApple/`

    OmniC_R1.fastq.gz
    OmniC_R2.fastq.gz
 
**QC and trimming of the the OmniC reads**

For this task use fastqc and fastp as demonstrated the day before 

**Mapping OmniC reads to the reference genome**

Create a soft link of the genome to your workind directory 

`ln -s /data01/dataRepository/StarApple/StarApple.p.purged2.fa .`

Create an index file of the reference genome

`samtools faidx StarApple.p.purged2.fa`

Get the sizes of the contigs 

`cut -f1,2 StarApple.p.purged2.fa.fai > StarApple.p.purged2.fa.fai.sizes`


`mkdir tmp` this will be used as a temporary directory to store temporary output files. 

Now create _new screen_ to run the OmniC reads mapping script

All the tools have been compiled in a python environment, activate the environment

`source /data01/utilities/mapOmnic/bin/activate`

Copy below chuck of code and save it as script

`nano mapOmniC.sh`

Copy paste. We will discuss each line of code as we wait for the mapping to complete.
**Change the path of the tmp folder**

```
bwa index StarApple.p.purged2.fa 
bwa mem -SP -T0 -t 10 StarApple.p.purged2.fa \
/data01/dataRepository/StarApple/OmniC_R1_trim.fastq.gz \
/data01/dataRepository/StarApple/OmniC_R2.trim.fastq.gz | \
pairtools parse --min-mapq 0 --walks-policy 5unique \
--max-inter-align-gap 30 --nproc-in 4 --nproc-out 4 \
--chroms-path StarApple.p.purged2.fa.fai.sizes | \
pairtools sort --tmpdir=/data01/workshop/workshop/omniC/tmp --nproc 8 \
| pairtools dedup --nproc-in 4 \
--nproc-out 4 --mark-dups --output-stats stats.txt | pairtools split --nproc-in 4 \
--nproc-out 4 --output-pairs AsmOmniCmapped.pairs --output-sam - | samtools view -bS -@8 | \
samtools sort -@8 -o AsmOmniCmapped.bam ;samtools index AsmOmniCmapped.bam
```

**Scaffolding using YAHS**

`yahs StarApple.p.purged2.fa AsmOmniCmapped.bam -o StarAppleScaf`

**Genome evaluation** 

Run BUSCO and QUAST on the scaffolded genome. Compare `StarApple.p.purged2.fa` genome with the scaffolds. What is the difference? Discuss. 


|[Previous](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/03_assembly.md)|[Next](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/05_GenomeAnnotation.md)|
|---|---|




