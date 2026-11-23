# STAGE 1.02 — Functional and Non-Functional Requirements

Imagine building Book Flight.

What should the application do? I might say:

1. Allow the user to search for flights.
2. Display available flights.
3. Allow the user to select a flight.
4. Collect passenger information.
5. Allow the user to pay.
6. Confirm the booking.

All these are reasonable expectations that describe things the system must be capable of doing.

Now, here are another set of questions.

- How quickly should flight search results appear?
- What happens if the flight provider takes 20 seconds to respond?
- Can two users book the same available offer simultaneously?
- How should the application behave when the internet connection drops?
- How secure should passenger information be?
- How many users should the application support simultaneously?

The first question describes what the system does.

The second set describes the conditions and qualities under which the system is expected to operate.

This is the foundation of functional and non-functional requirements.

## Functional requirements (FR)

A functional requirement describes a behaviour, capability, or operation that a system must provide.

In simpler terms:

### What must the system do?

For Book Flight, examples include:

| Requirement           | Expected system behaviour                                                            |
| --------------------- | ------------------------------------------------------------------------------------ |
| Flight search         | Users can search for flights using origin, destination, date, and passenger details. |
| Flight results        | The system displays flight offers returned by the inventory provider.                |
| Offer selection       | Users can select an available flight offer.                                          |
| Passenger information | Users can provide the required information for each traveler.                        |
| Seat selection        | Users can retrieve and select available seats where supported.                       |
| Payment               | Users can initiate payment for a booking.                                            |
| Booking confirmation  | Users receive confirmation after a successful booking and ticket issuance.           |

In each case, the requirement is describing something observable that the system should accomplish.

**A useful way to identify functional requirements is to ask:**

- Who interacts with the system?
- What action do they need to perform?
- What should the system do in response?
- What outcome should the interaction produce?

### An important distinction: Requirements vs Implementation

Suppose I write my functional requirement as: "The system must use Appwrite Functions to search for flight."

That's not really a functional requirement. It's an implementation decision.

The actual functional requirement is expressed as: "The system must allow users to search for available flights."

This is because I could implement this same capability using Appwrite Functions, AWS Lambda, a traditional backend server, or another approach.

This distinction between a functional requirement and an implementation decision is important because requirements describe the problem and expected outcome. Architecture and implementation decisions describe how I intend to solve it.

I should avoid making my initial implementation the definition of what the product must be.

## Non-functional requirements (NFR)

A non-functional requirement describes a quality, constraint, or operation expectation of a system.

In simpler terms:

**How well must the system perform its responsibilities, and under what conditions?**

Take the FR: "The system must allow users to search for flights."

But that leaves many unanswered questions.

- Should the search take 2 seconds or 30 seconds?
- Should it support 10 users or 10,000 simultaneous users?
- Should it continue functioning when Duffel is unavailable?
- Should flight searches be encrypted in transit?
- Should the system retain search history?

Here are examples of how non-functional requirements help answer these questions:

| Quality attribute | Example requirement                                                                                           |
| ----------------- | ------------------------------------------------------------------------------------------------------------- |
| Performance       | 95% of flight searches should return results within 5 seconds under an agreed test load.                      |
| Availability      | Core booking services should maintain 99.9% monthly availability.                                             |
| Security          | Sensitive passenger information must be protected in transit and at rest.                                     |
| Scalability       | The system should support increasing concurrent users without unacceptable degradation.                       |
| Reliability       | A failed payment must not result in an incorrectly confirmed booking.                                         |
| Usability         | Users should be able to identify the current state of a booking without interpreting technical errors.        |
| Maintainability   | Flight inventory providers should be replaceable without requiring changes throughout the mobile application. |

These examples are hypothetical targets, not requirements established for Book Flight.

Also, notice something important: many of these requirements can be measured or tested.

"Make the application fast" is too vague.

"95% of searches should return results withing 5 seconds under 500 concurrent users" is much more useful.

This is where non-functional requirements become especially interesting for system design.

## Distinguishing between FR and NFR

Consider the requirement:

"The system must allow users to search for flights even when they temporarily lose internet connectivity."

Is that functional or non-functional?

It sounds like a quality expectation because it concerns availability and resilience.

However, it also describes an explicit capability the system must provide.

Similarly, the requirement:

"The system must encrypt passenger information."

It is generally classified as a security NFR, even though implementing it involves specific functional behaviour.

The lesson is that the distinction isn't always perfectly clean.

Here's a mental model to help:

- Functional requirements describe capabilities and behaviours.
- Non-functional requirements describe quality attributes and constraints on those capabilities.

I shouldn't spend too much time arguing over the classification of an ambiguous requirement. I should rather focus on understanding what it expects from the system.

## Why this matters for architecture

This is where the concept becomes directly relevant to system design.

Taking a functional requirement:

**FR-01: Users must be able to search for available flights.**

On it's own, this doesn't tell much about the architecture.

Now introducing some non-functional requirements.

**NFR-01: Performance**

95% of searches should return results within 5 seconds.

**NFR-02: Reliability**

Temporary failures from the flight inventory provider should not crash the entire application.

**NFR-03: Security**

The mobile application must not contain the flight provider's secret credentials.

**NFR-04: Maintainability**

The application should be able to integrate another flight inventory provider without rewriting its entire flight-search experience.

Suddenly, one functional requirement, has led to multiple architectural implications.

The FR: **Search for available flights**

**Performance:** may influence request handling, caching, database queries, timeouts, and concurrency.

**Reliability:** may influence retry strategies, timeouts, error handling, circuit breakers, and graceful degradation.

**Security:** may influence trust boundaries, authentication, authorization, and secret management.

**Maintainability:** may influence provider adapters, modularity, API contracts, and separation of concerns.

This establishes the important relationship: **FRs tells me what capabilities to provide. NFRs influence the architectural decisions I make when providing them.**

Two systems can satisfy exactly the same FRs but have dramatically different architectures because their NFRs differ.

For example, a flight-search application supporting 100 users per day might not need the same infrastructure as one supporting 500,000 users per day.

The functionality could be identical. The quality expectations and operating constraints are not.

## A few concepts worth distinguishing

### A. NFRs are not optional enhancements

I already had this mindset rooted inside of me, that FRs are essential and NFRs are improvements to consider later.

That's a dangerous way to approach architecture.

Imagine Book Flight can successfully process payments but occasionally confirms bookings without verifying whether payment succeeded.

This behaviour satisfies the basic payment functionality in some situations, but its reliability and correctness are unacceptable.

NFRs can determine whether a system is fit for its intended purpose.

### B. Requirements can conflict

Consider the two requirements:

- The application should return search results as quickly as possible.
- The application should always display the latest available flight prices.

Caching search results might improve performance which will satisfy the first requirement, but cached results can become stale and fail the second requirement.

Now, there's a tradeoff.

I might decide to cache certain information while requiring a fresh provider validation before booking.

This introduces an important system design habit: requirements aren't independent. Satisfying one can make another harder to satisfy.

### C. Requirements need context and measurable expectations

Compare:

- The system must be highly available.
- The system should maintain 99.9% monthly availability for flight search, excluding agreed maintenance windows.

The second is much more useful.

Likewise:

- The system should support many users.
- The system should support 5,000 concurrent authenticated users while maintaining the agreed response-time targets.

A requirement becomes useful for architectural reasoning when I can evaluate whether the system satisfies it.

**I won't always have exact numbers at the beginning of a project. That's fine. I should record assumptions and identify what needs to be clarified.**
