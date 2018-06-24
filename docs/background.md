# Background

The original code, to build "the same" Docker and Singularity containers, took the build strategy of having a Singularity recipe
and Dockerfile in the same repository. Since we can import Docker into Singularity, I decided to only write one recipe (Dockerfile)
and thus kill two birds with one stone.

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

## Addtional Notes

This is some brief (additional) background to give rationale for this approach.

 - note the modular nature of the apps, I can now know that the container has bowtie, samtools, without seeing the recipe or listing executables on the path.
 - software install goes into respective bins of the application folder. Before we were installing to opt, and had to add these to the path.
 - Note that for some software, I chose to install from source code in the /scif base, because I can then make a strong assumption that the files there belong to the software. But let's say that I for some reason want to use the system package manager, but still reveal the executable as a scif entrypoint? The apps here show this example with samtools. I define the entry point to execute it.

