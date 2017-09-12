# System Design
The System Design Primer

Author Info
-----------
Author: Andrew Gurung <br>
URL: http://www.andrewgurung.com/

Table of Contents
-----------------
- Scalability
- High-level trade-off
- Consistency patterns
- Availability patterns
- Domain name system
- Content delivery network
- Load balancer
- Reverse proxy (web server)
- Application layer
- Database
	- Relational database management system (RDBMS)
	- NoSQL
	- SQL or NoSQL
- Cache
- Asynchronism
- Communication
- Security
-----------------

## Scalability
- Clones:
	- Clone is an exact replica of a codebase. Image files can be made from one server and new instances/clones can be produced
	- Useful in load balancing where a user Steve can login at server1. Then the other request can be served by some other available servers like server2, server3 or server4
	- Scalability Rule: Never store session information in local disc or memory. Otherwise Steve must always be served by server1
	- Sessions should be stored in a central external database or caches that are close to application server
- Databases
	- Path 1: Hire a DBA and tell him to do Master-Slave replication. Slave is for read-only, Master is for write-only. Add more RAM to Master server for performance. In the long run, it is better to go with Path 2
	- Path 2: Redesign your application to not use JOINS. Either stay with MySQL or switch to scalable NoSQL database like MongoDB. When the application gets slower due to your app doing all the dataset joins, it's time to introduce cache
- Caches
	- Cache is a simple key-value store that sits as a buffer between application and data store. Cache are lightning fast
	- Always opt for in-memory cache like Redis. Never do file-based cache, it makes cloning and scaling a pain
	- User sessions should always be cached
	- There are two patterns of caching data
		- Cached Database Queries: The resulset is stored in cache where the hashed query acts as key. Has issues with expiration and complex query. If a column is added to a table, all queries that uses that table has to be deleted.
		- Cached Objects: Better option where data is treated as an object. When the application has completed assembling the data, it can be directly stored in cache. The application just consumes the latest cached object and nearly never touches the databases anymore
- Asynchronism
	- Asynchronism is useful to avoid the user waiting for a service. While the requested service is being processed, user can navigate to other pages on the website
	- A messaging system like RabbitMQ can take a queue of task that a worker can process
	- There are two ways asynchronism can be done:
		- Async 1: Process time-consuming task in advance and serve the finished work in low request time. Eg convert dynamic site content to static. Results in a super responsive website
		- Async 2: When a very consuming task is requested, the job is added to queue and frontend immediately notifies the user that the task will take some time, please continue to browse the page. When the job is finished, the messaging system notifies the frontend to handle the response.
-----------------
## High-level trade-off
### Performance vs scalability
- Performance problem: System is slow for a single user
- Scalability problem: System is fast for a single user, but slows down under heavy load 

### Latency vs throughput
- Latency is the time required to perform some action or to produce some result
- Throughput is the number of such actions executed or results produced per unit of time
- Aim for maximal throughput with acceptable latency

### Availability vs consistency
- According to CAP theorem, only two out of the following three guarantees across a write/read pair
	- Consistency: A read will guarantee the most recent write
	- Availability: A non-failing node will return available data that could be stale
	- Partition Tolerance: The system will continue to function when network partitions occur
- Since network failure is inevitable and out of our reach, we should always consider Partition Tolerance
- Taking Partition Tolerance into consideration, we can have two combination
	- CP (Consistency/Partition Tolerance):  Wait for a response from the partitioned node which could result in a timeout error. Useful when your business requirements dictate atomic reads and writes
	- AP (Availability/Partition Tolerance):  Return the most recent version of the data you have, which could be stale. This system state will also accept writes that can be processed later when the partition is resolved. Useful when your business allows some flexibility around when the data in the system synchronizes (shopping carts)
-----------------

## Consistency patterns
1. Weak Consistency
	- After a write, reads may or may not see it
	- VoIP, video chat. What was spoken during connection loss, will not be repeated
2. Eventual Consistency
	- After a write, reads will eventually see it (typically within milliseconds)
	- Search Engine Indexing, Email services
3. Strong Consistency
	- After a write, reads will see it. Data is replicated synchronously
	- File systems and RDBMSes. Works well where transactions are needed
-----------------

## Availability patterns
