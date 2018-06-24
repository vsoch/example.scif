# Scientific Filesystem Container Builder

[![scif](https://img.shields.io/badge/filesystem-scientific-green.svg?style=for-the-badge)](https://sci-f.github.io)
[![CircleCI](https://circleci.com/gh/vsoch/example.scif.svg?style=svg)](https://circleci.com/gh/vsoch/example.scif)

A proof of concept of a RNA-Seq pipeline. Here we combine three technologies to handle each of the following:

 - **Reproducibility** of software is handled by container technology
 - **Discoverability** and **Transparency** are handled by installing our software in the container via a [Scientific Filesystem](https://sci-f.github.io)[[cite](https://academic.oup.com/gigascience/article/7/5/giy023/4931737#116684246)]. SCIF also lets us install the same dependencies across container technologies.
 - **Testing** is handled by continuous integration, which understands how to interact with a scientific filesystem.

Each of these components plays a slightly different and equally important role. Without SCIF, we could have the same container with commands to execute known software inside, but the container would largely remain a black box with software mixed amongst the base operating system. Without container technologies, you could install software on your host, but (as we all know) this would likely not be a portable solution. Without testing, we couldn't be sure that our software works as we intended, and is ready to plug into some pipeline tool. To get started, follow each of the links below to learn how to generate your own his will be consolidated into a tutorial, and for now follow the links below

 - [1. Clone the Repository](docs/setup.md): The first step is to clone this repository to your Github account.
 - [2. Write Your Recipe](docs/recipes.md): The container build and testing is driven by defining your Scientific Filesystem in a [recipe.scif](recipe.scif)
 - [3. Build a Container](docs/bulid.md): (optional) you will likely want to build a container, either to develop or run locally.
 - [4. Testing Criteria](docs/testing.md): Once you push to Github, you will need to connect to a Continuous Integration service ([CircleCI](https://circleci.com/gh/vsoch/example.scif/) is used for this repository.

After you write your recipe, making sure to add your scripts and other dependencies to this repository, connect it to test on Circle, and the tests pass, the container will be deployed for others to use.
