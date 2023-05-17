# Several main calling processes of the service discovery component Eureka

## foreword

The now popular microservice architecture is changing the way we build applications, from a single monolithic service to smaller and smaller individually deployable services (called `microservices`), which together make up our application . When conducting a business, there will inevitably be calls between multiple services. If a service A wants to access service B deployed on another server, the premise is that service A needs to know the IP address of the machine where service B is located and the service correspondence. The easiest way is to let service A maintain a configuration of service B (including information such as IP address and port), but this method has several obvious disadvantages: as the number of services we call increases, How to maintain the configuration file; lack of flexibility, if service B changes the IP address or port, service A must also modify the corresponding file configuration; another is that it is inconvenient to dynamically expand or shrink the service.
A better solution is `Service Discovery`. It abstracts a registration center. When a new service goes online, it will register its own IP and port to the registration center, and will perform regular heartbeat detection on the registered service. When it finds that the service status is abnormal, it will be removed from the The registration center removes offline. Service A only needs to obtain the information of service B from the registration center. Even when the IP or port of service B changes, service A does not need to be modified, which decouples services to a certain extent. Service discovery There are many open source implementations in the industry, such as `apache` [zookeeper](https://github.com/apache/zookeeper), `Netflix` [eureka](https://github.com/Netflix/eureka), `hashicorp` [consul](https :github.comhashicorpconsul), `CoreOS`'s [etcd](https://github.com/etcd-io/etcd).

## What is Eureka

`Eureka` 在 [GitHub](https://github.com/Netflix/eureka) 上对其的定义为
`Eureka` is defined on [GitHub](https://github.com/Netflix/eureka) as

> Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers.

At Netflix, Eureka is used for the following purposes apart from playing a critical part in mid-tier load balancing.

`Eureka` is open-sourced by [Netflix](https://www.netflix.com), designed in the Client Server mode, based on the http protocol and the service registration and discovery component developed using Restful Api, providing a complete service registration And service discovery, can be seamlessly integrated with `Spring Cloud`. Among them, the server side plays the role of the service registration center, which mainly provides functions such as service registration and discovery for the client side, maintains the service registration information of the client side, and regularly checks the registered service by heartbeat. The end can obtain the registration information of the service it depends on through the server end, so as to complete the call between services. Unfortunately, it can be found from its official [github wiki](https://github.com/Netflix/eureka/wiki) that version 2.0 is no longer open source. But it does not affect our in-depth understanding of it. After all, service registration and service discovery are relatively basic and general, and the ideas of other open source implementation frameworks can also be figured out.

## Service Registry（Eureka Server）

We introduce the dependencies of `Eureka Server` in the project, and then add the annotation `@EnableEurekaServer` to the startup class to use it as a registration center. After starting the service, visit the page as follows:

![eureka-server-homepage.png](./images/eureka-server-homepage.png)

We continue to add two modules `service-provider`, `service-consumer`, and then annotate `@EnableEurekaClient` in the startup class and specify the address of the registration center as the `Eureka Server` we just started. Visit again and you can see two All services are registered.

![eureka-instance-registered-currently.png](./images/eureka-instance-registered-currently.png)

It can be seen that `Eureka` is very simple to use. It only needs to add a few annotations and configurations to realize service registration and service discovery. Next, let's see how it realizes these functions.

### service registration（Register）

The registration center provides a service registration interface, which is used to call to realize service registration after a new service is started, or to change the status of the corresponding service when the heartbeat detects that the service status is abnormal. Service registration is to send a `POST` request with the current instance information to the `addInstance` method of the class `ApplicationResource` for service registration.

![eureka-server-applicationresource-addinstance.png](./images/eureka-server-applicationresource-addinstance.png)

You can see that the method calls the `register` method of the class `PeerAwareInstanceRegistryImpl`, which is mainly divided into two steps:

1. Call the `register` method of the parent class `AbstractInstanceRegistry` to register the current service to the registry
2. Call the `replicateToPeers` method to synchronize service registration information with other `Eureka Server` nodes in an asynchronous manner

Service registration information is stored in a nested `map`, which has the following structure:

![eureka-server-registry-structure.png](./images/eureka-server-registry-structure.png)

The `key` of the first layer `map` is the application name (corresponding to `SERVICE-PROVIDER` in `Demo`), and the `key` of the second layer `map` is the instance name corresponding to the application (corresponding to `Demo` in `mghio-mbp:service-provider:9999`), an application can have multiple instances, and the main calling process is shown in the following figure:

![eureka-server-register-sequence-chart.png](./images/eureka-server-register-sequence-chart.png)

### Service Renewal（Renew）

Service renewal will be called periodically by the service provider (such as `service-provider` in `Demo`), similar to heartbeat, used to inform the registration center `Eureka Server` of its own status, so as to avoid being considered by `Eureka Server` as service time limit Take it offline. Service renewal is to send a `PUT` request with the current instance information to the `renewLease` method of the class `InstanceResource` for service renewal operation.

![eureka-server-instanceresource-renew.png](./images/eureka-server-instanceresource-renew.png)

Entering the `renew` method of `PeerAwareInstanceRegistryImpl`, you can see that the service renewal steps are generally consistent with the service registration, first update the status of the current `Eureka Server` node, and then use the asynchronous method to synchronize the status to other nodes after the service renewal is successful. On the `Eureka Server` section, the main calling process is shown in the following figure:

![eureka-server-renew-sequence-chart.png](./images/eureka-server-renew-sequence-chart.png)

### Service offline（Cancel）

When the service provider (such as `service-provider` in `Demo`) stops the service, it will send a request to inform the registration center `Eureka Server` to perform service removal and offline operations, preventing service consumers from calling from the registration center to non-existent ones Serve. The service offline is to send a `DELETE` request with the current instance information to the `cancelLease` method of the class `InstanceResource` to perform the service removal offline operation.

![eureka-server-instanceresource-cancellease.png](./images/eureka-server-instanceresource-cancellease.png)

Entering the `cancel` method of `PeerAwareInstanceRegistryImpl`, you can see that the service renewal steps are generally consistent with the service registration. First remove the offline service from the current `Eureka Server` node, and then synchronize in an asynchronous manner after the service is successfully offline. Status to other `Eureka Server` sections, the main call process is shown in the following figure:

![eureka-server-cancellease-sequence-chart.png](./images/eureka-server-cancellease-sequence-chart.png)

### Service Elimination (Eviction)

Service culling means that the registration center `Eureka Server` will start a daemon thread `evictionTimer` to execute the detection service periodically (the default is `60` seconds) when it starts. The expiration time is `90` seconds, that is to say, when a registered service does not renew the service contract (Renew) to the registry `Eureka Server` within `90` seconds, it will be removed from the registry. The expiration time can be modified by configuring `eureka.instance.leaseExpirationDurationInSeconds`, and the periodic execution of the detection service can be modified by configuring `eureka.server.evictionIntervalTimerInMs`. The main calling process is shown in the following figure:

![eureka-server-evict-sequence-chart.png](./images/eureka-server-evict-sequence-chart.png)

## Service Provider（Service Provider）

For service providers (such as the `service-provider` service in `Demo`), there are three main types of operations, namely `service registration (Register)`, `service renewal (Renew)`, `service offline (Cancel)`, let's see how these three operations are implemented.

### Service Registration (Register)

To provide external services, a service must first register service-related information in the registration center `Eureka Server`. The prerequisite for this step is that you need to configure `eureka.client.register-with-eureka=true`, the default value is `true`, the registration center does not need to register itself to the registration center, set this configuration to `false`, this call is relatively simple, the main call process is shown in the following figure:

![eureka-service-provider-register-sequence-chart.png](./images/eureka-server-register-sequence-chart.png)

### Service Renewal (Renew)

Service renewal is initiated by the service provider on a regular basis (the default is `30` seconds), which is mainly used to inform the registration center `Eureka Server` that its status is normal and still alive. You can configure `eureka.instance.lease -renewal-interval-in-seconds` to modify, of course, the premise of service renewal is to configure `eureka.client.register-with-eureka=true`, register the service to the registration center, the main calling process is as follows Shown:

![eureka-service-provider-renew-sequence-chart.png](./images/eureka-service-provider-renew-sequence-chart.png)

### Service offline (Cancel)

When the service on the service provider side stops, it should send a `DELETE` request to inform the registration center `Eureka Server` that it has gone offline, so that the registration center will remove itself from the offline, preventing the service consumer from obtaining unavailable services from the registration center . The implementation of this process is relatively simple. Add the annotation `@PreDestroy` to the `shutdown` method of the class `DiscoveryClient`. When the service stops, it will automatically trigger the service to be removed offline and execute the service offline logic. The main calling process is shown in the following figure:

![eureka-service-provider-cancel-sequence-chart.png](./images/eureka-service-provider-cancel-sequence-chart.png)

## Service Consumer

If the service consumer here does not need to be called by other services, it will only involve two operations, namely `Get the service list (Fetch)` from the registry and `Update the service list (Update)`. If you also need to register with the registration center to provide external services, then the rest of the process is the same as the service provider mentioned above, and will not be elaborated here. Next, let's see how these two operations are realized.

### Get Service List (Fetch)

After the service consumer is started, it must first obtain the list of available services from the registry `Eureka Server`, and a copy will be cached locally. The operation of obtaining the list of services is performed when the `DiscoverClient` class is instantiated after the service is started.

![eureka-service-consumer-fetchregistry.png](./images/eureka-service-consumer-fetchregistry.png)

It can be seen that the premise of obtaining the service list is to ensure that `eureka.client.fetch-registry=true` is configured. The default value of this configuration is `true`. The main calling process is shown in the following figure:

![eureka-service-consumer-fetch-sequence-chart.png](./images/eureka-service-consumer-fetch-sequence-chart.png)

### Update service list (Update)

It can be seen from the operation process of `Get Service List (Fetch)` above that a copy will be cached locally, so here you need to go to the registry `Eureka Server` regularly to get the latest configuration of the service, and then compare and update the local cache. The interval can be modified by configuring `eureka.client.registry-fetch-interval-seconds`, the default is `30` seconds, the premise of updating the service list in this step is that you have to configure `eureka.client.register-with-eureka =true`, the default value is `true`. The main calling process is shown in the figure below:

![eureka-service-consumer-update-sequence-chart.png](./images/eureka-service-consumer-update-sequence-chart.png)

## Summarize

The project at work uses the `Spring Cloud` technology stack, which has a very complete set of open source code to integrate `Eureka`, which is very convenient to use. Before, it was done directly by adding annotations and modifying several configuration properties. I didn’t have a deep understanding of the source code implementation. This article mainly expounds the related processes and implementation methods of service registration and service discovery, and has a closer look at `Eureka` service discovery components. step by step understanding.

---

reference article

[Netflix Eureka](https://github.com/Netflix/eureka)

[Service Discovery in a Microservices Architecture](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture)
