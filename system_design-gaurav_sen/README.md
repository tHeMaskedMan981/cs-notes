# System Design by Gaurav sen 

## Important concepts
- optimize processes and increase throughputs using the same resource
- Preparing before-hand at non-peak hours. Pre-processing
- Create backups and avoid single point failures
- hire more resources (Horizontal scaling)
- Master slave architectures (creating backups)
- Distributed systems
- Load Balancers
- Decoupling
- Logging and metrics
- Make your system extensible. Decoupled modules.


## Horizontal vs Vertical Scaling

Horizontal Scaling - 
- load balancer required
- resilient *
- network calls used (RPC) (slower)
- Data consistency issues
- scales well with increase in users *

Vertical Scaling - 
- no load balancing required
- single point of failure
- inter-process communication (faster) *
- consistent *
- Hardware limitations

best option - use a hybrid of both the solutions taking the best features(*) from both the models. 


## Load Balancing 

Lets assume we have N servers for running an application. If we have to balance the load we can have the following approach - 
- generate the request Id (rid) and send it to the server. 
- find sno = H(rid)%N, and send the request to the server with sno

sometimes the request id is not at random, and follows some structure. for example it can be the user id of a user. That means all the request from the user will go to the same server. 
In that case we can have some cache stored to make the response faster. 

Now if the number of servers increase, the new way to balance becomes - 
find sno_new = H(rid)%(N+1), and send to the respective server. 

This will result in changing the request location, thus making all the cache stored useless. 

Therefore the problem is not load balancing, but adding and removing servers. 
 
### Consistent Hashing

- Instead of linear hashing, use cicular hashing.
- arrange the range as 0 to M-1 in a circle. 
- Hash the server indexes as well and place them on the circle. Can use same or different hash function. If using different hash function, use %M. 
- Now hash the request, place it on the circle, and move clockwise. find the first server and that server will handle the request. 
- This approach works and can give a uniform distribution on avg, but can also lead to some skew results
- If adding a new server, hash it and place it in the circle. Only the server next in clockwise direction is affected, all the other servers receive the same requests. 
- If removing a server, only the server before it in anticlockwise direction is affected (load is increased). Other servers are not affected.
- But these 2 situations can lead to very low or very high load on a particular server, ie. skewed distribution. 
- We can use multiple hash functions, h1,h2,..,hk. all of these functions map to a value on a circle, and we follow the same approach for assigning requests. 
- In this case chances of the distribution getting skewed decreases. 

<Insert Image>

## Message Queues

The basic idea for message queues is asynchronous working. If a task is going to take longer, reply the client OK, and add the task to the queue. the client can use the time to do other tasks and spend the resources somewhere else. when the task is completed, you send the response to the client. 

This way, when you are working on a request, you can also register more requests from users. 

Features of message queues:
- Notifier - checks the servers for heartbeat. sends messages at regular intervals and if server is not responding, will distribute the its requests to live servers. To do that, the requests should be persistent, and therefore must be stored in the database.
- Load balancing - If new requests are coming, or a server goes down, these requests needs to be distributed among servers. One way is to get all the requests which are not completed, and distribute them among the live servers. When distributing, it should not change the distribution of already allocated requests. Therefore live servers should only get new requests, should not have their already requests changed. We can use consistent hashing to acheive this.  
- Persistant storage - the requests should be stored in a database so that they can be retrieved in the case of failures.

very important concept to learn. Example - RabbitMQ

## Microservices 

**Monolith architecture** - Having big machines handle the requests. They perform all the fuctions required to serve the client. These machines can scale horizontally. 

**Microservice architecture** - Here the whole service provided to the client is broken down into smaller parts, call microservices.  Each microservice contains all the functions, variables, data, etc. relevant to that particular function performed by the microservice. These microservices then interact with each other to complete the request. 


### Monolith architecture - 
- Advantages : 
    - **Better for small Teams** - If the team is small, monolith architecture is better, as it is faster to build and easy to maintain. 
    - **Simple System** - easy to deploy and run, as less complex system. 
    - **Less Duplication** - code for setting up tests, connections, etc. need not be duplicated to every service. everything at one place. 
    - **Faster** - No RPC calls. Interprocess calls are much faster.
- Disadvantages: 
    - **More Context Required** - A new member of the team need to know complete system before starting building. 
    - **More frequent deployements** - Any small change in the code leads to building the code again and deploying again.
    - **Testing is difficult** - Tests have to be written for much larger code. Complex and time taking.
    - **Single Point of Failure** - If even a small part fails, have to restart and test everthing. 


### Microservice architecture - 
- Advantages:
    - **Easier to Scale** - Can add horizontally multiple microservices, based on load. 
    - **Less Context Required** - A new member joining the team need not know everything. He just needs to focus on that particular service
    - **Parallel Development** - The microservices are seperated from each other through APIs and therefore need not worry about internal implementation. Therefore multiple services can be developed in parallel
    - **Easier to Reason About** - can scale up only the specific service getting highest load. Dont need to scale up the entire application. More streamlined and focused approach. 
- Disadvantages: 
    - Difficult to design
    - Need skilled architects
    - If a microservice is talking to only one microservice, they can be combined to make a bigger microservice.
    

## Database sharding

**Horizontal Partioning** - The data or requests is partitioned based on a key, which is an attribute of the data, across multiple database servers. 

**Vertical Partitioning** - The data is partitioned based on columns. 

Things to consider while doing sharding:
- consistency is important. 
- Availability
- deciding on the key

Problems:
- joins across shards
- inflexible - fixed number of shards. (consitent hashing can be used to make flexible). 
also, can use herarchial sharding. breaking a big slice into smaller ones. solves the problem of inflexibility.


should use indexing within the shards. it can be on a completely different attribute 
It makes the queries fast. read and write performance goes up 

to take care of failures, can have master slave architecture - 
- all the write requests go to the master. 
- The slaves poll the master to update themselves. 
- all the read requests can be distributed across slaves.
- if the master fails, the slaves choose a master among themselves. 

Quite tough to implement in practice

