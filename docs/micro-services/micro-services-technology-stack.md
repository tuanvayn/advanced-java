# Microservices technology stack

## Technology stack

### Microservices development

Role: Rapid development of services.

-   Spring
-   Spring MVC
-   Spring Boot

[Spring](https://spring.io/) Currently an essential framework for Java Web developers, SpringBoot simplifies the configuration of Spring development and is now the mainstream development framework in the industry.

### Microservices registration discovery

Role: Discovery service, registration service, centralized management service.

#### Eureka

-   Eureka Server : The service registration service is provided, and after each node is started, it is registered in the Eureka Server.
-   Eureka Client : Simplify interaction with Eureka Server.
-   Spring Cloud Netflix : [GitHub](https://github.com/spring-cloud/spring-cloud-netflix)，[documentation](https://cloud.spring.io/spring-cloud-netflix/reference/html/)

#### Zookeeper

> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services.

[Zookeeper](https://github.com/apache/zookeeper) is a centralized service that maintains configuration information, naming, provides distributed synchronization, and provides group services.

#### The difference between Zookeeper and Eureka

Zookeeper guarantees CP, Eureka guarantees AP:

-   C: Data consistency;
-   A: Service availability;
-   P: The fault tolerance of the service to network partition failures, these three characteristics cannot be met at the same time in any distributed system, at most two at the same time.

### Microservices configuration management

Function: Unified management of configuration information of one or more services, centralized management.

#### [Disconf](https://github.com/knightliao/disconf)

Distributed Configuration Management Platform (distributed configuration management platform), which is a common component platform focusing on the configuration management of various distributed systems, provides unified configuration management services, and is a complete set of distributed configuration unified solutions based on zookeeper.

#### [SpringCloudConfig](https://github.com/spring-cloud/spring-cloud-config)

#### [Apollo](https://github.com/ctripcorp/apollo)

Apollo is a distributed configuration center developed by Ctrip's framework department, which can centrally manage the configuration of different environments and clusters of applications, and can be pushed to the application side in real time after configuration modification, and has standardized permissions, process governance and other characteristics for microservice configuration management scenarios.

### Permission authentication

Function: According to the security rules or security policies set by the system, users can access and only access their authorized resources, no more, no less

#### [Spring Security](https://spring.io/projects/spring-security)

#### [Apache Shiro](http://shiro.apache.org/)

> Apache Shiro™ is a powerful and easy-to-use Java security framework that performs authentication, authorization, cryptography, and session management. With Shiro’s easy-to-understand API, you can quickly and easily secure any application – from the smallest mobile applications to the largest web and enterprise applications.

### Batch processing

What it does: Batch process data or things of the same type

#### [Spring Batch](https://spring.io/projects/spring-batch)

### Timed tasks

> What it does: What to do regularly.

#### [Quartz](http://www.quartz-scheduler.org/)

### Microservice invocation (protocol)

> Communication protocol

#### Rest

-   Send REST requests over HTTP/HTTPS for data interaction

#### RPC

-   Remote Procedure Call
-   It is a protocol that requests services from remote computer programs over a network without requiring knowledge of the underlying network technology. RPC does not depend on specific network transport protocols, TCP, UDP, etc. can be.

#### [gRPC](https://www.grpc.io/)

> A high-performance, open-source universal RPC framework

The so-called RPC (Remote Procedure Call) framework actually provides a mechanism for applications to communicate with each other, and also follows the serverclient model. When used, the client calls the interface provided by the server as if it were calling a local function.

#### RMI

-   Remote Method Invocation
-   Pure Java calls

### Service interface call

> Role: Communication between multiple services

#### [Feign(HTTP)](https://github.com/OpenFeign/feign)

Spring Cloud Netflix's microservices are exposed in the form of HTTP interfaces, so they can be called with Apache's HttpClient or Spring's RestTemplate, and Feign is a more convenient HTTP client to use, using it like calling its own engineering methods, without feeling like calling remote methods.

### Service circuit breaker

> What it does: Do not allow the request to continue when it reaches a certain threshold.

#### [Hystrix](https://github.com/Netflix/Hystrix)

> Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.

#### [Sentinel](https://github.com/alibaba/Sentinel)

> A lightweight powerful flow control component enabling reliability and monitoring for microservices. (Lightweight flow control、Circuit breaker downgrades Java libraries)

### Load balancing for services

> Function: Reduce service pressure and increase throughput

#### [Ribbon](https://github.com/Netflix/ribbon)

> Spring Cloud Ribbon is a client-side load balancing tool based on HTTP and TCP that is based on the Netflix Ribbon

#### [Nginx](https://github.com/nginx/nginx)

Nginx (engine x) is a high-performance http and reverse proxy web server that also provides IMAP/POP3/SMTP services

#### Nginx is different from ribbon

Nginx belongs to server-side load balancing, and ribbon belongs to client-side load balancing. Nginx acts with Tomcat, and Ribbon acts with calls between individual services (RPC).

### Message Queuing

> Function: Decouple the business and process data asynchronously

#### [Kafka](http://kafka.apache.org/)

#### [RabbitMQ](https://www.rabbitmq.com/)

#### [RocketMQ](http://rocketmq.apache.org/)

#### [activeMQ](http://activemq.apache.org/)

### Log collection (elk)

> Function: Collects the logs of each service to provide log analysis and user portraits

#### [Elasticsearch](https://github.com/elastic/elasticsearch)

#### [Logstash](https://github.com/elastic/logstash)

#### [Kibana](https://github.com/elastic/kibana)

### API gateway

> Function: External requests are intercepted through API Gateway and forwarded to the real service

#### [Zuul](https://github.com/Netflix/zuul)

> Zuul is a gateway service that provides dynamic routing, monitoring, resiliency, security, and more.

### Service monitoring

> Function: Display the operation status of each service (CPU, memory, access, etc.) in a visual or non-visual form

#### [Zabbix](https://github.com/jjmartres/Zabbix)

#### [Nagios](https://www.nagios.org/)

#### [Metrics](https://metrics.dropwizard.io)

### Service link tracing

> Role: Clarify the call relationship between services

#### [Zipkin](https://github.com/openzipkin/zipkin)

#### [Brave](https://github.com/openzipkin/brave)

### Data storage

> What it does: Stores data

#### Relational databases

##### [MySql](https://www.mysql.com/)

##### [Oracle](https://www.oracle.com/index.html)

##### [MsSQL](https://docs.microsoft.com/zh-cn/sql/?view=sql-server-ver15)

##### [PostgreSql](https://www.postgresql.org/)

#### Non-relational databases

##### [Mongodb](https://www.mongodb.com/)

##### [Elasticsearch](https://github.com/elastic/elasticsearch)

### cache

> What it does: Stores data

#### [redis](https://redis.io/)

### Partition databases and tables

> Function: Database database sharding and table sharding scheme.

#### [ShardingSphere](http://shardingsphere.apache.org/)

#### [Mycat](http://www.mycat.io/)

### Service deployment

> Role: Quickly deploy, go online, and continuously integrate the project.

#### [Docker](http://www.docker.com/)

#### [Jenkins](https://jenkins.io/zh/)

#### [Kubernetes(K8s)](https://kubernetes.io/)

#### [Mesos](http://mesos.apache.org/)
