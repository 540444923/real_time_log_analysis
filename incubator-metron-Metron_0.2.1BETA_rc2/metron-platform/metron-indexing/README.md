#Indexing

## Introduction

The `indexing` topology is a topology dedicated to taking the data
from the enrichment topology that have been enriched and storing the data in one or more supported indices
* HDFS as rolled text files, one JSON blob per line
* Elasticsearch
* Solr

By default, this topology writes out to both HDFS and one of
Elasticsearch and Solr.

Indices are written in batch and the batch size is specified in the
[Enrichment Config](../metron-enrichment) via the `batchSize` parameter.
This config is variable by sensor type.

## Indexing Architecture

![Architecture](indexing_arch.png)

The indexing topology is extremely simple.  Data is ingested into kafka
and sent to 
* An indexing bolt configured to write to either elasticsearch or Solr
* An indexing bolt configured to write to HDFS under `/apps/metron/enrichment/indexed`

Errors during indexing are sent to a kafka queue called `index_errors`

# Notes on Performance Tuning

Default installed Metron is untuned for production deployment.  By far
and wide, the most likely piece to require TLC from a performance
perspective is the indexing layer.  An index that does not keep up will
back up and you will see errors in the kafka bolt.  There
are a few knobs to tune to get the most out of your system.

## Kafka Queue
The `indexing` kafka queue is a collection point from the enrichment
topology.  As such, make sure that the number of partitions in
the kafka topic is sufficient to handle the throughput that you expect.

## Indexing Topology
The enrichment topology as started by the `$METRON_HOME/bin/start_elasticsearch_topology.sh` 
or `$METRON_HOME/bin/start_solr_topology.sh`
script uses a default of one executor per bolt.  In a real production system, this should 
be customized by modifying the flux file in
`$METRON_HOME/flux/indexing/remote.yaml`. 
* Add a `parallelism` field to the bolts to give Storm a parallelism
  hint for the various components.  Give bolts which appear to be bottlenecks (e.g. the indexing bolt) a larger hint.
* Add a `parallelism` field to the kafka spout which matches the number of partitions for the enrichment kafka queue.
* Adjust the number of workers for the topology by adjusting the 
  `topology.workers` field for the topology. 

Finally, if workers and executors are new to you or you don't know where
to modify the flux file, the following might be of use to you:
* [Understanding the Parallelism of a Storm Topology](http://www.michael-noll.com/blog/2012/10/16/understanding-the-parallelism-of-a-storm-topology/)
* [Flux Docs](http://storm.apache.org/releases/current/flux.html)
