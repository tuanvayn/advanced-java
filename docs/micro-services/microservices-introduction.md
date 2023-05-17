# Microservices

> Translated from [Martin Fowler](https://martinfowler.com/) website [Microservices](https://martinfowler.com/articles/microservices.html). The article is long and requires a little patience to read. <br> My level is limited, if there is anything wrong, please help correct it, thank you.

Over the past few years, the term "microservices architecture" has emerged, describing a specific approach to designing a software application as several suites of services that can be deployed independently. While this architectural style is not precisely defined, it shares certain characteristics around organizations such as business capabilities, automated deployment, endpoint intelligence, and decentralized control of language and data.

"Microservices" – another new term on the crowded streets of software architecture. While our natural tendency is to glimpse it with disdain, the term describes an increasingly attractive style of software systems. We've seen many projects use this style over the past few years, and the results so far have been so positive that it has become the default style for many of our colleagues at ThoughtWorks to build enterprise applications. Unfortunately, however, there isn't much information to outline the style of microservices and how to implement them.

In short, the microservices architectural style[1] is an approach to developing a single application as a suite of small services, each running in its own process and communicating in a lightweight mechanism, typically an HTTP resource API. These services are built around business functions and can be deployed independently through a fully automated deployment mechanism. These services share a minimal centralized management, and they can be written in different programming languages and use different data storage technologies.

Before starting to explain the microservices style, it is useful to compare it to the monolithic style: monolithic applications are built as monolithic units. An enterprise application typically consists of three parts: a client-side user interface (consisting of HTML pages and Javascript running in a browser on the user's machine), a database (consisting of many tables, usually managed in a relational database), and a server-side application. The server-side application processes the HTTP request, performs some logical processing, retrieves and updates data from the database, selects the data, and populates it into an HTML view to be sent to the browser. This server-side application is a whole - a logical executable [2]. Any changes to the system involve building and deploying a new version of the server-side application.

This monolithic server is a natural way to build such a system. All the logic that processes a request runs in a single process, allowing you to divide your application into classes, functions, and namespaces using the basic features of the language. It is important to note that you can run and test your application on a developer's laptop and use deployment pipelines to ensure that changes to your program are properly tested and deployed to production. You can horizontally scale a monolithic block by running many instances behind a load balancer.

Monolithic applications can succeed, but more and more people are unhappy with them—especially as more applications are deployed to the cloud. Change cycles are bundled together—even if only a small part of the application is changed, the entire monolithic application needs to be rebuilt and deployed. Over time, it is often difficult to maintain a good modular structure, and it is also more difficult to maintain changes that should affect only one module in that module. When you scale a system, you have to scale the entire application, not just those parts of the system that require more resources.

![sketch](./images/sketch.png)

These grievances gave rise to a microservices architectural style: building applications as suites of services. In addition to the fact that services can be deployed independently and scaled independently, each service provides a solid module boundary and even allows different services to be written in different programming languages. They can also be managed by different teams.

We don't think the microservices style is novel or innovative, and its roots can be traced back at least to Unix design principles. But we don't think enough people think about microservices architecture, and a lot of software development would be better if it were used.

## Characteristics of a microservices architecture

While it is not possible to say that there is a formal definition of microservices architectural styles, we can try to describe some of the common characteristics that we believe they have in architecture that fits this label. As with any definition that outlines common characteristics, not all microservices architectures have all characteristics, but we do expect most microservices architectures to have most characteristics. While our writers have been active members of this rather permissive community, our intention is to try to describe what the two of us see in the work of ourselves and the team we know. In particular, we do not have some relevant definitions.

### Componentization through services

Ever since we've been involved in the software industry, we've always wanted to build systems by integrating components together, just like the way things we're seen in the physical world are built. Over the past few decades, we've seen tremendous progress in public software repositories for most language platforms.

When talking about components, one of the defining dilemma is, what is a component? We define components as software units that can be replaced and upgraded independently.

A microservices architecture also uses software libraries, but the primary way to componentize software is to split it into multiple services. We define a library as a component that is linked to a program and invoked using memory function calls, while a service is an out-of-process component that communicates through mechanisms such as Web service requests or remote procedure calls. (This is different from the concept of a service object in many object-oriented programs [3]. ）

One of the main reasons to treat services as components rather than libraries is that services can be deployed independently. If you have an application[4] that consists of multiple libraries in a single process, a change to any one component will result in a redeployment of the entire application. However, if the application can be split into multiple services, changes to a single service only require redeployment of the service. Of course, this is not absolute, some service interface modifications may lead to collaborative modifications between multiple services, but the purpose of a good microservices architecture is to minimize these collaborative modifications by cohesive service boundaries and service protocol evolution mechanisms.

Another result of using services as components is a more explicit component interface. Most languages do not have a good mechanism for defining explicitly published interfaces. Often, it's just documentation and rules to prevent clients from breaking the encapsulation of components, which results in too tight coupling between components. By using an explicit remote call mechanism, services can more easily avoid this.

There are indeed some downsides to using services like this. Remote calls are more expensive than in-process calls, and remote APIs need to be designed to be coarsely granular, which is often more difficult to use. If you need to change the distribution of responsibilities between components, this change in component behavior becomes more difficult to implement when you cross process boundaries.

Approximately, we can map a service to a runtime process, but this is only an approximation. A service might include multiple processes that are always developed and deployed together, such as an application process and a database used only by the service.

### Organize around business capabilities

When splitting a large application into parts, management tends to focus on the technical aspects, resulting in a division of UI teams, server-side logic teams, and database teams. When teams are separated in these ways, even simple changes can lead to time and budget approvals for cross-team projects. A smart team will optimize around this, "the lesser of two evils" – simply force the logic to any application they can access. In other words, logic is everywhere. This is an example of Conway's Law [5].

> Any organization that designs a system (in a broad sense) produces a design whose structure is a copy of the organization's communication structure. <br> —Melvin Conway, 1967

![conways-law](./images/conways-law.png)

Microservices are divided differently, which is to split the system into multiple services around business functions. These services employ a wide range of software implementations for this business area, including user interfaces, persistent storage, and any external collaboration. As a result, teams are cross-functional and include all the skills needed for development: user experience, database, and project management.

![PreferFunctionalStaffOrganization](./images/PreferFunctionalStaffOrganization.png)

One company formed in this way is [www.comparethemarket.com](http://www.comparethemarket.com/). Cross-functional teams build and operate each product, each split into separate services that communicate with each other via a message bus.

Large monolithic applications can also be modularized around business functions, although this is not a common scenario. Of course, we urge large teams building monolithic applications to break themselves down into smaller teams based on lines of business. The main problem we see here is that they tend to be organized around too much context. If monoliths cross module boundaries, it is difficult for individual members of the team to fit them into short-term memory. In addition, we see that modular production lines require a large number of rules to enforce. The clearer separation required by service components makes it easier to keep team boundaries clear.

### It's a product, not a project

Most of the application development efforts we see use a project model where the goal is to deliver some software and then it's done. Once completed, the software is handed over to the maintenance organization, and the project team that built it is disbanded.

Microservices proponents tend to avoid this pattern, arguing instead that the team should be responsible for the entire lifecycle of the product. A common takeaway is Amazon's concept of [“you build, you run it”](https://queue.acm.org/detail.cfm?id=1142065), where the development team takes full responsibility for the software in production. This gives developers frequent exposure to how their software works in production and increases contact with their users, who must undertake at least part of the support work.

The product mindset is closely linked to business capabilities. Keep an eye on how software helps users improve their business capabilities, rather than treating it as a set of functions to be accomplished.

There's no reason why this approach can't be used for a single application, but the smaller granularity of the service makes it easier to establish a personal relationship between the service developer and the user.

### Smart endpoints and dumb tubes

We've seen many products and approaches that emphasize putting a lot of intelligence into the communication mechanism itself when it comes to establishing communication between different processes. A good example is the Enterprise Service Bus (ESB), where ESB offerings typically include sophisticated tools for message routing, orchestration, transformation, and application of business rules.

The microservices community tends to take a different approach: smart endpoints and dumb tubes. The goal of applications built on top of microservices is to be as decoupled and cohesive as possible—they have their own domain logic, and they behave more like filters in classic UNIX ideas—receiving requests, applying appropriate logic, and producing responses. Use simple RESTful protocols to orchestrate them, rather than complex protocols like WS-Choreography or BPEL or orchestration.

The two most commonly used protocols are HTTP request-response with resource APIs and lightweight messaging[8]. The best formulation of the first agreement is:

> The web itself, not hidden behind it. <br> ——[Ian Robinson](http://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829)

The same rules and protocols used by microservices teams are the same rules and protocols for building the World Wide Web (and to a greater extent UNIX). From the perspective of developers and operators, commonly used resources can be easily cached.

The second common method is to pass messages over a lightweight message bus. The infrastructure chosen is typically dumb (dumb only acts as a message router here)—simple implementations like RabbitMQ or ZeroMQ simply provide a reliable asynchronous switching structure—and in services, intelligence still exists in the endpoints that produce and consume many messages, that is, in each service.

In a monolithic application, components are all executed within the same process, and they communicate with each other through method calls or function calls. The biggest problem with turning monoliths into microservices is the change in communication patterns. A naïve transition is to transition from memory method calls to RPC, which results in frequent communication and poor performance. Instead, you need to replace fine-grained communication with coarse-grained communication.

### Decentralized governance

One consequence of centralized governance is the trend towards standardization of a single technology platform. Experience shows that this approach is shrinking – not every problem is a nail, not every problem is a hammer. We prefer to use the right tools to get the job done, and it's not common for monolithic applications to take advantage of language to some extent.

Splitting monolithic components into services gives you your own choices when building those services. Do you want to develop a simple reporting page using Node.js? Go ahead. Implement a particularly coarse, near-real-time component in C++? Great. Do you want to switch to a different style of database that is better suited for components reading and manipulating data? We have the technology to rebuild it.

Of course, just because you can do something, doesn't mean you should do it – but dividing the system that way means you have a choice.

Teams also prefer a different approach to building microservices. They prefer the idea of producing useful tools rather than standards written down on paper so that other developers can use them to solve similar problems they face. Sometimes, these tools are often harvested in implementation and shared with a wider community, but not entirely using an in-house open source model. Now that Git and GitHub have become the de facto version control system of choice, the practice of open source internally is becoming more common.

Netflix is a great example of following this philosophy. In particular, sharing useful and market-tested code in the form of libraries incentivizes other developers to solve similar problems in similar ways, while also opening the door to different approaches. Shared libraries tend to focus on common issues such as data storage, inter-process communication, and infrastructure automation that we'll discuss in depth next.

For the microservices community, overhead is particularly unattractive. That's not to say that the community doesn't value service contracts. Quite the opposite, because they have more contracts. It's just that they're looking for different ways to manage these contracts. Patterns like [Tolerant Reader](https://martinfowler.com/bliki/TolerantReader.html) and [Consumer-Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html) are commonly used for microservices. These aid service contracts are evolving independently. Executing consumer-driven contracts as part of the build increases confidence and provides faster feedback on whether the service is working. In fact, we know that a team in Australia uses a consumer-driven contract model to drive new business builds. They use simple tools to define contracts for services. This has become part of the automatic build, even if the code for the new service has not yet been written. Services are created only when the contract is fulfilled - an elegant way to avoid the "YAGNI"[9] dilemma when building new software. The technologies and tools that have grown around these have limited the need for central contract management by reducing temporary coupling between services.

Perhaps the pinnacle of decentralized governance is Amazon's widespread idea of Build ItRun It. Teams are responsible for all aspects of the software they build, including 24/7 operations. This level of decentralization is definitely not standardized, but we're seeing more and more companies holding development teams accountable. Netflix is another company that has adopted this philosophy[11]. Being woken up by a pager at 3 a.m. every day is certainly a powerful incentive to focus on quality when writing code. Here are some ideas about moving as far away from the traditional centralized governance model as possible.

### Decentralized data management

The decentralization of data management comes in many different ways. At the most abstract level, this means a conceptual model of the world that makes differences between systems. When consolidating a large enterprise, it is a common problem that the customer's view of sales will be different from the view of support. Some things in the customer's sales view may not appear in the support view. They may indeed have different properties and (worse) common properties that are subtly different in semantics.

This problem is common between applications, but can also occur within an application, especially if the application is divided into separate components. A useful way to think about it is the Domain-Driven Design (DDD) philosophy within [Bounded Context] (http://martinfowler.com/bliki/BoundedContext.html) (Bounded Context). DDD divides a complex domain into bounded contexts and maps the relationships between them. This process is useful for both monolithic and microservices architectures, but there is a natural correlation between service and context boundaries, which help clarify and reinforce separation, as described in the Business Capabilities section.

Like the decentralized decision-making of the conceptual model, microservices also decentralize data storage decisions. While monolithic applications prefer a single logical database for persistent storage, enterprises tend to prefer a single database for a single set of applications—decisions driven by the vendor-licensed business model. Microservices prefer to have each service manage its own database, or a different instance of the same database technology, or a completely different database system — this is called Polyglot Persistence (https://martinfowler.com/bliki/PolyglotPersistence.html). You can use hybrid persistence in monolithic applications, but it is more common in servings.

![decentralised-data](./images/decentralised-data.png)

For data across microservices, decentralized responsibility has implications for management upgrades. A common way to handle updates is to use transactions to guarantee consistency when updating multiple resources. This method is usually used in monomers.

像这样使用事务有助于一致性，但会产生显著地临时耦合，这在横跨多个服务时是有问题的。分布式事务是出了名的难以实现，因此微服务架构强调[服务间的无事务协作](http://www.eaipatterns.com/ramblings/18_starbucks.html)，对一致性可能只是最后一致性和通过补偿操作处理问题有明确的认知。
Using transactions like this helps with consistency, but creates significant temporary coupling, which is problematic when spanning multiple services. Distributed transactions are notoriously difficult to implement, so microservices architectures emphasize [transactionless collaboration between services](http://www.eaipatterns.com/ramblings/18_starbucks.html), with a clear understanding that consistency may only be final consistency and handling problems through compensating operations.

Choosing to manage inconsistencies in this way is a new challenge for many development teams, but it often matches business practices. Often the business deals with some degree of inconsistency to respond quickly to demand, while there are some types of reversal processes to deal with errors. This trade-off is worth it, as long as the cost of fixing the error is less than the cost of losing business with greater consistency.

### Infrastructure automation

Infrastructure automation technologies have changed dramatically over the past few years—in particular, the evolution of the cloud and AWS has reduced the operational complexity of building, deploying, and running microservices.

Many products or systems built with microservices are built by teams with extensive experience in continuous delivery and continuous integration. Infrastructure automation techniques are widely used by teams that build software in this way. This is shown in the build pipeline shown below.

![basic-pipeline](./images/basic-pipeline.png)

Since this is not an article about continuous delivery, we will focus on only a few key features of continuous delivery here. We want to have as much confidence as possible to ensure that our software is working properly, so we do a lot of **automated testing**. Getting software to reach the "Promotion" state and "push" the pipeline means **automating the deployment** of software in every new environment.

A monolithic application can be very pleasant to build, test, and push through these environments. It turns out that once you've put in automated overall production for monoliths, deploying more applications doesn't seem so scary anymore. Keep in mind that one of the goals of continuous delivery is to make "deployment" "boring", so whether it's one or three applications, as long as the deployment is still "boring", then there is nothing to worry about[12].

Another area where we see a lot of infrastructure automation for teams is managing microservices in production. Compared to our assertion above (as long as the deployment is boring), there isn't much difference between monolithic and microservices, but the environment in which each deployment runs can be very different.

![micro-deployment](./images/micro-deployment.png)

### Be prepared for failures at design time

As a result of using services as components, applications need to be designed so that they can tolerate the failure of the service. If the service provider is unavailable, any service call may fail and the customer must respond to this as gracefully as possible. This is a disadvantage compared to monolithic designs, as it introduces additional complexity to handle it. The result is that microservices teams are constantly rethinking how service failures affect the user experience. Netflix's [Simian Army](https://github.com/Netflix/SimianArmy) can test the resiliency and visibility of applications by causing service and even data center failures to fail during the business day.

This automated testing in production is enough to make most operations teams shudder with excitement, as if just before the week's long holiday is just around the corner. That's not to say that monolithic architecture can't build advanced surveillance systems — it's just that in our experience, it's not common in monolithic systems.

Because services can fail at any time, it's critical to be able to detect failures quickly and automatically recover if possible. Microservices applications place great emphasis on real-time monitoring of applications, such as examining schema elements (how many requests the database gets per second) and business-related metrics (such as how many orders are received per minute). Semantic monitoring can provide an early warning system for problems, triggering follow-up and investigation by development teams.

This is especially important for microservices architectures, which prefer orchestration and event writing, which leads to some emergency situations. While many pundits are positive about the value of chance events, the truth is that "sudden behavior" can sometimes be a bad thing. Monitoring is critical to quickly spot bad emergency behavior and fix it.

Monolithic systems can also be monitored as transparently as microservices—in fact, they should. The difference is that you must be able to know when a service running in a different process is disconnected. For libraries in the same process, this transparency is not very useful.

Microservices teams want to see sophisticated monitoring and logging for each service, such as dashboards that show "up and down" status and a variety of operational and business-related metrics. Details about circuit breaker status, current throughput, and latency are other examples that we often encounter in our work.

### Evolutionary design

Microservices practitioners often have a background in evolutionary design and see service decomposition as a further tool that enables application developers to control changes in applications without slowing down the speed of change. Change control doesn't necessarily mean fewer changes – with the right attitude and tools, you can make frequent, fast, and well-controlled changes to your software.

Whenever you're trying to break down a software system into components, you're faced with the decision of how to split – what are the principles by which we decided to split the application? The key properties of components are characterized by independent substitution and upgradeability[13]—which means that we look for these points and imagine how we can rewrite the component without affecting its collaborators. In fact, many microservice groups think this further by explicitly expecting many services to be retired rather than evolving over the long term.

The Guardian website is a great example of designing and building monolithic applications, but it is also evolving in the direction of microservices. The original monolithic system is still the core of the website, but they prefer to add new functionality by building some microservice APIs. This approach is especially handy for features that are temporary in nature, such as dedicated pages that deal with sporting events. This section of the website can be quickly grouped together using the rapid development language and removed immediately after the event. We've seen similar approaches in financial institutions, adding new services to market opportunities and discarding them months or even weeks later.

This emphasis on fungibility is a special case of the general principle of modular design, which drives the realization of modularity through changing patterns [14]. Everyone is willing to put those things that change at the same time in the same module, and the system modules that rarely change should be in a different service than the system that is currently undergoing a lot of changes. If you find yourself repeatedly changing two services, it's a sign that they should be merged.

Putting components into a service can add opportunities for a more granular release schedule. For monolith, any change requires a complete build and deployment of the entire application. However, with microservices, you only need to redeploy the services you modified. This simplifies and speeds up the publishing process. The downside is that you have to worry about a change in a service breaking its consumers. The traditional integration approach is to try to solve this problem using version control, but the preference in the microservices world is to use version control only as a last resort. We can avoid a lot of versioning by designing the service to be as tolerant of changes in the service provider as possible.

## Are microservices the future?

Our main purpose in writing this article is to explain the main ideas and principles of microservices. By taking the time to do this, it's clear to us that the microservices architecture style is an important idea that deserves serious consideration when developing enterprise systems. We've recently built several systems this way, and we've learned that other teams agree with this style.

We learned about the pioneers who pioneered this architectural style to some extent, including Amazon, Netflix, The Guardian, UK Government Digital Services, realestate.com.au, Forward, and comparethemarket.com. The 2013 tech conference was filled with examples of companies that are turning to companies that can be classified as microservices, including Travis CI. Also, there are a lot of organizations that have been doing what we call microservices for a long time, but haven't used that name. (Often this is labeled SOA—although, as we said, SOA has many contradictory forms.) [15]）

However, despite these positive experiences, this does not mean that we are convinced that microservices are the future of software architecture. While our experience so far has been positive compared to the overall application, we realize that there is not enough time for us to make a full and complete judgment.

Often, the real impact of an architectural decision is not really visible until years after the decision is made. We've seen projects done by great teams with a strong desire for modularity end up building a monolithic architecture that continues to decay within a few years. Many believe that this corruption is unlikely if microservices are used because the boundaries of the service are clear and difficult to mess with. However, for systems that have been developed long enough, we can't really evaluate how mature a microservices architecture is unless we've seen enough of it.

Some people think that microservices can be difficult to mature, and there is certainly a reason for that. The success of any work done on componentization depends on how well the software matches the component. Figuring out exactly where the location of a component's boundary should appear is a difficult task. Evolutionary design acknowledges that it is difficult to position boundaries correctly, so it focuses its work on the ease of reconstructing boundaries. However, when each component becomes a service that communicates remotely, it becomes more difficult to refactor than to make calls between software libraries in a single process. Code movement across service boundaries becomes difficult. Any change in the interface needs to be coordinated among its various participants. Layers of backward compatibility also need to be added. Testing also becomes more complex.

Another problem is that if these components don't fit together cleanly into a system, all that is done is to transfer the complexity within the components to the connections between them. The consequence of this is not just to move complexity, it also shifts complexity to boundaries that are no longer clear and difficult to control. When looking inside a small and simple component, it's easy to think things have gotten better, but they ignore the messy connections between services.

Finally, there is the factor of team skills. New technologies tend to be adopted by more technically competent teams. A more effective technique for a more technically competent team may not work for a team that is less technically skilled. We've seen plenty of examples where teams with slightly less technical skills build messy monolithic architectures. What happens when this clutter happens to microservices? It takes time to observe. A bad team will always build a bad system — in which case it's hard to tell whether microservices reduce clutter or make things worse.

A reasonable argument we've heard is that you shouldn't start with a microservices architecture, but rather with the whole, keep it modular, and split it into microservices when something goes wrong with the whole. (This advice is not ideal, because a good in-process interface is usually not a good service interface.) ）

So we write this cautiously optimistically. We've seen enough microservices styles so far that we feel it might be a worthwhile path to take. We can't be sure where it will end up in the end, but one of the challenges of software development is that you can only make decisions based on imperfect information that you must currently have.

## footnote

1: The term "microservice" was discussed at a workshop of software architects near Venice in May, 2011 to describe what the participants saw as a common architectural style that many of them had been recently exploring. In May 2012, the same group decided on "microservices" as the most appropriate name. James presented some of these ideas as a case study in March 2012 at 33rd Degree in Krakow in [Microservices - Java, the Unix Way](http://2012.33degree.org/talk/show/67) as did Fred George [about the same time](http://www.slideshare.net/fredgeorge/micro-service-architecure). Adrian Cockcroft at Netflix, describing this approach as "fine grained SOA" was pioneering the style at web scale as were many of the others mentioned in this article - Joe Walnes, Dan North, Evan Botcher and Graham Tackley.

2: The term monolith has been in use by the Unix community for some time. It appears in [The Art of Unix Programming](https://www.amazon.com/gp/product/B003U2T5BA?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=B003U2T5BA) to describe systems that get too big.

3: Many object-oriented designers, including ourselves, use the term service object in the [Domain-Driven Design](https://www.amazon.com/gp/product/0321125215?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321125215) sense for an object that carries out a significant process that isn't tied to an entity. This is a different concept to how we're using "service" in this article. Sadly the term service has both meanings and we have to live with the polyseme.

4: We consider [an application to be a social construction](https://martinfowler.com/bliki/ApplicationBoundary.html) that binds together a code base, group of functionality, and body of funding.

5: The original paper can be found on Melvyn Conway's website [here](http://www.melconway.com/Home/Committees_Paper.html).

6: We can't resist mentioning Jim Webber's statement that ESB stands for ["Egregious Spaghetti Box"](http://www.infoq.com/presentations/soa-without-esb).

7: Netflix makes the link explicit - until recently referring to their architectural style as fine-grained SOA.

8: At extremes of scale, organisations often move to binary protocols - [protobufs](https://code.google.com/p/protobuf/) for example. Systems using these still exhibit the characteristic of smart endpoints, dumb pipes - and trade off transparency for scale. Most web properties and certainly the vast majority of enterprises don't need to make this tradeoff - transparency can be a big win.

9: "YAGNI" or "You Aren't Going To Need It" is an [XP principle](http://c2.com/cgi/wiki?YouArentGonnaNeedIt) and exhortation to not add features until you know you need them.

10: It's a little disengenuous of us to claim that monoliths are single language - in order to build systems on todays web, you probably need to know JavaScript and XHTML, CSS, your server side language of choice, SQL and an ORM dialect. Hardly single language, but you know what we mean.

11: Adrian Cockcroft specifically mentions "developer self-service" and "Developers run what they wrote"(sic) in [this excellent presentation](http://www.slideshare.net/adrianco/flowcon-added-to-for-cmg-keynote-talk-on-how-speed-wins-and-how-netflix-is-doing-continuous-delivery) delivered at Flowcon in November, 2013.

12: We are being a little disengenuous here. Obviously deploying more services, in more complex topologies is more difficult than deploying a single monolith. Fortunately, patterns reduce this complexity - investment in tooling is still a must though.

13: In fact, Dan North refers to this style as _Replaceable Component Architecture_ rather than microservices. Since this seems to talk to a subset of the characteristics we prefer the latter.

14: Kent Beck highlights this as one his design principles in [Implementation Patterns](https://www.amazon.com/gp/product/0321413091?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0321413091).

15: And SOA is hardly the root of this history. I remember people saying "we've been doing this for years" when the SOA term appeared at the beginning of the century. One argument was that this style sees its roots as the way COBOL programs communicated via data files in the earliest days of enterprise computing. In another direction, one could argue that microservices are the same thing as the Erlang programming model, but applied to an enterprise application context.
