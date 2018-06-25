---
title: RNA-Seq Example
sidebar: main_sidebar
permalink: example
folder: docs
---


Let's walk through using the container that is provided by this repository. As a reminder, here is how we built it:

```bash
docker pull vanessa/example.scif
docker build -t vanessa/example.scif .
```

Now let's review examples of how to run each of the applications using SCIF with the container. For the following steps, scif ensures that `/scif/data` exists in the container, so we can use it as a working directory (to mount from the hose) if needed. The weird name is chosen specifically to ensure that the same path doesn't exist on the host you
are mounting from.


## Bowtie
Here is what we are starting with: this is dat in the repository.

```
ls data/ggal
ggal_1_48850000_49020000.bed.gff               ggal_gut_1.fq  ggal_liver_1.fq
ggal_1_48850000_49020000.Ggal71.500bpflank.fa  ggal_gut_2.fq  ggal_liver_2.fq
```

and let's define what these paths will look like in the container. The `/scif/data` folder
that we know to exist with scif we will map to "data" in the present working directory.

```
genome=/scif/data/ggal_1_48850000_49020000.Ggal71.500bpflank.fa
genomeIndex=${genome}.index
```
```
docker run -v $PWD/data/ggal:/scif/data vanessa/example.scif exec bowtie bowtie2-build --threads 1 $genome $genomeIndex
singularity run -B $PWD/data/ggal:/scif/data rnatoy exec bowtie bowtie2-build --threads 1 $genome $genomeIndex
```

In the above, notice that I am:

 1. defining a genome and index output to be in the /scif/data folder in the container
 2. which is mapped to my host $PWD/data folder that has the data files

The output is verbose, but I get the result on my local machine!

```
ls data/ggal/
ggal_1_48850000_49020000.bed.gff                           ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.3.bt2      ggal_gut_1.fq
ggal_1_48850000_49020000.Ggal71.500bpflank.fa              ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.4.bt2      ggal_gut_2.fq
ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.1.bt2  ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.rev.1.bt2  ggal_liver_1.fq
ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.2.bt2  ggal_1_48850000_49020000.Ggal71.500bpflank.fa.index.rev.2.bt2  ggal_liver_2.fq
```

## Tophat
Now let's do the next step, and we will do the same sort of deal. Note I'm not sure if I am executing this correctly, I've never used tophat.

```
reads="/scif/data/ggal_gut_1.fq /scif/data/ggal_gut_2.fq /scif/data/ggal_liver_1.fq /scif/data/ggal_liver_2.fq"
annot=/scif/data/ggal_1_48850000_49020000.bed.gff
```
```
docker run -v $PWD/data/ggal:/scif/data vanessa/example.scif exec tophat tophat2 -p 1 --GTF $annot $genomeIndex $reads
singularity run -B $PWD/data/ggal:/scif/data example.simg exec tophat tophat2 -p 1 --output-dir /scif/data --GTF $annot $genomeIndex $reads
```
