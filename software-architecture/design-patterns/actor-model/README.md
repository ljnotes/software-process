# Actor model
- The **Actor model** is a mathematical and conceptual model for concurrent computation.
- Actor model is used to solve the challenges of OOP model and meet the challenges of distributed systems.

## Why Actor model and what problem does it solve?

### Encapsulation issues in OOP and multi-threading
- The main concept of OOP is **encapsulation** - internal data of an object should not be visible and directly accessible from outside, it should be changed by used methods intented for it.
- Say, *Object A*, has *method A* which changes some state - in a single thread of execution the state gets changed correctly. **What if there were two threads trying to access the method at the same time?** - it can lead to inconsistent state.
- Yes this can be **solved by using locks** which then means that we are **blocking one thread**, it can introduce **deadlocks**
- Additionaly **locks work very well only locally**; what about **distributed system**? We **need to use distributed lock protocols** - which creates **heavy-cost and latency issues as we need to communicate over network**
- **To summarize**:  

```
  Objects cannot guarantee Encapsulation rule and thus the internal state of the object
  to be maintained correctly in case of multiple threads.
```

### Shared memory concept
-  The below code execution logic over the period from 80-90's, to multi-core architecture:  
`var inputNumber = 11;`
Computer concept in 80-90's with single CPU = **we are writing directly to the register**  
With multi-core architecture, **it is written to local cache of a core - to the cache lines**  
- In order for another core to gets this information and make changes, the cache needs to be shipped from this core to another
- `Volatile` and `Atomic` markers (to mark it as a shared variable) is to give information that it is possible to modify this variable by multiple threads in multiple cores - but then again it is expensive as the **cache needs to be shipped and shared**.
- **To summarize**:  

```
  There is no real shared memory anywhere - CPU core pass chunks of data (cache lines) explicitly to each other.
  Instead of making variable as shared, more disciplined and principled approach is to keep the state local and
  propagate data / events across entities via messages.
```

### Call stack
- We **cannot take the call stack for granted** - it was invented when concurrent programming was not much of an use case
- What happens if our *main thread delegates* some work to another *background thread by placing in some shared memory* and continue with its operation, and the *some background thread* picks up this task from the shared memory and work on it. If an **exception happens or upon completion, how does the main thread get notified?**; **Where will the exception propagate?**
- One way to pass this error code to the main thread is to **place it where the `caller` expects it results**. What happens if the *worker thread* completely exceptions out - in this case the main thread doesn't have the error information in its call stack.
- **To summarize**:  
```
  We need to introduce new task-completion or exception propagation mechanism and cannot solely depend on call stack
```

### Actor model to deal with distributed computing concerns
- Actor model **enforces encapsulation without resorting to locks** - each actor work in single thread and holds it's own state
- Cooperating entities interact via signals - using **actor sent message to another actor to process it** - message is executed sequentially inside an actor - only one message is processed at a time.
- There is **no return value for a message sent**, the actor acts on it based on its current state and the **response is sent via a reply message**
- Once the actor finish processing the message **it returns its execution** - not the response like in OOP model.

## Fundamental concepts of Actor Model
- Like in OOP model, where **everything is an object**, the actor model adopts the philosophy that **everything is an actor**
- An **actor** is a primitive unit of computation which receive a **message** and respond to it by doing some king of **computation**
- It is similar to objects in OOP languages, which receives messages (method calls) and does something with it (the method action); The main difference is that **actors are isolated from each other** and they **never share memory**
- An actor is a **computational entity** which in response to a **message** it receives can concurrently:
  - **Send a finite number of messages to other actors**
  - **Create a finite number of new actors**
  - **designate the behavior to be used for the next message it receives**
- Actor model **decouples the sender from the communications sent**, enabling asynchronous communication
- The **recipient of a message are identified by an address**, sometimes called **mailing address**. Thus an actor can communicate with only those actors whose address it has which they can get from the message it receives, or if the address is for an actor that itself created.

## Terms in Actor model
- Actor is a container for **State**, **Behavior**, a **Mailbox**, **Child Actors** and a **Supervisor Strategy**
- All of this are encapsulated behind an **Actor reference**

### Actor Reference
- In order to benefit from the Actor model, the actors needs to be shielded from the outside. Therefore, the actors are represented to the outside as **actor reference** which are **objects which can be passed around freely without restrictions**.
- This split into **inner and outer objects** provides transparency for different operations:
  - Restarting an actor without needing to update the reference elsewhere
  - Placing the actual actor object on remote hosts
  - Sending messages to actors in completely different applications
- Main idea of **actor reference** is the **prevent looking inside of an actor and get hold of its state from outside**

### State
- Every actor has a set of variables which represent the current state it is in. This can be:
  - Explicit state machine
  - counter
  - set of listeners
  - pending requests
- As the **state** is vital to an actor, when the actor becomes faulty and is re-started by its supervisor - the state will be created from scratch
- Optionally an actor's state can be automatically recovered to the state before restart by persisting received messages and replaying them back after restart

### Behavior
- Every time a message is processed, it is **matched against the current behavior of the actor**
- **Behavior** means a **function which defines the actions to be taken to the message at that point in time**, say, forward a request if the client is authorized or deny otherwise. This behavior can change over time, for example, the client may get autorized over time.

### Mailbox
- An actor's purpose is to process messages. These messages were sent to the actor from other actors (or from outside the actor system).
- The piece which connects the sender and the receiver is the **actor's mailbox**. Each actor **has exactly one mailbox** to which the sender **enqueue the message**
- There are different mailbox implementations like - **FIFO**, **Priority Queue**, etc.

### Child Actors
- Each actor is potentially a **supervisor** if it has created child actors for delegating sub-tasks - it will then automatically supervise them.
- This list of children is maintained in the actor's context and it has access to it.

### Supervisor Strategy
- Is the **strategy for handling faults of its children**.

### References
- [The actor model by Brian Storti](https://www.brianstorti.com/the-actor-model/)
- [Actor model, Wikipedia](https://en.wikipedia.org/wiki/Actor_model)
- [Actor model, Akka](https://doc.akka.io/docs/akka/2.5/general/actors.html)
- [CPU Cache](https://en.wikipedia.org/wiki/CPU_cache)
- [Introduction of Actors, Akka](https://doc.akka.io/docs/akka/2.5.3/scala/guide/actors-intro.html)
