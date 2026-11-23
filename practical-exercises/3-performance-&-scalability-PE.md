# STAGE 1.03 — Performance and Scalability

**Today is November 23, 2026.**

## Practical exercise: Performance and scalability in Book Flight

I'm about to reason about Book Flight as though I'm investigating its ability to support growing demand. I don't need to know the exact limits of third-party services. Where I lack information, I should state my assumptions.

I shouldn't try to sound like an experienced distributed-systems engineer, though that's where I'm growing to get to. My objective right now isn't to produce the most sophisticated architecture. It's to develop my ability to understand a system's behaviour, identify its constraints, and reason about what should change.

## Part A — Identify performance characteristics

```txt
Consider the following operations:

1. Searching for flights.
2. Retrieving passenger information.
3. Retrieving a seat map.
4. Initiating a booking or payment.

For each operation:

- What would I consider a reasonable response time?
- What factors could influence its latency?
- Which components or dependencies could introduce delays?

I can propose hypothetical response-time targets. I should clearly identify them as assumptions rather than established product requirements.

I do recognize that different operations may have different performance expectations.
```

My responses here are assumptions and reasoning about the performance concept.

1. A reasonable response time for this operation will be 2 seconds. Factors that could influence the search latency are:
   - time taken for the app to prepare and send its request to the backend
   - time taken for the Appwrite function to execute its flight-search logic
   - time taken for Duffel API to respond with the flight offers
   - time taken to process and render the response
     The component and dependency to introduce delays will be the Duffel and the results display components.
2. For retrieving passenger information, a reasonable response time could be 500 milliseconds. This also depends on where the information is being retrieved. Assuming it's cached information, the response time will be probably faster than when the information is retrieved from a long-term storage unit. Factors that could influence the operation's latency are:
   - time taken for the app to prepare and send its request to the backend
   - time taken for the Appwrite function to execute its database query
     What could introduce delay would be the database query.
3. A reasonable response time for retrieving seat maps will be 1 second. Factors to influence its latency are very similar to the flight search, since its an operation that passes through a look-alike pathway. Delays are also most likely to come from Duffel and processing and displaying the seat map response.
4. When initiating a booking or payment, I'll assume a response time of about 5 seconds. This operation seem to execute a number of processing, validation, and external services. What could introduce delays here will be the execution by the payment provider.

In my responses, I assumed factors influencing latency to be the significant executions involved in the overall operation mentioned. I also considered components or dependencies introducing delays to be those with the highest latency.
I tried not to list out in full the factors influencing latency for the passenger information retrieval cos I consider them very similar with those for flight search.

## Part B — Understand workload and capacity

```txt
Assume Book Flight experiences the following growth:

| Metric                            | Current workload | Future workload |
| --------------------------------- | ---------------: | --------------: |
| Registered users                  |            1,000 |          50,000 |
| Daily active users                |              100 |          10,000 |
| Peak concurrent users             |               20 |           2,000 |
| Flight searches per minute (peak) |               10 |           1,000 |

Answer these questions:
1. Which metrics would I pay the most attention to when estimating system demand? Why?
2. Why wouldn't registered user count alone be sufficient?
3. Which parts of Book Flight do I expect to experience increased demand?
4. Would every component necessarily experience the same increase in workload?
```

1. When estimating system demand, I will pay attention to all four metrics. But the metrics to gain my most attention and influence my estimate the most are the peak concurrent users and the peak flight searches per minute. These metrics actually put the most workload on the system, and also infers the highest the system may experience.
2. Registered user count alone will be insufficient because the metric itself don't impose a strong workload on the system, and a system can maintain same number of registered users but have the other metrics change dramatically to make the system workload significantly different.
3. In this case where the workload for flight searches per minute gets an increase, the parts I expect to experience increased demand will be the flight-search function and Duffel API.
4. Every component won't necessarily experience the same increase in workload since it doesn't have to be that a user making a flight search requires the user to retrieve passenger information a initiate a booking.

## Part C — Investigate a performance problem

```txt
Imagine Book Flight is experiencing these conditions:

- Mobile application requests reach the backend successfully.
- Appwrite Functions are executing normally.
- Duffel API responses take an average of 8 seconds.
- The overall flight-search experience takes approximately 10 seconds.
- Database operations take less than 200 milliseconds.

Answer:

1. Where is the most obvious bottleneck?
2. Would increasing the number of Appwrite Function instances necessarily solve the problem?
3. What would you investigate before proposing an architectural change?
4. What options might improve the user experience without pretending that Duffel has become faster?
```

1. I suggest that the most obvious bottleneck is the Duffel API because it causes the most delay.
2. Increasing the number of Appwrite Function instances won't necessarily solve the problem and will have little to no impact on the operation latency. As an addition, it won't solve the problem because the Appwrite Function instances are outside the Duffel scope where changes from the Appwrite Function won't influence the Duffel contract. Besides, Duffel is the dependency to the backend and not the other way round (I don't know why I said this though).
3. Before I propose an architectural change, I should investigate the cause of the delay to identify the bottleneck.
4. What I will suggest to improve the user experience without pretending that Duffel has become faster will be caching responses, especially the less sensitive fields.

## Part D — Scaling decisions

```txt
Consider these two hypothetical situations.

SITUATION 1

A backend instance is running out of memory and CPU resources as demand increases.

Would vertical or horizontal scaling be worth investigating? Explain your reasoning and the trade-offs.

SITUATION 2

You have added four backend instances, but all of them depend on the same database. The database is now struggling to handle incoming requests.

What does this tell you about the scaling strategy? What would you investigate next?
```

For Situation 1, if at all, I'll try to investigate which point or which action is causing the backend to run out of resources with the increasing demand. To make a decision, any scaling can be worth investigating, but I'll start with vertical scaling, where I get to increase the capacity of my existing machines. I'll try this because it's quite straightforward and requires fewer changes. However, when I've reached a limit of improving my existing machines, I can then go ahead to add more machines to accommodate the growth.

In Situation 2, after scaling the backend horizontally and the database becomes the bottleneck. This tells me that my scaling strategy didn't impact the whole system, but only improved the backend. I'll now investigate the database next to discover what's causing the struggle in the component. It could be a connection limit, ineffective query, or whatever.
