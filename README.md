# RNA-Seq toy pipeline 

[![scif](https://img.shields.io/badge/filesystem-scientific-green.svg?style=for-the-badge)](https://sci-f.github.io)

A proof of concept of a RNA-Seq pipeline. Here we are (trying to) combine three technologies to handle each of the following:

 - **Reproducibility** of software is handled by container technology
 - **Discoverability** and **Transparency** are handled by installing our software in the container via a [Scientific Filesystem](https://sci-f.github.io). SCIF also lets us install the same dependencies across container technologies.
 - **Testing** is handled by continuous integration, which understands how to interact with a scientific filesystem.

Each of these components plays a slightly different and equally important role. Without SCIF, we could have the same container with commands to execute known software inside, but the container would largely remain a black box with software mixed amongst the base operating system. Without container technologies, you could install software on your host, but (as we all know) this would likely not be a portable solution. Without testing, we couldn't be sure that our software works as we intended, and is ready to plug into some pipeline tool.

# Recipes
What are we looking at in the repository? When building pipelines, you can think of it like baking a cake. We have entire recipes for creating our final products (containers), and within those recipes ingredients (software) that we need to add. In this first part, we will talk about the three recipes in this repository, the [Dockerfile](Dockerfile) for the Docker container, and the [recupe.scif](recipe.scif) for the Scientific Filesystem.

## The Scientific Filesystem Recipe
A scientific filesystem is useful because it allows me to write one recipe for my various software, and then install easily in different containers or on my host. How do you know when you find a recipe? When you find a recipe for a scientific filesystem (SCIF), you will see a file with extension *.scif. For example, in this repository:

 - [recipe.scif](recipe.scif) is the recipe for the scientific filesystem that will be installed in the container. SCIF is flexible in that there are **many** different internal applications defined in this one file, however if we wanted we could put them in individual files and install them equivalently. For example, given the apps "samtools" "tophat" and "bowtie" and using a single recipe file `recipe.scif`, I would install like:

```
/usr/local/bin/scif install recipe.scif
```

but I could also define the different applications in separate files, and these commands would give me the same result.

```
/usr/local/bin/scif install samtools.scif
/usr/local/bin/scif install bowtie.scif
/usr/local/bin/scif install tophat.scif
```

Why might you want to do this? You might want to combine recipes, or have different maintainers and keep the files separate for a clean
distinction. This level of modularity is up to the user. For this repository, I've decided to provide the core applications in [recipe.scif](recipe.scif). 

## The Docker Recipe
If you are familar with Docker, you will know that the [Dockerfile](Dockerfile) is the recipe for building our container. You will also notice the installation is simple - we start with a container base that was equivalently used by the creator of the pipeline with system / host dependencies, and then simply install the SCIF recipe to it. That comes down to these three commands:

```
RUN /usr/local/bin/pip install scif    # Install scif from pypi
ADD *.scif /                           # Add recipes to the container
RUN scif install /recipe.scif          # Install it to the container
```

We could build this via an automated build by connecting it to Docker Hub (so other users don't need to also build the container locally) or we can build locally ourselves:

```
docker build -t vanessa/example.scif .
```

## Singularity
A cool thing about Singularity is that you can import a Docker container into a Singularity image. Thus, for the following examples we will show usage with
both Docker and Singularity commands. Given an image `vanessa/example.scif` hosted on Docker Hub, we can create a Singularity container called `rnatoy` with the 
conversion command:


```bash
singularity pull --name rnatoy docker://vanessa/example.scif
```

# The Scientific Filesystem
For each of the following examples, we show commands with Docker and with a Singularity container called `rnatoy`. If we want to interact with our filesystem, we can just run the container:

```
$ docker run vanessa/example.scif
$ ./rnatoy
```
```
Scientific Filesystem [v0.0.71]
usage: scif [-h] [--debug] [--quiet] [--writable]
            
            {version,pyshell,shell,preview,help,install,inspect,run,apps,dump,exec}
            ...

scientific filesystem tools

optional arguments:
  -h, --help            show this help message and exit
  --debug               use verbose logging to debug.
  --quiet               suppress print output
  --writable, -w        for relevant commands, if writable SCIF is needed

actions:
  actions for Scientific Filesystem

  {version,pyshell,shell,preview,help,install,inspect,run,apps,dump,exec}
                        scif actions
    version             show software version
    pyshell             Interactive python shell to scientific filesystem
    shell               shell to interact with scientific filesystem
    preview             preview changes to a filesytem
    help                look at help for an app, if it exists.
    install             install a recipe on the filesystem
    inspect             inspect an attribute for a scif installation
    run                 entrypoint to run a scientific filesystem
    apps                list apps installed
    dump                dump recipe
    exec                execute a command to a scientific filesystem
```

## Inspecting Applications
The strength of SCIF is that it will always show you the applications installed in a container, and then provide predictable commands for inspecting, running, or otherwise interacting with them. For example, if I find the container, without any prior knowledge I can reveal the applications inside:

```
$ docker run vanessa/rnatoy apps
$ ./rnatoy apps
```
```
    bowtie
 cufflinks
    tophat
  samtools
```

We can look at an application in detail, including asking for help:

```
$ docker run vanessa/example.scif help samtools
$ ./rnatoy help samtools
```
```
    This app provides Samtools suite
```

and then inspecting

```
$ docker run vanessa/example.scif inspect samtools
$ ./rnatoy inspect samtools
```
```
{
    "samtools": {
        "apprun": [
            "    exec /usr/bin/samtools \"$@\""
        ],
        "apphelp": [
            "    This app provides Samtools suite"
        ],
        "applabels": [
            "VERSION 1.7",
            "URL http://www.htslib.org/"
        ]
    }
}
```

The creator of the container didn't write any complicated scripts to have this happen - the help text is just a chunk of text in a block of the recipe. The labels that are parsed to json, are also just written easily on two lines. This means that the creator can spend less time worry about exposing this. If you can write a text file, you can make your applications programatically parseable.


## Interacting with Applications
I can easily shell into the container in the context of an application, meaning that the
environment is sourced, etc. 

```
$ docker run -it vanessa/example.scif shell samtools
$ ./rnatoy shell samtools
```
```
[samtools] executing /bin/bash 
root@d002e338b88b:/scif/apps/samtools# env | grep PATH
LD_LIBRARY_PATH=/scif/apps/samtools/lib
PATH=/scif/apps/samtools/bin:/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Notice how I'm in the app's context (in it's application folder) and that it's bin is added to the path? I can also shell in without a specific application context, but still have all the SCIF [global variables](https://sci-f.github.io/spec-v1#environment-namespace) available to me.

```
$ docker run -it vanessa/example.scif shell
$ ./rnatoy shell
```
```
WARNING No app selected, will run default ['/bin/bash']
executing /bin/bash 
root@055a34619d17:/scif# ls
apps
data
```

The same kind of functionality exists with the python shell, `pyshell`, but you interact directly with the scif client:

```
$ docker run -it vanessa/example.scif pyshell
$ ./rnatoy pyshell
```
```
Found configurations for 4 scif apps
cufflinks
samtools
bowtie
tophat
[scif] /scif cufflinks | samtools | bowtie | tophat | nextflow-docker-config | nextflow-singularity-config
Python 3.6.2 |Anaconda, Inc.| (default, Sep 22 2017, 02:03:08) 
[GCC 7.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
client.apps()
['cufflinks', 'samtools', 'bowtie', 'tophat']
```

## Running Applications
Before we get into creating a pipeline, look how easy it is to run an application. Without scif, we would have to have known that samtools is installed, and then executed the command to the container. But with the scientific filesystem, we discovered the app (shown above) and then we can just run it. The `run` command maps to the entrypoint, as was defined by the creator:

```
$ docker run vanessa/example.scif run samtools
$ ./rnatoy run samtools
```
```
Program: samtools (Tools for alignments in the SAM format)
Version: 0.1.18 (r982:295)

Usage:   samtools <command> [options]

Command: view        SAM<->BAM conversion
         sort        sort alignment file
         mpileup     multi-way pileup
         depth       compute the depth
         faidx       index/extract FASTA
         tview       text alignment viewer
         index       index alignment
         idxstats    BAM index stats (r595 or later)
         fixmate     fix mate information
         flagstat    simple stats
         calmd       recalculate MD/NM tags and '=' bases
         merge       merge sorted alignments
         rmdup       remove PCR duplicates
         reheader    replace BAM header
         cat         concatenate BAMs
         targetcut   cut fosmid regions (for fosmid pool only)
         phase       phase heterozygotes

[samtools] executing /bin/bash /scif/apps/samtools/scif/runscript
```

And executing any command in the context of the application is possible too:

```
$ docker run vanessa/example.scif exec samtools env | grep PATH
$ ./rnatoy exec samtools env | grep PATH
```
```
LD_LIBRARY_PATH=/scif/apps/samtools/lib
PATH=/scif/apps/samtools/bin:/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Note that for above, you will get more output with the Singularity container, as it shares the environment with the host. Whether we are using Docker or Singularity, the actions going on internally with the scientific filesystem client are the same. Given a simple enough pipeline, we could stop here, and just issue a series of commands to run the different apps.


## Run Using Docker + Scientific Filesystem
Here are examples of how to run each of the applications using SCIF with the container. For the following steps, scif ensures that `/scif/data` exists in the container, so we can use it as a working directory with confidence.


### Bowtie
Here is what we are starting with:

```
ls data/ggal
ggal_1_48850000_49020000.bed.gff               ggal_gut_1.fq  ggal_liver_1.fq
ggal_1_48850000_49020000.Ggal71.500bpflank.fa  ggal_gut_2.fq  ggal_liver_2.fq
```

and let's define what these paths will look like in the container. The /scif/data folder
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

### Tophat
Now let's do the next step, and we will do the same sort of deal. Note I'm not sure if I am executing this correctly, I've never used tophat.

```
reads="/scif/data/ggal_gut_1.fq /scif/data/ggal_gut_2.fq /scif/data/ggal_liver_1.fq /scif/data/ggal_liver_2.fq"
annot=/scif/data/ggal_1_48850000_49020000.bed.gff
```
```
docker run -v $PWD/data/ggal:/scif/data vanessa/example.scif exec tophat tophat2 -p 1 --GTF $annot $genomeIndex $reads
singularity run -B $PWD/data/ggal:/scif/data rnatoy exec tophat tophat2 -p 1 --output-dir /scif/data --GTF $annot $genomeIndex $reads
```

### Cufflinks

Finally, this one!

```
bam_file=/scif/data/unmapped.bam
```
```
docker run -v $PWD/data/ggal:/scif/data vanessa/example.scif exec cufflinks cufflinks --no-update-check -q -p 1 -G $annot $bam_file
singularity run -B $PWD/data/ggal:/scif/data rnatoy exec cufflinks cufflinks --no-update-check -q -p 1 -G $annot $bam_file
```

I don't think that was right (There are two bam files, accepted_hits and unmapped and I have no idea?), but I can't spend more time on this! Note that the rest of this description is wrong, I can't figure out for the life of me how Nextflow works. I'm going to try other workflow managers instead.


## Additions / Notes:
These are additional notes:

 - note the modular nature of the apps, I can now know that the container has bowtie, samtools, without seeing the recipe or listing executables on the path.
 - software install goes into respective bins of the application folder. Before we were installing to opt, and had to add these to the path.
 - I added `python-setuptools` to install scif from PyPi.
 - Note that for some software, I chose to install from source code in the /scif base, because I can then make a strong assumption that the files there belong to the software. But let's say that I for some reason want to use the system package manager, but still reveal the executable as a scif entrypoint? The apps here show this example with samtools. I define the entry point to execute it.

The original code, to build "the same" Docker and Singularity containers, took the build strategy of:

## Why use a SCIF recipe?
When we build a container, we usually need to start with a Dockerfile (or Singularity recipe) and it looks like this:

```
Dockerfile --> Dockerhub --> Docker Image
               Dockerhub --> Singularity Recipe --> Singularity Container
```

But what if we want to build both Docker and Singularity? We can either create one recipe type (and convert to the other) or have an installation of a recipe file
that can be done  **any** container technology. That is SCIF.  Our workflow instead looks like this:

```
SCIF Recipe --> Dockerfile --> Dockerfile --> Docker Image
            --> Singularity Recipe --> Singularity Container

```


## Development
You likely need to develop your SCIF recipe, meaning testing commands to install software. During the development of the container, I took a strategy to start with a base, interactively shell into it, and test installation and running of things. To do that you might want to build the container first from the Dockerfile, but comment
out the file that does the `scif install ...`. This will build the container base without installing applications. Then you can build the image:

```
docker build -t vanessa/example.scif .
```

and shell inside

```
docker run -it --entrypoint bash vanessa/example.scif
```

And manually run commands to test. When you are happy with a command, write it into your [recipe.scif](recipe.scif). Keep in mind
that during the actual install, the present working directory will always be `/scif/apps/<appname>` and the environment (`%env` section) will be sourced. The folders "bin" and
"lib" will also exist for you in the `$SCIF_APPROOT` environment variable. Actually, there are [lots of environment variables](https://sci-f.github.io///spec-v1#environment-namespace) that you can use!
