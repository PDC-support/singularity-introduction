---?image=img/first_slide.png

@snap[north-west]
<h2>How to use singularity in a HPC environment</h2>
<b>Author:</b> Henric Zazzi, PDC Center for High Performance Computing
@snapend

---

## Overview

- What are containers
- Docker, the most popular container
- Singularity: Containers for the HPC environment
- installation of singularity
- Using the container
- How to build containers
- Running your container in an HPC environment
- Creating recipes for singularity

---

## What are containers

![borderless](img/container.png)
---

@snap[north-west]
A container is standalone executable file with everything needed to run application(s)
@snapend

@snap[south-west size40 border-image]
Virtual Machine<br>
![](img/container-vm-whatcontainer.png)
@snapend

@snap[south-east size40 border-image align-left]
Container<br>
![](img/container-what-is-container.png)
@snapend

---

## Containers: How are they useful

- Reproducibility
- Portability
- Depending on application and use-case, simple extreme scalability
- Next logical progession from virtual machines

---

## Why do we want containers in HPC?

- Escape “dependency hell”
- Load fewer modules
- Local and remote code works identically every time
- One file contains everything and can be moved anywhere

---

## Docker, the most popular container

![](img/docker.png)

---

## The Docker container software

- The most know and utilized container software
- Facilites workflow for creating, maintaining and distributing software
- Easy to install, well documented, standardized
- Used by many scientist

---

## Docker on HPC: The problem

- Incompabilities with scheduling managers (SLURM...)
- No support for MPI
- No native GPU support
- Docker users can escalate to root access on the cluster
- @color[red](Not allowed on HPC clusters)

---

## Singularity: Containers for the HPC environment

- Package software and dependencies in one file
- Use same container in different SNIC clusters
- Limits user’s	privileges,	better security
- Same user inside container as on host
- No need for most modules
- **Negligable performance decrease**

---

## But I want to keep using docker

- Works great for local and private resources.
- No HPC centra will install docker for you
- **Singularity can import Docker images**

---

## Singularity hub

https://singularity-hub.org/
@snap[with-border]
![](img/hub.png)
@snapend

---

## Singularity Versions

- Latest version: 3.2.1-1 (2019-06-18)
- Installed on Tegner: 3.2.1-1
- Installed on VM: 3.2.1-1

---

@snap[north-west]
<h2>Singularity workflow</h2>
@snapend

@snap[west with-border]
**Local computer**<br>
**(ROOT)**<br>
Create container<br>
@css[lightgreen](singularity build)<br>
Install software<br>
Install libraries<br>
@snapend

@snap[kthblue]
@fa[arrow-right fa-4x]
@snapend

@snap[east align-left with-border]
**HPC cluster**<br>
**(USER)**<br>
@css[lightgreen](singularity shell)<br>
@css[lightgreen](singularity exec)<br>
@css[lightgreen](singularity help)<br>
@css[lightgreen](singularity run)<br>
@snapend

---

## Install singularity in Linux

* Install dependencies
  ```
  build-essential libssl-dev uuid-dev libgpgme11-dev
  squashfs-tools libseccomp-dev wget pkg-config git
  ```
* Install go (version 1.12.6, 2019-06-19)
* Install singularity (version 3.2.1-1, 2019-06-18)
* Follow instructions at https://www.sylabs.io/guides/3.2/user-guide/installation.html
* @color[red](Singularity cannot be installed on Mac or Windows, but can be used via VMs)

---

## Launching a container

* Singularity sets up the container environment and creates the necessary
  namespaces.
* Directories, files and other resources are shared from the host into the
  container.
* All expected I/O is passed through the container: pipes, program arguments,
  std, X11
* When the application(s) finish their foreground execution process, the
  container and namespaces collapse and vanish cleanly

---

## Using the container

---

## Download and test an image

Download and test the latest UBUNTU image from docker hub

```
$ sudo singularity build my_image.sif docker://ubuntu:latest
INFO:    Starting build...
Getting image source signatures
...
Writing manifest to image destination
Storing signatures
INFO:    Creating SIF file...
INFO:    Build complete: my_image.sif 
$ singularity shell my_image.sif
Singularity my_image.sif:~> cat /etc/*-release
Singularity my_image.sif:~> exit
```

@snap[align-right]
@color[red](Do it yourself:)
@snapend

---

## @color[red](Exercise 1: Download a container)

@ol[](false)
- Go to singularity hub and find the hello-world container (https://singularity-hub.org/collections)
- build the container using singularity
  (shub://[full name of container])
- Use the container shell and get acquainted with it 
@olend

---

## How to build containers

---

## Why must I be root?

Same permissions in the container as outside...

To be root in the singularity image you must be root on the computer

---

## Build a writeable image

Since there are memory limitation on writing directly to image file,
it is better to create a sandbox

```
$ sudo singularity build --sandbox my_sandbox my_image.sif
INFO:    Starting build...
INFO:    Creating sandbox directory...
INFO:    Build complete: my_sandbox 
$ sudo singularity shell -w my_sandbox
Singularity: Invoking an interactive shell within container...
Singularity my_sandbox:~>
```

---

## How do I execute commands in singularity

Commands in the container can be given as normal.

```
singularity exec my_image.sif ls
```
```
$ singularity shell my_image.sif
Singularity: Invoking an interactive shell within container...
Singularity my_image.sif:~> ls
```

---

## Transfer files into container

**Read mode:** You can read/write to file system outside container and
read inside container.

**write mode:** You can read/write inside container.

@color[darkgreen](**Remember:** In write mode you are user ROOT, home folder: /root)

---

## How to transfer files into the container

my_folder is a local folder on your computer

```
$ sudo singularity exec -w my_sandbox mkdir singularity_folder
$ sudo singularity shell -B my_folder:/root/singularity_folder -w my_sandbox
Singularity my_sandbox:~> cp singularity_folder/file1 .
```

---

## How to execute scripts outside image

* All the dependencies in image
* Script outside image

```
$ singularity exec -B my_folder:/root/singularity_folder my_sandbox singularity_folder/script
```

@snap[align-right]
@color[red](Do it yourself:)
@snapend

---

## @color[red](Exercise 2: Create your own container)

@ul[](false)
- Go to docker hub and find the official latest ubuntu
- build the container using singularity
- Build a writeable sandbox
- Install necessary tools into the container (Compiler etc...)
  - apt-get update
  - apt-get install build-essential
@ulend

---

## singularity.d folder

Startup scripts etc... for your singularity image

```
$ singularity exec my_image.sif ls -l /.singularity.d
total 7
-rw-r--r-- 1 root root   39 Jun 18 11:26 Singularity
drwxr-xr-x 2 root root 4096 Jun 18 11:22 actions
drwxr-xr-x 2 root root  173 Jun 18 11:26 env
-rw-r--r-- 1 root root  309 Jun 18 11:26 labels.json
drwxr-xr-x 2 root root    3 Jun 18 11:26 libs
-rwxr-xr-x 1 root root 1084 Jun 18 11:26 runscript
-rwxr-xr-x 1 root root   19 Jun 18 11:26 runscript.help
-rwxr-xr-x 1 root root   10 Jun 18 11:26 startscript 
```

@color[darkgreen](**Important:** Scripts must be executable and owned by root)

---

## Creating a script

@snap[align-left]
runscript
@snapend

```
#!/bin/sh

ls -l
```
@snap[align-left]
command
@snapend
```
$ singularity run my_image.sif
total 1
drwxr-xr-x 2 root root  76 Sep 11 17:05 file1
drwxr-xr-x 2 root root 139 Sep 11 17:23 file2
```

---

## What is a help file and how is it used

@snap[align-left]
runscript.help
@snapend

```
This is a text file
```
@snap[align-left]
command
@snapend
```
$ singularity run-help my_image.sif
This is a text file
```

---

## Build a new container from a sandbox

```
$ sudo singularity build my_new_image.sif my_sandbox
INFO:    Starting build...
INFO:    Creating SIF file...
INFO:    Build complete: my_new_image.sif 
```

@snap[align-right]
@color[red](Do it yourself:)
@snapend

---

## @color[red](Exercise 3: Edit your own container)

@ol[](false)
- Create a help file
- Create/Edit the runscript running hello world
- Create a new container from the sandbox
@olend

@color[darkgreen](**Tip:** You can use the editor in your VM and then transfer the file)

---

## Running your container in an HPC environment

---

## Requirements

- OpenMPI version must be the same in container and cluster
- You need to bind to the LUSTRE file system at PDC so it can be detected

---

## What are the required tools

wget, build-essential, lzip, m4, libgfortran3, gmp, mpfr, mpc,
zlib, gcc, openmpi, cmake, python, cuda

- On AFS
  - /afs/pdc.kth.se/pdc/vol/singularity/3.2.1/shub.backup
- On Lustre
  - /cfs/klemming/pdc.vol.tegner/singularity/3.2.1/shub
- **Image:** ubuntu-18.04.2-gnu-basic.sif
- https://www.pdc.kth.se/software

---

## Send in a singularity batch job and execute

```
#!/bin/bash -l
#SBATCH -J myjob
#SBATCH -A <Allocation ID>
#SBATCH --reservation=<Reservation ID>
#SBATCH -t 1:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH -o output_file.o
module add gcc/8.2.0 openmpi/4.0-gcc-8.2 singularity/3.2.1
mpirun -n 8 singularity exec -B /cfs/klemming hello_world.sif hello_world_mpi
```

@snap[align-right]
@color[red](Do it yourself:)
@snapend

---

## @color[red](Exercise 4: Run a HPC container)

@ol[](false)
- Login into tegner.pdc.kth.se
- send in a job for the hello-world image
  - Use the hello_world image on PDCs singularity repository
@olend

@color[darkgreen](**Tip:** With the singularity module use the **Path:** $PDC_SHUB)

---

## How about GPUs?

**Flag:** --nv

Finds the relevant nVidia/CUDA libraries on your host.
 
```
salloc -t <time> -A <Allocation ID> --gres=gpu:K420:1
srun -N 1 singularity exec --nv -B /cfs/klemming cuda.sif cuda_device
Device Number: 0
  Device name: Quadro K420
  Memory Clock Rate (KHz): 891000
  Memory Bus Width (bits): 128
  Peak Memory Bandwidth (GB/s): 28.512000
```

---

## Creating recipes for singularity

---

## Singularity Recipes

A Singularity Recipe is the driver of a custom build, and the starting point
for designing any custom container. It includes specifics about installation
software, environment variables, files to add, and container metadata.

Singularity recipes is also a good practice for keeping your image up-to-date

---

## How to build from a recipe

A recipe is a textfile explaining what should be put into the container

```
$ sudo singularity build my_image.sif my_recipe.def
```

---

## Recipe format

```
# Header
Bootstrap: docker
From: ubuntu:latest
# Sections
%help
  Help me. I'm in the container.
%files
    mydata.txt /home
%post
    apt-get -y update
    apt-get install -y build-essential
%runscript
    echo "This is my runscript"
```

---

## Header

What image should we start with?

- *Bootstrap:*
  - shub
  - docker
  - localimage
- *From:*
  - The name of the container
```
# Header
Bootstrap: docker
From: ubuntu:latest
```

---

## Section: %help

Some information about your container.
Valuable to put information about what software and versions
are available in the container

```
%help
  This container is based on UBUNTU 16.04. GCC v6.2 installed
```

---

## Section: %post

What softwares should be installed in my container.

```
%post
    apt-get -y update
    apt-get install -y build-essential
```

@snap[align-left]
@color[darkgreen](No interaction in the scripts)<br>
@color[darkgreen](We do not need sudo in the container)
@snapend

---

## Section: %files

What local files should be copied into my container

```
%files
    # <filename> <singularity path>
    myfile.txt /opt
```

---

## Section: %runscript

What should be executed with the run command.

```
%runscript
    mysoftware -param1 -param2
    
```

@snap[align-right]
@color[red](Do it yourself:)
@snapend

---

## How to inspect an image

How to examine an image to see what recipy has created it

```
$ singularity inspect --deffile my_image.sif
```

---

## @color[red](Exercise 5: Create a recipe)

@ol[](false)
- Based on UBUNTU
- Install compilers
- Create a help text
- Create a runscript
- Run the recipe
@olend

@color[darkgreen](**Tip:** You can use the editor in your VM and then transfer the file)

---

## Useful links

- https://www.pdc.kth.se/software/software/singularity/index_general.html
- https://www.sylabs.io/guides/3.2/user-guide/
- https://gitpitch.com/PDC-support/singularity-introduction#/
- https://github.com/PDC-support/singularity-recipes

Answers to Exercises
- https://github.com/PDC-support/singularity-introduction/blob/master/facit.md
