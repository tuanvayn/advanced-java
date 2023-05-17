# Develop a monolithic application

Suppose you are preparing to develop a taxi dispatch software to compete with Uber and Hailo. After an initial meeting and requirements analysis, you may start this new project manually or using a generator based on Rails, Spring Boot, Play or Maven. Its The hexagonal architecture is modular, and the architecture diagram is as follows:

![monolithic-application-architecture-diagram](./images/monolithic-application-architecture-diagram.png)

At the heart of the application is the business logic, completed by modules that define services, domain objects, and events. Surrounding the core are adapters that deal with the outside world. Adapters include database access components, message components that produce and process messages, and web modules that provide API or UI access support, etc.

Although it is also modular logic, in the end it will be packaged and deployed as a monolithic application. The exact format depends on the application language and framework. For example, many Java applications are packaged as WARs for deployment on Tomcat or Jetty, while others are packaged as self-contained JARs. Likewise, Rails and Node.js are packaged as hierarchical directories.

This style of application development is very common because IDEs and other tools are good at developing a simple application, which is also easy to debug. You only need to simply run the application and use Selenium to link the UI to complete the end-to-end test. Monolithic applications are also easy to deploy. You only need to copy the packaged application to the server side, and you can easily scale the application by running multiple copies on the backend of the load balancer. In the early days such applications worked very well.

# Disadvantages of monolithic applications

Unfortunately, this simple approach has significant limitations. A simple application can grow over time. In each sprint, the development team faces a new "story" and develops a lot of new code. After a few years, this small and simple application can turn into a huge monster. Here's an example I was recently discussing with a developer who was writing a tool to analyze dependencies between JAR files in one of their multi-million-line-of-code applications. I'm pretty sure this code is a monster that many developers have worked on over the years.

Once your application becomes a big and complex monster, it must be a pain for the development team. Agile development and deployment struggles, chief among them the fact that the application is too complex for any single developer to understand. Therefore, fixing bugs and correctly adding new features becomes very difficult and time-consuming. Plus, team morale can go downhill. If the code is difficult to understand, it cannot be corrected. It will eventually lead to a huge, incomprehensible quagmire.

Monolithic applications also slow down development. The larger the app, the longer it will take to start up. For example, a recent survey showed that sometimes apps take longer than 12 minutes to launch. I've also heard that some apps take 40 minutes to start up. If the developer needs to restart the application frequently, most of the time will be spent waiting, and the production efficiency will be greatly affected.

In addition, complex and huge monolithic applications are not conducive to continuous development. Today, the norm for SaaS applications is to change many times a day, which is very difficult for monolithic application models. Also, the impact of this change was not well understood, so a lot of manual testing had to be done. Then, continuous deployment will be very difficult.

When resource conflicts occur in different modules of a monolithic application, it will be very difficult to expand. For example, a module that completes a CPU-sensitive logic should be deployed on AWS EC2 Compute Optimized instances, while another in-memory database module is more suitable for EC2 Memory-optimized instances. However, since these modules are deployed together, a compromise has to be made in hardware selection.

Another problem with monolithic applications is reliability. Because all modules run in one process, a bug in any one module, such as a memory leak, will likely bring down the entire process. In addition, since all application instances are unique, this bug will affect the reliability of the entire application.

Finally, monolithic applications make adopting new architectures and languages very difficult. For example, imagine you have 2 million lines of code written using the XYZ framework. If you want to change to the ABC framework, both time and cost are very expensive, even if the ABC framework is better. Therefore, this is an insurmountable gap. You have to bow your head before your initial choice.

To sum it up: what starts out as a successful business-critical application turns into a giant, incomprehensible monster. Hiring potential developers is difficult because of outdated, inefficient technologies. Applications cannot scale, reliability is low, and ultimately, agile development and deployment becomes impossible.

So how to deal with it?

# Microprocessing architecture - dealing with complex things

Many companies, such as Amazon, eBay, and NetFlix, have solved the above problems by adopting the microprocessing architectural pattern. The idea is not to develop a huge monolithic application, but to decompose the application into small, interconnected microservices.

A microservice generally completes a specific function, such as order management, customer management, and so on. Each microservice is a tiny hexagonal application with its own business logic and adapters. Some microservices also publish APIs for use by other microservices and application clients. Other microservices complete a Web UI. At runtime, each instance may be a cloud VM or a Docker container.

比如，一个前面描述系统可能的分解如下：
For example, a possible decomposition of the system described above is as follows:

![deal-with-complex things-1](./images/deal-with-complex-things-1.png)

Each functional area of the application is done using microservices, and the web application is split into a series of simple web applications (say one for passengers and one for taxi drivers). Such a split is easier to deploy for different users, devices, and special application scenarios.

Each backend service exposes a REST API, and many services themselves use APIs provided by other services. For example, driver management uses a notification service that notifies drivers of a potential need. The UI service activates other services to update the Web pages. All services use asynchronous, message-based communication. The internal mechanism of microservices will be discussed in subsequent series.

Some REST APIs are also open to mobile apps used by passengers and drivers. These applications do not directly access background services, but pass intermediate messages through API Gateway. API Gateway is responsible for load balancing, caching, access control, API billing monitoring and other tasks, which can be easily realized through NGINX. Subsequent articles will introduce API Gateway.

![deal-with-complex-things-2](./images/deal-with-complex-things-2.png)

The microservices architectural pattern in the diagram above corresponds to the Y-axis representing the Scale Cube, a three-dimensional scaling model described in The Art of Scalability. The other two axes of scalability, the X-axis consists of multiple copies of the application running on the backend of the load balancer, and the Z-axis is about routing demand to related services.

The application can basically be represented by the above three dimensions, and the Y axis represents the decomposition of the application into microservices. At runtime, the x-axis represents running multiple instances hidden behind a load balancer, providing throughput. Some applications may still use the Z axis to partition services. The diagram below demonstrates how the trip management service is deployed on Docker running on AWS EC2.
![deal-with-complex-things-3](./images/deal-with-complex-things-3.png)

At runtime, the trip management service consists of multiple service instances. Each service instance is a Docker container. In order to ensure high availability, these containers generally run on multiple cloud VMs. In front of the service instances is a layer of load balancers such as NGINX, which are responsible for distributing requests among the instances. The load balancer also handles other requests, such as caching, access control, API statistics and monitoring.

This microservice architecture model has profoundly affected the relationship between applications and databases. **Unlike traditional multiple services sharing a database, each service in the microservice architecture has its own database**. In addition, this line of thinking has also affected enterprise-level data models. At the same time, this pattern means multiple copies of data, but if you want to get the benefits of microservices, a unique database for each service is necessary, because this architecture requires this kind of loose coupling. The following diagram illustrates the sample application database schema.

![deal-with-complex-things-4](./images/deal-with-complex-things-4.png)

Each service has its own database. In addition, each service can use a more suitable database type, which is also called a multilingual consistency architecture. For example, driver management (discovering which driver is closer to the passenger), must use a database that supports geographic information queries.

On the surface, the microservices architectural pattern looks a bit like SOA in that they are composed of multiple services. However, looking at this from another perspective, the Microservices Architecture pattern is a SOA without Web Services (WS-) and ESB services. Microservices applications prefer simple, lightweight protocols like REST instead of WS-, avoiding ESB and ESB-like features inside microservices. The microservice architectural pattern also rejects SOA concepts such as canonical schema.

# Benefits of Microservices Architecture

The microservice architectural pattern has many benefits. First, complexity is resolved by decomposing a monolithic application into multiple service methods. Under the condition of unchanged function, the application is decomposed into multiple manageable branches or services. Each service has a well-defined boundary with an RPC- or message-driven API. The microservice architectural pattern provides a modular solution to functions that are difficult to implement with monolithic coding, whereby individual services are easy to develop, understand, and maintain.

Second, this architecture enables each service to be developed by a dedicated development team. Developers are free to choose development technologies and provide API services. Of course, many companies try to avoid confusion by only offering certain technology options. This freedom, however, means that instead of being forced to use the outdated technology that a project started with, developers can choose the technology that is today. Even, because services are relatively simple, it is not very difficult to rewrite previous code using current technology.

Third, the microservice architecture pattern is an independent deployment of each microservice. Developers no longer need to coordinate the impact of other service deployments on this service. This change can speed up deployment. UI teams can use AB testing to quickly deploy changes. The microservice architecture pattern makes continuous deployment possible.

Finally, the microservice architectural pattern enables each service to scale independently. You can deploy the scale that meets your needs according to the scale of each service. Even more, you can use hardware that is better suited to the service's resource needs. For example, you can deploy CPU-intensive services on EC2 Compute Optimized instances, and deploy in-memory databases on EC2 memory-optimized instances.

# Disadvantages of microservice architecture

Fred Brooks wrote 30 years ago, "there are no silver bullets", and like any other technology, microservices architecture has its shortcomings. One of them is similar to his name, "microservice" emphasizes the size of the service. In fact, some developers advocate the establishment of a slightly larger service group of 10-100 LOC. Although small services are more likely to be adopted, don't forget that this is only the choice of the terminal and not the ultimate goal. The purpose of microservices is to effectively split applications and achieve agile development and deployment.

Another major disadvantage is that microservice applications are distributed systems, which brings inherent complexity. Developers need to choose between RPC or message passing and implement the interprocess communication mechanism. What's more, they have to write code to deal with local failures such as slow or unavailable message delivery. Of course, this is not difficult, but compared to the language-level method or process call in the monolithic application, this technology under the microservice is more complicated.

Another challenge with microservices comes from partitioned database architectures. It is common to update news to multiple business sub-entities at the same time in business transactions. This kind of transaction is easy for monolithic applications because there is only one database. In microservice architecture applications, different databases used by different services need to be updated. Using distributed transactions is not necessarily a good choice, not only because of the CAP theory, but also because today's highly scalable NoSQL databases and messaging middleware do not support this requirement. In the end, you have to use an eventual consistency method, which puts higher requirements and challenges on developers.

Testing an application based on microservice architecture is also a complex task. For example, using the popular Spring Boot architecture, it is very easy to test the REST API of a monolithic web application. In turn, the same service test needs to start all services related to it (at least the stubs of these services are required). Again, the complexity of adopting a microservices architecture cannot be underestimated.

Another challenge is that changes in the application of the microservice architecture pattern will affect multiple services. For example, suppose you are completing a case and need to modify services A, B, and C, and A depends on B, and B depends on C. In a monolithic application, you only need to change the relevant modules, integrate the changes, and deploy. In contrast, the microservice architecture pattern needs to consider the impact of related changes on different services. For example, you need to update service C, then B, and finally A. Fortunately, many changes generally affect only one service, and changes that need to coordinate multiple services are rare.

Deploying a microservice application is also very complicated. A distributed application only needs to deploy its own servers behind the complex balancer. Each application instance needs to configure basic services such as database and message middleware. In contrast, a microservice application typically consists of a large number of services. For example, according to Adrian Cockcroft, Hailo is made up of 160 different services, and NetFlix has around 600 services. Each service has multiple instances. This results in a lot of configuration, deployment, scaling, and monitoring. In addition, you also need to implement a service discovery mechanism (published in a follow-up article) to discover the address of the communication service with it (including server address and port). Traditional problem-solving methods cannot be used to solve such complex problems. Next, successful deployment of a microservice application requires developers to have sufficient control over the deployment method and a high degree of automation.

One approach to automation is to use a PaaS service such as Cloud Foundry. PaaS provides developers with an easy way to deploy and manage microservices, and it solves all these problems in a package. At the same time, systems and network experts configuring PaaS can adopt best practices and policies to simplify these issues. Another way to automatically deploy microservice applications is to develop the most basic PaaS system for you. A typical starting point is to use a clustering solution such as Mesos or Kubernetes with Docker. In the following series, we will look at how to provide caching, permission control, API statistics and monitoring at the microservice level based on software deployment methods such as NGINX.

# Summarize

Building complex applications is really hard. Monolithic architecture is more suitable for lightweight and simple applications. It's really bad if you're using it for complex applications. The microservice architecture pattern can be used to build complex applications. Of course, this architecture model also has its own shortcomings and challenges.
