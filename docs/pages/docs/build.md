---
title: Build
sidebar: main_sidebar
permalink: build
folder: docs
---

For the build step, this is what happens:

 - you issue a command to the Docker daemon to build a container
 - the Dockerfile is the build specification for the container, and within this template it will use the scientific filesystem client to install your recipe
 - Upon successful build, your container is good to go!

First we will review these different kinds of recipes, and then we will show you how to do this build step (when you get errors of some kind).

## The Docker Recipe
If you are familar with Docker, you will know that the [Dockerfile](https://github.com/vsoch/example.scif/blob/master/Dockerfile) is the recipe for building our container. You will also notice the installation is simple - we start with a container base that was equivalently used by the creator of the pipeline with system / host dependencies, and then simply install the SCIF recipe to it. That comes down to these three commands:

```
RUN /usr/local/bin/pip install scif    # Install scif from pypi
ADD *.scif /                           # Add recipes to the container
RUN scif install /recipe.scif          # Install it to the container
```

We could build this via an automated build by connecting it to Docker Hub (so other users don't need to also build the container locally) or we can build locally ourselves:

```
docker build -t vanessa/example.scif .
```

## A Singularity Recipe?
You'll notice that we don't have a Singularity recipe in this repository, and the reason is because we can generate a Singularity image directly from a Docker container. Thus,
if we just build a Docker container, we kill two birds with one stone.  For your FYI, to build a Singularity container from Docker you can do any of the following:

```bash
singularity pull --name container.simg docker://vanessa/example.scif
singularity build container.simg docker://vanessa/example.scif
```



## Development Flow

>> My container didn't build!

It's really rare, even for an experienced developer, that everyone works in one go. You likely need to test the commands that ultimately wind up in your SCIF recipe, and
you want to do so in a way that doesn't require you to re-compile many times (and have to wait). Docker will cache some of the layers to save you time, but I still
want to show you a development strategy that is faster than that. What you can do is create a skeleton base image, and then interactively shell inside to test
commands as you do. To do this you can start with an empty recipe that has your apps defined. It might look like this:

```bash

%appinstall bwa
   echo "Steps to instll bwa go here!"

%appinstall samtools
   echo "Steps to instll samtools go here!"

%appinstall main
   echo "Steps to instll some primary thing go here!"
```

How do you know what apps to define? This is what we call the modularity of your container, and depending on its use case, you might do this in several ways.
You can read about [different strategies for this here](https://academic.oup.com/gigascience/article/7/5/giy023/4931737#116684246). The reason you want to add the names like
this is because it will create the base of the scientific filesystem for these apps. The only thing you need to know after that is that the installation steps (`%appinstall`)
happen in context of the application folder under `/scif/apps/<appname>` and that you have a rich suite of environment variables available to you (see [Table 2 here](https://academic.oup.com/gigascience/article/7/5/giy023/4931737#tbl2))

If you've already written steps in your recipe and some of them are erroring out, you can comment out these steps that trigger the error, and then build the container. You can also choose to completely comment out the `scif install ...` and create the directories manually to just test commands (`mkdir -p /scif/apps/<appname>`). This will build the container base without installing applications. Either way, when you are ready to build the container:

```
docker build -t vanessa/example.scif .
```

you can then shell inside:

```bash
docker run -it --entrypoint bash vanessa/example.scif
```

And manually run commands to test. When you are happy with a command, write it into your [recipe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif). Keep in mind
that during the actual install, the present working directory will always be `/scif/apps/<appname>` and the environment (`%env` section) will be sourced. The folders "bin" and
"lib" will also exist for you in the `$SCIF_APPROOT` environment variable. Actually, there are [lots of environment variables](https://sci-f.github.io///spec-v1#environment-namespace) that you can use!

When you are done, always be sure to do a final build from start to finish with the entire recipe. If you don't want to use the cache:


```bash
docker build -t --no-cache vanessa/example.scif
```

The cache can get in the way if, for example, you are using a remote resource (and the command in the file has changed) but the resource has.  At this point, you can read more about [interacting with your scif](usage) container, or go straight to best practices for [testing](testing).
