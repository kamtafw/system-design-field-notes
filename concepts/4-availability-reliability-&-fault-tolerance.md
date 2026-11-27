# STAGE 1.04 — Availability, Reliability, and Fault Tolerance

**Today is November 27, 2026.**

Still using Book Flight as my working example to understand the problem, build the mental model, examine the trade-offs, and apply the concepts.

**What happens when a component my application depends on stops working?**

Say a user opens Book Flight and tries to search for flights, but Duffel's API is unavailable. Here are few possible outcomes:

- The entire app crashes.
- The search request fails, but the rest of the app remains usable.
- The app displays previously retrieved results, clearly indicating that they may be outdated.
- The app retries the request and eventually succeeds.
- The app informs the user that flight search is temporarily unavailable.

The outcomes represents different approaches to handling failure. They also introduce an important architectural consideration: **A system's quality isn't determined by only how well it works when everything is functioning. It's also determined by how it behaves when things go wrong.**

## 1. Three concepts to understand

### A. Availability

Availability describes whether a system or component is operational and accessible when it is needed.

For example:

- If Book Flight's flight-search functionality is accessible and can process requests, that capability is available.
- If Appwrite Functions is unavailable and Book Flight cannot execute its search function, flight search is unavailable.

Availability is often expressed as a percentage over a period.

Here's for illustration:

| Availability | Approximate downtime per year |
| ------------ | ----------------------------: |
| 99%          |              3 days, 15 hours |
| 99.9%        |           8 hours, 46 minutes |
| 99.99%       |                    52 minutes |

The important thing is that availability needs a defined scope. The entire application might be available while one particular feature is not.

### B. Reliability

Reliability describes the ability of a system to perform its intended function correctly and consistently over time.

Consider two scenarios:

1. Flight search responds successfully 99 times out of 100, returning valid results.
2. Flight search responds 99 times out of 100, but frequently returns incomplete or incorrect information.

Both systems might appear operational, but the second system has a reliability problem.

Availability asks whether the system is operational. Reliability asks whether it continues to perform its intended function correctly.

### C. Fault Tolerance

Fault tolerance describes a system's ability to continue providing some or all of its intended functionality despite failures in individual components.

Imagine Book Flight depends on:

- Appwrite Functions for backend execution
- Duffel for flight inventory
- A database for application data

If Duffel becomes unavailable, Book Flight might still allow users to:

- Access previously saved passenger information.
- View existing booking records.
- Navigate other parts of the application.

The app doesn't necessarily need to become entirely unusable because one dependency has failed.

Though, fault tolerance doesn't mean every feature must continue working regardless of what fails. Some failures will inevitably prevent certain capabilities from functioning.

## 2. The relationship between the concepts

To make the distinction clear.

- **Availability:** Is the system operational?
- **Reliability:** Does the system perform its intended function correctly and consistently?
- **Fault tolerance:** Can the system continue operating when some components fail?

These concepts are related, but they are not interchangeable.

_A system can be available but unreliable. It can also have fault-tolerant mechanisms that improve its availability._
