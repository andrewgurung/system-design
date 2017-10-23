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
There are two main patterns to support high availability: fail-over and replication

### Fail-over
1. Active-passive
	- Heartbeats are sent between active and passive server on standby
	- If there is any interruption in heartbeat, the passive server will take over active server's IP and starts service
	- Downtime can vary depending on the status of passive server. Cold standby will result in more downtime than hot standby 
2. Active-active
	- Both servers are managing traffic, spreading the load between them
	- For public facing servers, DNS has to know IP addresses of both servers
	- For internal facing servers, application logic would need to know about both servers

### Replication
- Master-slave replication
- Master-master replication
-----------------

## Domain name system (DNS)
- DNS translates a domain name such as www.example.com to an IP address
- Your ISP provides information about which DNS to contact when doing lookup
- Some DNS can route traffic based on latency, geolocation or weighted round robin
- DNS are complex systems which can sometimes become a victim of DDoS attack

### Some DNS terms
- NS record (name server) - Specifies the DNS servers for your domain/subdomain
- MX record (mail exchange) - Specifies the mail servers for accepting messages
- A record (address) - Points a name to an IP address
- CNAME (canonical) - Points a name to another name or CNAME (example.com to www.example.com) or to an A record
-----------------

## Content delivery network (CDN)
- A CDN is a globally distributed network of proxy servers that serves content from location closer to the user
- Content are mostly static like HTML, CSS, JS, photo, video etc. But Amazon CDN can also serve dynamic content

### Types of CDN
1. Push CDN
- Developer has to upload content to CDN server whenever there are any new changes
- Minimizes traffic load to your server, but increases the storage on CDN server which can be costly

2. Pull CDN
- When user requests content for the first time, it goes to your server directly
- This results in slower request until the content is cached on CDN
- A time-to-live (TTL) determines how long content is cached
- Useful for heavy traffic websites
- Minimal storage on CDN
-----------------

## Load balancer
- Distribute incoming client request to workers (application servers and databases)
- Prevents request going to unhealthy server
- SSL termination: Decrypt request and encrypt response to avoid expensive operation for servers
- Session persistence: Issue cookies and route a specific client's request to a particular instance. Web apps can outsource session persistence to load balancers

### Layer 4 vs Layer 7 load balancing
- Layer 4: Transport Layer
	- Involves source and destination IP addresses, ports etc. But not the content
- Layer 7: Application Layer
	- Involves content of header, message and cookies to open connection to selected server
	- Use case: Route video traffic to regular video hosting server and direct sensitive billing traffic to security-hardened servers

### Horizontal VS Vertical scaling
- Horizontal Scaling
	- Add more machines or clones
	- Database horizontal scaling: Partitioning of data where each node only contains part of the data. Cassandra, MongoDB

- Vertical Scaling
	- Add more RAM or memory on same machine
	- Database vertical scaling: Data resides in a single node. Scaling is done through multi-core. MySQL - Amazon RDS. Switching vertically from smaller to bigger machine in cloud
-----------------

## Reverse proxy (web server)
![Image](https://camo.githubusercontent.com/e88216d0999853426f72b28e41223f43977d22b7/687474703a2f2f692e696d6775722e636f6d2f6e3431417a66662e706e67)
- A webserver that acts as a mediator to public request and provides a unified interface
- Has multiple benefits:
	- Increased security: Hide information about backend servers and block blacklisted IPs
	- SSL termination
	- Caching
	- Serve static content
- Load balancer is normally used when you have multiple servers serving the same function
- Reverse proxy is used with just one web or application server for the benefits highlighted above
-----------------

## Application layer
- Web layer can be separated out from application layer
- Having a separate application or platform layer allows you to scale and configure independently
- Service Discovery tools like Consul and Zookeeper can be used to find services through registered names, addresses and ports
- Use case: Adding a new API results in adding application servers without necessarily adding additional web servers (for presentation)
-----------------

## Database
### Relational database management system (RDBMS)
- A relational database is a collection of data items organized in tables
- ACID is a set of properties of relation database transactions
	- Atomicity: Each transaction is all or nothing
	- Consistency: Any transaction will bring database from one valid state to another
	- Isolation: Transaction can run both concurrently as well as serially
	- Durability: A committed transaction will remain so

### Scaling a relational database
1. Master-slave replication
- Master: Read-write
- Slave: Read only which replicates data from master
- If master is offline, slave serves data in read-only mode until one of the slaves is promoted as Master
- Cons:
	- Additional logic is needed to promote a slave to a master
	- More writes leads to more slave replication which bogs down the read performance of slaves
2. Master-master replication
- Both masters serve read and writes
- Co-ordinates with each other on writes for replication
- If one master goes down, the other will continue to serve both reads and writes
- Cons:
	- Need load balancer to decide where to write
	- Increased write latency due to synchronization

3. Federation
- Federation splits up databases by function
- Split a single monolithic database into smaller databases like forums, users and products
- Reduces replication lag
- Smaller database result can fit in memory and improves cache locality
- Cons:
	- Joining data from two databases is more complex with a server link
	- Not effective if your schema requires huge functions or tables

4. Sharding
- Breaks a data into different subsets and distribute them into different databases
- For instance: Break User database into User[A-F], User[G-I] and User[X-Z]
- Similar to the advantages of Federation
- Cons:
	- Joining data from multiple shards is more complex
	- Data distribution can become lopsided in a shard

5. Denormalization
- Redundant copies of data are written in multiple tables to avoid expensive joins
- Attempts to improve read performance at the expense of some write performance
- In most systems Read:Write ratio is between 100:1 to 1000:1, so read query must be optimized to avoid complex database joins
- PostgreSQL and Oracle support materialized views which stores redundant information
- Cons:
	- Data is duplicated
	- A denormalized databases performs worse under heavy writes

6. SQL tuning
- Benchmarking and profiling can point to the optimization ideas
- Benchmark: Simulate high-load situations with tools such as `ab`
- Profile: Enable tools such as the `slow query log` to help track performance issues
- Tips:
	- Use `CHAR` instead of `VARCHAR` for fixed-length fields
	- Use `TEXT` for blog posts which supports boolean search
	- Use `DECIMAL` for currency to avoid floating point representation errors
	- Use location to object instead of large `BLOBS`
	- Set the `NOT NULL` constraint where applicable to improve search performance
	
### NoSQL
- NoSQL is a collection of data items represented in key-value store, document-store, wide column store or a graph database
- Lacks true ACID
- BASE properties:
	- Basically available: The system guarantees availability
	- Soft state: The state of system may change over time even without input
	- Eventual Consistency: System will be consistent over a period of time, given there is no input during that period

1. Key-value store
- Allows O(1) for read and write
- Often backed by memory or SSD
- Simple data models for rapidly changing data

2. Document-store
- All information for a given object is stroed in a document
- Document-store provide APIs or query language to search based on the internal structure of the document itself
- High flexibility and often used with occasionally changing data
- MongoDB, CouchDB

3. Wide column store
![Wide column store](https://camo.githubusercontent.com/823668b07b4bff50574e934273c9244e4e5017d6/687474703a2f2f692e696d6775722e636f6d2f6e3136694f476b2e706e67)
- Basic unit of data is a column (name/value) pair
- Columns can be grouped in column families which can be further grouped as super column family
- Offer high availability and high scalability
- HBase, Cassandra, BigTable

4. Graph database
- Each node is a record and each edge is a relationship between two nodes
- Offer high performance for data models with complex relationships like social network
- Relatively new and many graphs can only be accessed with REST APIs

### SQL or NoSQL
Reasons for SQL
- Structured data
- Strict schema
- Relational data
- Need for complex joins
- Transactions
- More established: developers, community, code, tools, etc
- Lookups by index are very fast

Reasons for NoSQL
- Semi-structured data
- Dynamic or flexible schema
- Non-relational data
- No need for complex joins
- Store many TB (or PB) of data
- Very data intensive workload

-----------------

## Cache
- Caching improves page load time and reduces load on servers and databases
- If dispatcher finds a previous result it returns that information to prevent the actual execution on your server/database
- Types of Caching
	- Client caching: Browsers
	- CDN caching
	- Web server caching: Reverse proxies can serve static and dynamic content directly
	- Database caching
	- Application caching: In-memory caches such as Memcached and Redis are key-value stores between your application and your data storage
	
### When to update the cache	
- You can only store a limited about of data in cache. So be careful when choosing cache update strategy
1. Cache-aside
	- Lazy loading where data is only cached upon request
	- Steps:
		- Look for entry in cache, if not found (cache miss)
		- Load entry from database
		- Add entry to cache
		- Return entry
2. Write-through
	- Cache acts as the primary data store, reading and writing data to it. Synchronously the cache reads/writes data into database
	- Steps:
		- Application adds/updates entry to cache
		- Cache synchronously writes entry to database
		- Return
3. Write-behind
	- Cache acts as the primary data store, reading and writing data to it. Asynchronously the cache reads/writes data into database
	- Steps:
		- Application adds/updates entry to cache
		- Cache asynchronously writes entry to database improving write performance
		- Return
4. Refresh-ahead
	- Cache is configured to be automatically refreshed prior to its expiration	

-----------------

## Asynchronism
- Asynchronism is useful to avoid the user waiting for a service
1. Message queues
	- Receive, hold and deliver messages
	- User is not blocked and job is processed in background
	- Redis, RabbitMQ, Amazon SQS
2. Task queues
	- Recieve tasks and their related data, runs them and delivers their results
	- Supports scheduling and runs intensive jobs in the background
	- Celery: Python support
-----------------

## Communication

-----------------

## Security
