---
layout: pattern
title: Database per service
---
# Pattern: Database per service

## Context

Let's imagine you are developing an online store application using the [Microservices pattern](/patterns/microservices.html).
Most services need to persist data in some kind of database.
For example, the `Order Service` stores information about orders and the `Customer Service` stores information about customers.

<img class="img-responsive" src="/i/customersandorders.png"/>

## Problem

What's the database architecture in a microservices application?

## Forces

* Services must be loosely coupled so that they can be developed, deployed and scaled independently

* Some business transactions need to update data that is owned by multiple services.
For example, the `Place Order` use case updates the Customer to reserve credit for the Order and creates an Order.

* Some queries must join data that is owned by multiple services.
For example, finding customers in a particular region and their recent orders requires a join between customers and orders.

* Databases must sometimes be replicated and sharded in order to scale. See the [Scale Cube](/articles/scalecube.html).

* Different services have different data storage requirements.
For some services, a relational database is the best choice.
Other services might need a NoSQL database such as MongoDB, which is good at storing complex, unstructured data, or Neo4J, which is designed to efficiently store and query graph data.

## Solution

Keep each microservice’s persistent data private to that service and accessible only via its API.
The following diagram shows the structure of this pattern.

<img class="img-responsive" src="/i/databaseperservice.png"/>

The service's database is effectively part of the implementation of that service.
It cannot be accessed directly by other services.

There are a few different ways to keep a service’s persistent data private.
You do not need to provision a database server for each service.
For example,  if you are using a relational database then the options are:

* Private-tables-per-service – each service owns a set of tables that must only be accessed by that service
* Schema-per-service – each service has a database schema that’s private to that service
* Database-server-per-service – each service has it’s own database server.

Private-tables-per-service and schema-per-service have the lowest overhead.
Using a schema per service is appealing since it makes ownership clearer.
Some high throughput services might need their own database server.

It is a good idea to create barriers that enforce this modularity.
You could, for example, assign a different database user id to each service and use a database access control mechanism such as grants.
Without some kind of barrier to enforce encapsulation, developers will always be tempted to bypass a service’s API and access it’s data directly.

## Resulting context

Using a database per service has the following benefits:

* Helps ensure that the services are loosely coupled.
Changes to one service's database does not impact any other services.

* Each service can use the type of database that is best suited to its needs.
For example, a service that does text searches could use ElasticSearch.
A service that manipulates a social graph could use Neo4j.

Using a database per service has the following drawbacks:

* Implementing business transactions that span multiple services is not straightforward.
Distributed transactions are best avoided because of the CAP theorem.
Moreover, many modern (NoSQL) databases don't support them.
The best solution is to use an eventually consistent, event-driven architecture.
Services publish events when they update data.
Other services subscribe to events and update their data in response.

* Implementing queries that join data that is now in multiple databases is challenging.
There are various solutions:

 * Application-side joins - the application performs the join rather than the databse.
 For example, a service (or the API gateway) could retrieve a customer and their orders by first retrieving the customer from the order service and then querying the order service to return the customer's most recent orders.

 * Command Query Responsibility Segregation (CQRS) - maintain one or more materialized views that contain data from multiple services.
 The views are kept by services that subscribe to events that each services publishes when it updates its data.
 For example, the online store could implement a query that finds customers in a particular region and their recent orders by maintaining a view that joins customers and orders.
 The view is updated by a service that subscribes to customer and order events.

* Complexity of managing multiple SQL and NoSQL databases

## Related patterns

* [Microservices pattern](/patterns/microservices.html) creates the need for this pattern
* Event-driven architecture pattern is a useful way to implement eventually consistent transactions
* Command Query Responsibility Segregation (CQRS) is a useful way to implement complex queries
* The Shared Database anti-pattern describes the problems that result from microservices sharing a database
