## Scaffolding tutorial

We now have a draft assembly that we will now scaffold using the OmniC read. First, we will map the OmniC read to the reference genome, then scaffold the genome using [YAHS](https://github.com/c-zhou/yahs) “yet another Hi-C scaffolding tool”

Before mapping the OmniC reads follow [QC tutorial](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/01_QC.md) - check for the quality and trim the reads based on the QC report

The data is avalable: `/data01/dataRepository/rawData/StarApple/OmniC2`

    FC2001607-BCC_L01_Read1_Sample_Library_LA_African_Star_Apple_lane_1.fastq.gz
    FC2001607-BCC_L01_Read2_Sample_Library_LA_African_Star_Apple_lane_1.fastq.gz
 

**Mapping OmniC reads to the reference genome**

Create a soft link of the genome to your workind directory 

`ln -s /data01/dataRepository/StarApple/StarApple.p.purged2.fa .`

Create an index file of the reference genome

`samtools faidx StarApple.p.purged2.fa`

Get the sizes of the contigs 

`cut -f1,2 StarApple.p.purged2.fa.fai > StarApple.p.purged2.fa.fai.sizes`


`mkdir tmp` this will be used as a temporary files

All the tools have been compiled in a python enviroment, activate the enviroment
`source /data01/utilities/mapOmnic/bin/activate`

Copy below chuck of code and save it as script

`nano mapOmniC.sh`

Copy paste. We will discuss each line of code as we wait for the mapping to complete


```
bwa index StarApple.p.purged2.fa 
bwa mem -SP -T0 -t 10 StarApple.p.purged2.fa \
/data01/mlandi/Inqaba/data/OmniC_filtered/new/trimmed_omniC_R1.fastq.gz \
/data01/mlandi/Inqaba/data/OmniC_filtered/new/trimmed_omniC_R2.fastq.gz | \
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


|[Previous](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/03_assembly.md)|[Next](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/05_GenomeAnnotation.md)|
|---|---|




