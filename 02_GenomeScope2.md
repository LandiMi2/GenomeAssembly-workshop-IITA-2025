## GenomeScope2 Tutorial

When creating de novo assemblies, key factors to consider include the organism's ploidy, genome size, heterozygosity rate, and repeat content. These parameters can be estimated using [GenomeScope2](https://github.com/tbenavi1/genomescope2.0). Before running GenomeScope2, you must first compute the histogram of k-mer frequencies. We will use [jellyfish](https://github.com/gmarcais/Jellyfish)

K-mer Counting with Jellyfish <br>
`jellyfish count -C -m 21 -s 1000000 -t 2 cel1_50K_hifi_trimmed.fastq -o star.jr`

Histogram Generation from Jellyfish Output <br>
`jellyfish histo star.jr > star.histo`

Run GenomeScope to visualize <br>
`Rscript /usr/bin/genomescope.R -i star.histo -o star -k 21`

Genomscope can be installed as bellow:

```
git clone https://github.com/tbenavi1/genomescope2.0.git
cd genomescope2.0/
Rscript install.R
genomescope.R
```

Here is an example of StarApple Genome GenomeScope2 output
![genomescopePlot](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/transformed_linear_plot.png)

|[Previous](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/01_QC.md)|[Next](https://github.com/LandiMi2/GenomeAssemblyTut/blob/main/03_assembly.md)|
|---|---|
