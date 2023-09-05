+++
title = "Monolith, SOA, Microservices, when to use what"
date = "2023-09-05T05:00:46+04:00"
author = "Danny Hawkins"
authorTwitter = "dannyhawkins"
categories = ["architecture"]
tags = ["architecture", "monolith", "soa","microservices"]
showFullContent = false
readingTime = true
hideComments = false
+++

There has been a lot of hype around microservice architecture in the last decade. More often than not, discussions focus on its numerous advantages. However, diving into a microservices architecture without careful consideration and awareness of its complexities could set you up for a nasty surprise.

In this article I want to discuss the pros and cons of 4 different architectural approaches, the **tightly coupled monolith**, the **loosly coupled monolith**, **service oriented architecture (SOA)** and **microservices**, as well as my recommendations on when to use what and tooling / approaches that can help

## Tightly Coupled Monolith

A tightly coupled monolith refers to a software architecture where different components or functionalities are deeply interwoven and dependent on each other within a single codebase. Changes or failures in one component can affect others, making the system less modular and flexible. This architecture contrasts with more decoupled or modular systems where components interact through well-defined interfaces and can often be developed, scaled, or replaced independently.

### Pros

- **Simplicity**: Monolithic applications are often easier to develop initially since everything is in one place, without the need for complex communication between services.
- **Single Deployment**: You deploy a single application, which can simplify the deployment process and avoid complexities of managing multiple services.
- **Atomic Changes**: Changes can be made atomically, meaning everything can be updated at once. This can reduce inconsistencies and version mismatch issues.
- **Tight Integration**: All components are naturally integrated, so there's less chance of incompatibilities between modules.
- **Efficiency**: There's no network overhead of calling between different services, which can boost performance.
- **Simplified Testing**: Testing can be more straightforward since you're only dealing with a single application.
- **Transactional Simplicity**: Transactions across modules are simpler to manage without having to handle distributed transactions.
- **No Service Coordination**: There's no need to manage coordination or communication between multiple services.
- **Immediate Consistency**: Data consistency is often easier to ensure with a monolithic architecture.
- **Simplified Scaling**: Initially, it's often simpler to scale a monolithic application by just replicating the entire application.

### Cons

- **Limited Scalability**: Monolithic applications can become a bottleneck, especially as they grow, making it harder to scale specific parts of the app.
- **Lengthy Builds**: As the codebase grows, build and deployment times can become significantly longer.
- **High Impact of Failures**: If one part of the system fails, the entire system could go down.
- **Inflexibility**: Making changes or adding features can be cumbersome as the codebase grows.
- **Barrier to New Technologies**: It's challenging to integrate new technologies or frameworks into a large monolithic codebase.
- **Tough Parallel Development**: When many developers work on the same codebase, it can lead to merge conflicts and integration issues.
- **Hard to Understand**: A large codebase can be overwhelming and challenging to understand for new developers.
- **Slower Iteration**: Larger monolithic systems can slow down the development cycle and make continuous delivery or continuous integration harder.
- **Technical Debt**: Over time, the monolith can accumulate more technical debt, leading to more complexities and making refactoring challenging.
- **Vendor Lock-in**: Monolithic architectures might lead to being locked into specific technology choices made early in the project.


## Loosley Coupled Monolith

A loosely coupled, modular monolith refers to a software architecture where the application is organized into distinct and independent modules within a single codebase. These modules interact through well-defined interfaces, ensuring minimal dependencies between them. While still a monolithic application in terms of deployment, its design emphasizes modularity and separation of concerns, allowing for easier maintenance, testing, and potential scaling of individual modules without affecting others.


### Pros

- **Modularity**: Encourages cleaner code organization based on functionality or domain, which improves maintainability.
- **Isolation**: Failures or bugs in one module are less likely to affect other parts of the application.
- **Easier Scaling**: Specific modules that require more resources can be scaled independently, even within the confines of a monolithic deployment.
- **Flexibility**: Modules can be developed, replaced, or refactored without affecting other parts of the system significantly.
- **Easier Parallel Development**: Different teams can work on different modules with minimal interference.
- **Reusability**: Modules can be reused across different parts of the application or even in other projects.
- **Optimized Performance**: Individual modules can be optimized for their specific tasks without compromising the performance of other modules.
- **Incremental Refactoring**: Transitioning to a different architecture, like microservices, can be done incrementally, one module at a time.
- **Better Testability**: Individual modules can be tested in isolation, which can improve test coverage and reliability.
- **Tech Stack Flexibility**: Different modules can potentially be written in different technologies or frameworks, provided they expose consistent interfaces.

### Cons

- **Complexity**: While more modular than tightly coupled monoliths, they can still grow complex over time.
- **Interface Management**: The interfaces between modules must be carefully managed to ensure compatibility.
- **Deployment Overhead**: While it's simpler than deploying many microservices, it might be more complex than deploying a tightly coupled monolith.
- **Potential for Hidden Coupling**: Poor discipline can lead to hidden dependencies and unintentional coupling.
- **Integration Testing**: Ensuring that all modules work harmoniously can require extensive integration testing.
- **Performance Overhead**: Inter-module communication, even within a single codebase, might introduce some performance overhead.
- **Learning Curve**: Developers might need to adapt to the modularity and the best practices required to maintain loose coupling.
- **Possible Inconsistencies**: If not managed well, different modules might implement slightly different behaviors or data models.
- **Deployment Consistency**: Ensuring all modules are compatible during deployment can introduce challenges, especially when modules are being updated at different rates.
- **Transition Challenges**: Moving from a tightly coupled monolith to a modular one can be a time-consuming process.

## Service Oriented Architecture (SOA)

Service-Oriented Architecture (SOA) is a software design pattern where components provide services to other components through well-defined interfaces and protocols, typically over a network. It emphasizes modularity, interoperability, and reusability, allowing different services to communicate seamlessly and facilitating easier integration of diverse systems.

### Pros

- **Scalability**: Services can be independently scaled based on demand, allowing for more efficient use of resources.
- **Flexibility**: New services or changes to existing ones can be introduced without disrupting other services or clients.
- **Reusability**: Services are designed to be reused across different applications or parts of the same application.
- **Interoperability**: SOA often employs standard communication protocols, allowing for integration with a wide range of services and platforms.
- **Modularity**: SOA promotes modularity in design, which can lead to better organized and more maintainable systems.
- **Improved Fault Isolation**: If one service fails, it does not necessarily mean the entire system will fail.
- **Parallel Development**: Different teams can work on separate services simultaneously, speeding up development.
- **Location Independence**: Services can be located anywhere, facilitating distributed development and deployment.
- **Optimized Maintenance**: Fixing or updating a specific service can be done without taking down the entire system.
- **Easier Integration**: Integrating with third-party systems or legacy applications can be more straightforward with SOA, especially when standards like SOAP or REST are used.

### Cons

- **Complexity**: Introducing services, especially if overdone, can add a level of complexity in terms of setup, maintenance, and monitoring.
- **Performance Overhead**: The constant communication between services, especially if over a network, can introduce latency.
- **Increased Network Dependency**: Relying heavily on network communication can introduce potential points of failure.
- **Security Concerns**: More communication endpoints mean more potential entry points for malicious actors.
- **Data Consistency**: Managing data consistency across services can be challenging, especially in distributed systems.
- **Versioning Issues**: As services evolve, ensuring that different versions of services interact correctly can become problematic.
- **Potential for Tight Coupling**: Poorly designed services can become tightly coupled, defeating one of the main purposes of SOA.
- **Testing Challenges**: Testing in an SOA environment can be more complicated, especially when considering integration testing across services.
- **Deployment and Monitoring**: Deploying and monitoring multiple services can require more sophisticated tools and processes.
- **Initial Setup Time**: Setting up an SOA, especially in environments not used to it, can require a significant initial time investment.

## Microservices

Microservices is an architectural style that structures an application as a collection of small, independent services, each focused on a specific business capability. These services are loosely coupled, can be developed, deployed, and scaled independently, and they communicate with each other using lightweight protocols, typically over HTTP. This approach aims to enhance scalability, flexibility, and maintainability compared to traditional monolithic architectures.

### Pros

- **Scalability**: Individual microservices can be scaled independently based on their specific demand.
- **Flexibility in Technology Stack**: Different microservices can use different technologies, frameworks, and data stores.
- **Resilience**: A failure in one service doesn't necessarily bring down the entire system.
- **Easier Deployment and Rollback**: Changes or fixes can be deployed for a single service without affecting the entire system. If issues arise, rollbacks affect only the updated service.
- **Organizational Alignment**: Microservices can align well with small development teams, each taking ownership of specific services.
- **Granularity**: Focused on single business capabilities, which can lead to more manageable and understandable codebases.
- **Independent Development and Release Cycles**: Each service can be developed, deployed, and versioned independently.
- **Optimized for Continuous Deployment/Delivery**: Fits well with modern CI/CD pipelines, allowing frequent releases.
- **Improved Fault Isolation**: A problematic service can be isolated and dealt with without affecting others.
- **Potential for Reusability**: Some services might be reusable across different projects or parts of the system.

### Cons

- **Complexity**: Introduces complexities in terms of service coordination, data consistency, and network reliance.
- **Network Latency**: Inter-service communication over a network can introduce latency.
- **Data Consistency Challenges**: With each service managing its own database, ensuring data consistency can be complex, especially without distributed transactions.
- **Operational Overhead**: Requires sophisticated monitoring, logging, and alerting to keep track of numerous services.
- **Service Discovery**: As services scale or change, mechanisms are needed to track and locate them.
- **Integration Testing**: Testing the interactions between multiple services can be challenging.
- **Deployment Complexity**: Requires advanced deployment orchestration tools and practices.
- **Potential for Overengineering**: Not every application needs a microservices approach, and using it unnecessarily can introduce unneeded complexity.
- **Versioning**: Managing different versions of services and ensuring compatibility can be tricky.
- **Security Concerns**: More services mean more potential points of attack. Ensuring secure communication and isolation can be demanding.

## Wait, so whats the difference between SOA and Microservices

Service-Oriented Architecture (SOA) and microservices are both architectural patterns that emphasize building applications as collections of services. However, they have different design philosophies, characteristics, and implementations. Here's a comparison to highlight the differences:

1. Scope and Granularity:
   * **SOA:** SOA often revolves around enterprise-level services, which might lead to larger, more encompassing services often referred to as "enterprise services". These services might group multiple functionalities.
   * **Microservices:** As the name implies, microservices focus on breaking down the application into small, single-functionality services. Each service typically corresponds to a specific business capability.
2. Communication:
   * **SOA:** Traditionally uses Enterprise Service Bus (ESB) for communication which can handle routing, transformation, and applying business rules. This could sometimes introduce a centralized bottleneck.
   * **Microservices:** Tends to use simpler communication mechanisms like HTTP/REST or lightweight messaging queues. The idea is to keep inter-service communication decoupled and direct.
3. Data Storage:
   * **SOA:** Services in SOA might share a centralized database.
   * **Microservices:** Each microservice typically manages its own database to ensure decoupling and maintain its bounded context.
4. Size and Complexity:
   * **SOA:** Components in SOA might be larger and can be heterogeneous in nature.
   * **Microservices:** Components are smaller, focused, and are typically homogeneous in terms of technology and design.
5. Service Independence:
   * **SOA:** Though services are modular, they might still have dependencies on shared components like the ESB.
   * **Microservices:** Services are designed to be fully independent, capable of being developed, deployed, and scaled individually.
6. Development and Deployment:
   * **SOA:** Might have a more centralized governance and often requires coordinated deployments.
   * **Microservices:** Advocates for decentralized governance, allowing individual teams to develop, deploy, and scale their services independently.
7. Failure Isolation:
   * **SOA:** A failure in the ESB or a core service can impact multiple consumers.
   * **Microservices:** Services are isolated, so the failure of one microservice (in theory) shouldn't impact others.
8. Technology Stack:
   * **SOA:** Might be more prescriptive in terms of technology due to centralized governance.
   * **Microservices:** Provides flexibility for individual services to choose the technology stack that fits best for their requirements.

## When to use what

There are many factors to consider when deciding which of these approaches to adopt. While some factors are more evident than others, let's delve into specific profiles:

### Startup

When starting a new company, it's best to opt for the simplest approach. This will facilitate rapid development, from MVP to your fully realized initial product. Given that startups often have small development teams, it's unlikely you'd want separate product teams for every aspect of your product or service.

A loosely-coupled modular monolith is advisable here. But why not a tightly-coupled monolith? In my view, there's seldom a reason to develop a tightly-coupled monolith. It's essential to conceptualize distinct modules from the outset, even if they aren't perfect initially, refactoring is always an option. Overlooking boundaries at the beginning with the notion of revisiting them later can cause substantial issues down the line.

With a loosely-coupled monolith, you benefit from simpler development and deployment processes, all while avoiding challenges like networking, versioning, and transactional consistency. Plus, the modular approach lays the groundwork for transitioning into SOA or microservices as your business scales.

### Re-architecture

If you're contemplating transitioning from a monolith to a more distributed system, I'd advocate for SOA. Employ domain-driven design (DDD) to pinpoint your business's domains. A heuristic I favor is to identify areas of your business that could standalone with a unique value proposition. For instance, at Quiqup, we designated "fleet" as a domain, covering all facets concerning driver fleet management—a domain that could very well be an independent business.

When transitioning to SOA or microservices, it's crucial to establish contracts between services. This could entail agreed-upon JSON/Avro schemas for integration events or well-defined OpenAPI specs. Proper contracts accelerate development and facilitate testing at the entry and exit points of individual systems.

### Many teams / domains

If you're in the advantageous position of overseeing multiple teams and domains, your choice should oscillate between SOA and microservices. These architectures empower teams with autonomy and the authority over their software, granting them the liberty to decide how to best deliver business value. However, managing numerous teams or services necessitates robust platform architecture. A software catalog can be invaluable at this juncture.

Proficient platform engineering and DevOps practices are vital when working with microservices, and even SOA depending on your service count. If you lack effective mechanisms to handle:

- New service creation
- Service Versioning
- Contract / API management
- Dependency versioning
- Network failures

You may encounter challenges with SOA or microservices.

## Tooling

### Service Weaver

*Note: Golang only!*

[Service Weaver](https://serviceweaver.dev/docs.html) courtesy of Google, is a framework for designing, deploying, and managing distributed applications. It facilitates local application testing and debugging and offers effortless cloud deployment.

The platform allows for single-repo code development that can be deployed as separate services. Thus, you enjoy the convenience of a monolith during development and the advantages of microservices during deployment. Moreover, it supports comprehensive local testing spanning all concerns.

### Temporal.io

[temporal.io](https://temporal.io/)
is an open-source programming paradigm designed to streamline your code, bolster application reliability, and expedite feature delivery.

I'm currently navigating through the Temporal 101 and 102 courses, but my initial impressions are positive. Temporal accommodates multiple programming languages and oversees task coordination, albeit all code execution remains local. More details are available on their [website](https://temporal.io/how-temporal-works).