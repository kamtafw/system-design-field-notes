# STAGE 1.04 — Availability, Reliability, and Fault Tolerance

**Today is November 27, 2026.**

I had a good exercise. One thing to like is that I'm beginning to reason beyond the immediate failure. I'm asking which other capabilities depend on the affected component, whether the application can continue operating, and what compromises a proposed solution introduces.

That's what I need to develop.

There a few gaps I'll address. They are particularly around the distinction between a component failing and the entire operation failing.

## Scenario A: Duffel becomes unavailable

My central reasoning for this correct: if Duffel is unavailable, Book Flight cannot retrieve authoritative flight inventory through that dependency. Consequently, the flight-search operation cannot be completed normally.

Here's an important distinction in my interpretation of the scenario.

I wrote:

> The backend responds successfully to the mobile app, even though there was no response from Duffel.

But that's not necessarily the case. The backend function can execute successfully without the overall flight-search operation succeeding. The function gets executed, but the flight search did not succeed.

Note: _A successful execution doesn't necessarily mean I get the response that I want from the request. It could also mean the request didn't break down from it's end, but rather came to me with a response._

This distinction matters because **successful execution of a component does not automatically mean successful completion of the operation it participates in.**

For example, my backend could return:

- A successful search response containing flight offers.
- A controlled error indicating that the flight provider is temporarily unavailable.
- A timeout response because the provider did not respond within the permitted time.

All three are possible outcomes of an executed backend function.

### About my proposed solution

I suggested a friendly message and a retry button. These are good.

But let's examine the retry mechanism.

Suppose Duffel is experiencing an outage that lasts 10 minutes, and a user happens to click the retry every two seconds. What happens?

The user now keeps generating requests against a dependency that is already struggling. If I have many users doing that simultaneously, Book Flight gets to contribute to the pressure on Duffel and potentially makes the situation worse.

This should introduce and important design consideration: **Retrying a failed operation is not always the right response.**

I'll explore this more deeply when I get to retries, backoff, and circuit breakers.

Here are few principles I should stick with for now:

- Don't retry indefinitely.
- Give requests a reasonable time limit.
- Avoid making users repeatedly initiate requests without feedback.
- Don't present old flights offers as though they're current.
- Revalidate an offer before proceeding with a booking.

**Assessment:** My reasoning here was sound. The main gap was distinguishing between successful backend execution from successful completion of the business operation.

## Scenario B: Appwrite Functions becomes unavailable

I correctly identified that live flight search becomes unavailable because the mobile app cannot reach the backend function responsible for executing it.

I also made an important observation:

> However, the other capabilities with other dependencies that exclude Appwrite Functions or aren't tightly coupled with it will still be accessible.

That's so right.

This highlights an important architectural principle: **A failure should ideally affect only the capabilities that depend on the failed component.**

To make this concrete, imagine Book Flight has the following capabilities:

| Capability                              | Depends on Appwrite Functions?          | Expected behaviour during outage |
| --------------------------------------- | --------------------------------------- | -------------------------------- |
| Live flight search                      | Yes                                     | Unavailable                      |
| Retrieve live seat maps                 | Yes, assuming the same backend boundary | Unavailable                      |
| View locally stored search preferences  | No                                      | Available                        |
| View locally stored passenger form data | No                                      | Available                        |
| View cached reference information       | No                                      | Available, subject to freshness  |

### Graceful Degradation

_Graceful degradation means that when a component or capability fails, the system reduces functionality in a controlled manner rather than allowing the failure to unnecessarily disrupt everything else._

For example:

- Live flight search is unavailable.
- Previously saved passenger information remains accessible.
- The user can still navigate the application.
- Existing locally stored information remains visible.
- The application communicates the unavailable functionality clearly.

The system isn't fully operational, but it remains useful.

This is an important part of designing resilient applications.

**Assessment:** I showed strong reasoning. My next step is to become more precise about what information remains safe to display and what operations must be disabled when live dependencies are unavailable.

## Scenario C: The database becomes unavailable

What's peculiar about my response for this scenario is that I considered the actual importance of the affected functionality.

I wrote:

> I find this problem very lightweight. It don't seem to affect the primary objective of the Book Flight application itself.

It's a useful observation because I'm distinguishing between a core capability and a supporting capability.

However, I should challenge one assumption in my response.

> A user may still be able to search for flights, select a flight, enter their information, make a booking, and still get issued a ticket without any breakdown.

I can't conclude that yet. The scenario only tells me that the database is used to retrieve saved passenger information. It doesn't establish whether the database is involved in:

- Persisting booking information
- Tracking booking status
- Recording payment transactions
- Storing booking references
- Maintaining an audit trail

If the database is genuinely only responsible for retrieving saved passenger profiles, then that makes my conclusion reasonable.

But, if it's also responsible for recording booking transactions, its unavailability could affect the core booking process.

This illustrates something important: **The impact of a failure depends on the responsibilities of the affected component, not merely its name.**

A database isn't inherently a supporting component. Its importance depends on what the system uses it for.

### My proposed solution: Client-side storage and background persistence

I suggested allowing users to enter their information and storing it locally if persistence fails. This is temporary ownership of data.

Say a user enters passenger information while the database is unavailable and the app retains the information locally and attempts to persist it when the database becomes available again.

That is a reasonable approach for certain types of data, but there are important considerations.

#### 1. Data durability

What happens if the user closes the app before the information is successfully persisted?

Will the information survive?

#### 2. Data security

Passenger information contains personal data. Storing it locally introduces security and data-retention considerations.

#### 3. Data consistency

What happens if the user modifies the information locally while the database becomes available and another version already exists?

Which version should be considered authoritative?

#### 4. Business importance

Is this information merely a saved passenger profile, or is it part of an active booking transaction? The distinction matters enormously.

For example, retaining a passenger profile is different from assuming that a booking has been successfully created because its details were saved locally.

I don't need to solve all four concerns in this exercise. I just need to recognize that introducing local persistence solves one problem while potentially introducing others.

**Assessment:** I gave good identification of the limited impact. But, I need to avoid assuming that the database has no other responsibilities beyond the one specified in the scenario.

## Scenario D: Intermittent failures

This is where my reasoning needs the most development, although my initial diagnosis is correct.

I wrote:

> I'll first consider this an unreliable system. This is because some requests get successful responses, while others time out for a period. It isn't said that Duffel is unavailable.

That's correct. I distinguished between complete unavailability and intermittent failures.

Duffel is reachable, but it isn't consistently completing requests.

This is a reliability problem, and it may also reduce availability depending on how availability is measured.

Availability and reliability are related. A service that fails to respond successfully is less dependable for its users, even if it is technically reachable.

### My investigation approach

I mentioned several plausible causes, and I'm right not to jump straight into implementing a solution before understanding the failure.

Here's another consideration.

#### Where is the failure occurring?

The problem could be:

1. Between Book Flight and Duffel.
2. Within Duffel itself.
3. Because Book Flight is sending too many requests.
4. Because requests are taking longer than the timeout configured by Book Flight.
5. Because some particular request types are more expensive to process than others.

All these possibilities have different solutions.

For example, if the problem is a temporary network interruption, the appropriate response might be different from what I'd do if Duffel were rejecting requests because of rate limits.

### How should Book Flight handle it?

I said I don't know how to handle the situation, and that's fine.

Introducing a few mechanisms at a conceptual level.

#### A. Timeouts

_A timeout establishes how long a system is willing to wait for a dependency to respond._

Without a timeout, a request could remain pending for an unreasonable period.

In this case, setting a timeout doesn't fix Duffel. It prevents it as a dependency from holding up my application indefinitely.

#### B. Retries

For certain temporary failures, the system can attempt the request again.

However, retries need to be controlled. Retrying every failed request immediately and indefinitely can make an already struggling dependency worse.

#### C. Circuit Breakers

Imagine Book Flight repeatedly sending requests to Duffel and most of them timing out.

Instead of continuing to send requests that are likely to fail, Book Flight can temporarily stop making calls to Duffel. That is the basic idea behind a circuit breaker.

_The circuit breaker prevents my application from continuously depending on a component that is currently unhealthy._

After a period, Book Flight can then allow a limited number of requests through to determine whether Duffel has recovered.

#### D. Graceful Degradation

While Duffel is experiencing intermittent failures, Book Flight can communicate the problem and preserve functionality that doesn't depend on successful Duffel responses.

For example, saved passenger information and locally stored preferences can remain accessible.

**Assessment:** I showed good diagnosis and investigation instincts. The gap I had was me moving from identifying possible causes to considering how the app should behave while the problem remains unresolved.

## Takeaway

Here's how I want to consolidate the most important lessons.

| Concept              | What it helps us reason about                                                   | Book Flight example                                                         |
| -------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Availability         | Whether a capability is operational when needed                                 | Can users perform live flight searches?                                     |
| Reliability          | Whether a capability performs correctly and consistently                        | Does flight search consistently return valid results?                       |
| Fault tolerance      | Whether the system can continue functioning despite component failures          | Can users access other features when Duffel is unavailable?                 |
| Graceful degradation | How functionality is reduced without unnecessarily disrupting the entire system | Can users still access saved information when live search fails?            |
| Timeout              | How long the system waits for an operation                                      | How long should Book Flight wait for Duffel?                                |
| Retry                | Whether a failed operation should be attempted again                            | Should Book Flight retry a timed-out search?                                |
| Circuit breaker      | Whether calls to an unhealthy dependency should temporarily stop                | Should Book Flight temporarily stop calling Duffel after repeated failures? |

A final distinction:

**Fault tolerance is a system-level capability. Timeouts, retries, circuit breakers, redundancy, and graceful degradation are among the mechanisms and strategies that can help achieve it.**

I shouldn't confuse the desired architectural property with the individual mechanisms used to support it.
