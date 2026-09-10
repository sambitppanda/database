# Introduction

## About This Workshop

Run this hands-on workshop to learn how Oracle True Cache improves scalability by offloading read queries and reducing the number of requests and connections sent to the primary database. The workshop uses a compute instance running an online transaction processing application and a primary database configured with Oracle True Cache. The demo application is a Java program that uses the 26ai JDBC driver to simulate a heavy transaction workload and show how offloading read-only queries to True Cache affects application performance.

### About Oracle True Cache

Oracle Database True Cache is an in-memory, consistent, and automatically managed SQL and key-value cache. It is conceptually a diskless Active Data Guard (ADG) database. Large-scale web applications can experience performance issues when the primary database becomes a bottleneck. True Cache improves scalability by offloading read queries and reducing the number of requests and connections sent to the primary database.

### Why Use True Cache

True Cache can scale a read-heavy application without requiring data partitioning. When the primary database becomes a bottleneck, True Cache offloads read queries and helps the application handle more work. Data remains consistent and current within a single query, which is important for joins across multiple rows and complex objects with nested relationships.

*Estimated Workshop Time:* 1 hour 

![True Cache introduction](https://oracle-livelabs.github.io/database/truecache/introduction/images/truecache-intro.png " ")

The diagram shows the application using one logical connection to both databases. Read-only queries can be served by True Cache, while read-write operations continue to use the Primary database; changes are replicated from Primary to keep the cache consistent.

### Objectives
Run this hands-on workshop to learn the basics of True Cache.

Once you complete your setup, the next lab will cover:

- Creating and loading data to a transaction based schema
- Running a Java based application using JDBC to connect to the database and run different transactions against the primary database first and True Cache after that, to show how True Cache can improve application performance.


### Prerequisites

- Familiarity with Oracle Database is required
- Familiarity with Java and JDBC is desirable, but not required
- Some understanding of cloud and database terms is helpful
- Familiarity with Oracle Cloud Infrastructure (OCI) is helpful
- Familiarity with podman/docker is helpful

## Choose Your Workshop Path

The DBW26 workshop provides two ways to learn the same True Cache workflow:

- **FastLab:** use the visual command center for a quick guided demonstration.
- **Full LiveLab:** use the terminal to run the database, Java, and Podman commands directly.

The workflow covers environment validation, JDBC routing, cache KEEP and warmup, Primary versus True Cache read performance, availability while Primary is stopped, and semantic payment search with Oracle AI Vector Search. The Full LiveLab path starts with one hidden password prompt and one `sudo -s` host session; later steps continue in the same sessions.

## Learn More
- [True Cache documentation](https://docs.oracle.com/en/database/oracle/oracle-database/23/odbtc/overview-oracle-true-cache.html)

## Acknowledgements
* **Authors** - Sambit Panda, Consulting Member of Technical Staff, Oracle Database Product Management
* **Contributors** - Pankaj Chandiramani, Shefali Bhargava, Jyoti Verma, Nithin Thekkupadam Narayanan
* **Last Updated By/Date** - Sambit Panda, Consulting Member of Technical Staff, Sep 2026
