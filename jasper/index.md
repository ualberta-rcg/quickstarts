# Jasper Quickstart

## See also ...

* [Jasper technical specifications](technical.md)

## Intended purpose and usage of system

**NOTE: This system was “defunded” in Fall 2017. Researchers affiliated with the University of Alberta should contact research.support@ualberta.ca for information about ongoing use of local systems.**

Jasper cluster is intended for general purpose serial and MPI-based parallel computing. The system is optimized for whole node scheduling, giving higher priority to jobs asking for 12 cores per node. The short maximum walltime (3 days) allows for quicket turnaround of jobs. Larger memory serial jobs (<48gb) can also be accomodated but as the number of nodes with more than 24gb are relatively small, these jobs can be expected to take longer to run.

## Request for Access
**NOTE: This system was “defunded” in Fall 2017. Researchers affiliated with the University of Alberta should contact research.support@ualberta.ca for information about ongoing use of local systems.**

## Connecting and logging in
Log in to Jasper by connecting to the host name `jasper.westgrid.ca` using an ssh (secure shell) client.

## Batch job policies
Batch jobs are handled by a combination of TORQUE and Moab software. For more information about submitting jobs, see Running Jobs.

| Resource                                         | Policy or limit |
| :----------------------------------------------- | --------------: |
| Maximum walltime                                 | 72 hours        |
| Maximum number of running jobs for a single user | 2880            |
| Maximum number of jobs submitted                 | 2880            |
| Maximum jobs in Idle queue                       | 5               |

Some of the former Checkers compute nodes have been added to the Jasper cluster. The former Checkers nodes have 2, 4-core Xeon L5420 processors with about 16000mb of memory available, while the original Jasper Nodes have 2, 6-core Xeon X5675 processors with about 24000mb or 48000mb of memory available.  Please note that serial jobs longer than 12 hours will run on L5420 (old checkers) nodes only. Shorter serial jobs may also run on  X5675 nodes. The shorter the walltime of a serial job the better chance it will have of running on X5675 nodes. Parallel jobs may run on either nodes. Howevever, jobs with resource requests of the form `nodes=n:ppn=12` will run exclusively on the X5675 nodes. Altenatively, if you have larger memory requirements that require whole nodes, `naccesspolicy=singlejob` can be used. For example

```bash
#PBS -l naccesspolicy=singlejob
#PBS -l nodes=2:ppn=4
#PBS -l pmem=6000mb
```

## Interactive jobs

Except for compiling programs and small tests(less than 1 hour, no more than 2 cores, less than 2000mb of memory/core), interactive use of Jasper should be through the '-I' option to qsub.

## Storage Information on Jasper

* `/home`
	* **Size: 356 TB**
	* **Quota**: 1 TB; Files: 500,000
	* **Command to checkquota**: `lfs quota -u <your username> /lustre`
	* **Purpose**: Home directory. Also serves as a global scratch directory on Jasper.
	               Note: Jasper and Hungabee share the same `/home` file system.
	* **Backup policy**: Not backed up. Users are encouraged to backup their own data.

## Program Information on Jasper
### OpenMP programs
The Intel compilers include support for shared-memory parallel programs that include parallel directives from the OpenMP standard. Use the `-openmp` compiler option to enable this support, for example:

```bash
module load compiler/intel/13.0.1
icc -o prog -openmp prog.c
ifort -o prog -openmp prog.f90
```

Before running an OpenMP program, set the `OMP_NUM_THREADS` environment variable to the desired number of threads using bash-shell syntax:

```bash
export OMP_NUM_THREADS=12
```

or C-shell (tsch-shell) syntax:

```csh
setenv OMP_NUM_THREADS 12
```

according to the shell you are using. Then, to test your program interactively, launch it like you would any other:

```bash
./prog
```

Here is a sample TORQUE job script for running an OpenMP-based program.

```bash
#!/bin/bash
#PBS -S /bin/bash
#PBS -l pmem=2000mb
#PBS -l nodes=1:ppn=12
#PBS -l walltime=12:00:00
#PBS -m bea
#PBS -M yourEmail@address
cd $PBS_O_WORKDIR
export OMP_NUM_THREADS=$PBS_NUM_PPN
./prog
```

### MPI Programs

MPI programs can be compiled using the compiler wrapper scripts `mpicc`, `mpicxx`, and `mpif90`. These scripts invoke the gnu compilers. For the Intel compilers, use mpiicc, mpiicpc, and mpifort. Use the mpirun command to launch an MPI program, for example:

```
module load library/openmpi/1.6.5-intel
mpicc -o prog prog.c
mpirun -np 8 ./prog
```

After your program is compiled and tested, you can submit large-scale production runs to the batch job system. Here are some sample TORQUE batch job scripts for MPI-based programs. People are encouraged to use entire nodes for MPI jobs. So requests should be in the form `nodes=x:ppn=12`. Requests in the form `procs=n` are still accepted but these jobs will have lower priority.

```bash
#!/bin/bash
#PBS -S /bin/bash
#PBS -l pmem=2000mb
#PBS -l nodes=2:ppn=12
#PBS -l walltime=12:00:00
#PBS -m bea
#PBS -M yourEmail@address
cd $PBS_O_WORKDIR
module load library/openmpi/1.6.5-intel
mpirun ./prog > out
```

 If you need more than pmem=2000mb, then you can use `naccesspolicy=singlejob` to reserve entire nodes as follows:

```bash
#!/bin/bash
#PBS -S /bin/bash
#PBS -l pmem=4000mb
#PBS -l nodes=4:ppn=6
#PBS -l naccesspolicy=singlejob
#PBS -l walltime=12:00:00
#PBS -m bea
#PBS -M yourEmail@address
cd $PBS_O_WORKDIR
module load library/openmpi/1.6.5-intel
mpirun ./prog > out
```

### Compiling and running programs

The latest Intel compilers are available on Jasper. The compiler commands are `icc` (C compiler), `icpc `(C++ compiler), and `ifort` (Fortran compiler). Please note that modules need to be loaded to use the compilers and MPI. In the sections below, basic use of the compilers is shown for OpenMP and for MPI-based parallel programs. Additional compiler directives for optimization or debugging should often be used.
