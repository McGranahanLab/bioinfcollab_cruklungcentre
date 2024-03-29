# Anatomy of Singularity's recipe

## Video tutorial is available
Video tutorial accompanies this text tutorial. It is available [here](https://disk.yandex.ru/i/B4vw60S2bBkXqQ)

## What is Singularity?
> [Singularity](https://sylabs.io/) is a program which packages (aka *containerization*) almost any software in a file and therefore makes it **portable** and **reproducible**. 

- Portability means that a file with a software can be copied to *any* computer, server, workstation, HPC, *etc* and stay the same.
- Reproducibility means that given a singularity container with a software, results of that software execution on *any* computer, server, workstation, HPC, will stay *exactly* the same. 

## Why use Singularity (especially as bioinformatician)?
1. **Reproducibility**. It's a common knowledge that sometimes results of a software execution are highly dependable on software versions (in the field of bioinformatics especially, but not exclusive). Despite that exact software versions may be noted in the methods, some users may have difficulties retrieving or installing it. Containers solve this issue effortlessly as a future user can just download a container, exactly the same one you performed your analysis with, and run it of their data. So, if done right, containers are the road to **ultimate reproducibility**.
2. **No installation hustle**. With Singularity containers there is no struggles with software installation and version management. Containers are good to go a second after a download or creation. Need to test other software version? No problem! Just switch to another container, no need to worry about compatibility of various version on a computer/server/HPC. Containers also make you less dependent on cluster support team if you work on HPC as you won't need to bother them every time you project takes a need turn and you need to use yet another software.

## What is a Singularity container?
Container is like a light, which has properties of both a wave and a particle: container can be viewed as a file, a software and a whole remote computer. 

- Like a **file**, a container can be moved from folder to folder, copied, deleted, downloaded, uploaded, encrypted, *etc*.
- Like a **software**, a container harbors inside it a code which can be used to analyse data.
- Like a **remote computer**, a container has inside it a whole operating system (OS), i.e. Ubuntu. As to a remote server, it is possible to log in *into it* and interact with the installed software *inside* it.

Hopefully, by now all the advantages of software containerization with Singularity are well outlined and it is clear that containers are useful and indispensive part of future fully reproducible and easy to manage research. The question "how do I create a container and use it?" is open now. Let's dive into it!

## What is a Singularity definition file aka recipe?
To understand what is a Singularity Definition File (or recipe for short) an analogy of a container as a computer is very useful. Imagine you got a job as IT engineer and your company just bought 100 new shiny computers which you're tasked with configuring. You need to install exactly the same OS and software on all 100 computers and there is nothing on them to begin with. As a good IT engineer you will write a code containing a set of instructions which will install OS and all the software needed across the computers. This code is in essence a Singularity recipe. 

> So, Singularity recipe is a text file (it is also piece of code) which describes the container: which exact OS is at its base, which, where and how various software were installed, where are any helpful files or data bases, etc. 

Since a recipe is just a text file, it can be put under version control like [Git](https://github.com/) allowing to track any changes. It can also be spread across the community a bit easier than a container file itself and the container created each time from this particular recipe will be exactly the same (if the recipe is written well, of course). By convention, recipe files have a `.def` extension, but you can use any extension you fancy, i.e. `txt`.

## Recipe backbone
Let's have a look at example of the simplest definition file:
```
Bootstrap: docker
From: ubuntu:20.04

%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'

```
This recipe upon execution will produce a container which has naked Ubuntu v20.04 as OS and it will print `Hello World! Hello from our custom Singularity image!")` upon launch.

In general, Singularity definition file can be divided into 2 core parts: **header** and **sections**. 

### Header Section
* **Header** part is usually composed out of 2 lines with which recipe starts. In the example above it is 
```
Bootstrap: docker
From: ubuntu:20.04
```
> Header determines the operating system for the future container, which in turn controls what will already be pre-installed in the container without need for our intervention.

To continue the logic of example with IT engineer configuring 100 computers, header would represent the first thing an IT engineer does with the computer, which is OS installation.

`Bootstrap` and `From` here are key words, like name of parameters, and `docker` and `ubuntu:20.04` are the values for those parameters. The recipe should always start from the `Bootstrap:` key word. In 99.9% of the cases line `Bootstrap: docker` will be the 
first line of your recipe. It says that the base OS (determined by `From:` line) will be pulled (downloaded) from [Docker Hub](https://hub.docker.com/). You can also put `Bootstrap: library`, and then the base OS will be pulled from [Singularity Cloud](https://sylabs.io/). In the majority of the cases, they are interchangeable. There are more various values you can give to `Bootstrap:`, you can read about them [here](https://sylabs.io/guides/3.5/user-guide/definition_files.html#header).

The `From: ` line is quite powerful. Ordinary `From: ubuntu:20.04` tells Singularity engine that you'd like to use "naked" Ubuntu v20.04 as a basis of your container, meaning that there will be nothing else, except Ubuntu v20.04 itself, installed. However, we can change value `ubuntu:20.04` to something else to allow some weight being lifted for us. For example, if this value will be `continuumio/miniconda3` it would mean that basis of our container will already have ubuntu + python 3 + conda installed and we won't need to worry about python or conda installation. Table below lists some useful values for `From:`. Unfortunately, the full list of all possible values does not exist.


| Value of `From:`           |  What is already inside                              | Note |
| :-------------------------|:----------------------------------------------------| :-------------------|
| `ubuntu:14.04`             | 'naked' Ubuntu 14.04 + Python 3.4.3 (no conda)       | Unless you're 100% sure, it's better to use more up-to-date Ubuntu version |
| `ubuntu:18.04`             | 'naked' Ubuntu 18.04, no python                      | Same as above        |
| `ubuntu:20.04`             | 'naked' Ubuntu 20.04, no python                      | **Used in 50% of times** |
| `continuumio/anaconda2`    | Debian GNU/Linux 10 + Python v2.7.16 + conda v4.7.12 |       |
| `continuumio/miniconda2`   | Debian GNU/Linux 10 + Python v2.7.16 + conda v4.7.12 | More lightweight in terms of GB version of anaconda2. Should be proffered to `continuumio/anaconda2` |
| `continuumio/anaconda3`    | Debian GNU/Linux 11 + Python 3.9.7 + conda v4.11.0   |               |
| `continuumio/miniconda3`   | Debian GNU/Linux 11 + Python 3.9.7 + conda v4.11.0   | **Used in 49% of times**. More lightweight in terms of GB version of anaconda3. Should be proffered to `continuumio/anaconda3` |
| `rstudio/r-base:4.0-focal` | Ubuntu 20.04.4 + R v.4.0.5, no python                | [Check this link](https://hub.docker.com/r/rstudio/r-base) to see how specify your favorite R version |
| `ibmjava`                  | Ubuntu 18.04.6 + Java v8.0.7.6, no python            |
| `julia:1.3`                | Debian GNU/Linux 10 + Julia v.1.3.1, no python       |       |

> Header is the only essential part of the recipe. 

A definition file which consists just out of header is valid. However, then the container is created there will be not much inside as can be deduced from the table above. You can create a test container just with a header, play with it, explore and see which programs comes in package with operating system (spoiler: not much).

### Sections
In the [example above](LINK) you may have noticed lines starting with `%` followed by commands:
```
%post
    apt-get -y update && apt-get install -y python

%runscript
    python -c 'print("Hello World! Hello from our custom Singularity image!")'
```
those are sections. 

To continue the logic of example with IT engineer configuring 100 computers, sections would represent actual commands the engineer would use to download and install software, download and configure data base and so on.

> Section(s) of a singularity recipe determines which/where/how software is installed, environmental variables, meta information, *etc*. To keep things neat, commands are divivded into sections with a specific purpose. As such, `%post` section tells which/where/how software is installed. `%environment` keeps track of environmental variables and `%labels` stores meta information (creator name and contact, *etc*) of the container. 

Each section is defined by `%` followed by name and a code black. Section names are predefined: %post, %environment, %labels, %files, %help, %runscript, %startscript, %test, %setup, %app. In this tutorial only some sections will be considered. 

> Order of sections in the recipe does not matter.

Example of a singularity recipe above is not representative. Therefore, let's switch to the other one. The recipe below is a working Singularity definition file for a widely used bioinformatics software [samtools](http://www.htslib.org/).

```
Bootstrap: docker
From: ubuntu:20.04

%post
  # STEP 1 [REQUIRED] make installation not require user interaction
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get upgrade -y 
  
  # STEP 2 [OPTIONAL] install some basic ubuntu/linux libraries
  apt-get install -y wget curl unzip zip bzip2 tabix git \
                     gfortran perl gcc g++ make cmake build-essential \
                     software-properties-common autoconf
  
  # STEP 3 [SOFTWARE - SPECIFIC] installation commands for a specific software
  apt-get install -qq -y libncurses5-dev zlib1g-dev libbz2-dev liblzma-dev
  wget https://github.com/samtools/samtools/releases/download/1.10/samtools-1.10.tar.bz2
  tar -xf samtools-1.10.tar.bz2
  cd samtools-1.10
  ./configure
  make
  make install
  cd ../
  
  # STEP 4 [JUST AN EXAMPLE] creation of folder, file and bash script
  mkdir tiny_scripts
  echo '#!/bin/bash' > tiny_scripts/say_cheese.sh
  echo 'echo CHEESE!' >> tiny_scripts/say_cheese.sh

%environment
  export PATH=$PATH:/samtools-1.10/bin:$PATH
  PI=3.14

%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1    

%help
  Main software:
  samtools        v.1.10          http://www.htslib.org/
  
  Example run:
  singularity exec samtools_in_a_box.sif samtools --help

```

#### %post
> `%post` section stores commands dedicated to software installation and file/database download and is the most important section. In essence, the absolute minimal meaningful recipe would consist of header and post section. 

`%post` section is written in `bash` and contains all the same commands used during software installation on a machine running Linux/Ubuntu. In plain Ubuntu `sudo` is often required to install a software system wide. In the `%post` section `sudo` is never used as while building a container root privileges are given by default. Let's have a closer look at the `%post` section in the example recipe:

1. In my practice I found that numerous software require some user input during installation, i.e. "Please type y if you agree for installation". During container creation, there will not be an opportunity to give any input. Therefore, all requests from the computer must be disabled and a default response to them given. This is exactly what `export DEBIAN_FRONTEND=noninteractive` command does.
```
  # STEP 1 [REQUIRED] make installation not require user interaction
  export DEBIAN_FRONTEND=noninteractive
```
2. In step 2 some basic Linux/Ubuntu packages are installed. The listed packages are my personal preferences and may not be required dependencies for software of your choice. Yet, I find that they are quite useful and are in the list of required dependencies for some software. Therefore this command with some modifications appeared in the majority of my singularity recipes. Table below describes each package function. 
```
  # STEP 2 [OPTIONAL] install some basic ubuntu/linux libraries
  apt-get install -y wget curl unzip zip bzip2 tabix git \
                     gfortran perl gcc g++ make cmake build-essential \
                     software-properties-common autoconf
```
| Package           |  Function                              |
| :-------------------------|:----------------------------------------------------|
| `wget` `curl`             | to download files from the internet |
| `unzip zip bzip2 tabix`   | to zip/unzip/compress files |
| `git`            | github |
| `gfortran perl`            | languages, fortran and perl. Python and R are not installed here! |
| `gcc g++ make cmake build-essential`            | compilers|
| `software-properties-common`            | package manager |
| `autoconf`           | automatically configure software source |


> Ideally, during the installation process a **specific version**, i.e wget=1.20.3, of each package/software/program should be installed.

In practice, _all_ software do not have specific versions of dependencies noted and therefore knowing in advance a package version could be difficult. There is a trick for such case: first, build a container without requiring specific versions, then write down versions of the installed in the container software and re-build the container with specific versions required. So, the fully reproducible version of step 2 would look like:
```
  # STEP 2 [OPTIONAL] install some basic ubuntu/linux libraries
  apt-get install -y wget curl \ # to download from internet
                     unzip zip bzip2 tabix \ # to zip/unzip/compress files
                     git \ # speaks for itself!
                     gfortran perl \ # languages, fortran and perl. Python and R are not installed here!
                     gcc g++ make cmake build-essential \ # compilers
                     software-properties-common \ # package manager
                     autoconf \ # automatically configure software source
```
**ADVICE:** Always use `-y` option after `apt-get install`, it sets a 'yes' response to any computer request.

3. Step 3 is an actual installation of software, `samtools` in the case of the example recipe. This step is completely software - specific. Usually a software does provide commands for its installation in the guide. Those commands just needed to be copied in `%post` section, omitting `sudo`. 
```
  # STEP 3 [SOFTWARE - SPECIFIC] installation commands for a specific software
  apt-get install -qq -y libncurses5-dev zlib1g-dev libbz2-dev liblzma-dev
  wget https://github.com/samtools/samtools/releases/download/1.10/samtools-1.10.tar.bz2
  tar -xf samtools-1.10.tar.bz2
  cd samtools-1.10
  ./configure
  make
  make install
  cd ../
```
4. Step 4 is not a necessary step for `samtools` installation and functioning. It is just an example that one can create directories (`tiny_scripts`) and files (`say_cheese.sh`, it's even a script!) inside the container. 
```
  # STEP 4 [JUST AN EXAMPLE] creation of folder, file and bash script
  mkdir tiny_scripts
  echo '#!/bin/bash' > tiny_scripts/say_cheese.sh
  echo 'echo CHEESE!' >> tiny_scripts/say_cheese.sh
```

#### %environment
> `%environment` section defines environmental variables in your container.

In case you're not familiar with the environmental variables here is a little example. It is very nice that you can just open your terminal, navigate to any folder, and then type `samtools` and it will work, isn't it? But how does your system knows where to look for `samtools` executable? Though environmental variable of course! Usually, on _actual computer_ they are defined in `.bashrc` file. This file is one of the first files your computer reads during booting and therefore each time you switch your computer on it knows where `samtools` is. So, the common format for the environmental variable to define a path to binary executable is: `export PATH=/where/to/install/bin:$PATH`. Obviously, environmental variables may not only define path to a certain software (or file), they can also define a variable, i.e. `PI=3.14`.
In container, they are defined in environment section. 

```
%environment
  # example of environmental variable as path to samtools executable
  export PATH=$PATH:/samtools-1.10/bin:$PATH
  # example of environmental variable defining a global variable
  PI=3.14
```

**Important note:** in container all environmental variables are accessible only **after** container is created, but **not during its creation** (*not* during execution of `%post` section). So, if `samtools` or `PI` variable are needed during container creation, then they have to be defined `%post` section as well. For example:
```
%post
    export PATH=/where/to/install/bin:$PATH
    PI=3.14
```

#### %labels
> %labels section provides a meta data for the container. 

Conventionally, one puts there a general information about the container, such as who have created it, organization, email to contact, `etc` to `%labels` section. It's a free form section, there is no restrictions. The general format is a name-value pair. 
```
%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1
```

#### %help
> %help section contains information which helps a user to interact with the container, i.e. how to run it.

This section is also a  free text and usually provides information about main and support software installed in this container as well as gives example execution command.

```
%help
  Main software:
  samtools        v.1.10          http://www.htslib.org/
  
  Example run:
  singularity exec samtools_in_a_box.sif samtools --help
```

#### %files
> %files section is used to copy files from the host computer (where a container is build) inside the container. 

**This section can only be used if you build container on your computer. Can not be used during online build on Singularity cloud**. The general convention is that each line of `%files` is a source and destination pair. Source has to be a valid path on host computer and destination is a path inside container. However, this section significantly reduces reproducibility of a container.
```
 %files
/usr/bin/Desctop/my_test_file /olala
```

This list of sections is not all inclusive. For the full list, please check with [official documentation](https://sylabs.io/guides/3.5/user-guide/definition_files.html#sections).

Example above was dedicated to show how containerization of Linux/Ubuntu command line tool is performed. Yet, there is a lot of bioinformatics software which comes as `R` or `python` packages. More about their containerization in the sections below.

## Containerization of python and conda packages
Due to the existence of `continuumio/miniconda3` header creating "pure python" containers is actually one of the easiest tasks. `continuumio/miniconda3` assures that `python3`, `conda` and `pip` are already present in in container and there is no need to do anything to install them. Quite nice! In comparison the usua python packages installation, there is only one difference: `conda` and `pip` are not accessible in the root folder, because environmental variables were not set up. So in order to be able just type `conda` and `pip` as usual an environmental variables need to be set up for them. In the demo recipe below it is done with following command: `export PATH=/opt/conda/bin/:$PATH`. It covers both `pip` and `conda`.

Here is an example of numpy containerized:
```
Bootstrap: docker
From: continuumio/miniconda3

%post
    # STEP 1: environmental variables for conda and pip
    export PATH=/opt/conda/bin/:$PATH
    
    # STEP 2: install python version you need (this is optional, but improves reproducibility)
    conda install python=3.8
    
    # STEP 3: install python packages with conda.
    conda install --channel conda-forge --channel bioconda -y \
                  numpy=1.21.5
%environment
    export PATH=/opt/conda/bin/:$PATH
    
%help
    Main software:
    numpy v.1.21.5  https://anaconda.org/anaconda/numpy
    
    Example run:
    # create script test.py which loads numpy and displys version
    echo 'import numpy' > test.py
    echo 'version = numpy.__version__' >> test.py
    echo 'print(version)' >> test.py
    # run it
    singularity exec numpy.sif python3 test.py

%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1
```
Please note that for the enhanced reproducibility a certain version of numpy (1.21.5) was requested. 

Some python packages recommend creation of python environment for installation. With containers it's not necessary. Usually python environments are created to isolate different, sometimes incompatible, packages from each other. If a container will have installed only python packages compatible with each other, there will be no need to isolate them from one another. Yet, if some packages are not compatible, my recommendation is to create two separate containers for them.

The example above is the recommended way to create a container with python packages installed. However, if `continuumio/miniconda3` header can not provide with a needed functionality, and one chooses to use `ubuntu:20.04` instead, here is how `conda` can be installed into container from scratch:

```
Bootstrap:docker
From:ubuntu:20.04

%post
  # STEP 1 [REQUIRED] make installation not require user interaction
  export DEBIAN_FRONTEND=noninteractive
  
  apt-get update
  apt-get upgrade -y 
  
  # STEP 2 [OPTIONAL] install some basic ubuntu/linux libraries
  apt-get install -y wget curl unzip zip bzip2 tabix git \
                     gfortran perl gcc g++ make cmake build-essential \
                     software-properties-common autoconf
  
  # STEP 3: download miniconda v3.4 (check out https://repo.continuum.io/miniconda/ for other versions)
  cd /opt
  rm -fr miniconda
  wget https://repo.continuum.io/miniconda/Miniconda3-4.7.12-Linux-x86_64.sh -O miniconda.sh
  
  # STEP 4: install conda
  bash miniconda.sh -b -p /opt/miniconda
  export PATH="/opt/miniconda/bin:$PATH"
  
  # now conda is available and can be used, i.e.
  conda install --channel conda-forge --channel bioconda -y numpy=1.21.5

%environment
  export PATH="/opt/miniconda/bin:$PATH"
  
%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1  
  
%help
  A test container to show installation of conda on Ubuntu 20.04 from scratch
  
  singularity exec conda_from_scratch.sif conda
```

## Containerization of R libraries
For creation of containers with R libraries there is also a trick with header `rstudio/r-base:4.0-focal` which simplifies the build due to pre-installed R. In you want another version of R, scroll to section above and choose proper value to give to `From: `. In general, to install `R` libraries inside the container, the same `install.packages` command is used. However, it is prefixed with `Rscript -e` (see example below). Also, `repos` argument of `install.packages` is used to indicated CRAN mirror. An example for bioconductor packages is also given below.

```
Bootstrap: docker
From: rstudio/r-base:4.0-focal

%post 
  # installation of data.table package from CRAN
  Rscript -e "install.packages(c('data.table'), repos = 'http://cran.us.r-project.org', quietly = TRUE)"
  
  # installation of biocondactor & limma package from bioconductor 
  Rscript -e "install.packages(c('BiocManager'), repos = 'http://cran.us.r-project.org', quietly = TRUE)"
  Rscript -e "BiocManager::install(c('limma'), ask = FALSE, update = FALSE)"

%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1  
                                         
%help
  A test container showing how to install R packages
  echo 'library(data.table)' > test.R
  echo 'library(limma)' >> test.R
  singularity exec limma.sif Rscript --vanilla test.R 
```

The example above is the recommended way to create a container with `R` libraries installed. However, if `rstudio/r-base:4.0-focal` header can not provide with a needed functionality, and one chooses to use `ubuntu:20.04` instead, here is how `R` can be installed into container from scratch:

```
Bootstrap: docker
From: ubuntu:20.04

%post
  # STEP 1 [REQUIRED] make installation not require user interaction
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get upgrade -y 
  
  # STEP 2 [REQUIRED] install some basic ubuntu/linux libraries
  apt-get install -y wget curl unzip zip bzip2 tabix git \
                     gfortran perl gcc g++ make cmake build-essential \
                     software-properties-common autoconf
  
  # STEP 3 install R
  apt-get -qq -y install libncurses5-dev zlib1g-dev libbz2-dev gnupg2 \
                         libcurl4-openssl-dev libssl-dev libxml2-dev
  apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9
  add-apt-repository 'deb https://cloud.r-project.org/bin/linux/ubuntu bionic-cran40/'
  apt-get -qq -y dist-upgrade
  apt-get -qq -y install r-base r-base-core r-recommended
        
%labels
  CREATOR     Maria Litovchenko
  ORGANIZATION    TEST_WORKING_PLACE       
  EMAIL
  VERSION v0.0.1  
                                         
%help
  A test container showing how to install R from scratch

```

## Compound containers: command line software + R libraries + python packages
If you'd like to create a frankenstein Singularity container which will contain bash command line software, R libraries and python packages, the best way to go is to use `continuumio/miniconda3` header rather than `ubuntu:20.04`. This is because `continuumio/miniconda3` will already provide you with Linux and `conda`, and `R` can be installed via `conda`: `conda install -c conda-forge r-base`. Also, a lot of command line tools, i.e. `bwa` aligner [can also be installed thought `conda`](https://anaconda.org/bioconda/bwa)! Always check out [https://anaconda.org/](https://anaconda.org) for availability of your command line tools before you start container design! It can greatly simplify container design and take care of numerous dependencies!

Now, then we have Singularity definition file written, we can finally build our container. 

## Building container offline
*It is implied that singularity is installed on your computer/server and you have root privileges. For MAC OS installation guide, check our this link, and for Ubuntu - this one. If you have a Windows machine, you can build you container in the cloud or install a virtual machine with Ubuntu and Singularity.*

Building container on computer is extremely easy! It's essentially a one-liner:
```
sudo singularity build name_of_the_container.sif definition_file_name.def 
```
In case of the container - example from above for `samtools`: 
```
sudo singularity build samtools_in_a_box.sif samtools.def 
```
Once the command is executed, a file `samtools_in_a_box.sif` will appear. It is a container fully ready to use!

## Building container online on Singularity cloud.
Singularity provides us with opportunity to build containers in the cloud, which takes away hustle of Singularity installation on personal computer. [Please follow this tutorial to build your container in the cloud]().

## Using containers
_This section implies that you have singularity installed on your computer/server. For MAC OS installation guide, check our this link, and for Ubuntu - this one. If you have a Windows machine, you're on your own._

If the container was build in the cloud, pull it using corresponding command:
```
singularity pull --arch amd64 library://marialitovchenko/marialitovchenko/samtools_in_a_box:latest
```
If the container was build offline, you already have it!

Once your container is created/pulled/downloaded, you can interact with it in several modes. 
1. **Checking integrity.** As described above a container is something like remote server in the shell of a sif file. If it's like server, we can log in _inside_ it and check installation of the software. To get inside the container, use _shell_ command to tell _singularity_ to let you in:
```
singularity shell samtools_in_a_box_latest.sif
```
This will lead to a change in your terminal to:
```
Singularity>
```
Congratulations! You are inside the container! Command:
```
ls -l /
```
will display all the files and folders inside the container:
```
total 4615
lrwxrwxrwx   1 root    root          7 Apr 26 19:49 bin -> usr/bin
drwxr-xr-x   2 root    root          3 Apr 15  2020 boot
drwxr-xr-x  18 root    root       3800 May 11 16:51 dev
lrwxrwxrwx   1 root    root         36 May 11 16:13 environment -> .singularity.d/env/90-environment.sh
drwxr-xr-x  53 root    root       1819 May 11 16:14 etc
drwxr-xr-x   1 vagrant vagrant      60 May 11 16:59 home
lrwxrwxrwx   1 root    root          7 Apr 26 19:49 lib -> usr/lib
lrwxrwxrwx   1 root    root          9 Apr 26 19:49 lib32 -> usr/lib32
lrwxrwxrwx   1 root    root          9 Apr 26 19:49 lib64 -> usr/lib64
lrwxrwxrwx   1 root    root         10 Apr 26 19:49 libx32 -> usr/libx32
drwxr-xr-x   2 root    root          3 Apr 26 19:49 media
drwxr-xr-x   2 root    root          3 Apr 26 19:49 mnt
drwxr-xr-x   2 root    root          3 Apr 26 19:49 opt
dr-xr-xr-x 102 root    root          0 May 11 16:52 proc
drwx------   2 root    root         64 May 11 16:14 root
drwxr-xr-x   8 root    root        136 May 11 16:13 run
drwxrwxr-x   9 root    root       2371 May 11 16:15 samtools-1.10
-rw-rw-r--   1 root    root    4721173 Dec  7 22:42 samtools-1.10.tar.bz2
lrwxrwxrwx   1 root    root          8 Apr 26 19:49 sbin -> usr/sbin
lrwxrwxrwx   1 root    root         24 May 11 16:13 singularity -> .singularity.d/runscript
drwxr-xr-x   2 root    root          3 Apr 26 19:49 srv
dr-xr-xr-x  13 root    root          0 May 11 16:51 sys
drwxrwxr-x   2 root    root         36 May 11 16:15 tiny_scripts
drwxrwxrwt   8 root    root       4096 May 11 16:59 tmp
drwxr-xr-x  14 root    root        217 May 11 16:14 usr
drwxr-xr-x  11 root    root        160 Apr 26 19:52 var
```

You can even see the folder `tiny_scripts` which we created in the recipe above!
```
ls -l /tiny_scripts/
```

Now, if just `ls -l` command is executed (without the slash!), you'll see files which are located in the same folder as your container (and even container itself):
```
total 238104
drwxr-xr-x 3 root    root         4096 Nov 16 21:31 go
-rw-rw-r-- 1 vagrant vagrant         0 May 11 17:04 samtools.def
-rwxrwxr-x 1 vagrant vagrant 243810304 May 11 16:56 samtools_in_a_box_latest.sif
```
This shows that container has an access not only to the files which are located _inside_ it, but also to the ones located _outside_ it too.

Now, you can check on software installation and in general interact with the terminal in the same way as you would just as in usual terminal. To check `samtools` installation, run:
```
samtools --help
```
Result should be:
```
Program: samtools (Tools for alignments in the SAM format)
Version: 1.10 (using htslib 1.10)

Usage:   samtools <command> [options]

Commands:
  -- Indexing
     dict           create a sequence dictionary file
     faidx          index/extract FASTA
     fqidx          index/extract FASTQ
     index          index alignment
...
```

To log out of the container, type
```
exit
```

2. **Executing software from container.** Singularity makes it very easy to run the software from the container. Essentially, all the commands used in terminal to run the software will stay the same. The only change one needs to make is to put `singularity exec your_container.sif` in front of the command. For example, to execute `samtools` which is installed in our test container, the following command is used:
```
singularity exec samtools_in_a_box_latest.sif samtools --help
```
Following message should appear on your screen proving that `samtools` worked:
```
Program: samtools (Tools for alignments in the SAM format)
Version: 1.10 (using htslib 1.10)

Usage:   samtools <command> [options]

Commands:
  -- Indexing
     dict           create a sequence dictionary file
     faidx          index/extract FASTA
     fqidx          index/extract FASTQ
     index          index alignment
...
```
Basically, you can put any bash command after `singularity exec samtools_in_a_box_latest.sif` and it will be executed inside the container. For example, to list files _inside_ the container:
```
singularity exec samtools_in_a_box_latest.sif ls -l /
```
2a. **Executing software from container if it is not globally accessible**. In the example above `samtools` is globally accessible meaning that regardless of the folder _inside_ the container, if a `samtools` command is called, `samtools` will be executed. However, it can happen that a software you (or someone else) put inside the container was not made globally accessible. In such case, a full path to software _inside_ the container should be given after `samtools_in_a_box_latest.sif `. During creation of `samtools_in_a_box_latest.sif` a `say_cheese.sh` script was created in folder _tiny_scripts_. In order to execute it:
```
singularity exec samtools_in_a_box_latest.sif bash /tiny_scripts/say_cheese.sh
```
Result should be:
```
CHEESE!
```
The **slash** in front of the `tiny_scripts/say_cheese.sh` is **imperative** as it indicates the root directory and that file to be executed is _inside_ the container.

2b. **Binding**. All commands above did not require files from your computer to serve as an input, which of course will not be the case then you'll use the container for data analysis. There is a small detail in the usage of singularity containers associated with location of your input files. Let's create a small sam file to serve as test one:
```
wget https://raw.githubusercontent.com/samtools/samtools/develop/examples/toy.sam
```
Now, let's use our container with `samtools` to convert that sam to bam format:
```
singularity exec samtools_in_a_box_latest.sif samtools view -bS toy.sam > small_test.bam
```
If you check now your computer with `ls -l`, you'll see that `small_test.bam` was created. Container worked! Hooray! However, it is unlikely that all of your input files will be located in the same directory as a container. So let's move our `toy.sam` to the directory just outside directory which contains our container:
```
# create the directory
mkdir ../sam_files_dir/
# move
mv toy.sam ../sam_files_dir/
```
and let's try to execute the same command again:
```
singularity exec samtools_in_a_box_latest.sif samtools view -bS /sam_files_dir/toy.sam > small_test_1.bam
```
Error occurred:
```
FATAL:   could not open image /home/vagrant/sam_files_dir/olala/samtools_in_a_box_latest.sif: failed to retrieve path for /home/vagrant/sam_files_dir/olala/samtools_in_a_box_latest.sif: lstat /home/vagrant/sam_files_dir/samtools_in_a_box_latest.sif: no such file or directory.
```
Despite that a full path to `/sam_files_dir/toy.sam` was given! The error message informs that file `small_test.sam` does not exist! Of course, in reality it does, just the container doesn't "see" it meaning that it can't detect it. This problem is solved with _binding_. Singularity allows you to map directories on your host system to directories within your container using _bind_ mounts. Binding is very similar with mounting directories of the remove server to your own computer. Binding is performed via `--bind` option given to singularity followed by a full path to the **folder** on your computer you'd link to bind and full path to the folder in the container you'd like the original folder be bound to separated by ":". For example: `singularity --bind path_on_your_computer:path_to_be_used_inside_the_container`. The folder inside the container you'd like your folder on the computer to be bound to does not have to exist. In fact, I find it **the most convenient to use exactly the same full path to the folder on computer as the binding path inside the container**. This will ensure that all your code which uses absolute paths to the input files will runs smoothly. For example, the container with samtools we've created above is located in _containers_ folder, and small_test.sam is located in _sam_files_dir_. Both _containers_ folder and _sam_files_dir_ are subfolders of _singularity_showcase_ folder. Let's use it for binding. I  general, it's a good idea to use as bind path a directory which is parent to both directory containing your container (sif file) and your input files.
``` 
singularity —bind /singularity_tutorial/software/:/singularity_tutorial/software/ exec samtools_in_a_box_latest.sif samtools view -bS /singularity_showcase/sam_files_dir/small_test.sam > small_test_1.bam
```
Worked!

## Tips, tricks 
How to see recipe of already build container?
singularity run-help my_container.sif
External files to download into package: dropbox, google drive 
Library of recipies

## Sources:
https://sylabs.io/guides/3.5/user-guide/definition_files.html
