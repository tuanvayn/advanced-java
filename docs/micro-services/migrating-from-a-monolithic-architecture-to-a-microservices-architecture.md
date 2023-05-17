# Overview of migrating to microservices

Migrating monolithic applications to microservices architectures means a series of modernization processes, a bit like what these generations of developers have been doing, and in real time, when migrating, we can reuse some ideas.

One strategy is not to rewrite the code on a large scale (only if you are responsible for rebuilding a whole new microservices-based application). Rewriting code sounds great, but it's actually fraught with risks and can eventually fail, as Martin Fowler puts it:

> “the only thing a Big Bang rewrite guarantees is a Big Bang!”

Instead, you should adopt a strategy of gradually migrating monolithic applications, by gradually building new microservices applications, integrating with old monolithic applications, and over time, the proportion of monolithic applications in the overall architecture gradually decreases until they disappear or become part of the microservices architecture. This strategy is a bit like maintaining the car at a highway speed limit of 70 miles, and although it is challenging, it is much less risky than rewriting.

Martin Fowler called this modern strategy the Strangler application, named after the strangler vine in the rainforest, also known as the strangler fig. In order to climb to the top of the forest, the strangled vine had to grow around a large tree, and after a while, the tree died, leaving behind the tree-shaped vine. This application uses the same pattern, and new microservice applications are developed around traditional applications, which will gradually fade from the stage.

Let's take a look at other possible strategies。

# Strategy 1 – Stop mining

The Law of Holes says that when you enter a hole, you should stop digging. This is the best recommendation for monolithic applications when they are not manageable. In other words, monolithic applications should stop growing, meaning that when developing new features, you should not add new code to old monolithic applications, and the best way to develop new features into independent microservices. As shown in the following figure:

![1](./images/Law-of-Holes.png)

In addition to the new services and legacy applications, there are two modules, one of which is the request router, which handles the ingress (http) requests, a bit like the API gateway mentioned earlier. The router sends new feature requests to newly developed services, while traditional requests are sent back to monolithic applications.

The other is glue code, which integrates microservices and monolithic applications, which rarely exist independently and often access the data of monolithic applications. The glue code, which may be applied as a monolith, as a service, or both, is responsible for data integration. Microservices read and write data from monolithic applications through glue code. ​

Microservices have three ways to access monolithic application data：

-   Remote API provided by the ventilation monomer application
-   Directly access the monolithic application database
-   Maintain a copy of the data synced from a monolithic application yourself

Glue code is also known as the anti-corruption layer because the glue code protects the new domain model of microservices from the traditional monolithic application domain model. The glue code provides translation capabilities between the two models. The term anti-corruption layer first appeared in Eric Evans' must-read _Domain Driven Design_ and was later distilled into a white paper. Developing a disaster tolerance layer may not be very important, but it is a necessary part of avoiding monolithic quagmire.

Implementing new features as lightweight microservices has many advantages, such as preventing monolithic applications from becoming more unmanageable. Microservices themselves can be developed, deployed, and scaled independently. Adopting a microservices architecture makes a difference for developers.

However, this method does not solve any monomer itself problem, in order to solve the monomer itself problem must be deeply applied to make changes. Let's take a look at the strategy for doing so.

# Strategy 2 – Separate the front-end and back-end

The strategy to reduce the complexity of monolithic applications is to separate the presentation layer from the business logic and data access layers. A typical enterprise application consists of at least three distinct elements:

1. Presentation layer – handles HTTP requests, either in response to a RESTAPI request or by providing an HTML-based graphical interface. For a complex user interface application, the presentation layer is often an important part of the code.

2. Business logic layer - the core of the application that completes the business logic.

3.Data access layer – Access foundational elements such as databases and message brokers.

There is a clear separation between the presentation layer and the business data access layer. The business layer has a coarse-grained API composed of several aspects, which contains business logic elements. APIs are natural boundaries that can split a monolithic business into two smaller applications, one of which is the presentation layer and the other is the business and data access logic. After the split, the logic app remotely calls the business logic app, and the following diagram shows the different architecture before and after migration: ​

![2](./images/Before-and-after-migration.png)

This division of monolithic applications has two advantages, one of which makes the development, deployment and extension of the two parts of the application independent, in particular, allowing presentation layer developers to quickly select on the user interface for AB testing; Second, it enables some remote APIs to be called by microservices.

However, this strategy is only part of the solution. It is likely that one or both parts of the application are unmanageable, so a third strategy is required to eliminate the remaining monolithic architecture.

# Strategy 3 – Pull out services

The third migration strategy is to extract certain modules from monolithic applications to become independent microservices. Whenever a module is extracted into a microservice, the monolithic application becomes simpler; Once enough modules are converted, the monolithic application itself is no longer a problem, either disappears or is simple enough to become a service.

## The sequencing module should be converted to a microservice

A huge complex monolithic application consists of dozens or hundreds of modules, each of which is an extracted object. Deciding on the first module to be extracted is generally a challenge, and it is generally best to start with the easiest module to extract, which will allow developers to accumulate enough experience that can bring great benefits to subsequent modularization efforts.

Converting modules into microservices is generally time-consuming and can generally be sorted according to the degree of benefit, generally starting with frequently changing modules to benefit the most. Once a module is transformed into a microservice, its development can be deployed as a standalone module, speeding up the development process.

Extracting large resource consumers first is also one of the sorting criteria. For example, it can be useful to extract an in-memory database to become a microservice that can be deployed on a large memory host. Similarly, it is beneficial to extract algorithmic applications that are sensitive to computing resources, and such services can be deployed on hosts with a lot of CPUs. By transforming resource-consuming modules into microservices, you can make your application easily scalable.

It is also beneficial to look at existing coarse-grained boundaries to decide which module should be extracted, which makes porting easier and simpler. For example, a module that only synchronizes messages asynchronously with other apps is a clear boundary that can be easily transformed into a microservice.

## How to extract modules

The first step in extracting a module is to define a coarse-grained interface between the module and the monolithic application, which is more like a bidirectional API because the monolithic application requires data from microservices and vice versa. Because of the balance that must be balanced between responsible dependencies and fine-grained interface patterns, developing such APIs is challenging, especially for the business logic layer that uses the domain model pattern, so it is often necessary to change the code to solve the dependency problem, as shown in the figure:

Once the coarse-grained interface is complete, the module is transformed into a standalone microservice. To achieve this, code must be written to enable the exchange of information between monolithic applications and microservices through APIs that use the interprocess communication (IPC) mechanism. The comparison before and after migration is shown in the figure:

![3](./images/30103116_ZCcM.png)

In this example, the Z module that is using the Y module is an alternative extraction module, and its elements are being used by the X module, and the first step in migration is to define a set of coarse-grained APIs, and the first interface should be the internal interface used by the X module to activate the Z module; The second interface is the external interface used by the Z module to activate the Y module.

The second step in the migration is to convert the module into a standalone service. The internal and external interfaces use code based on the IPC mechanism, and the Z module is generally integrated into a microservice infrastructure framework to solve problems in the cutover process, such as service discovery.

Once the module is extracted, you can develop, deploy, and extend another service that is independent of the monolithic application and other services. You can write code from scratch to implement the service; In this case, the API code that integrates the service and the monolithic application becomes the disaster recovery layer, translating between the two domain models. With each service extracted, one step in the direction of microservices. Over time, monolithic applications will become simpler and users will be able to add more independent microservices.
Migrating an existing application to a modern application with a microservices architecture should not be achieved by rewriting code from scratch, but rather by gradual migration. There are three strategies to consider: implementing new features as microservices; Decouple the presentation layer from the business data access layer; Extract existing modules into microservices. Over time, the number of microservices will increase, and the resiliency and efficiency of development teams will increase significantly.
