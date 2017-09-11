# System Design
The System Design Primer

Author Info
-----------
Author: Andrew Gurung <br>
URL: http://www.andrewgurung.com/

Table of Contents
-----------------
- Performance vs scalability
- Latency vs throughput
- Availability vs consistency
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
- Asynchronism
