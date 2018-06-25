---
title: The Scientific Filesyste
sidebar: main_sidebar
permalink: usage
folder: docs
---

How do you use a container with a scientific filesystem? Here we will use the container built by this repository to learn this! First, obtain the container. You can build
or pull it.

```bash
docker pull vanessa/example.scif
docker build -t vanessa/example.scif .
```

For each of the following examples, we show commands with Docker and with a Singularity container called `example.simg`. 

## Interact with SCIF client
If we want to interact with our filesystem, we can just run the container:

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

this works because the scif is the entrypoint to the container.

## Inspecting Applications
The strength of SCIF is that it will always show you the applications installed in a container, and then provide predictable commands for inspecting, running, or otherwise interacting with them. For example, if I find the container, without any prior knowledge I can reveal the applications inside:

```
$ docker run vanessa/example.scif apps
$ ./example.simg apps
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
$ ./example.simg help samtools
```
```
    This app provides Samtools suite
```

and then inspecting

```
$ docker run vanessa/example.scif inspect samtools
$ ./example.simg inspect samtools
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
$ ./example.simg shell samtools
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
$ ./example.simg shell
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
$ ./example.simg pyshell
```
```
Found configurations for 4 scif apps
cufflinks
samtools
bowtie
tophat
[scif] /scif cufflinks | samtools | bowtie | tophat
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
$ ./example.simg run samtools
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
$ ./example.scif exec samtools env | grep PATH
```
```
LD_LIBRARY_PATH=/scif/apps/samtools/lib
PATH=/scif/apps/samtools/bin:/opt/conda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Whether we are using Docker or Singularity, the actions going on internally with the scientific filesystem client are the same. Given a simple enough pipeline, we could stop here, and just issue a series of commands to run the different apps. But more likely you would then integrate these app entrypoints into some pipeline. If you are a developer, you may not even have a pipeline, but want to provide your software for others to use (and integrate into their pipelines!)

Now that you understand usage, take a look at the [example](example) provided in this repository.
