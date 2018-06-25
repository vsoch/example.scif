---
title: Recipes
sidebar: main_sidebar
permalink: recipes
folder: docs
---

What the heck is a recipe? It's a text file that has instructions for install, run, environment, and test of your container. Within a recipe you will define different entrypoints, or functions that your container provides that someone would want to interact with. This means that your recipe might define several of these apps, and this is an important note because traditionally containers only provide a single entrypoint.


## Quick Start

 1. Edit the [recipe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif) file to include an app section (described below) for each of the functions in your container. This means chunks of text to add files, describe install sequences, write help, and test. You can read about the [different sections in Table 1 here](https://academic.oup.com/gigascience/article/7/5/giy023/4931737#tbl1).
 2. Add any external file dependencies to this repository. (We will soon be writing instructions here to describe a protocol for adding data (and download / mount on CI to test)


## Detailed Start
What are we looking at in the repository? When building pipelines, you can think of it like baking a cake. We have entire recipes for creating our final products (containers), and within those recipes ingredients (software) that we need to add. In this first part, we will talk about the three recipes in this repository, the [Dockerfile](https://github.com/vsoch/example.scif/blob/master/Dockerfile) for the Docker container, and the [recupe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif) for the Scientific Filesystem.


### The Scientific Filesystem Recipe
A scientific filesystem is useful because it allows me to write one recipe for my various software, and then install easily in different containers or on my host. How do you know when you find a recipe? When you find a recipe for a scientific filesystem (SCIF), you will see a file with extension *.scif. For example, in this repository:

 - [recipe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif) is the recipe for the scientific filesystem that will be installed in the container. SCIF is flexible in that there are **many** different internal applications defined in this one file, however if we wanted we could put them in individual files and install them equivalently. For example, given the apps "samtools" "tophat" and "bowtie" and using a single recipe file `recipe.scif`, I would install like:

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
distinction. This level of modularity is up to the user. For this repository, I've decided to provide the core applications in [recipe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif). 
Let's take a quick look at an example section:

```bash
%applabels bowtie
    VERSION 2.2.7
%appinstall bowtie
    curl -Lk -o bowtie.zip https://sourceforge.net/projects/bowtie-bio/files/bowtie2/2.2.7/bowtie2-2.2.7-linux-x86_64.zip/download
    unzip bowtie.zip
    mv bowtie2-2.2.7/* bin/
%apptest bowtie
    exec /scif/bowtie/bin/bowtie2 "$@"
```

In the example above:

 - the application name is "bowtie"
 - each section is in the format `%app<section> <name>`
 - I write steps to install things under `%appinstall`
 - I write steps to test things under `%apptest`
 - labels go under `%applabels`, and environment variables should be defined and exported under `%appenv`

For environment, this section will be sourced when the application context is active. More about this later - give a go at writing your first recipe, and then move on to read about [building](build).
