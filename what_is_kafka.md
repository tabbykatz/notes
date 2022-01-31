# What is Kafka?

Decoupling of data streams & systems.

## Apache Kafka is a high-throughput messaging system created by LinkedIn maintained by Confluent

Scalable- 100s of brokers, millions of messages per second. High performance. Low-latency real-time

- messaging systems
- activity tracking
- metrics
- logs
- stream processing
- system dependencies
- integration

Kafka is just a transport system.

## What is Kafka theory?

### What are Kafka topics?

A particular stream of data.
Similar to a table in a database.
You can have as many topics as you need.
A topic is identified by its name.

### What are partitions within a topic?

- ordered but it only matters within a partition
- each message in a partition has an incremental id called an offset (zero indexed)
- when a topic is created you declare the number of partitions
- data is only kept for a limited amount of time (default 1 week)
- immutable data. offsets never start over from 0

kafka_topic_name ---> every 20s, msg ---> topic_name with 10 partitions --->

### What are kafka brokers?

- A Kafka cluster is composed of multiple brokers (servers)
- Identified with and ID (int)
- Each broker contains certain topic partitions
- After connecting to one broker you are connected to entire cluster
- Number of brokers varies (3 - over 100)
- A topic is spread / distributed across all brokers

### What is Topic Replication Factor?

- 2 - 3 (gold standard) copies / replicas of each partition so that things don't get lost
- There is a leader broker for a given partition (Primary). The others are passive in-sync replicas / ISR
- If you lose a leader an ISR will become the leader
- All this is handled by Zookeeper

/end notes for 1/31/22
