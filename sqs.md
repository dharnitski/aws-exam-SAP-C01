# Simple Queue Service

## Features (vs Kinesis)

* Retry for failed processing after a configured visibility timeout
* Individual message delay
* Dynamically increasing concurrency/throughput at read time

## Access pattern

Messaging semantics (such as message-level ack/fail) and visibility timeout.

## Scaling

auto-scale transparently