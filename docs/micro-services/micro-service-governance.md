# Microservice Governance Strategy

## Service Registration and Discovery

Problem Solving: Centrally Managed Services

Solution：

-   Eureka
-   Zookeeper

## load balancing

Problem Solving：Reduce server hardware pressure

Solution：

-   Nginx
-   Ribbon

## 通讯

Problem Solving：Communication bridge between various services

Solution：

-   REST（Synchronize）
-   RPC（Synchronize）
-   MQ（asynchronousp）

## configuration management

Problem Solving： With the increase of services, the configuration is also increasing, how to manage the configuration of each service.

Solution：

-   Nacos
-   Spring Cloud Config
-   Apollo

## Fault tolerance and service degradation

Problem Solving：In microservices, a request often involves invoking several services. If one of the services fails and there is no service fault tolerance, it is very likely that a series of services will be unavailable. This is the avalanche effect.

Solution：

-   Hystrix

## service dependencies

Problem Solving：Multiple services rely back and forth, and the startup relationship is not clear.

Solution：Apply layers.

## service documentation

Problem Solving：Reduce communication costs

Solution：

-   Swagger
-   Java doc

## Service Security Issues

Problem Solving：Security of Sensitive Data

Solution：

-   Oauth
-   Shiro
-   Spring Security

## flow control

Problem Solving：Avoid excessive traffic on one service from dragging down the entire service system

Solution：

-   Hystrix

## automated test

Problem Solving： Predict abnormalities in advance to determine whether services are available

Solution：

-   junit

## Service online and offline process

Problem Solving： Avoid random online and offline services

Solution： When a new service goes online, it needs to be reviewed by the management personnel. When the service goes offline, it needs to notify each caller to make modifications. The service can not be offline until the service is not called.

## compatibility

Problem Solving：How to achieve compatibility in continuous service development.

Solution：It is managed in the form of version number, and the modification is completed for regression testing.

## service orchestration

Problem Solving：A way to solve the service dependency problem

Solution：

-   Docker
-   K8s

## Resource Scheduling

Problem Solving：The resource usage of each service is different, how to allocate

Solution：

-   JVM isolation
-   Classload isolation
-   hardware isolation

## capacity planning

Problem Solving：As time grows, calls gradually increase, when to add machines.

Solution：Count the daily call volume and response time, set the threshold according to the machine condition, and add the machine if the threshold is exceeded.
