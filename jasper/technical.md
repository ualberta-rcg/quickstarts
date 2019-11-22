# Jasper Technical Specifications

* **System Location**: University of Alberta
* **Production Date**: Friday, June 1, 2012
* **Cores**: 4160
* **Login Node**: jasper.westgrid.ca

## Technical Specifications

**NOTE: This system was “defunded” in Fall 2017. Researchers affiliated with the University of Alberta should contact research.support@ualberta.ca for information about ongoing use of local systems.**

## Processors

Jasper is an SGI Altix XE cluster with an aggregate 400 nodes, 4160 cores and 8320 GB of memory. 240 nodes have Xeon X5675 processors, 12 cores (2 x 6) and 24 GB of memory. Of these, 32 have additional memory for a total of 48 GB.  160 nodes, formerly part of the Checkers cluster, have Xeon L5420 processors, 8 cores (2 x 4) and 16 GB of memory.

## Interconnect

Jasper uses an InfiniBand interconnect.  X5675 nodes are connected at 40 Gbit/s, with a 1:1 blocking factor, which is the fastest interconnect currently in WestGrid.  L5420 nodes are connected at 20 Gbit/s, with a 2:1 blocking factor.  Parallel jobs have priority on the X5675 nodes.

## Storage

A Lustre parallel distributed file system is attached to Jasper via the InfiniBand interconnect. (This system is also attached to Hungabee.)  Housed in an SGI IS16000 disk array with 250 drives, it provides a single 356 TB filesystem.
