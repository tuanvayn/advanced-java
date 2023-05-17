# foreword

Deploying a monolithic application means running multiple copies of a large application, typically providing several (N) servers (physical or virtual), running several (M) application instances. Deploying a monolithic application won't be straightforward, but it's certainly easier than deploying a microservices application.

A microservice application consists of hundreds of services, which can be written in different languages and frameworks. Each service is a single application that can have its own deployment, resource, scaling and monitoring requirements. For example, several service instances can be run according to the service requirements, in addition, each instance must have its own CPU, memory and IO resources. Even more challenging, though complex, is that service deployment must be fast, reliable, and cost-effective.

There are some patterns of microservice deployment, first discuss the pattern of multiple service instances per host.

# Single host multi-service instance mode

One way to deploy microservices is the single-host multi-service instance mode. Using this mode, several physical or virtual machines need to be provided, and multiple service instances run on each machine. In many cases, this is the traditional approach to application deployment. Each service instance runs one or more well-known ports of the host, which can be regarded as a pet.

The diagram below shows this architecture：

![deployment-strategy-1](./images/deployment-strategy-1.png)

This mode has some parameters, one parameter represents how many processes each service instance consists of. For example, a Java service instance needs to be deployed on Apache Tomcat Server as a web application. A Node.js service instance may consist of a parent process and several child processes.

Another parameter defines how many instances of the service are running within the same process group. For example, you can run multiple Java web applications on the same Apache Tomcat Server, or run multiple instances of OSGI bundles inside the same OSGI container.

The single-host multi-service instance mode also has advantages and disadvantages. The main advantage is resource utilization efficiency. Multiple service instances share the server and operating system. It is more efficient if the process group runs multiple service instances, for example, multiple web applications share the same Apache Tomcat Server and JVM.

Another advantage is that deploying service instances is fast. Just copy the service to the host and start it. If the service is written in Java, just copy the JAR or WAR file. For other languages, such as Node.js or Ruby, you need to copy the source code. That is, the network load is low.

Since there is not much load, starting the service is fast. If the service is a self-contained process, it only needs to be started; otherwise, if it is a service instance running in a container process group, it needs to be dynamically deployed into the container, or the container needs to be restarted.

In addition to the above advantages, single-host multiple service instances also have drawbacks. One of the main disadvantages is that there is little or no isolation between service instances unless each service instance is a separate process. If you want to accurately monitor the resource usage of each service instance, you cannot limit the resource usage of each instance. So it is possible for a bad service instance to take up all the memory or CPU of the host.

There is no isolation for multiple service instances within the same process. All instances may, for example, share the same JVM heap. A bad service instance can easily attack other services in the same process; even more, it may not be possible to monitor the resources used by each service instance.

Another serious problem is that the operation and maintenance team must know the detailed steps of how to deploy. Services can be written in different languages and frameworks, so the development team must have a lot to communicate with the operation and maintenance team. The complexity increases the possibility of errors during the deployment process.

As you can see, despite the familiarity, single host multiple service instances have a number of serious flaws. Let's see if there are other ways to deploy microservices that can avoid these problems.

# Single host single service instance mode

Another way to deploy microservices is the single-host single-instance mode. When using this mode, each service instance on each host is independent. There are two different implementation modes: single-VM single-instance and single-container single-instance.

## Single VM Single Instance Mode

But with the single virtual machine single instance mode, the service is usually packaged as a virtual machine image (image), such as an Amazon EC2 AMI. Each service instance is a VM (for example, an EC2 instance) launched using this image. The following diagram shows this architecture:

![deployment-strategy-2](./images/deployment-strategy-2.png)

Netflix uses this architecture to deploy video streaming service. Netflix uses Aminator to package each service into an EC2 AMI. Each running service instance is an EC2 instance.

There are many tools that can be used to set up your own VMs. A continuous integration (CI) service (for example, Jenkins) can be configured to prevent Aminator from packaging the service into an EC2 AMI. packer.io is another option for automated VM image creation. Unlike Aminator, it supports a range of virtualization technologies such as EC2, DigitalOcean, VirtualBox and VMware.

Boxfuse has an innovative approach to creating virtual machine images that overcomes the following drawbacks. Boxfuse packages java applications into minimal virtual machine images, which are created quickly and started quickly, and are safer because there are fewer service interfaces exposed to the outside world.

The CloudNative company has a SaaS application for creating EC2 AMIs, Bakery. After the user microservice architecture has passed the test, you can configure your own CI server to activate Bakery. Bakery packages services as AMIs. Using a SaaS application like Bakery means users don't need to waste time setting up their own AMI creation architecture.

The service instance per virtual machine model has many advantages, the main VM advantage is that each service instance runs completely independently, has its own independent CPU and memory and will not be occupied by other services.

Another benefit is that users can use mature cloud architectures, such as those provided by AWS, and cloud services provide useful functions such as load balancing and scalability.

Another benefit is that the service implementation technology is self-contained. Once a service is packaged into a VM it becomes a black box. The VM's management API becomes the deployment service's API, and deployment becomes a very simple and reliable affair.

The single-VM single-instance model also has disadvantages. One disadvantage is the inefficient use of resources. Each service instance occupies the resources of the entire virtual machine, including the operating system. Moreover, in a typical public IaaS environment, virtual machine resources are standardized and may not be fully utilized.

Moreover, public IaaS charges based on the VM, regardless of whether the virtual machine is busy; for example, AWS provides an automatic scaling function, but lacks a quick response to on-demand applications, forcing users to deploy more virtual machines, thereby increasing deployment costs.

Another disadvantage is that deploying new versions of services is slow. The virtual machine image is relatively slow to create due to its size. For the same reason, the virtual machine initialization is also relatively slow, and the operating system startup also takes time. But this is not always the case, some lightweight VMs, such as those created with Boxfuse, are faster.

The third disadvantage is that for the operation and maintenance team, they are responsible for a lot of customization work. Unless you use a tool like Boxfuse that can help relieve a lot of the work of creating and managing virtual machines; otherwise, a lot of time will be spent on work that is not very irrelevant to the core business.

So let's take a look at another microservice deployment method that still has the characteristics of a virtual machine but is relatively lightweight

# Single container single service instance pattern

When using this pattern, each service instance runs in its own container. A container is a virtualization mechanism that runs at the operating system level. A container contains several processes running in a sandbox. From the process point of view, they have their own namespace and root file system; the memory and CPU resources of the container can be limited. Certain containers also have IO constraints, such container technologies include Docker and Solaris Zones.

The diagram below illustrates this pattern:

![deployment-strategy-3](./images/deployment-strategy-3.png)

Using this pattern requires packaging the service as a container image. A container image is a file system that runs the libraries and applications that contain the services required. Some container images consist of a full linux root filesystem, others are lightweight. For example, to deploy a Java service, a container image needs to be created that contains the Java runtime, and perhaps the Apache Tomcat server, as well as the compiled Java application.

Once the service is packaged into a container image, several containers need to be started. Generally, multiple containers are run on a physical machine or a virtual machine, and a cluster management system, such as k8s or Marathon, may be required to manage the containers. The cluster management system regards the host as a resource pool, and decides which host to schedule the container to according to the resource requirements of each container.

The single container single service instance mode also has advantages and disadvantages. The advantages of containers are similar to those of virtual machines. Service instances are completely independent, making it easy to monitor the resources consumed by each container. Similar to virtual machines, containers use isolation technology to deploy services. The container management API is also available as an API for management services.

However, unlike virtual machines, containers are a lightweight technology. Container images are created quickly, for example, packaging a Spring Boot application into a container image takes only 5 seconds on a laptop. Because no operating system startup mechanism is required, the container startup is also very fast. When the container starts, the background service is started.

There are also some downsides to using containers. Although the container architecture is developing rapidly, it is still not as mature as the virtual machine architecture. And because the host OS kernel is shared between containers, it is not as secure as a virtual machine.

In addition, container technology will put forward many customized requirements for managing container images. Unless you use Google Container Engine or Amazon EC2 Container Service (ECS), users will need to manage container architecture and virtual machine architecture at the same time.

Third, containers are often deployed on an architecture that is charged according to the virtual machine. Obviously, customers will also increase deployment costs to cope with load growth.

Interestingly, the distinction between containers and VMs is getting blurred. As mentioned earlier, Boxfuse virtual machine startup and creation are very fast, and Clear Container technology is aimed at creating lightweight virtual machines. The technology of unikernel companies has also attracted everyone's attention, and Docker recently acquired Unikernel companies.

In addition to these, server-less deployment technology, which avoids the defects of the aforementioned container and VM technology, has attracted more and more attention. Let's take a look below.

# Serverless deployment

AWS Lambda is an example of serverless deployment technology, supporting Java, Node.js and Python services; services need to be packaged into a ZIP file and uploaded to AWS Lambda for deployment. Metadata can be provided, providing the name of the function that handles the service request (an event). AWS Lambda automatically runs microservices that handle enough requests, but is only billed based on the running time and the amount of memory consumed. Of course, the details determine success or failure, and AWS Lambda also has limitations. But you don't need to worry about servers, virtual machines, or anything in containers that is absolutely attractive.

Lambda functions are stateless services. Requests are typically handled by activating AWS services. For example, when an image is uploaded to an S3 bucket to activate a Lambda function, an entry can be inserted into the DynamoDB image table, a message can be published to the Kinesis stream, and an image processing action can be triggered. Lambda functions can also be activated through third-party web services.

There are four ways to activate a Lambda function:

- Direct way, using web service request
- Automatically respond to events generated by AWS S3, DynamoDB, Kinesis or Simple Email Service, etc.
- Automatically handle HTTP requests from application clients through AWS API gateway
- Timing mode, response via cron --much like timer mode

It can be seen that AWS Lambda is a very convenient way to deploy microservices. The request-based billing method means that users only need to bear the load of processing their own business; in addition, because they do not need to understand the infrastructure, users only need to develop their own applications.

However, there are still many limitations. It does not need to be used to deploy long-term services, such as to consume messages forwarded from third-party agents, the request must be completed within 300 seconds, and the service must be stateless, because in theory AWS Lambda will generate an independent instance for each request ; must be done in one of the supported languages, and the service must start quickly, otherwise, it will be stopped due to a timeout

Deploying microservice applications is also a challenge. There are hundreds of services written in various languages and frameworks. Each service is a mini-application with its own unique deployment, resource, scaling, and monitoring needs. There are several microservice deployment models, including single-VM single-instance and single-container single-instance. Another optional mode is AWS Lambda, a serverless approach.