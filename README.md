# 2022SC-Blaze-artifact
This repository is intended to be used by SC'22 artifact evaluation committee.

## Abstract

Blaze is an out-of-core graph processing framework optimized for modern, fast NVMe drives (FNDs) such as Intel Optane SSD. Blaze achieves high-performance on representative graph workloads (e.g., BFS, PageRank on highly-skewed graphs) by constantly saturating FND with the low-overhead <em>online binning</em> mechanism.  Blaze runs on any block devices although its ability to fully utilize IO bandwidth is best highlighted when running on FNDs.

## Description

### How to access
For easy reproduction, we provide a docker container image, which can be downloaded by

```
$ docker pull junokim8/blaze:1.0
```

The docker image contains binary files corresponding to each graph query implemented in Blaze. We also plan to open source the code once the paper is accepted.

### Hardware dependencies

We evaluated Blaze with the following machine configuration.

- CPU : Intel Xeon Gold 6230 processor (2.1 GHz) with 20 physical cores in a single socket (no hyperthreading)
- Memory : 96GB of DRAM
- Storage : 1.9TB Intel NAND SSD (model: DC S3520), 960GB Intel Optane SSD (model: DC P4800X)
- OS: Linux 5.12

Blaze is not yet optimized for multi-socket processors so we recommend to use single-socket machine or similar setting to reproduce similar results in the paper.

Also, while Blaze runs on any type of block device, its high performance is best judged when running on fast NVMe SSDs such as Intel Optane SSD.
If desired, we will provide proper guidelines to allow access to our testbed.


### Software dependencies
We provide a docker image containing pre-built binaries so it is not necessary to explicitly build the dependencies to run Blaze. As for the docker version, we used the version 20.10.10, so we expect the same or a newer version to work.

### Data sets

We evaluated Blaze's performance on the following six input graphs. 

- rmat27: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/rmat27.zip (13GB)
- rmat30: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/rmat30.zip (102GB)
- uran27: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/uran27.zip (16GB)
- twitter: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/twitter.zip (8.5GB)
- sk2005: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/sk2005.zip (2.3GB)
- friendster: https://storage.googleapis.com/nvsl-aepdata/graphdata/sc22/friendster.zip (13GB)

After unzipping, each dataset consists of four files: Two of them are `.gr.index` (index file) and `.gr.adj.0` (adjacency list file), collectively representing a CSR format that stores outgoing neighbors of each vertex. Additional two files, `.tgr.index` and `.tgr.adj.0`, represent a transpose of the given CSR graph.

## Installation

### Storage setup
The target storage should be mounted in the target machine to place the input graphs. An example of mounting our target disk `/dev/nvme0n1` to `/mnt/nvme` using Ext4 file system is as follows.

```
$ sudo mkfs.ext4 -F /dev/nvme0n1
$ mkdir -p /mnt/nvme
$ sudo mount /dev/nvme0n1 /mnt/nvme
```

Then, place the downloaded input graphs under /mnt/nvme.

### Getting inside the docker container

As Blaze binaries are available in the provided docker image, it is necessary to run and get into the container as follows.

```
$ docker run --rm -it -v "/path/to/your/storage":"/mnt/nvme" junokim8/blaze:1.0 /bin/bash
```

Inside the docker container console, the Blaze binaries are available at /home/blaze/build.

## Evaluation workflow

### Major claims

The major claims and key results made in Blaze paper are listed as follows.

1. Blaze outperforms FlashGraph and Graphene with significant speedups [Figure 7].
2. Blaze's online binning mechanism is the key to saturating Intel Optane SSD [Figure 8].
3. Blaze scales well with more threads as long as IO is not saturated [Figure 9].

### Running target queries

For instance, the following command runs BFS on rmat27 graph. The example calculates BFS using 17 threads (16 for computation and 1 for IO) starting from vertex 0.

```
$ ./bin/bfs -computeWorkers 16 -startNode 0 /mnt/nvme/rmat27.gr.index /mnt/nvme/rmat27.gr.adj.0
```

Certain queries require a transpose of the input graph as well. For instance, our Betweenness Centrality implementation falls into this case. To give the transpose graph as additional input, use `-inIndexFilename` and `-inAdjFilenames` as follows.

```
$ ./bin/bc -computeWorkers 16 -startNode 0 /mnt/nvme/rmat27.gr.index /mnt/nvme/rmat27.gr.adj.0 -inIndexFilename /mnt/nvme/rmat27.tgr.index -inAdjFilenames /mnt/nvme/rmat27.tgr.adj.0
```

The number of IO thread is automatically determined by Blaze depending on the number of given partitions. In the above examples, we use one partition.

## Reproducing results
