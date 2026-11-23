# STAGE 1.03 — Performance and Scalability

**Today is November 22, 2026.**

This is where system design gets more interesting as I begin examining what happens when a system has to do more work, serve more users, process more information, and operate under increasing demand.

I should understand these concepts before I go into studying caching, load balancing, database optimization, or distributed systems. Otherwise, it's easy for me to learn techniques without understanding the problems they're supposed to solve.

## 1. Start with a problem

Say I lunch Book Flight.

At start, I have 20 users. They can search for flights, view offers, select flights, provide passenger information, and proceed through the booking process.

Everything works reasonably well, and my app starts gaining traction.

I now have:

- 500 registered users.
- 100 users searching for flights simultaneously.
- 1,000 flight searches per hour.
- Increasing number of bookings.
- More records being stored in the database.

Suddenly, users begin reporting problems.

Some searches take 15 seconds. Others take 30 seconds. In some cases, the app returns an error. Other requests even appear to hang indefinitely.

I investigate and discover that my flight-search function is taking longer to complete because the number of incoming requests has increased.

This introduces two questions.

- How do I make the system respond faster?
- How do I ensure the system continues performing acceptably as demand increases?

Both questions introduce two related but distinct concepts: **performance and scalability.**

## 2. Understanding performance

_Performance describes how effectively a system executes its operations under a given workload and set of conditions._

In simpler terms:

**How well does the system perform the work it's expected to do?**

Take a user searching for flights. The request travels through several components:

```txt
User
  |
  v
Mobile Application
  |
  v
Appwrite Function
  |
  v
Flight Search Logic
  |
  v
Duffel API
  |
  v
Flight Search Results
  |
  v
Mobile Application
  |
  v
User sees results
```

The user doesn't really care about how many internal operations happen. They rather care about how long they wait before seeing the results.

That waiting time is one aspect of performance.

_However, performance, isn't limited to how quickly a single request complete._

With that, I need to understand a few related measurements.

### 2.1 Latency

_Latency measures the time taken for an operation or request to complete._

For example:

A user initiates a flight search at 10:00:00.

The app displays the results at 10:00:04.

The observed _end-to-end latency_ is approximately 4 seconds.

_Latency is usually expressed in milliseconds or seconds._

For Book Flight, I might measure:

- Time taken for the mobile app to receive a response from the backend.
- Time taken by the Appwrite Function to execute its flight-search logic.
- Time taken for Duffel to respond.
- Total time between the user's search action and the display of results.

All these measurements aren't interchangeable.

If say the total search takes 8 seconds, I will need to understand where those 8 seconds are being spent.

For example:

| Operation                         |     Duration |
| --------------------------------- | -----------: |
| Mobile request preparation        |       100 ms |
| Network request to Appwrite       |       200 ms |
| Function execution and processing |       300 ms |
| Duffel API response               |     6,500 ms |
| Response processing and rendering |       900 ms |
| **Total**                         | **8,000 ms** |

These figures illustrate that the entire system can feel slow even when some of its individual components are performing reasonably fast.

_This is why end-to-end latency isn't always sufficient. I also need to understand the contribution of individual components._

#### Latency percentiles

Suppose Book Flight processes 1,000 flight searches, and I measure their response times and discover:

| Percentile | Response time |
| ---------- | ------------: |
| P50        |   1.5 seconds |
| P90        |     3 seconds |
| P95        |     5 seconds |
| P99        |    12 seconds |

What do these numbers mean?

- **P50:** 50% of requests completed within 1.5 seconds.
- **P90:** 90% of requests completed within 3 seconds.
- **P95:** 95% of requests completed within 5 seconds.
- **P99:** 99% of requests completed within 12 seconds.

Why does this matter?

Imagine my report says that Book Flight's average search time is 2 seconds. It sounds great.

But perhaps some users regularly experience 12-second delays.

An average can hide these experiences.

_Percentiles helps understand the distribution of response times, particularly the slower requests._

For systems where user experience matters, P95 and P99 can reveal problems that averages conceal.

The mental note here is: **Don't rely exclusively on average response time when evaluating system performance.**

### 2.2 Throughput

_Throughput measures how much work a system successfully processes within a given period._

Suppose Book Flight processes 300 flight searches in one minute.

Its average throughput during that period is:

300 ÷ 60 = 5 searches per second.

_Throughput helps understand the volume of work a system handles._

Here's an important distinction:

**Latency measures how long an individual operation takes. Throughput measures how much work the system completes over time.**

However, this doesn't nullify that a system can have low latency but low throughput. It can also have high throughput but poor latency.

Consider two hypothetical systems:

| Measurement            |        System A |         System B |
| ---------------------- | --------------: | ---------------: |
| Average search latency |          500 ms |        5 seconds |
| Throughput             | 10 searches/sec | 100 searches/sec |

System A responds faster to individual requests. System B processes more searches per second.

Which is better?

It's not alright to just determine that from these two measurements (latency and throughput) alone. I will also need to understand the workload, user expectations, available resources, and business requirements.

### 2.3 Concurrency

_Concurrency describes how many operations are in progress during an overlapping period._

Say 100 users initiate flight searches around same time, their requests may overlap, meaning multiple searches are being processed concurrently.

Concurrency is important because systems don't necessarily handle simultaneous requests in the same way they handle sequential requests.

But as concurrency increases, the system may encounter:

- Resource contention
- Increased memory consumption
- Database connection limits
- Network congestion
- Request queues
- Provider rate limits

Concurrency is particularly relevant when discussing backend execution environments and external APIs.

Here's a distinction to remember: **100 concurrent users doesn't necessarily mean 100 concurrent requests.**

Some users may be idle. Others may make several requests. One user may initiate multiple operations.

This requires me to distinguish users, active sessions, requests, and concurrent operations when describing workloads.

## 3. Understanding scalability

Here's a different problem.

Book Flight is performing well with 100 concurrent users. The app grows, and I now have 10,000 concurrent users. My existing infrastructure can no longer maintain the performance targets.

What do I do?

I need to increase the system's ability to handle growing demand.

This is the problem scalability addresses.

**Scalability is a system's ability to accommodate increasing workloads by adapting its available resources or architecture while maintaining acceptable performance and other required qualities.**

_It isn't just about handling more users. It's handling more demand without unacceptable degradation._

There are two fundamental approaches worth understanding.

### 3.1 Vertical scaling

_Vertical scaling means increasing the resources available to an existing machine or instance._

For example:

Suppose my backend runs on a server with:

- 2 CPU cores
- 4 GB RAM

As demand increases, vertical scaling is increasing the capacity of that existing machine to say:

- 8 CPU cores
- 16 GB RAM

Think of vertical scaling as making an existing worker stronger.

**Advantages:**

- Relatively straightforward.
- Often requires fewer architectural changes.
- Can improve performance when resource limitations are the problem.
- May simplify operational management.

**Disadvantages:**

- Machines have practical resource limit.
- More powerful machines can become expensive.
- A single instance can remain a single point of failure.
- Increasing resources doesn't solve every bottleneck.

Vertical scaling is useful, but has it limits.

### 3.2 Horizontal scaling

_Horizontal scaling means increasing capacity by adding more machines or instances._

Instead of upgrading one backend instance, imagine running four instances.

```txt
                 Incoming Requests
                         |
                         v
                   Load Balancer
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          Server A    Server B    Server C
             |           |           |
             +-----------+-----------+
                         |
                         v
                      Database
```

Think of it as adding more workers rather than making one worker stronger. Now, the incoming load is distributed across multiple instances.

**Advantages:**

- Can accommodate substantial workload growth.
- Work can be distributed across instances.
- Can improve availability when properly designed.
- Individual instances can be replaced without necessarily taking down the entire service.

**Disadvantages:**

- Introduces coordination and distribution concerns.
- Shared resources, such as databases, can become bottlenecks.
- Requires careful handling of state and concurrent operations.
- Adds operational complexity.

Take note of this: **Adding more instances doesn't automatically make the system faster.**

If all four instances depend on one overloaded database, the databases continues to remain the bottleneck.

Likewise, if all four instances depend on a flight provider that only permits a certain number of requests per minute, adding more instances won't eliminate that restriction.

**Scaling one component doesn't necessarily scale the entire system.**

## 4. Performance and scalability are related, but not interchangeable

The distinction between the two NFRs is important enough to be examined.

Say Book Flight currently supports 100 concurrent users and its average search latency is 2 seconds. The number of user increases to 1,000, and the average search latency becomes 15 seconds.

It can be said that the system is experiencing performance degradation as demand increases.

_A scalable system should be able to accommodate increasing workloads without unacceptable degradation, subject to its capacity and design constraints._

But also consider another situation where Book Flight has only 10 users, and every search takes 20 seconds.

It won't necessarily be said that this app has a scalability problem. It might however have a performance problem caused by inefficient processing, a slow external API, or poor network conditions.

To distinguish the problems:

| Performance                                            | Scalability                                                                                |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| Concerns how effectively the system performs its work. | Concerns how well the system accommodates growing demand.                                  |
| Can be evaluated at a particular workload.             | Requires examining behaviour as workload changes.                                          |
| Focuses on latency, throughput, resource usage, etc.   | Focuses on capacity, growth, and maintaining acceptable quality under increasing load.     |
| Example: A flight search takes 10 seconds.             | Example: Search latency increases dramatically as concurrent users grow from 100 to 1,000. |

**A system can have good performance at a small workload but poor scalability. It can also be scalable but have poor performance at every workload.**

The goal is to understand both.

## 5. Workload and capacity: What exactly am I scaling?

Before I make decisions on scaling, I need to understand what work the system is expected to perform.

Consider the two applications:

**Application A:**

- 5,000 registered users
- 100 daily active users
- 20 searches per minute during peak periods

**Application B:**

- 5,000 registered users
- 4,000 daily active users
- 2,000 searches per minute during peak periods

The thing is both apps have same number of registered users, but their workloads are significantly different. And this is why registered user count alone is a poor way to estimate system demand.

When evaluating workload, I should consider:

| Dimension             | What it tells us                                      |
| --------------------- | ----------------------------------------------------- |
| Request rate          | How many requests arrive within a period.             |
| Concurrency           | How many operations overlap.                          |
| Data volume           | How much information the system stores and processes. |
| Request complexity    | How much work individual operations require.          |
| Workload distribution | When demand occurs and how it varies.                 |
| Resource consumption  | CPU, memory, network, database connections, etc.      |

For Book Flight, I should take it that flight search, passenger information retrieval, and booking creation may have very different resource requirements.

The search may involve an external API call.

The booking operation may involve multiple validations, payment processing, database writes, and external communication.

Trying to treat all requests as equivalent would give me an incomplete understanding of system capacity.

### Capacity

_Capacity refers to the amount of work a system can handle while satisfying its required operating conditions._

For example:

Suppose Book Flight has the following performance target:

- P95 flight-search latency must remain below 5 seconds.
- Search error rate must remain below an agreed threshold.
- The system must support 200 concurrent searches.

If the system can meet these targets at 200 concurrent searches, but fails at say 500, then 200 is within its demonstrated capacity under the tested conditions.

Capacity is not necessarily a fixed number. It depends on:

- The workload
- Available resources
- Dependencies
- Performance requirements
- System configuration

This is why capacity testing matters.

## 6. Bottlenecks: Why does a system become slow?

Suppose search latency in Book Flight increases from 2 seconds to 12 seconds.

I shouldn't immediately conclude that I need more servers.

I should investigate where the delay originates. Potential bottlenecks could be:

| Component           | Possible problem                                          |
| ------------------- | --------------------------------------------------------- |
| Mobile application  | Excessive rendering or unnecessary processing.            |
| Network             | High latency or poor connectivity.                        |
| Appwrite Functions  | Resource limitations or slow execution.                   |
| Database            | Inefficient queries or connection exhaustion.             |
| Duffel API          | Slow responses or rate limiting.                          |
| Response processing | Expensive transformations or unnecessary data processing. |

_A bottleneck is a component or resource that limits the overall performance or capacity of the system._

Here's an important architectural habit: **Identify the bottleneck before deciding how to optimize or scale.**

Suppose Duffel takes 9 seconds to return flight offers, while my function spends only 300 milliseconds processing the request, increasing my function's CPU resources might have little impact on the overall search latency, cos the external provider is responsible for most of the delay.

Again, this is why understanding system boundaries and dependencies is very important.

## 7. A few principles to stick with

Here are five observations.

### a. Faster doesn't always mean more scalable

Optimizing a system's operation might reduce its latency without increasing the system's ability to handle increasing demand or more concurrent work.

### b. More resources don't always solve performance problems

I can add CPU, memory, and other resources, while leaving the system's actual bottleneck untouched.

### c. Scaling introduces trade-offs

Horizontal scaling can increase capacity but introduces coordination, state management, and operational complexity.

### d. External dependencies influence my system's performance and capacity

Book Flight doesn't control Duffel's internal infrastructure. My system must account for the behaviour and limitations of its external dependencies.

### e. Performance requirements need workload context

Stating "flight searches should complete within 5 seconds" is an incomplete requirement if the conditions under which that target should be met isn't specified.
