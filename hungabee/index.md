# Hungabee Quickstart

## See also ...

* [Hungabee technical specifications](technical.md)

## Intended purpose and usage of system
NOTE: This system was “defunded” in Fall 2017. Researchers affiliated with the University of Alberta should contact research.support@ualberta.ca for information about ongoing use of local systems.

Hungabee is intended for large shared memory jobs that cannot run elsewhere. This includes threaded applications requiring more than 24 cores and serial programs requiring more than 256gb of memory. The machine is not intended for running MPI applications unless there is a very large memory requirement.

## Request for Access
NOTE: This system was “defunded” in Fall 2017. Researchers affiliated with the University of Alberta should contact research.support@ualberta.ca for information about ongoing use of local systems.

## Connecting and logging in
Log in to the UV100 by connecting to the host name hungabee.westgrid.ca using an ssh (secure shell) client.

## Batch job policies
Batch jobs are handled by a combination of TORQUE and Moab software. For more information about submitting jobs, see Running Jobs.

Although Hungabee is not a cluster, the batch job system treats it as such. This is a limitation of TORQUE and Moab, not of the hardware. From the point of view of the batch system, Hungabee is made up of 256 8-core (virtual) nodes, each with 64GB of memory. This implies that you can not request more than 64GB of memory per core (pmem). The maximum value is actually 65520MB, which is slightly less than 64GB (65536MB).

If you require more memory per core, request more cores. You do not have to use the extra cores. For example, to run a single-core job that requires 4TB of memory, you would have to request at least 512 cores.

| Resource                                            | Policy or limit |
| :-------------------------------------------------- | --------------: |
| Maximum walltime (hours)                            | 72              |
| Minimum number of cores (procs)                     | 32*             |
| Maximum memory resource request per core, pmem (MB) | 65520           |
| Maximum number of running jobs for a single user    | unlimited       |
| Maximum cores (sum for all jobs) for a single user  | unlimited       |
| Maximum jobs in Idle queue                          |         5       |
 

*Jobs with procs less than or equal to 16 will run on uv100. The maximum total mem should be 64000mb in this case.

## Interactive jobs

Except for compiling programs and small tests, interactive use of Hungabee should be through the `-I` option to `qsub`.

## Storage Information on Hungabee

Directory path	Size	Quota	Command to check quota	Purpose	Backup Policy


* `/home`
  * **Size**: 356 TB
  * **Quota**: 1 TB; Files: 500,000
  * **Command to check quota**: `lfs quota -u <your username> /lustre`
  * **Purpose**: Home directory.  Not good for scratch on UV1000; instead, use `/data` or `$TMPDIR`.
    Note: Hungabee and Jasper share the same `/home` filesystem.
  * **Backup Policy**: Not backed up.  Users are encouraged to backup their own data.
* `/data`
  * **Size**: 35 TB
  * **Quota**: 200 GB per user unless RAC allocated
  * **Purpose**: Working directory.
     For the best performance, jobs on Hungabee should be submitted from `/data`.
     This is a local disk on the uv1000 and there is an environmental variable, `UV_DATA`,
     that points to your personal directory on the local disk. Type
     `cd $UV_DATA`
     as soon as you log into Hungabee, or put this step in your login script.
     Please note that moving large amount of data to/from $UV_DATA should be done as part of a batch job.
  * **Backup policy**: Not backed up.
    Users are encouraged to backup their own data.
* `$TMPDIR`
  * **Size**: 18 TB
  * **Quota**: none. Request space when submitting job using `-l file=size`, where size < 2TB
  * **Purpose**: Scratch directory. Accessed by using `$TMPDIR`. The `$TMPDIR` directory is uniquely created at the beginning of each job and is deleted at the end.
  * **Backup policy**: Deleted at the end of the job (not backed up)

## Program Information on Hungabee

### Requesting memory
It is usually easiest to keep `pmem=8190mb` and adjust procs to insure that there is sufficient total memory available. For example, if you need 2000000mb of memory, 2000000 /8192 = 244.1. However, procs needs to be a multiple of 16 so procs should be 256. If there are  more procs than the calculations are using, then cores that are being used should be evenly spread out among the memory. This is done using the command omplace or dplace as shown below.

### OpenMP programs
The Intel compilers include support for shared-memory parallel programs that include parallel directives from the OpenMP standard. Use the `-openmp` compiler option to enable this support, for example:

```bash
icc -o prog -openmp prog.c 
ifort -o prog -openmp prog.f90
```

Before running an OpenMP program, set the `OMP_NUM_THREADS` environment variable to the desired number of threads using bash-shell syntax:

```bash
export OMP_NUM_THREADS=8
```

or C-shell (tsch-shell) syntax:

```csh
setenv OMP_NUM_THREADS 8
```

according to the shell you are using. Then, to test your program interactively, launch it like you would any other:

```bash
./prog
```

Here is a sample TORQUE job script for running an OpenMP-based program. Note the use of the omplace command to launch the program. This ensures that successive threads are pinned to unique cores for optimal performance. Please see the manpages for more information ('man omplace').

```bash
#!/bin/bash 
#PBS -S /bin/bash 
## procs should be a multiple of 16 
## pmem (per core memory) should avoid multiple of 8 GB 
#PBS -l pmem=8190mb 
#PBS -l procs=256 
#PBS -l walltime=12:00:00 
#PBS -m bea 
#PBS -M yourEmail@address 
cd $PBS_O_WORKDIR 
export OMP_NUM_THREADS=$PBS_NP 
omplace -nt $OMP_NUM_THREADS ./prog
```

### MPI programs
MPI programs can be compiled using the compiler wrapper scripts `mpicc`, `mpicxx`, and `mpif90`. These scripts invoke the Intel compilers and link against the SGI MPT (multi-processing toolkit) library. Use the mpirun command to launch an MPI program in interactive mode, for example:

```bash
mpicc -o prog prog.c 
mpirun -np 8 ./prog
```

After your program is compiled and tested, you can submit large-scale production runs to the batch job system. For batch scripts, use mpiexec_mpt instead of mpirun. It will not be neccessary to specify the number of cores as mpiexec_mpt will take this value from the batch resource request (`procs=n`). Here are some sample TORQUE batch job scripts for MPI-based programs.

```bash
#!/bin/bash 
#PBS -S /bin/bash

## procs should be a multiple of 16 
## pmem (per core memory) should avoid multiple of 8 GB

#PBS -l pmem=8190mb 
#PBS -l procs=256 
#PBS -l walltime=12:00:00 
#PBS -m bea 
#PBS -M yourEmail@address

cd $PBS_O_WORKDIR

mpiexec_mpt ./prog > out
```

This second example is for an MPI job using more than 64 GB of memory per core. The `dplace` command is used to distribute the MPI tasks evenly among the set of allocated cores and memory nodes. See the manpages for more details (`man dplace`).

```bash
#!/bin/bash 
#PBS -S /bin/bash

#PBS -l procs=32 
#PBS -l pmem=65520mb 
#PBS -l walltime=12:00:00

cd $PBS_O_WORKDIR

mpiexec_mpt -np 16 dplace -c 0-31:2 ./prog > out
```

The following example shows how MPI statistics could be obtained for performance tuning. The resulting output may indicate excessive retries allocating buffers. In this case, perfomance may be improved by setting `MPI_BUFS_PER_PROC` to a higher value(default: 32). Please see the man pages for mpi (`man mpi`) for more information.

```bash
mpiexec_mpt -v -stats -np $PBS_NP ./prog > out
```

