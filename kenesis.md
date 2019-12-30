# Kinesis data streams

## Features (vs SQS)

* Routing related records to the same record processor (as in streaming MapReduce)
* Ordering of records
* Ability to consume records in the same order a few hours later (replay).
* Ability for multiple applications to consume the same stream concurrently and independently.

## Access pattern

State-full iterator (cursor)

## Scalability

No auto-scaling, you need to provision enough shards ahead of time.

## Shard

Shard is the base throughput unit.

One shard provides a capacity of:

* 1MB/sec data input
* 2MB/sec data output
* up to 1000 PUT records per second

You can add or remove shards from your data stream dynamically as your data throughput changes by resharding the data stream.

number_of_shards = max (incoming_write_bandwidth_in_KB/1000, outgoing_read_bandwidth_in_KB/2000)

While the capacity limits are exceeded, the put data call will be rejected with a ProvisionedThroughputExceeded exception

## Record

A record is the unit of data stored in an Amazon Kinesis data stream. 

A record is composed of:

* sequence number
* partition key
* data blob

Data blob is the data of interest your data producer adds to a data stream. The maximum size of a data blob (the data payload before Base64-encoding) is 1 megabyte (MB).

Partition key is used to segregate and route records to different shards of a data stream.  A partition key is specified by your data producer.

A sequence number is a unique identifier for each record. Sequence number is assigned by Amazon Kinesis when a data producer calls PutRecord or PutRecords.

## Enhanced fan-out

Enhanced fan-out is an optional feature for Kinesis Data Streams consumers that provides logical 2 MB/sec throughput pipes between consumers and shards.

KCL version 2.x takes care of registering your consumers automatically

The HTTP/1 GetRecords API does not currently support enhanced fan-out

The use of enhanced fan-out does not impact the limits of shards for traditional GetRecords usage (2MB/sec data output)

There is an on-demand hourly cost for every combination of shard in a stream and consumer (a consumer-shard hour) registered to use enhanced fan-out, in addition to a data retrieval cost for every GB retrieved.

## Kinesis Client Library (KCL)

Amazon Kinesis Client Library (KCL) for Java | Python | Ruby | Node.js | .NET is a pre-built library that helps you easily build Amazon Kinesis Applications for reading and processing data from an Amazon Kinesis data stream.

## Kinesis Connector Library

Amazon Kinesis Connector Library is a pre-built library that helps you easily integrate Amazon Kinesis Data Streams with other AWS services and third-party tools.

provides connectors to:

* DynamoDB
* Redshift
* S3
* Elasticsearch

## Kinesis Producer Library (KPL)

Amazon Kinesis Producer Library (KPL) is an easy to use and highly configurable library that helps you put data into an Amazon Kinesis data stream. KPL presents a simple, asynchronous, and reliable interface that enables you to quickly achieve high producer throughput with minimal client resources.

KPL's core is built with C++ module and can be compiled to work on any platform with a recent C++ compiler. The library is currently available in a Java interface.

## Kinesis Agent

Kinesis Agent is a pre-built Java application that offers an easy way to collect and send data to your Amazon Kinesis data stream. You can install the agent on Linux-based server environments such as web servers, log servers, and database servers.

It monitors certain files on the disk and then continuously send new data to your Amazon Kinesis data stream

Kinesis Agent currently supports Amazon Linux or Red Hat Enterprise Linux.

## Error in Kinesis Firehose tutorial

https://aws.amazon.com/getting-started/projects/build-log-analytics-solution/

All records pushed to AWS ES timestamped with the same value.

This is the line that causes an issue:

    TIMESTAMP_TO_CHAR('yyyy-MM-dd''T''HH:mm:ss.SSS', LOCALTIMESTAMP) as datetime

`LOCALTIMESTAMP` to be replaced with `min("SOURCE_SQL_STREAM_001".ROWTIME)` as shown below

```sql
CREATE OR REPLACE STREAM "DESTINATION_SQL_STREAM" (
 datetime VARCHAR(30),
 status INTEGER,
 statusCount INTEGER);
CREATE OR REPLACE PUMP "STREAM_PUMP" AS INSERT INTO "DESTINATION_SQL_STREAM"
 SELECT STREAM   TIMESTAMP_TO_CHAR('yyyy-MM-dd''T''HH:mm:ss.SSS',
min("SOURCE_SQL_STREAM_001".ROWTIME)) as datetime, 
 "response" as status, COUNT(*) AS statusCount
 FROM "SOURCE_SQL_STREAM_001"
 GROUP BY
 "response",
 FLOOR("SOURCE_SQL_STREAM_001".ROWTIME TO MINUTE);
 ```
