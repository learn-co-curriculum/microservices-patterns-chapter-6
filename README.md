## # Ch 6 Developing business logic with event sourcing

### Intro

### 6.1 Developing business logic using event sourcing

- it persists an aggregate as a sequence of events

- app recreates the current state of an aggregate by replaying the events

- benefits:

  - preserves history (auditing, regulatory)

  - reliably publishes domain events 

- drawbacks:

  - learning curve

  - querying event store is often difficult (need to use CQRS pattern)

- trouble with traditional persistence

  - object-relational impedance mismatch(?)

    - mismatch between tabular relational schema and graph structure of rich domain model

    - "Object-Relational mapping is the Vietnam of Computer Science" [The Vietnam of Computer Science · Ted Neward's Blog](http://blogs.tedneward.com/post/the-vietnam-of-computer-science/)

  - lack of aggregate history

    - only updates current state, history is lost

  - implementing audit logging is tedious and error prone

    - audit logging! 

    - time consuming and also if audit logging code and business logic diverge ==> BUGS!

  - event publishing bolted onto business logic

    - can invoke application-provided callbacks when data objects change, but no support for auto publishing messages as part of transaction

- event sourcing

  - Event sourcing is an event-centric technique for implementing business logic and persisting aggregates

  - Event sourcing persists each aggregate as a sequence of events in the database, known as an event store

  - An application loads an aggregate from current store by retrieving events and replaying them e.g.

    - Load events for aggregate

    - Create an aggregate instance using constructor

    - Iterate through events and call apply (similar to a functional fold or reduce)

  - With event sourcing, the aggregate determines the events and their structure (as opposed to consumers of events)

  - An event must contain the data that the aggregate needs to perform the state transition (Order S.apply(Event E) == Order S')

  - Transaction log tailing is used to reliably publish events 

  - Long-lived aggregates can have a large number of events, so it is common to periodically persist a snapshot of the aggregate's state.

  - A snapshot can be the aggregate's JSON serialization (or use the Memento pattern)

  - Evolving domain events

    - things can change, the model may change. how do you handle it? adding new things is easy. removing things is hard.

    - in sql, you would run a migration.

    - in event sourcing, you would transform events when they're loaded from the store

    - the "upcaster" updates individual events from an old version to a newer one 

  - benefits of event sourcing:

    - reliably publishes domain events

      - state change => event

      - store identity of user who made the change

    - preserves history of aggregates

    - mostly avoids the O/R impedance mismatch problem

    - provides developers with a time machine

  - drawbacks:

    - different programming model

    - complexity of messaging-based app

    - evolving events is tricky

    - deleting data is tricky

      - traditional way to delete data is to soft delete 

      - application deletes an aggregate by setting a "deleted" flag (sounds familiar, see: user.deleted_at in Ironboard)

      - soft delete works for most stuff except GDPR (!)

      - you can use encryption key to encrypt any events containing user's personal information, store the encryption key in separate table

      - psuedoanonymization is another solution

    - querying event store is challenging

	- there aren't columns for easy lookup like credit_limit, you have to write a complex query by folding events over time 

### 6.2 Implementing an event store

  - An event store is a hybrid of a database and a message broker

  - Behaves as a db b/c it has an API for inserting and retrieving aggregate's events by primary key

  - Message broker b/c it has an API for subscribing to events

  - You can build your own event store with a RDBMS, poll the EVENTS table

  - OR use a special-purpose event store like Event Store, Lagom, Axon, Eventuate

  - Eventuate

    - EVENTS

    - ENTITIES

    - SNAPSHOTS

    - tail transaction log to publish events to message broker

### 6.3 Using sagas and event sourcing together

  - Event sourcing makes it easy to use choreography-based sagas

  - Event-sourcing with orchestration-based sagas is harder

  - Implementing choreography-based sagas using event sourcing

    - "straightforward"

    - aggregate updated, emits an event

    - event handler for different aggregate consumes that event and updates its aggregate

    - when combined with event sourcing, events now have a dual purpose

    - event sourcing uses events to represent state changes, but using events for saga choreography requires aggregate to emit an event even if there is no state change (as in, for errors)

  - creating an orchestration-based saga

  - Creating a saga orchestrator when using a NoSQL-based event store

  - Implementing an event-sourcing based saga participant

  - Implementing saga orchestrators using event sourcing

    - `SagaOrchestratorCreated`

    - `SagaOrchestratorUpdated`
<p class='util--hide'>View <a href='https://learn.co/lessons/microservices-patterns-chapter-6'>Microservices Patterns Chapter 6</a> on Learn.co and start learning to code for free.</p>
