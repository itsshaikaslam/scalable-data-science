# Spark Programs

This `dockerCompose/programs` directory is read-writable from inside the containers.

This can be used to keep program files that are:

- Scala projects built with `sbt` (with/out IDE from outside container) or 
- notebooks written from zeppelin or jupyter notebook servers inside container.

This is all one needs to be able to integrate your favourite Scala IDE to build packages from your platform inside the container. 

## 1. Setup your system for `sbt`

- [dockerCompose/readmes/toolsSetup.md](https://github.com/lamastex/scalable-data-science/blob/master/_sds/basics/infrastructure/onpremise/dockerCompose/readmes/toolsSetups.md) and 

We will mostly be using `sbt` from within the container/service.

## 2. A Quick `sbt` Tutorial

- [dockerCompose/readmes/sbt_tutorial.md](https://github.com/lamastex/scalable-data-science/blob/master/_sds/basics/infrastructure/onpremise/dockerCompose/readmes/sbt_tutorial.md) 

We will mostly work from notebooks to focus on concepts and take advantage of interactive visualisations in notebook REPLs.

Engineers are welcome to work on the corresponding local development using the instaructions above.

## 3. Go trough the exercises

- [dockerCompose/programs/exercises/tutorialSbtScalaSpark/README.md](https://github.com/lamastex/scalable-data-science/tree/master/_sds/basics/infrastructure/onpremise/dockerCompose/programs/exercises/tutorialSbtScalaSpark)
- [dockerCompose/programs/exercises/sparkSubmit/README.md](https://github.com/lamastex/scalable-data-science/tree/master/_sds/basics/infrastructure/onpremise/dockerCompose/programs/exercises/sparkSubmit)
