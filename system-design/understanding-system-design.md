## Distributed Systems 

- Group of systems working together in tandem 
- Keep the system simple (avoid distributed systems wherever possible)


### Fallacies 

- Reliability of network
- Security of network
- Bandwidth requirement changes 
- Topologies change
- Latencies need to be managed 

### Characteristics 

- No shared clock
- No shared memory
- Shared resources 
- Concurrency and consistency 

### Error Sources 

- Client cannot find the server 
- Server crashes mid request 
- Server response is lost 
- Client crashes 


## System Design Performance Metrics 

### Scalability 

- Ability of a system to grow and manage increased traffic 
- Increased volume of data or requests handling 

### Reliability

- Probability of a system failing during a a period of time 
- Harder to define for software reliability (bugs, deployment issues)
- Mean time between failure is used metrics 

### Availability 

- Amount of time system is operational in an interval 
- Poor design leads to low availability
- 99.999 (5 9's rule) - 5.26 minutes downtime in an year 
- Redundancy ensures more availability

### Reliability vs Availability 

- Reliable systems are always available (availability is subset of reliability)
- Availability can be solved with redundancy, same is not true for reliability
- Fleet of planes solves availability but a plane flying reliably is another major requirement 

### Efficiency 

- Throughput 
- Latencies 

### Manageability 

- Speed and difficulty involved in maintaining system 
- Identification and deployment processes should be easy 
- Data consistency need to looked into in detail 


## Numbers a programmer must know 

### Latency numbers 

| Resource    | Time             |
|-------------|------------------|
| CPU Cycles  | 0.3 ns           |
| CPU L1      | 1 ns             |
| CPU L2      | 3 ns             |
| CPU L3      | 3 ns             |
| Main Memory | 120 ns           |
| SSD         | 150 microseconds |
| HDD         | 10 ms            |
| SF to NYC   | 40 ms            |

- Avoid network calls 
- Keep frequently accessed data in memory 
- Replicate for disaster recovery 
- Use CDNs to reduce latencies 
- Example: MapReduce IRL by Google 

### Quick Maths for capacity estimates 

- Data conversions (bit -> byte -> KB -> MB -> GB -> TB -> PB)
- Common data types
  - Char: 1 Byte 
  - Int: 4 Bytes 
  - Unix Timestamp: 4 Bytes


- Time conversions 
  - 3600 seconds/hour 
  - 86400 seconds/day
  - 2500000 seconds/month 

### Traffic Estimates

- Estimate of total requests an application will receive
- Average daily active users (DAU) X Average reads/writes per user 

### Memory 

- Read requests per day X Average request size X 0.2 
- 80 percent rule applies here (frequent reads account for 80 percent of memory which can be cached)
- Include replication factor 

### Bandwidth 

- Requests per day X Requests Size (all included)
- Average data served per second X No of seconds 

### Storage 

- Writes per day X Size of writes X Time to store data 


## Horizontal vs Vertical Scaling 

- CPU, Memory, I/O and Bandwidth are major bottlenecks encountered

### Vertical Scaling 

- Easier way to solve the problem and scale application by increasing the resources 
- Diminishing returns, as resource increases benefits are limited
- Single point of failure 

### Horizontal Scaling 

- Long term efficiency but complex to implement initially 
- Redundancy built in  
- Need load balancer 
- K8s, Docker, Hadoop 
- Example: Low latency in horizontal scaling using CDN 

## System Design Components 

### Load Balancers

- Balancing incoming traffic to multiple servers 
- Nginx, HAProxy - Software load balancers
- F5, Citrix - Hardware load balancers 

#### Load balancers routing methods 

- Round Robin 
- Least connections (useful for chat and streaming applications)
- Least response time 
- IP Hash (useful for stateful sessions)

#### L4 vs L7 

**Layer 4**

- Only access to TCP and UDP data and protocol
- Faster

**Layer 7** 

- Full access to HTTP data and protocol 
- SSL termination 
- Check for authentication 
- Smarter routing options 

#### Redundant LB systems 

- Have active and passive load balancers 
- Active and passive load balancers communicate between them 

### Caching 

- Improve performance of application
- Saves memory 
- 50-200x Faster
- Precalculate and cache data 
- More reads vs writes - perfect for caching 

#### Caching Layers 

- DNS 
- CDN 
- Application 
- Database 

#### Distributed Cache 

- Replicate and Shard across servers and locate proper server for each key 
- Active and Passive cache 

#### Cache Eviction 

- Preventing stale data 
- Caching most frequently used data 
- TTL (Time to live)
- LRU/LFU 
  - Least Recently Used 
  - Least Frequently Used 

#### Caching Strategies 

- Cache Aside 
- Read Through 
- Write Through 
- Write Back 

#### Cache Consistency 

- DB and cache should be in consistent state 
- Minimal latency 


### Database Scaling 

- Performance bottleneck usually 
- Most applications are read heavy, 95% 

#### Vertical Scaling 

- Fast processors and more memory 

#### Indexes 

- Speed up read performance 
- Writes become slow 

#### Denormalization 

- Add redundant data to tables by making flat table structure
- Consistency is the challenge
- Writes are slow

#### Connection pooling 

- Allow multiple application threads to use same database connection thread 

#### Caching 

- Cache sits infront of database
- Can't create cache for everything 

#### Relication 

- Create replica server to handle reads 
- Master-Slave architecture 
- Fault tolerance 

#### Sharding 

- Horizontal partitioning 
- schema of table stays same but across multiple databases 
- hotkeys is problem in which optimal sharding condition is a major task, e.g. Justin Bieber issue in Instagram 

#### Vertical partitioning

- divide schema into separate tables 
- most columns which are not needed while quering, e.g. employee and employee_details 

#### When to use NoSQL

- Less transaction heavy tasks 
- Consistency can be compromised 
- Team is well versed with NoSQL 
- Language support is present



