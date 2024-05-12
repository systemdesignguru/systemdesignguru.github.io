# System Design Interview Guide

#### **1. [Feature Expectations [5 min]](#feature-expectations-5-min)**
   - [Use cases](#use-cases)
   - [Scenarios that will not be covered](#scenarios-that-will-not-be-covered)
   - [Who will use](#who-will-use)
   - [How many will use](#how-many-will-use)
   - [Usage patterns](#usage-patterns)

#### **2. [Estimations [5 min]](#estimations-5-min)**
   - [Throughput (QPS for read and write queries)](#throughput-qps-for-read-and-write-queries)
   - [Latency expected from the system (for read and write queries)](#latency-expected-from-the-system-for-read-and-write-queries)
   - [Read/Write ratio](#readwrite-ratio)
   - [Traffic estimates](#traffic-estimates)
       - [Write Traffic](#write-traffic)
       - [Read Traffic](#read-traffic)
   - [Storage estimates](#storage-estimates)
   - [Memory estimates](#memory-estimates)

#### **3. [Design Goals [5 min]](#design-goals-5-min)**
   - [Latency and Throughput requirements](#latency-and-throughput-requirements)
   - [Consistency vs Availability (Weak/strong/eventual => consistency | Failover/replication => availability)](#consistency-vs-availability-weakstrongeventual--consistency--failoverreplication--availability)

#### **4. [High-Level Design [10 min]](#high-level-design-10-min)**
   - [APIs for Read/Write Scenarios for Crucial Components](#apis-for-readwrite-scenarios-for-crucial-components)
   - [Database Schema](#database-schema)
   - [Basic Algorithm](#basic-algorithm)
   - [High-Level Design for Read/Write Scenario](#high-level-design-for-readwrite-scenario)

#### **5. [Deep Dive [15 min]](#deep-dive-15-min)**
   - [Scaling the algorithm](#scaling-the-algorithm)
   - [Scaling individual components](#scaling-individual-components)
       - [Availability, Consistency, and Scale Story](#availability-consistency-and-scale-story)
       - [Consistency and Availability Patterns](#consistency-and-availability-patterns)
   - [Component considerations](#component-considerations)
       - [DNS](#dns-domain-name-system)
       - [CDN (Content Delivery Network)](#cdn-content-delivery-network)
       - [Load Balancers](#load-balancers)
       - [Reverse Proxy](#reverse-proxy)
       - [Application Layer Scaling](#application-layer-scaling)
       - [DB (RDBMS, NoSQL)](#db-rdbms-nosql)
            - RDBMS: Master-slave, Master-master, Federation, Sharding, Denormalization, SQL Tuning
            - NoSQL: Key-Value, Wide-Column, Graph, Document
                - Fast-lookups:
                    - RAM (Bounded size) => Redis, Memcached
                    - AP (Unbounded size) => Cassandra, RIAK, Voldemort
                    - CP (Unbounded size) => HBase, MongoDB, Couchbase, DynamoDB
        - [Caches](#caches)
            - Client caching, CDN caching, Webserver caching, Database caching, Application caching, Cache @Query level, Cache @Object level
            - Eviction policies: Cache aside, Write through, Write behind, Refresh ahead
        - [Asynchronism](#asynchronism)
            - Message queues
            - Task queues
            - Back pressure
        - [Communication](#communication)
            - TCP, UDP, REST, RPC

#### **6. [Justify [5 min]](#justify-5-min)**
   - [Throughput of Each Layer](#throughput-of-each-layer)
   - [Latency Caused Between Each Layer](#latency-caused-between-each-layer)
   - [Overall Latency Justification](#overall-latency-justification)

#### **7. [Key Metrics to Measure [3 min]](#key-metrics-to-measure-3-min)**
   - [Search Hit Ratio](#search-hit-ratio)
   - [Search Queries per Minute](#search-queries-per-minute)
   - [Blank Search Results](#blank-search-results)

#### **8. [System Health Monitoring [3 min]](#system-health-monitoring-3-min)**
   - [Measuring App Index](#measuring-app-index)
   - [Latency of Microservices](#latency-of-microservices)

#### **9. [Log Systems [2 min]](#log-systems-2-min)**
   - [ELK (Elasticsearch, Logstash, Kibana)](#elk-elastic-logstash-kibana)
   - [Logtail](#logtail)

#### **10. [Security [2 min]](#security-2-min)**
   - [Firewall](#firewall)
   - [Encryption at Rest and Transit](#encryption-at-rest-and-transit)
   - [TLS (Transport Layer Security)](#tls-transport-layer-security)
   - [Authentication and Authorization (AuthN/Z)](#authentication-and-authorization-authnz)
   - [Limited Egress/Ingress](#limited-egressingress)
   - [Principle of Least Privilege](#principle-of-least-privilege)


</br>
</br>

# Concepts Explanation
## Feature Expectations [5 min]

In system design interviews, discussing feature expectations involves understanding the requirements and constraints of the system being designed. Here's how each aspect can be addressed:

### Use cases

- Identify and describe various scenarios where the system will be utilized.
- Discuss the primary functionalities and interactions that users expect from the system.
- Provide concrete examples of how users will interact with the system to achieve specific goals.

### Scenarios that will not be covered

- Acknowledge any limitations or edge cases that the system may not handle.
- Explain the reasons behind excluding certain scenarios, such as complexity, resource constraints, or low priority.
- Emphasize the importance of prioritizing features that align with the core objectives of the system.

### Who will use

- Define the target audience or user personas who will benefit from using the system.
- Discuss the demographics, behaviors, and needs of different user segments.
- Consider any external systems or third-party integrations that users may interact with in conjunction with the system.

### How many will use

- Estimate the expected user base or workload the system needs to support.
- Discuss factors such as concurrent users, peak usage times, and potential growth projections.
- Address scalability requirements to accommodate increases in user traffic over time.

### Usage patterns

- Analyze the typical usage patterns and behaviors of users interacting with the system.
- Consider factors such as frequency of usage, session duration, and common actions performed.
- Discuss how understanding usage patterns informs design decisions related to performance, scalability, and user experience optimization.

During the interview, articulate each aspect clearly, providing rationale behind design choices and considering trade-offs between different features. Demonstrating a comprehensive understanding of feature expectations showcases both technical proficiency and problem-solving skills in system design interviews.

</br>

## Estimations [5 min]

In system design interviews, demonstrate a strong understanding of system design principles and their practical application to real-world scenarios. Specifically, candidates should be able to:

### Throughput (QPS for read and write queries)
   - Throughput refers to the number of queries or operations a system can handle per second (QPS).
   - For read queries, it indicates how many read operations the system can perform in a second.
   - For write queries, it represents the number of write operations the system can handle in a second.
   - Throughput is a crucial metric for evaluating system performance and scalability.

### Latency expected from the system (for read and write queries)
   - Latency refers to the time delay between initiating a request and receiving a response.
   - Expected latency for read queries is the time it takes for the system to retrieve data in response to a read request.
   - Expected latency for write queries is the time it takes for the system to process and store data in response to a write request.
   - Lower latency is desirable as it indicates faster response times and better user experience.

### Read/Write ratio
   - The read/write ratio is the proportion of read operations to write operations in the system.
   - It indicates the balance between reading existing data and writing new data.
   - Understanding the read/write ratio helps in capacity planning, resource allocation, and optimizing system performance.

### Traffic estimates
   #### Write Traffic
   - Write traffic estimates include the number of write queries per second (QPS) and the volume of data generated by write operations.
   - It helps in determining the system's capacity to handle incoming write requests and the storage required for storing the written data.
   
   #### Read Traffic
   - Read traffic estimates include the number of read queries per second (QPS) and the volume of data read by these queries.
   - It helps in understanding the read workload and ensuring that the system can efficiently retrieve data for read requests.

### Storage estimates
   - Storage estimates refer to the amount of storage space required to store data generated by the system.
   - It includes estimating the total volume of data to be stored and selecting appropriate storage solutions (e.g., disk, SSD, cloud storage) to accommodate the data.
   - Storage estimates are essential for capacity planning and budgeting purposes.

### Memory estimates
   - Memory estimates involve determining the amount of memory (RAM) required by the system to operate efficiently.
   - For caching systems, memory estimates include deciding what data to store in cache and calculating the RAM requirements accordingly.
   - It also involves estimating the number of machines needed to achieve the desired memory capacity.
   - Additionally, memory estimates may include the amount of data to be stored in disk or SSD in case of overflow from memory-based caches.

</br>

## Design Goals [5 min]
In system design interviews, discussing design goals is essential for defining the system's objectives and guiding architectural decisions. Here's how each aspect can be addressed:

### Latency and Throughput Requirements:
- ***Latency:*** Refers to the time it takes for a system to respond to a request, measured from request initiation to response receipt. Lower latency signifies faster response times and better user experience.
- ***Throughput:*** Represents the rate at which a system can process requests within a specific time frame, often measured in queries per second (QPS). Higher throughput implies greater processing capacity.

   - ***Define Targets:*** Set desired latency targets for read and write operations, indicating maximum acceptable response times.
   - ***Specify Throughput:*** Establish throughput requirements in terms of QPS for both read and write operations.
   - ***Considerations:*** Take into account user expectations, system performance, and scalability when setting latency and throughput goals.

### Consistency vs Availability:
- **Consistency:** Ensures uniformity of data across distributed system nodes. Strong consistency guarantees all nodes see the same data simultaneously, while eventual consistency allows temporary discrepancies between replicas.
- **Availability:** Reflects the system's ability to remain operational despite failures or disruptions. High availability ensures users can access the system even during component failures.

   - **Determine Levels:** Decide on the desired consistency and availability levels for the system.
   - **Discuss Models:** Explore different consistency models (strong, eventual) and their implications.
   - **Evaluate Trade-offs:** Assess trade-offs between consistency and availability, considering factors such as data integrity, fault tolerance, and system responsiveness.
   - **Explore Strategies:** Investigate replication, failover mechanisms, and distributed consensus protocols for achieving consistency and availability goals.

### Performing in Interviews
In system design interviews, candidates should:
- Clearly define goals for latency, throughput, consistency, and availability based on project requirements.
- Understand trade-offs between these objectives to design scalable, reliable systems.
- Articulate the rationale behind latency, throughput targets, and consistency vs. availability trade-offs to align system architecture with project needs.

</br>


## High-Level System Design [10 min]

In system design interviews, outlining the high-level design is crucial for understanding how various components interact to fulfill system requirements. Here's how to approach each aspect:

### APIs for Read/Write Scenarios for Crucial Components:
   - Define APIs for interacting with crucial components such as data storage, processing, and retrieval.
   - Specify endpoints, request parameters, and response formats for read and write operations.
   - Consider RESTful principles for designing clear and intuitive APIs.

### Database Schema:
   - Design a database schema that reflects the structure of the data and supports required functionalities.
   - Define tables, columns, relationships, and indexes based on data access patterns and query requirements.
   - Optimize the schema for efficient data storage, retrieval, and manipulation.

### Basic Algorithm
   - Develop basic algorithms for key system operations such as data insertion, retrieval, update, and deletion.
   - Consider algorithmic complexity, scalability, and performance implications.
   - Use appropriate data structures and algorithms to meet system requirements.

### High-Level Design for Read/Write Scenario:
   - Outline the flow of read and write operations within the system architecture.
   - Describe how data is processed, stored, and retrieved in response to read and write requests.
   - Identify components involved in handling read and write scenarios and their interactions.

During the interview, discuss each aspect of the high-level design in a structured manner, focusing on clarity, coherence, and alignment with system requirements. Illustrate your understanding of system architecture and design principles by providing concrete examples and explaining design decisions.

</br>

## Deep Dive [15 min]

Discuss techniques to scale algorithms efficiently, considering factors like data size, computational complexity, and resource utilization.

###  Scaling Individual Components:
  - ***Availability, Consistency, and Scale Story:*** Analyze availability and consistency requirements for each component, outlining scaling strategies to meet increasing demands.
  - ***Consistency and Availability Patterns:*** Explore consistency models (strong, eventual) and availability patterns (failover, replication) suitable for each component.

### Component Considerations:
  - #### DNS (Domain Name System):
    - DNS is a hierarchical decentralized naming system for computers, services, or other resources connected to the Internet or a private network. It translates domain names (e.g., www.example.com) to IP addresses, enabling users to access websites using human-readable names.
    - Explain the role of DNS in system architecture and considerations for scalability and fault tolerance.

  - #### CDN (Content Delivery Network)
    - A CDN is a distributed network of servers strategically deployed in multiple locations worldwide to deliver web content (such as images, videos, scripts) more efficiently to users. CDNs cache content closer to the user's location, reducing latency and bandwidth consumption.
        1. **Push CDN:**
        - In a push CDN, content is uploaded and pushed to edge servers in the CDN network in advance, regardless of whether there is an immediate request from users.
        - Content providers typically determine which content to push and when to push it based on factors like popularity, anticipated demand, or scheduled updates.
        - Push CDN is suitable for scenarios where content is relatively static, and the provider has control over content distribution.

        2. **Pull CDN:**
        - In a pull CDN, content is only fetched from the origin server and cached at edge servers when there is a user request for that content.
        - When a user requests content, the edge server checks if it has a cached copy. If not, it pulls the content from the origin server, caches it locally, and serves it to the user.
        - Pull CDN is suitable for scenarios with dynamic content or where the origin server frequently updates content. It optimizes resource usage by caching only the content that is requested by users.
    - Compare push and pull CDN architectures and their impact on content distribution and system performance.

  - #### Load Balancers:
    - Load balancers distribute incoming network traffic across multiple servers or resources to ensure optimal resource utilization, maximize throughput, minimize response time, and avoid overload on any single server. They can operate at different layers of the OSI model, including Layer 4 (transport layer) and Layer 7 (application layer), Active-Passive, Active-Active.
    - Evaluate load balancing strategies and technologies to achieve high availability, scalability, and efficient traffic distribution.

  - #### Reverse Proxy: 
    - A reverse proxy sits between clients and servers, intercepting requests from clients and forwarding them to the appropriate backend servers. It enhances security, improves performance, and provides features like SSL termination, caching, and load balancing.
    - Discuss the use of reverse proxies for enhancing security, performance, and scalability of web applications.

  - #### Application Layer Scaling:
    - ***Microservices:*** Microservices architecture involves breaking down complex software applications into smaller, independent services that can be developed, deployed, and scaled independently. Each microservice typically focuses on a specific business capability and communicates with other services through well-defined APIs.
    - ***Service Discovery:*** Service discovery is the process of automatically locating and registering services in a distributed system. It enables dynamic, runtime discovery of available services, facilitating communication between microservices and ensuring resilience in dynamic environments.
    - Explore microservices architecture and service discovery mechanisms for horizontal scaling and fault isolation.

  - #### DB (RDBMS, NoSQL):
    - **RDBMS (Relational Database Management System):** 
        - RDBMS is a type of database management system based on the relational model. It organizes data into tables with rows and columns, enforces ACID properties (Atomicity, Consistency, Isolation, Durability), and supports SQL for querying and manipulating data.
        
        - **Master-slave replication:**
            - In master-slave replication, one database server (the master) is designated as the primary source of truth for data changes, while one or more other servers (the slaves) replicate data from the master.
            - The master receives write operations and propagates these changes to the slave servers asynchronously.
            - Slave servers can handle read operations, offloading read traffic from the master and providing scalability and fault tolerance.

        - **Master-master replication:**
            - In master-master replication (also known as dual-master replication), multiple database servers act as both master and slave simultaneously.
            - Each server can accept write operations, and changes made to any master are asynchronously propagated to other masters.
            - Master-master replication provides high availability and load balancing for both read and write operations but requires careful conflict resolution mechanisms to handle conflicts that may arise when the same data is modified on different masters.

        - **Federation:**
            - Federation involves distributing data across multiple database instances or servers while presenting a unified interface to clients.
            - Each database instance manages a subset of the overall data, typically partitioned by some criteria such as geographical location, user type, or data type.
            - Federation allows for horizontal scaling and can improve performance by distributing workload across multiple servers.

        - **Sharding:**
            - Sharding is a technique for horizontal partitioning of data across multiple database instances or servers (shards) to improve scalability and performance.
            - Data is divided into shards based on a shard key or partition key, and each shard is stored on a separate server.
            - Sharding distributes both the data storage and processing workload, allowing the system to handle larger volumes of data and higher query loads.

        - **Denormalization:**
            - Denormalization is the process of optimizing database performance by reducing the number of joins needed to retrieve data at the expense of data redundancy.
            - It involves storing redundant copies of data or aggregating data into fewer tables to eliminate the need for complex joins.
            - Denormalization can improve query performance, especially for read-heavy workloads, but it may increase storage requirements and introduce data consistency challenges.

        - **SQL Tuning:**
            - SQL tuning involves optimizing the performance of SQL queries and database operations to improve overall system performance.
            - Techniques for SQL tuning include optimizing query execution plans, creating appropriate indexes, rewriting inefficient queries, and tuning database configuration parameters.
            - SQL tuning aims to reduce query response times, minimize resource consumption, and enhance overall database performance.

    - Discuss strategies like master-slave, master-master, federation, sharding, denormalization, and SQL tuning for scaling relational databases.

    - **NoSQL:** 
      - NoSQL (Not Only SQL): NoSQL encompasses a wide range of database technologies designed to handle large volumes of unstructured or semi-structured data. NoSQL databases offer flexible schemas, horizontal scalability, and high availability, making them suitable for distributed, non-relational data storage.
      - Compare key-value, wide-column, graph, and document database models, highlighting their suitability for different use cases.
      - **Fast Lookups:**
        - ***RAM (Random Access Memory):*** 
            - RAM is a type of computer memory that stores data and machine code currently being used by the CPU. It provides fast, temporary storage for active programs and data, enabling quick access and retrieval of information.
            - Describe in-memory caching solutions like Redis and Memcached for fast, low-latency data access.

        - ***AP (Available and Partition-tolerant):*** 
            - AP databases prioritize availability and partition tolerance over consistency. They ensure that data remains available even in the presence of network partitions or failures, sacrificing strong consistency guarantees in favor of responsiveness and fault tolerance.
            - Explore AP databases like Cassandra, RIAK, and Voldemort for distributed, highly available architectures.

        - ***CP (Consistent and Partition-tolerant):*** 
            - CP databases prioritize consistency and partition tolerance over availability. They guarantee strong consistency across distributed systems, ensuring that all nodes see the same data at the same time, even in the presence of network partitions or failures.
            - Discuss CP databases such as HBase, MongoDB, Couchbase, and DynamoDB for consistency and partition tolerance at scale.

  - #### Caches:
    - Caches are temporary storage locations that store frequently accessed data to reduce access latency and improve performance. They can exist at various levels of the system stack, including client-side, CDN, web server, database, and application layers.
    - Eviction Policies: Eviction policies determine how cache entries are removed or replaced when the cache reaches its capacity limit. Common eviction policies include cache-aside, write-through, write-behind, and refresh-ahead, each optimizing cache performance and behavior in different scenarios.
    - Message Queues: Message queues are communication channels that decouple components in a distributed system by allowing them to exchange messages asynchronously. They enable reliable, scalable, and loosely coupled communication between producers and consumers, supporting features like message persistence, routing, and prioritization.
    - Task Queues: Task queues manage asynchronous processing of background jobs or tasks within an application. They enable efficient execution and scaling of time-consuming or resource-intensive tasks, ensuring optimal resource utilization and responsiveness.
    - Explain various caching layers (client, CDN, web server, database, application) and caching strategies (query level, object level).
    - Discuss eviction policies (cache aside, write through, write behind, refresh ahead) to optimize cache performance and resource utilization.

  - #### Asynchronism:
    - Back Pressure: Back pressure is a flow control mechanism used in distributed systems to prevent overload or congestion by regulating the rate of data transmission or processing. It allows systems to gracefully handle bursts of incoming requests or workload spikes without causing performance degradation or failure.
    - Explore message queues and task queues for asynchronous communication and workload decoupling, managing back pressure to prevent overload.


  - #### Communication:
    - Communication Protocols: Communication protocols define rules and conventions for exchanging data between systems or components. TCP (Transmission Control Protocol) and UDP (User Datagram Protocol) are transport-layer protocols that provide reliable, connection-oriented and connectionless communication, respectively. REST (Representational State Transfer) and RPC (Remote Procedure Call) are architectural styles for designing web services and distributed systems, each with its own characteristics and trade-offs.
    - Compare communication protocols (TCP, UDP) and architectural styles (REST, RPC) for efficient data exchange and service interaction within the system.

</br>

## Justify [5 min]

- #### Throughput of Each Layer:
  Evaluate the throughput (requests processed per second) at each layer of the system architecture to ensure optimal performance and resource utilization.

- #### Latency Caused Between Each Layer:
  Analyze the latency introduced between each layer of the system architecture to identify potential bottlenecks and optimize data flow for minimal delay.

- #### Overall Latency Justification:
  Justify the overall latency of the system by considering the cumulative effect of latency introduced at each layer, ensuring that it meets the defined performance requirements and user expectations.

</br>

## Key Metrics to Measure [3 min]

- In designing a search system, key metrics may include:
  - #### Search Hit Ratio: 
    - Ratio of successful search queries to total search queries, indicating the effectiveness of the search algorithm and index.
  - #### Search Queries per Minute: 
    - Rate of incoming search queries, providing insights into system load and user activity.
  - #### Blank Search Results: 
    - Number of search queries resulting in no matching results, highlighting potential areas for improvement in indexing or query optimization.
- For example, in the case of designing a search system, we will be interested to see how many keywords are resulting in blank search results. Here the ratio of search hits is important so we also need to know the number of search queries we are receiving per minute.
- Tools like Grafana with Prometheus, AppDynamics, etc., can be utilized for monitoring and visualizing these metrics in real-time, enabling proactive system management and performance optimization.


</br>

## System Health Monitoring [3 min]

- Monitoring the health of the system involves:
  - #### Measuring App Index: 
    - App index measurement involves assessing the status and progress of indexing processes within the system. It ensures that all relevant data is properly indexed and available for retrieval. Monitoring app indexing helps maintain data consistency and completeness, crucial for accurate search results and efficient data access.
    - Tracking the indexing process to ensure data consistency and completeness.

  - #### Latency of Microservices: 
    - Monitoring the latency of microservices involves tracking the response time of individual microservices within the system. It measures the duration between sending a request to a microservice and receiving the corresponding response. High latency can indicate performance issues, such as network congestion, resource contention, or inefficient code execution. Monitoring microservices latency helps identify bottlenecks and optimize service performance to ensure timely responses to user requests.
    - Monitoring the response time of microservices to detect performance bottlenecks and ensure optimal service delivery.

- Tools like New Relic and AppDynamics offer comprehensive monitoring solutions for analyzing application performance, identifying issues, and optimizing system health.
    - New Relic and AppDynamics are leading application performance monitoring (APM) solutions that offer comprehensive monitoring capabilities for measuring and analyzing system health. These tools provide real-time insights into application performance, infrastructure metrics, and user experience, enabling organizations to identify and resolve performance issues proactively. They offer features such as transaction tracing, code-level diagnostics, anomaly detection, and customizable dashboards for monitoring and optimizing application performance.


</br>

## Log Systems [2 min]

- Log systems, such as ELK (Elasticsearch, Logstash, Kibana) or Logtail, are essential for:
    - #### ELK (Elasticsearch, Logstash, Kibana):
        - ***ELK stack:*** ELK stack is a popular open-source log management platform used for centralized logging, log analysis, and visualization.
        - ***Elasticsearch:*** Elasticsearch is a distributed search and analytics engine that stores and indexes log data for fast and scalable search capabilities.
        - ***Logstash:*** Logstash is a data collection pipeline that ingests, processes, and forwards log data from various sources to Elasticsearch.
        - ***Kibana:*** Kibana is a data visualization and exploration tool that provides a user-friendly interface for querying and analyzing log data stored in Elasticsearch. It offers features like dashboards, visualizations, and real-time monitoring for gaining insights into system behavior and performance.
    - ***Logtail:*** Logtail is a log management and analysis tool provided by Alibaba Cloud. It offers log collection, storage, search, and analysis capabilities, allowing organizations to monitor and troubleshoot their systems effectively. Logtail supports various log formats and provides features like real-time log streaming, keyword searching, and log visualization for comprehensive log management.

  - ***Collecting Logs:*** Gathering and centralizing log data from various components of the system.
  - ***Analysis and Visualization:*** Analyzing log data to gain insights into system behavior, troubleshoot issues, and detect anomalies.
  - ***Monitoring and Alerting:*** Setting up alerts based on predefined thresholds or patterns to proactively identify and address issues.

- These log management platforms provide powerful capabilities for searching, filtering, and visualizing log data, enabling efficient log management and analysis.

</br>

## Security [2 min]

Security measures are crucial for protecting the system from threats and vulnerabilities:
### Firewall: 
- A firewall is a network security device that monitors and controls incoming and outgoing network traffic based on predetermined security rules. It acts as a barrier between a trusted internal network and untrusted external networks, preventing unauthorized access and protecting against malicious attacks.
- Implementing firewalls to control incoming and outgoing network traffic, safeguarding against unauthorized access and malicious attacks.

### Encryption at Rest and Transit: 
- Encryption is the process of encoding data to make it unreadable to unauthorized users.
    - ***Encryption at Rest:*** Encrypting data stored in databases, files, or other storage mediums to protect it from unauthorized access if the storage device is compromised.
    - ***Encryption in Transit:*** Encrypting data as it travels between client and server or between different components of the system to prevent interception and eavesdropping by unauthorized parties.
- Encrypting data both at rest (stored data) and in transit (data transmission) to prevent unauthorized access and maintain data confidentiality.

### TLS (Transport Layer Security): 
- TLS is a cryptographic protocol that ensures secure communication over a computer network. It encrypts data transmitted between clients and servers to prevent eavesdropping, tampering, and forgery of data. TLS is widely used to secure web browsing, email, instant messaging, and other network communications.
- Securing communication channels by implementing TLS protocols to encrypt data exchanged between clients and servers.

### Authentication and Authorization (AuthN/Z): 
- Authentication verifies the identity of users or systems attempting to access the system, while authorization determines the permissions and privileges granted to authenticated users.
    - ***Authentication:*** Verifying user identities through credentials such as passwords, biometrics, or digital certificates.
    - ***Authorization:*** Enforcing access control policies to restrict user access to authorized resources based on their roles, permissions, and privileges.
- Implementing robust authentication mechanisms to verify user identities and authorization policies to control access to resources based on user roles and permissions.

### Limited Egress/Ingress: 
- Egress and ingress filtering control the flow of network traffic in and out of a network or system.
    - ***Egress Filtering:*** Restricting outbound traffic leaving the network to prevent data leakage and unauthorized communication with malicious or unauthorized destinations.
    - ***Ingress Filtering:*** Filtering incoming traffic entering the network to block malicious or unwanted traffic, such as denial-of-service (DoS) attacks, malware, or unauthorized access attempts.
- Restricting outbound (egress) and inbound (ingress) network traffic to specific destinations and sources, reducing the attack surface and mitigating security risks.

### Principle of Least Privilege: 
- The principle of least privilege (PoLP) is a security best practice that advocates granting users or systems only the minimum level of access and permissions required to perform their tasks. It reduces the risk of privilege escalation, unauthorized access, and data breaches by limiting the scope of access to sensitive resources and functionalities.