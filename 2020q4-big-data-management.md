# Big data management

## 21-4-2021

'1st generation':

* Hadoop MapReduce
* Hadoop HDFS (Hadoop Distributed File System, object storage)

2nd generation:

* Apache Spark
* Google Pregel
* Microsoft Dryad

Hadoop ecosystem, from bottom to top:

* Object storage, Hadoop Distributed File System (HDFS)
* Table storage: HCatalog+HBase (meta data and col. storage)
* Computation: MapReduce (Distributed Programming Framework)
* Programming languages: Pig (Data Flow), Hive (SQL)

Why storage layer?

* Scalability
* Simplicity (abstraction)
* Efficiency (parallel)
* Fault-tolerance (redundancy)

HDFS:

* Files partitioned into blocks (typically 64MB)
* Blocks indentified by URI, distributed and replicated
* Nodes:
  * Name nodes: keep locations of blocks
  * Secondary name nodes: snapshot+log of changes
  * Data nodes: actual blocks
* Detect failed data nodes with heartbeats. On failure, data node is removed from index and lost partitions are re-replicated.
* Copy: `hdfs dfs -cp <srcPath> <targetPath>`
* Remove: `hdfs dfs -rm <path>`
* Fast RAM for hot data for higher speed

Resilient distributed dataset (RDDs):

* Immutable
* Distributed
* Lazily evaluated
* Cacheable
* Replicated

MapReduce + Combine:

* Mapper produces key-value pairs
* Reducer consumes and produces K,V pairs
* Combiner locally merges K,V pairs to reduce shuffling (number of cross-node messages)

