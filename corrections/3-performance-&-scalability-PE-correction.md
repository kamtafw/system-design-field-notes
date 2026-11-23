# STAGE 1.03 — Performance and Scalability

**Today is November 23, 2026.**

My answers to the exercise shows I'm beginning to investigate the problem before proposing s solution. I'm beginning to reason about system components behave under different workloads and how their behaviours affects the overall work system.

My weakest area is not necessarily my understanding of performance; it's my ability to distinguish between the different things that contribute to performance and the things that limit capacity.

## Part A — Identify performance characteristics

_I'm correct to recognize that different operations have different performance expectations and that external dependencies can significantly influence latency._

Here are things worth correcting.

### A1. Searching for flights

My response:

> A reasonable response time for this operation will be 2 seconds.

> Factors that could influence the search latency are:
>
> - time taken for the app to prepare and send its request to the backend
> - time taken for the Appwrite function to execute its flight-search logic
> - time taken for Duffel API to respond with the flight offers
> - time taken to process and render the response

It's a good breakdown cos I identified the major stages involved in the request lifecycle.

However, I'll challenge my two-second target.

Is two seconds a reasonable target for the entire flight-search operation, or just the time between the mobile app sending a request and receiving a response?

These are different measurements.

Now, consider this simplified sequence:

```txt
User initiates search
        |
        v
Mobile prepares request
        |
        v
Network request
        |
        v
Appwrite Function executes
        |
        v
Duffel API responds
        |
        v
Backend processes response
        |
        v
Mobile receives response
        |
        v
Mobile renders results
        |
        v
User sees results
```

There are two useful measurements in the sequence.

**Backend response latency:** The time between the mobile app sending a request and receiving the backend response.

**End-to-end user-perceived latency:** The time between the user initiating the search and being able to see and interact with results.

The distinction matters because an operation can complete on the backend while the mobile app still needs time to process and display the results.

My two-second target is reasonable initial hypothesis, but to define it more precisely:

> Assumption: 95% of flight searches should display their results within 5 seconds under an agreed workload and normal provider conditions.

Why P95? Because an average can hide the experience of users who consistently encounter slow searches.

Also, asides Duffel and the results display components as potential sources of delay, to expand slightly:

- Mobile network conditions
- Network latency between Appwrite and Duffel
- Appwrite Function execution and resource constraints
- Duffel response time
- Responses transformation and processing
- Mobile rendering and UI processing

Here's a subtle point: **the component that performs an operation isn't necessarily the component that introduces delay.**

For example, my Appwrite Function might execute correctly in 100 milliseconds, but spend eight seconds waiting for Duffel.

That distinction will become increasingly important as I study performance optimization.

### A2. Retrieving passenger information

My response:

> Assuming it's cached information, the response time will be probably faster than when the information is retrieved from a long-term storage unit.

There's a distinction between **temporary session data, saved passenger profiles, and completed booking records.** They may have different storage and retrieval requirements.

Identifying the database as a bottleneck is correct.

I'd expand the possible latency contributors to include:

- Mobile-to-backend network latency
- Authentication and authorization checks
- Appwrite Function execution
- Database query execution
- Database connection availability
- Response processing and rendering

It's not like I need to list every possible contributor, but I should understand that a database query is only one part of the complete operation.

### A3. Retrieving a seat map

My response:

> A reasonable response time for retrieving seat maps will be 1 second. Factors to influence its latency are very similar to the flight search, since its an operation that passes through a look-alike pathway.

My reasoning is sound. Both operations may involve:

- The mobile app
- Appwrite Functions
- Server-side logic
- Duffel API
- Response processing

Although, I shouldn't automatically assume they have identical performance characteristics.

A flight search might return several flight offers, while a seat-map request might return a considerably larger and complex data structure for a response that could contain:

- Multiple cabins
- Seat rows and columns
- Seat availability
- Seat characteristics
- Additional charges
- Passenger-specific information

Consequently, even if both operations use same external provider, their response sizes and processing requirements may differ.

Here's another consideration: seat availability is dynamic.

A seat that was available when the seat map was retrieved might no longer be available when the user attempts to select it.

This doesn't necessarily make seat-map retrieval slow, but it influences how I handle the information.

My answer is correct in principle. I just need to avoid assuming that similar request pathways imply similar performance.

### A4. Initiating a booking or payment

My response:

> When initiating a booking or payment, I'll assume a response time of about 5 seconds. This operation seem to execute a number of processing, validation, and external services. What could introduce delays here will be the execution by the payment provider.

I gave a good start, but there's a particularly important distinction for me to understand.

**Initiating payment is not the same as completing payment.**

Imagine this sequence:

1. The user confirms their booking details.
2. Book Flight validates the booking information.
3. Book Flight requests payment initialization from the payment provider.
4. The payment provider returns the information required to continue payment.
5. The user completes the payment process.
6. The payment provider processes the transaction.
7. Book Flight receives and verifies the payment outcome.

Some of these operations can happen within seconds. Others might take considerably longer.

For example, the user might be redirected to a bank authentication page or asked to approve a transaction. So what exactly should the five-second target measure?

Let me distinguish:

| Operation            | What we measure                                                                       |
| -------------------- | ------------------------------------------------------------------------------------- |
| Payment initiation   | Time taken to establish a payment session or obtain payment instructions.             |
| Payment processing   | Time taken for the payment provider to process the transaction.                       |
| Payment confirmation | Time taken for Book Flight to establish and communicate the verified payment outcome. |
| Booking confirmation | Time taken to establish and communicate the actual booking outcome.                   |

These operations don't necessarily complete together.

For Book Flight, this distinction is particularly important because payment and booking are separate stages of the overall booking lifecycle.

Identifying the payment provider as a possible source of delay is correct. Other contributors could include booking validation, database operations, network communication, and payment-status verification.

### What I need to remember

My approach was good, but here's a more complete mental model:

**Latency is the time an operation takes, while the individual stages of that operation contributes to its total latency.**

When investigating latency, I should ask:

1. What is the operation's starting point?
2. What is its completion point?
3. What stages occur between those points?
4. Which stages consume the most time?
5. Are the stages sequential or can some execute concurrently?

The fifth will become especially important when learning about asynchronous processing and parallel execution.

## Part B — Understand workload and capacity

In this section, I demonstrated a good understanding of how system workload differs from registered user count.

Let's examine my answers.

### B1. Which metrics deserve the most attention?

My response:

> The metrics to gain my most attention and influence my estimate the most are the peak concurrent users and the peak flight searches per minute.

These metrics tells me much more about actual system demand than registered user count.

But, let me introduce a distinction.

**Peak concurrent users and peak request rate describe different dimensions of workload.**

Peak concurrent users tell me how many users are active simultaneously. Peak flight searches per minute tells me how many search operations arrive during a given period.

They are related, but not interchangeable.

Imagine:

- 2,000 users are concurrently using Book Flight.
- Only 100 of them initiate flight searches during that minute.

Now imagine another situation:

- 500 users are concurrently using Book Flight.
- Each initiates several flight searches during that minute.

The second situation could generate more flight-search requests despite having fewer concurrent users.

This is why I should examine multiple workload metrics.

Let me also introduce **request distribution**.

Suppose Book Flight receives 1,000 searches per minute on average. That sounds manageable.

But what if 800 of those searches arrive within the first 10 seconds of every minute?

That creates a very different workload from receiving 1,000 evenly distributed requests throughout the minute.

This gives the difference between average demand and bursty demand.

For capacity planning, I need to understand not just how much work arrives, but how that work is distributed over time.

### B2. Why isn't registered user count sufficient?

My response:

> Registered user count alone will be insufficient because the metric itself don't impose a strong workload on the system, and a system can maintain same number of registered users but have the other metrics change dramatically to make the system workload significantly different.

This is an excellent response. But just a minor correction to the wording.

_Registered users don't directly impose continuous workload on the system. Their activity generates workload._

A system with 50,000 registered users might have only 200 active users on a particular day. Another system also with 50,000 registered users might have 30,000 active users. While their registered user count is identical, their actual workload differs considerably.

I have understood this correctly.

### B3. Which components experience increased demand?

My response:

> The parts I expect to experience increased demand will be the flight-search function and Duffel API.

This response is correct given the increase in flight-search requests.

But following the request beyond these two components, a flight-search request may also involve:

- Mobile app networking
- Appwrite Function execution capacity
- Authentication services, depending on the request flow
- Database access, if the operation reads or writes data
- Logging and monitoring infrastructure
- Network resources between services

Not all of these will necessarily experience a significant increase, but they are worth investigating.

Also, Duffel may have its own request limits. My Appwrite Functions might be able to process an increased demand, but Duffle might impose a rate limit.

My system's capacity is therefore influenced by constraints of its dependencies.

This introduces an important principle: **The capacity of a system can be constrained by a component outside the system's direct control.**

I don't necessarily have to scale that external component. I however need to understand its limits and design my application around them.

### B4. Will every component experience the same workload increase?

My response:

> Every component won't necessarily experience the same increase in workload since it doesn't have to be that a user making a flight search requires the user to retrieve passenger information and initiate a booking.

This is a very important observation: different operations have different request patterns.

For example:

| Operation                   | Potential demand                                        |
| --------------------------- | ------------------------------------------------------- |
| Flight search               | Very high                                               |
| Seat-map retrieval          | Moderate                                                |
| Passenger profile retrieval | Dependent on booking activity                           |
| Payment initiation          | Dependent on users proceeding to payment                |
| Booking retrieval           | Dependent on existing customers checking their bookings |

To extend this.

Suppose 1,000 users search for flights.

Perhaps 300 select an offer.

Perhaps 200 proceed to passenger information.

Perhaps 150 initiate payment.

The workload isn't uniform across the system.

This is an example of a **workload distribution** or **request mix**.

It becomes useful when deciding which components need additional capacity and where optimization will have the greatest impact.

### Conclusion

Here's an additional idea to add to my notes. When estimating workload, I should examine:

- Request rate.
- Concurrency.
- Request distribution and bursts.
- Operation mix.
- Data volume.
- Resource consumption.

I don't necessarily need all of these metrics for every system. The relevant ones depend on the workload I'm investigating.

## Part C — Investigate a performance problem

I correctly identified the bottleneck, and my instinct not to scale Appwrite Functions immediately was right.

However, my explanation of the relationship between Appwrite and Duffel contains a conceptual mistake.

Let's unpack it.

### C1. Where is the bottleneck?

My response:

> I suggest that the most obvious bottleneck is the Duffel API because it causes the most delay.

This is correct.

Let's revisit the figures:

| Component          | Observed behaviour                         |
| ------------------ | ------------------------------------------ |
| Mobile application | Requests reach backend successfully        |
| Appwrite Functions | Executing normally                         |
| Duffel API         | Average response time of 8 seconds         |
| Database           | Operations take less than 200 milliseconds |
| Overall search     | Approximately 10 seconds                   |

Duffel really accounts for most of the observed latency, and that makes Duffel interaction the most obvious bottleneck. But there's a subtle distinction here.

Duffel is the most obvious source of latency, but I should still investigate whether the eight-second response is normal for the provider, unusually high, or caused by something in our integration.

For example:

- Is the request taking eight seconds consistently?
- Does it happen only during peak periods?
- Are we making unnecessary sequential requests?
- Are network conditions between Appwrite and Duffel contributing?
- Is Duffel experiencing degraded performance?

Identifying the bottleneck is the beginning of an investigation, not necessarily the end.

### C2. Would increasing Appwrite Function instances solve the problem?

My response:

> Increasing the number of Appwrite Function instances won't necessarily solve the problem and will have little to no impact on the operation latency.

My conclusion is correct, but my explanation needs some refinement.

My addition:

> Besides, Duffel is the dependency to the backend and not the other way round (I don't know why I said this though).

This gives an opportunity to revisit something from the first concept.

#### What is a dependency?

A dependency is something a component relies on to perform its responsibilities.

In Book Flight:

- The flight-search logic depends on Duffel to retrieve flight inventory.
- Duffel does not depend on Book Flight to provide its flight inventory service.
  The relationship looks like this:

```txt
Book Flight
     |
     | Requests flight inventory
     v
Duffel API
     |
     | Returns flight offers
     v
Book Flight
```

The correct direction of dependency is my backend depends on Duffel.

I brought this up when explaining scaling, trying to reason that changing Appwrite Function capacity cannot directly change Duffel's processing capacity.

My instinct is correct, but the dependency relationship itself doesn't prove that adding function instances will have no effect.

Here's why.

#### The difference between application capacity and dependency latency

Imagine Book Flight receives 100 simultaneous search request, but my existing function capacity only allows 20 requests to execute at a time, while the remaining requests wait.

Suppose each request takes eight seconds because Duffel is slow, even if the function itself isn't doing much work, its limited execution capacity may create additional queues.

Now imagine increasing the number of function instances. I might be able to process more requests concurrently, and that could reduce waiting time caused by my own execution constraints.

However, this doesn't make Duffel respond faster. In fact, increasing the number of simultaneous requests could overwhelm Duffel or trigger rate limits.

So I need to distinguish:

| Problem                                                    | Would adding function instances help? |
| ---------------------------------------------------------- | ------------------------------------- |
| Function CPU or memory exhaustion                          | Potentially                           |
| Requests waiting because function concurrency is exhausted | Potentially                           |
| Duffel taking 8 seconds to process each request            | Not directly                          |
| Duffel rate limiting requests                              | Could make things worse               |
| Network latency between Appwrite and Duffel                | Not necessarily                       |

Here's the distinction I need to understand:

**Scaling your application can increase its ability to send and process requests. It doesn't necessarily increase the processing capacity of the external services those requests depend on.**

This makes my original conclusion right. But, my reasoning need to be more precise.

### C3. What would you investigate?

My response:

> Before I propose an architectural change, I should investigate the cause of the delay to identify the bottleneck.

My response here is correct, but too general. I need to develop the habit of translating these statements into specific investigations.

For this scenario, I'd investigate:

| Investigation                     | Why it matters                                                                                                 |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Duffel response-time distribution | Determine whether the eight seconds is consistent or concentrated in slower requests.                          |
| Request timing                    | Establish when the request leaves Appwrite and when Duffel responds.                                           |
| Network conditions                | Determine whether communication overhead contributes to the delay.                                             |
| Request structure                 | Check whether we're requesting unnecessary information.                                                        |
| Request sequence                  | Determine whether multiple provider requests are being executed sequentially when some could run concurrently. |
| Provider rate limits              | Determine whether we are approaching or exceeding the provider's limits.                                       |
| Function resource usage           | Confirm whether the function is actually under resource pressure.                                              |

These are not proposals for architectural changes. It's just gathering evidence to understand the problem. That's exactly the habit I should grow.

### C4. How can I improve the user experience?

My response:

> What I will suggest to improve the user experience without pretending that Duffel has become faster will be caching responses, especially the less sensitive fields.

I'm taking a good direction, but here are two corrections.

#### First: Caching isn't automatically the right solution

Suppose Duffle takes eight seconds to return flight offers, caching the entire response might make subsequent identical searches faster.

However, flight prices and availability can change, which now brings about a correctness problem.

I must distinguish between two categories of information.

| Information            | Caching consideration                                                     |
| ---------------------- | ------------------------------------------------------------------------- |
| Airport reference data | Generally suitable for caching.                                           |
| Airline information    | Generally suitable for caching.                                           |
| Flight search offers   | Potentially suitable for short-lived caching, with clear freshness rules. |
| Prices                 | Must be validated before booking.                                         |
| Seat availability      | Must be revalidated before relying on it.                                 |

Also, the reason to cache some fields isn't that they're less sensitive. It's that they're relatively stable and can be reused.

#### Second: There are other ways to improve the user experience without reducing Duffel's actual latency

For example:

1. Display a clear loading state so the user understands that the search is processing.
2. Avoid making duplicate requests when the user repeatedly taps the search button.
3. Preserve previous results, clearly identifying them as potentially outdated.
4. Provide a meaningful timeout and retry experience.
5. Use short-lived cached results where appropriate.
6. Avoid blocking unrelated application features while the search is processing.

I could also investigate whether some work can be performed concurrently, reducing the total time spent waiting on sequential operations.

However, I should be careful with retries. Automatically retrying every slow request can increase load on an already struggling provider.

Here's an important distinction: **Some improvements reduce actual operation latency. Others improve perceived performance or preserve useful functionality while the operation is still taking place.**

Both matter, but they're not the same thing.

## Part D — Scaling decisions

My reasoning in this section was good and I showed I understand that scaling a single component doesn't necessarily increase the capacity of the entire system.

### D1. Backend instance running out of CPU and memory

My response:

> To make a decision, any scaling can be worth investigating, but I'll start with vertical scaling, where I get to increase the capacity of my existing machines. I'll try this because it's quite straightforward and requires fewer changes.

I gave a reasonable start, and my response before discussing the scaling decision was excellent.

> I'll try to investigate which point or which action is causing the backend to run out of resources with the increasing demand.

Investigating should come before choosing vertical or horizontal scaling.

Image the backend running out of memory. Possible causes can include:

- Excessively large objects held in memory.
- Memory leaks.
- Large responses being processed.
- Too many simultaneous operations.
- Inefficient data processing.

Increasing memory might temporarily relieve the problem, but it won't necessarily address the underlying cause.

The same applies to CPU consumption.

If the application performs an unnecessarily expensive operation for every request, adding CPU resources might only postpone the next resource limitation.

#### Vertical scaling

My proposal to start with vertical scaling is reasonable when:

- The workload is concentrated on one instance.
- That instance is genuinely resource-constrained.
- A larger instance is available.
- The cost is acceptable.
- The application architecture doesn't need to change substantially.

The advantage is simplicity. I can increase available resources without immediately introducing the complexity of distributing work across multiple instances.

There are limits. I may eventually reach the maximum available resource for instance, or the cost of further upgrades may become unreasonable.

#### Horizontal scaling

My next step is also reasonable.

I could introduce additional instances to distribute incoming requests. But horizontal scaling works best when my application can handle multiple instances independently.

For example, imagine each instance stores important information only in its local memory.

Now, a user's first request reaches InstanceA, and their next request reaches InstanceB. Then, InstanceB doesn't know the state stored by InstanceA. Now there's a problem.

This is one reason stateless application instances are useful when designing horizontally scalable systems. More on this later.

For now, I should always remember:

**Vertical scaling increases the capacity of an existing instance. Horizontal scaling increases capacity by adding instances. Both require an understanding of the actual resource constraint.**

My answer is correct. However, I shouldn't turn "vertical first, horizontal later" into a universal rule. It's a possible strategy, not a law.

### D2. Four backend instances, one struggling database

My response:

> This tells me that my scaling strategy didn't impact the whole system, but only improved the backend.

I gave a strong answer, and showed that I've recognized that the database has become the limiting component despite the additional backend capacity.

Imagine this:

```txt
                 Incoming Requests
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          Backend A  Backend B   Backend C
             |           |           |
             +-----------+-----------+
                         |
                         v
                      Database
                    (Overloaded)
```

Here, I have improved the backend's capacity to receive and process requests. But all those instances still depend on the same database.

Consequently, the database may now receive more simultaneous requests than it can handle efficiently.

This affirms that adding backend instances doesn't automatically increase database capacity.

#### What should I investigate?

My suggestions were:

> It could be a connection limit, ineffective query, or whatever.

The first two are good possibilities. Let's make the investigation more systematic.

| Potential issue       | What to investigate                                             |
| --------------------- | --------------------------------------------------------------- |
| Connection exhaustion | Number of active database connections and connection limits.    |
| Inefficient queries   | Query execution plans, indexes, and unnecessary data retrieval. |
| Lock contention       | Whether concurrent transactions are blocking each other.        |
| Excessive reads       | Whether the same information is repeatedly retrieved.           |
| Excessive writes      | Whether unnecessary updates or writes are occurring.            |
| Resource saturation   | Database CPU, memory, storage, and I/O utilization.             |
| Large transactions    | Whether transactions are holding resources for too long.        |

## Consolidation

I shouldn't just have definitions that I can recite but struggle to apply. Here's something to help bring the concepts together.

| Concept     | What it helps us understand                                         | Book Flight example                                            |
| ----------- | ------------------------------------------------------------------- | -------------------------------------------------------------- |
| Latency     | How long an operation takes.                                        | Flight search takes 8 seconds.                                 |
| Throughput  | How much work is completed over time.                               | 100 searches completed per second.                             |
| Concurrency | How many operations overlap.                                        | 200 searches being processed simultaneously.                   |
| Workload    | The demand placed on a system.                                      | Search requests, booking requests, database operations.        |
| Capacity    | How much workload the system can handle under agreed conditions.    | 500 concurrent searches while meeting performance targets.     |
| Performance | How effectively the system performs its work.                       | P95 search latency remains below 5 seconds.                    |
| Scalability | How well the system accommodates increasing demand.                 | Search latency remains acceptable as request volume increases. |
| Bottleneck  | The component or resource limiting overall performance or capacity. | Duffel's response time dominates search latency.               |

There are three relationships I should understand.

### Relationship 1: Latency and throughput

Latency and throughput are different measurements, but they influence each other.

A system can have high throughput while individual requests take a relatively long time.

Likewise, a system can have low latency but process relatively few requests per second.

The workload and concurrency matter when interpreting these measurements.

### Relationship 2: Concurrency and capacity

As concurrent operations increase, the system consumes more resources.

Eventually, some resource becomes constrained.

This could be:

- CPU.
- Memory.
- Database connections.
- Network bandwidth.
- External provider capacity.

Understanding those constraints helps to estimate capacity.

### Relationship 3: Performance and scalability

Performance tells how well the system operates under a given workload.

Scalability tells how its behaviour changes as the workload increases.

This is why I need to measure performance at different workload levels to understand scalability.

## Refinements

Here are five refinements worth documenting.

### 1. Response-time targets need measurement boundaries

When I say a flight search should complete within two seconds, I should clarify whether I'm measuring backend response latency or the complete user-perceived operation.

### 2. Different operations have different performance characteristics

Similar request pathways don't guarantee similar latency. Request size, processing complexity, external dependencies, and data volume matter.

### 3. Dependency latency and application capacity are different problems

Adding Appwrite Function instances might increase my application's processing capacity without making Duffel respond faster.

### 4. Caching has a freshness trade-off

A faster response isn't necessarily a correct response. Cached flight offers, prices, and seat availability need appropriate validation.

### 5. Scaling is not an isolated component decision

A system is composed of interacting components. Increasing the capacity of one component can expose limitations in another.
