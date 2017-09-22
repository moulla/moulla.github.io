---
layout: default
title: Database Architecture for ML
title_desc: Adapted from a Robert Blumen interview with Dr Michael Stonebreaker
header_image : /images/dbase_market.png
pub_date: 17 Sep 2017
abstract: Databases(Dr Michael Stonebraker)
tags:
- Database
- OLTP
- MMDB
- ACID
- MapReduce
- Hadoop
- RDBMS
---
Many database solutions exist, which one is the 'best'? Which database vendor/philosophy should you choose? What are the architecture choices we need to understand? This article attempts to explain the underlying mechanisms used in both traditional and alternative technologies.

Applied Data-basing technology is somewhat lacking from academic data science and machine learning education. In reading about what [Tamr](https://www.tamr.com) is trying to achieve as a prerequisite to data science and applied machine learning solutions I was introduced to Dr Michael Stonebraker. Dr Stonebraker has multiple vested interests including VoltDB having started 9 VC backed companies but while opinionated provides excellent rationales for his views.This article summarises the useful insights from his talk but highly recommend you [listen yourself](http://www.se-radio.net/2013/12/episode-199-michael-stonebraker) if you want a concise and balanced overview on the state of modern database technology.

  The key piece that needs to be understood is that almost every efficiency gain is a trade-off and which is best depends on the problem the database is solving.  The primary concern for your choice should be high availability with scale out as you really don't want to change your architecture as you grow.

Examining the traditional Relational Database Management System(RDBMS) first:

Designed to work on disk in blocks/clusters in a serial manner with a buffer pool using locking, write ahead logs for recovery and B-trees for indexing and record locking for consistency. Generally RDBMS solutions use a serial disk block store, storing records sequentially on disk in formats like:

<div class='w3-card'>
<img src="/images/disk_block.png" alt="Disk Blocks" style="width:100%">
</div>

with a buffer pool in memory in disk block format with concurrency via record locking and write ahead log for ACID(atomicity, consistency, isolation, durability) and crash recovery using B trees for indexing, with query and query optimisation.

Dr Stonebraker's view is that where large scale multi-node architectures are required RDBMS will migrate to three segments, namely Data Warehousing ,Online Transaction Processing and the rest (nosql, graph db, array, map-Reduce).

<div class='w3-card'>
<img src="/images/dbase_market.png" alt="Database Segments" style="width:100%">
</div>


The first two pieces being optimally addressed by Column Stores and Main Memory Data-basing.  While the RDBMS Foreign Keys are powerful they are expensive in scalable applications.  This migration has yet to show up significantly in revenues with many RDBMS vendors adopting elements of the successful contenders into their products.

His arguments for this are that RDBMS are inefficient solutions to these problems with 90% overhead vs 10% find/reads/writes.  This overhead being made up approximately equally from buffer pool, locking, write ahead log, and latching threaded data structures.

The solutions he proposes are

* buffer pool : use main memory db

* locking - optimistic multi-version concurrency control(check only at end if there is a problem and repeat vs prevent problem) or transaction ordering(time-stamp or other).

* write ahead log: use command logging - which defers cost to post crash recovery time

* threaded: no threading/segment memory or latch free shared data structures

Addressing cluster consistency he compares eventual consistency (update, acknowledge then propagate), active/active replication (run on each node), active/passive (run on primary then replicate which requires 2 phase commit).  Active/Active does has the advantage that it does not require a log.

PS Stonebraker is not a fan of Hadoop except for 'embarrassingly' parallel NLP style applications which is OK for me as that's where I hope to use it and its in memory cousin Spark(which necessitates Tungsten due to Java's malloc overhead).

He describes Hadoop as the stack of HDFS file system, the mapReduce layer and Hive query tools and his critique is that other than for truly non-parallel loads queries in Hive on stack runs slower than alternative parallel data warehouse systems noting that Impala is Hive minus MapReduce.

<span class="w3-opacity" markdown="1">Adapted from the  [Software Engineering Radio Podcast interview with Michael Stonebraker](http://www.se-radio.net/2013/12/episode-199-michael-stonebraker) originally published 5 Dec 2013</span>

## Further Reading: ##

[In-Memory Big Data Management and Processing: A Survey](http://ieeexplore.ieee.org/document/7097722/)

[Stanford Computer Science Department Distinguished Computer Scientist Lecture lecture, November, 2010: Building Software Systems at Google and Lessons Learned Jeff Dean](https://static.googleusercontent.com/media/research.google.com/en//people/jeff/Stanford-DL-Nov-2010.pdf)

[Overview of Data Storage in RDBMS Physical Characteristics of Disks](http://users.csc.http://calpoly.edu/~dekhtyar/560-Fall2012/lectures/lec04.560.ps)



