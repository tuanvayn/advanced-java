# 1.1 Microservices and Distributed Data Management Issues

Monolithic applications generally have a relational database. The benefit of this is that the application can use ACID transactions, which can bring some important operational features:

1. Atomicity - any change is atomic
2. Consistency - the database state is always consistent
3. Isolation - Even though transactions execute concurrently, they appear to be serial
4. Durable – once a transaction is committed it cannot be rolled back

Given the above characteristics, the application can be simplified as: start a transaction, change (insert, delete, update) many rows, and then commit these transactions.

Another advantage of using a relational database is that it provides SQL (a powerful, declarative, table-transformed query language) support. Users can easily combine the data of multiple tables through queries, and the RDBMS query scheduler determines the best implementation method, and users do not need to worry about underlying issues such as how to access the database. In addition, because all application data is in one database, it is easy to query.

However, with a microservice architecture, data access becomes very complicated because the data is private to the microservice and the only way it can be accessed is through the API. This packaged data access method makes microservices loosely coupled and independent of each other. If multiple services access the same data, the schema updates access times and coordinates between all services.

What's more, different microservices often use different databases. Applications generate a variety of data, and relational databases are not necessarily the best choice. In some scenarios, a NoSQL database may provide a more convenient data model, providing better performance and scalability. For example, an application that generates and queries strings employs a character search engine such as Elasticsearch. Similarly, an application that generates social image data can use an image database, for example, Neo4j; therefore, microservice-based applications generally use a database that combines SQL and NoSQL, which is a method called polyglot persistence.

Partitioned, polyglot-persistent architectures for storing data have many advantages, including loosely coupled services and better performance and scalability. However, with that comes the challenge of distributed data management.

The first challenge is how to complete a transaction while maintaining data consistency among multiple services. The reason for this problem is that we take an online B2B store as an example. Customer service maintenance includes various information of customers, such as credit lines. The order service manages orders and needs to verify that a new order does not violate the customer's credit limit. In a monolithic application, the order service only needs to use ACID transactions to check available credit and create an order.

On the contrary, under the microservice architecture, the order and customer tables are private tables of the corresponding services, as shown in the following figure:

![service table](./images/Private-table-of-the-corresponding-service.png)

The order service cannot directly access the customer table, it can only be accessed through the API published by the customer service. The order service can also use distributed transactions, also known as two-phase commit (2PC). However, 2PC is not optional in current applications. According to CAP theory, a choice must be made between availability and ACID consistency, and availability is generally the better choice. However, many modern technologies, such as many NoSQL databases, do not support 2PC. Maintaining data consistency between services and databases is a very fundamental requirement, so we need to find other solutions.

The second challenge is how to accomplish searching data from multiple services. For example, imagine an application that needs to display a customer and his orders. If the order service provides an API to accept user order information, then the user can use the application-like join operation to receive the data. The application accepts user information from the user service, and accepts the user's order from the order service. Assuming that the order service only supports querying orders by private key (perhaps using a NoSQL database that only supports acceptance based on primary key), at this time, there is no suitable method to receive the required data.

# 1.2 event-driven architecture

For many applications, the solution is to use an event-driven architecture. In this architecture, a microservice publishes an event when something important happens, such as updating a business entity. When the microservices that subscribe to these events receive this event, they can update their own business entities, and may also trigger more event publications.

Events can be used to implement business transactions across multiple services. Transactions generally consist of a series of steps, each of which consists of a microservice that updates a business entity and publishes an event that activates the next step. The figure below shows how to use the event-driven approach to check credit availability when creating an order, and the microservices exchange events through the Message Broker (Messsage Broker).

1. The order service creates an Order with NEW status and publishes an "Order Created Event" event.

![Order-Created-Event](./images/Order-Created-Event.png)

2. Customer service consumes the Order Created Event, reserves credit for this order, and publishes a "Credit Reserved Event" event.

![Credit-Reserved-Event](./images/Credit-Reserved-Event.png)

3. The order service consumes the Credit Reserved Event and changes the status of the order to OPEN.

![Status-is-OPEN](./images/Status-is-OPEN.png)

More complex scenarios can introduce more steps, such as reserving inventory while checking user credit, etc.

Considering that (a) each service atomically updates the database and publishes events, then, (b) the message broker ensures that the event is delivered at least once, then a business transaction can be done across multiple services (this transaction is not an ACID transaction). This mode provides weak certainty, such as eventual consistency. This type of transaction is called the BASE model.

Events can also be used to maintain an implementation view of different microservices having data pre-joined. The service maintaining this view subscribes to relevant events and updates the view. For example, the customer order view update service (maintaining the customer order view) subscribes to events published by the customer service and order services.

![pre-join](./images/pre-join.png)

When the customer order view update service receives customer or order events, it updates the customer order view dataset. The customer order view can be implemented using a document database such as MongoDB, storing one document per user. The customer order view query service is responsible for responding to queries for customers and recent orders (by querying the customer order view dataset).

The event-driven architecture also has both advantages and disadvantages. This architecture can make transactions span multiple services and provide final consistency, and can enable applications to maintain the final view; the disadvantage is that the programming model is more complicated than the ACID transaction model: in order to invalidate from the application level In recovery, compensating transactions also need to be completed, for example, the order must be canceled if the credit check is unsuccessful; in addition, the application must deal with inconsistent data, because changes caused by temporary (in-flight) transactions are visible, and when Applications can also encounter data inconsistencies when reading final views that are not updated. Another disadvantage is that subscribers must detect and ignore redundant events.

# 1.3 Atomic Operations Achieving Atomicity

Event-driven architectures also run into issues of atomicity of database updates and publishing events. For example, an order service must insert a row into the ORDER table and then publish an Order Created event, both operations need to be atomic. If after updating the database, the service crashes (crashes) and causes the event to fail to be published, the system becomes inconsistent. The standard way to ensure atomic operations is to use a distributed transaction involving the database and message broker. However, based on the CAP theory described above, this is not what we want.

## 1.3.1 Publish events using local transactions

One way to achieve atomicity is to apply a multi-step process involving only local transactions for publishing events. The trick is an EVENT table that functions as a list of messages in the database storing the business entities. The application initiates a (local) database transaction, updates the business entity state, inserts an event into the EVENT table, and commits the transaction. Another independent application process or thread queries the EVENT table, publishes the event to the message broker, and then uses the local transaction to mark the event as published, as shown in the following figure:

![multi-step process](./images/multi-step-process.png)

The order service inserts a row into the ORDER table, and then inserts an Order Created event into the EVENT table. The event publishing thread or process queries the EVENT table, requests unpublished events, publishes them, and then updates the EVENT table to mark this event as published.

This method also has advantages and disadvantages. The advantage is that you can ensure that event publication does not depend on 2PC, and applications publish business-level events without inferring what happened to them; the disadvantage is that this method has the potential for errors because developers must keep in mind publishing events. In addition, this method is a challenge for some applications using NoSQL databases, because NoSQL itself has limited transaction and query capabilities.

This method does not require 2PC because the application uses local transactions to update the state and publish events. Now let's look at another method for simply updating the state of the application to obtain atomicity.

## 1.3.2 Mining database transaction logs

Another way to obtain atomicity of thread or process publishing events without 2PC is to mine database transactions or commit logs. The application updates the database, making changes in the database transaction log, and the transaction log mining process or thread reads these transaction logs and publishes the logs to the message broker. As you can see in the picture below:

![No-2PC-required](./images/No-2PC-required.png)

Examples of this approach are the LinkedIn Databus project, where Databus mines Oracle transaction logs and publishes events based on changes, and LinkedIn uses Databus to ensure consistency across records within the system.

Another example is: AWS's streams mechanism in AWS DynamoDB, which is a manageable NoSQL database. A DynamoDB stream is based on time-series changes (create, update, and delete operations) to database tables in the past 24 hours. Applications can learn from the stream Read those changes, then publish those changes as events.

Transaction log mining also has advantages and disadvantages. The advantage is to ensure that every update release event does not depend on 2PC. Transaction log mining can be simplified by separating publishing events from application business logic; the main disadvantage is that transaction logs have different formats for different databases, and even different database versions; and it is difficult to convert from low-level transaction log update records to high-level business event.

The transaction log mining method directly updates the database through the application without 2PC intervention. Let's look at a completely different approach: a method that does not need to be updated and only depends on events.

## 1.3.3 Using event sources

Event sourcing (event source) achieves atomicity without 2PC by using a fundamentally different event center approach to ensure the consistency of business entities. This kind of application saves a series of state change events of business entities, rather than storing the current state of entities. Applications can reconstruct the entity's current state by replaying events. Whenever a business entity changes, new events are added to the timetable. Because the save event is a single operation, it must be atomic.

To understand how event sourcing works, consider event entities as an example. Traditionally, each order is mapped to a row in an ORDER table, for example in the ORDER_LINE_ITEM table. But with the event sourcing approach, the order service stores an order in terms of event state changes: created, approved, shipped, canceled; each event includes enough data to reconstruct the order state.

![Event-sourcing](./images/Event-sourcing.png)

Events are stored in the event database for a long time, and APIs are provided to add and obtain entity events. The event store is similar to the message broker described earlier, providing an API to subscribe to events. The event store delivers events to all interested subscribers, and the event store is the backbone of the event-driven microservice architecture.

The event source method has many advantages: it solves the key problem of the event-driven architecture, so that events can be released reliably as long as there is a state change, and it also solves the data consistency problem in the microservice architecture. In addition, because it is a persistent event rather than an object, the object relational impedance mismatch problem is avoided.

The data source method provides 100% reliable business entity change monitoring logs, making it possible to obtain entity status at any point in time. In addition, the event sourcing approach enables business logic to be composed of loosely coupled business entities exchanging events. These advantages make it relatively easy to port a monolithic application to a microservices architecture.

The event source method also has many shortcomings, because it adopts different or unfamiliar transformation modes, which makes it difficult to relearn; event storage only supports primary key query business entities, and Command Query Responsibility Segregation (CQRS) must be used to complete the query business. Therefore, applications must deal with eventually consistent data.

# 1.4 Summarize

In a microservice architecture, each microservice has its own private dataset. Different microservices may use different SQL or NoSQL databases. Although the database architecture has strong advantages, it also faces the challenge of distributed data management. The first challenge is how to maintain business transaction consistency between multiple services; the second challenge is how to obtain consistent data from a multi-service environment.

The best solution is to use an event-driven architecture. One of the challenges encountered is how to atomically update state and publish events. There are several approaches to this problem, including treating the database as a message queue, transaction log mining, and event sourcing.
