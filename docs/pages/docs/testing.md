---
title: Testing
sidebar: main_sidebar
permalink: testing
folder: docs
---

You will need to set up testing for your repository before issuing any pull request. Testing means using a
"continuous integration" service like [CircleCI](https://circleci.com/workflow-run/a8dc69fa-fa42-4b47-8af5-611f924e175b) to
run a workflow. The workflow typically consists of the following steps:

 - Connect the repository, and add deployment credentials
 - Build your container, taking advantage of the cache of layers to speed things up
 - Download any data files you need (that maybe were too big to host in the repository)
 - Test the applications in your container
 - Upon successful build and test, deploy the container to Docker Hub (or else where)


These docs are still under development, since we have to define criteria for testing. I'll write brief notes here for now.

## 1. Connect the repository
You should be able to log in with your Github account, and click "Add a new project" to [select your repository](https://circleci.com/dashboard).
Once you have added the project, in the project settings (the gear icon in the upper right of the project main page) you should be able to
see a tab under "Build Settings" called "Environment Variables." Here you should define your Docker username and password
(or whatever deployment you decide to use, note that if you don't use Docker Hub you will need to edit the config.yml). Take a look
at other [deployment options here](https://circleci.com/docs/2.0/deployment-integrations/).

These are the two you should add. They will be encrypted. Yes, it's always risky even to put an encrypted password, but this is
true of passwords for any service.

```bash
DOCKER_USER
DOCKER_PASS
```

Given that these variables are found, a container will be pushed to Docker Hub on successful build. The tag will be the Circle CI
build tag. If not, it will just skip over the step. This is how others will be able to pull your container from Docker Hub:

```bash
docker pull vanessa/example.scif
```

## 2. The Configuration File

You'll notice the template has a "hidden" directory called `.circleci` with a `config.yml` inside. This is a file that defines the workflow
to define the steps that we described above. Importantly, you don't need to edit this, because the steps already know
to build your container from your Dockerfile, and how to interact with it via issuing commands to the scientific filesystem.
All you need to do is write tests for your applications, and this is done in the [recipe.scif](https://github.com/vsoch/example.scif/blob/master/recipe.scif)

## Step 1: Write tests
The testing section discovers the apps in your container like this:

```bash
$ docker run -it vanessa/example.scif apps
 cufflinks
  samtools
    bowtie
    tophat
```

And then logically we can loop through these apps, and run the test for each!

```bash
$ for app in $(docker run -it vanessa/example.scif apps)
    do
        docker run -it vanessa/example.scif test ${app} 
    done
```

Note that @vsoch hasn't fully set this up yet, as the SCIF client doesn't have the test command exposed (oups).

### You must write a test!
You should minimally have a test that runs your main executable, and returns a status of 0. Thus, your `%apptest` section
might look just like your `%apprun` section!


```bash

%apprun main
    exec /scif/apps/main/bin/run.sh "$@"

%apptest main
    exec /scif/apps/main/bin/run.sh "$@"
```
If the SCIF client returns that you don't have a test written, it will print a particular message
that testing will detect, and fail the test. Yes, that means that you must write something!


### Write a help section
As part of testing (TBD) it is required that you write a help section, instructing the users how to use
your functions. While we cannot assess the quality of the help, the tests ensure that it exists.


### Labels
As part of testing, we assess if you have defined some essential labels. We will add them here when we know what they are :)

More to be added soon! 
