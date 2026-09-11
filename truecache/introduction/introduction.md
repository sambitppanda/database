# Introduction

## About This Workshop

Run this hands-on workshop to learn how Oracle True Cache improves scalability by offloading read queries and reducing the number of requests and connections sent to the primary database. The workshop uses a compute instance running an online transaction processing application and a primary database configured with Oracle True Cache. The demo application is a Java program that uses the 26ai JDBC driver to simulate a heavy transaction workload and demonstrate how routing read-only queries to True Cache affects application performance.

### About Oracle True Cache

Oracle Database True Cache is a consistent, automatically managed, read-only replica designed to serve eligible SQL and key-value read workloads. It is conceptually similar to a diskless Active Data Guard replica. Large-scale web applications can experience performance issues when the primary database becomes a bottleneck. True Cache improves scalability by offloading read queries and reducing the number of requests and connections sent to the primary database.

### Why Use True Cache

True Cache can scale a read-heavy application without requiring data partitioning. When the primary database becomes a bottleneck, True Cache offloads read queries and helps the application handle more work. Each eligible query returns a transactionally consistent result. Because True Cache is maintained from the Primary database, availability of the most recent committed change depends on replication/apply state.

*Estimated Workshop Time:* 1 hour 

![True Cache introduction](https://oracle-livelabs.github.io/database/truecache/introduction/images/truecache-intro.png " ")

The diagram shows the application using one logical connection to both databases. Read-only queries can be served by True Cache, while read-write operations continue to use the Primary database; changes are replicated from the Primary database to keep True Cache current.

### Objectives
Run this hands-on workshop to learn the basics of True Cache.

Once you complete your setup, the next lab will cover:

- Reviewing the preloaded data in the TRANSACTIONS schema
- Running a Java-based JDBC application against the Primary database and then True Cache to compare application performance.


### Prerequisites

- Familiarity with Oracle Database is required
- Familiarity with Java and JDBC is desirable, but not required
- Some understanding of cloud and database terms is helpful
- Familiarity with Oracle Cloud Infrastructure (OCI) is helpful
- Familiarity with Podman or Docker is helpful.

## Choose Your Workshop Path

The DBW26 workshop provides two ways to learn the same True Cache workflow:

- **FastLab:** use the visual command center for a quick guided demonstration.
- **Full LiveLab:** use the terminal to run the database, Java, and Podman commands directly.

The workflow covers environment validation, JDBC routing, cache KEEP and warmup, Primary versus True Cache read performance, availability while Primary is stopped, and semantic payment search with Oracle AI Vector Search. The Full LiveLab path uses a hidden password prompt and terminal commands that require sudo access. Follow each lab's container-session instructions.

## Learn More
- [True Cache documentation](https://docs.oracle.com/en/database/oracle/oracle-database/23/odbtc/overview-oracle-true-cache.html)

## Acknowledgements
* **Authors** - Sambit Panda, Consulting Member of Technical Staff, Oracle Database Product Management
* **Contributors** - Pankaj Chandiramani, Shefali Bhargava, Jyoti Verma, Nithin Thekkupadam Narayanan
* **Last Updated By/Date** - Sambit Panda, Consulting Member of Technical Staff, Sep 2026
