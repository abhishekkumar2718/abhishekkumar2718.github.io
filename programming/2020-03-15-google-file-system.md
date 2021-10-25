---
layout: post
title: Google File System
parent: Programming
summary: Read more about the file system that used to power Google's infrastructure, from Gmail to Docs and Youtube to Drive.
---

# Google File System
{:.no_toc}

Google File System is a proprietary distributed file system developed by Google
to provide efficient, reliable access to data using large clusters of commodity
hardware. It was initially described in a 2003 paper titled “The Google File
System” by Sanjay Ghemawat, Howard Gobioff and Shun-Tak Leung [1].

Designed to meet Google’s rapidly growing needs, GFS explores radically
different design choices. It is widely deployed within Google as the storage
platform for the generation and processing of data used. The largest cluster
provides hundreds of terabytes of storage across thousands of disks over a
thousand machines and it is concurrently accessed by hundreds of clients.

In 2010, Google adopted Colossus as their revamped file system. Colossus is
specifically built for real time services, whereas GFS was built for batch
operations. Colossus, along with Caffeine, their new search infrastructure
power virtually all of Google’s web services from Gmail to Google Docs,
and Youtube to Google Cloud Storage.

> I was assigned GFS as a part of an Operating Systems Assignment, which is
> why I have skipped over most of distributed system aspects. Check out the
> [original paper](https://research.google.com/archive/gfs-sosp2003.pdf) for 
> more.

1. Table of Contents
{:toc .text-delta}

## Design Overview

### Assumptions

GFS has been designed by key observations about expected workload and
technological environment. It reflects a departure from some earlier file
system design assumptions.

- Component failures are the norm rather than exception. GFS clusters are
built using large clusters of commodity hardware. The quantity and quality of
component guarantee that some are not functional at a given time and might not
recover from their failures.

- Files are huge by traditional standards. Multi-GB files and fast growing data
sets of many TBs are typical. As a result, GFS uses a block size of 64 MB
compared to 4 KB of a traditional linux file system.

- Most files are mutated by appending new data rather than overwritting
existing data. Once written, files are only read and often only sequentially.
Given this access patern, appending becomes the focus of performance
optimization and atomicity guarantees.

- System must efficiently implement semantics for multiple clients that
concurrently append to same file. Atomicity with minimal synchronization
overhead is essential.

- High sustained bandwith is more important than low latency. Target
applications place a premium on processing data at high rate with little to no
response time requirements for individual read or write.

### Interface

GFS provides a familiar file system interface, although it does not implement
a standard API like POSIX. Usual operations to create, delete, open, close,
read and write files are supported. Addtionally, GFS supports  snapshot and
record append operations. Snapshot creates a copy of file or directory tree.
Record append allows multiple clients to append data to the same file
concurrently while guaranteeing atomicity.

## Architecture

A GFS cluster consists of a single master and mutliple chunkservers and is
accessed by mutliple clients. Each of these is typically a commodity linux
machine running a user level server process.

Files are divided into fixed-size chunks. Each chunk is identified by an
immutable and globally unique 64 bit chunk handle assigned by master at the
time of chunk creation. Chunkservers store chunks on local disk as Linux files.

Read/write operations is specified by a chunk handle and byte range. For
reliablity, each chunk is replicated on multiple chunkservers. By default,
three replicas ares stored, though users can designate different replication
levels.

Chunk size is one of the key design parameters. GFS choose 64 MB, which is much
larger than typical file system block size. Lazy space allocation avoids wasting
space due to internal fragmentation. A large chunk size reduces clients’ need
to interact with master, reduces network overhead by maintaing persistent TCP
connection to chunkserver and reduces size of metadata stored on master.

The master maintains all file system metadata. This includes namespace,
authorization, mapping of files to chunks and current location of chunks. It
controls system wide activities such as chunk lease management, garbage
collection and migration.

GFS client code linked into each application implements filesystem API and
communicates with the cluster. Clients interact with master for metadata but
all data-bearing communication goes directly to appropriate chunkserver.

## Performance

When used with a small number of servers (15), the file system achieves read
speeds comparable to a single disk (80-100 MB/s) but has a reduced write
performance (30 MB/s) and slow append rate (5 MB/s). Aggregating multiple
servers also allows greater capacity, reaching upto 583 MB/s for 342 nodes.

## Advantages

1. GFS provides a location independent namespace.
2. GFS spreads file’s data across storage servers, distributing read/writes.
3. GFS uses commodity machines, lowering infrastructure costs.
4. GFS stores minimal metadata in memory and has fast reboot times for master (1-2 seconds).

## Disadvantanges

1. GFS uses replication for redundancy and consumes more raw storage than xFS or Swift.
2. GFS uses a centralized approach and failure of master stops the service.
3. GFS is specialized highly for its workload and performs poorly as a general purpose file system.
4. GFS uses a relaxed consistency model and clients have to perform independent consistency checks.

## Summary

GFS is a highly customized file system, catering to Google’s specific needs.
It is an example of smart design - analysis of workload and expectations. It
prioritizes performance for its specific use over overall performance.

## References

1. [The Google File System](https://research.google.com/archive/gfs-sosp2003.pdf) - 2003 paper introducing GFS

2. [Google Remakes Online Empire With 'Colossus'](https://www.wired.com/2012/07/google-colossus/) - Article by wired.com

3. [Google File System](http://kaushiki-gfs.blogspot.com/2012/10/normal-0-false-false-false-en-us-x-none.html)
