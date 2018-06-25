# Scientific Filesystem Container Builder

[![scif](https://img.shields.io/badge/filesystem-scientific-green.svg?style=for-the-badge)](https://sci-f.github.io)
[![CircleCI](https://circleci.com/gh/vsoch/example.scif.svg?style=svg)](https://circleci.com/gh/vsoch/example.scif)

![docs/assets/img/circle.png](docs/assets/img/circle.png)

A proof of concept of a RNA-Seq pipeline. Here we combine three technologies to handle each of the following:

 - **Reproducibility** of software is handled by container technology
 - **Discoverability** and **Transparency** are handled by installing our software in the container via a [Scientific Filesystem](https://sci-f.github.io)[[cite](https://academic.oup.com/gigascience/article/7/5/giy023/4931737#116684246)]. SCIF also lets us install the same dependencies across container technologies.
 - **Testing** is handled by continuous integration, which understands how to interact with a scientific filesystem.

Each of these components plays a slightly different and equally important role. Without SCIF, we could have the same container with commands to execute known software inside, but the container would largely remain a black box with software mixed amongst the base operating system. Without container technologies, you could install software on your host, but (as we all know) this would likely not be a portable solution. Without testing, we couldn't be sure that our software works as we intended, and is ready to plug into some pipeline tool. Check out the [documentation](https://vsoch.github.io/example.scif) to get started to clone the template, write your container recipe, and then build --> test --> deploy!

If you have any questions or issues, please don't hesitate to [reach out](https://www.github.com/vsoch/example.scif/issues).
