# Exercise 1

```
$ singularity build my_image.sif shub://vsoch/hello-world
$ singularity shell my_image.sif
$ singularity run my_image.sif
```

# Exercise 2

```
$ sudo singularity build my_image.sif docker://ubuntu:latest
$ sudo singularity build --sandbox my_sandbox my_img.sif
$ sudo singularity shell -w my_sandbox
Singularity my_sandbox:~> sudo apt-get update
Singularity my_sandbox:~> sudo apt-get install build-essential
```

# Exercise 3

```
$ sudo singularity shell -w my_sandbox
Singularity my_sandbox:~> echo "This is my help file" > /.singularity.d/runscript.help
Singularity my_sandbox:~> exit
$ sudo singularity build my_new_image.sif my_sandbox
$ singularity run-help my_new_image.sif
```

# Exercise 4

```
#!/bin/bash -l
#SBATCH -J myjob
#SBATCH -A <Allocation ID>
#SBATCH --reservation=<Reservation ID>
#SBATCH -t 1:00
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH -o output_file.o
module add gcc/6.2.0 openmpi/3.0-gcc-6.2 singularity
mpirun -n 8 singularity exec -B /cfs/klemming $PDC_SHUB/hello_world.sif hello_world_mpi
```

# Exercise 5

```
# Header
Bootstrap: docker
From: ubuntu:latest
# Sections
%help
  Help me. I'm in the container.
%runscript
  echo "This is my runscript"
%post
  apt-get -y update
  apt-get install -y build-essential
```
