# STAGE 1.04 — Availability, Reliability, and Fault Tolerance

**Today is November 27, 2026.**

## Practical exercise: Availability, reliability, and fault tolerance in Book Flight

Here are four hypothetical scenarios involving Book Flight.

**Scenario A: Duffel becomes unavailable.**

A user attempts to search for flights. My backend function executes successfully, but Duffel doesn't respond.

**Scenario B: Appwrite Functions becomes unavailable.**

The mobile application can connect to the internet, but requests to my flight-search function fail.

**Scenario C: The database becomes unavailable.**

Assume Book Flight has a database dependency for retrieving saved passenger information. The database becomes inaccessible.

**Scenario D: Intermittent failures.**

Duffel responds successfully to some requests but times out on others. The problem lasts for approximately five minutes before resolving itself.

---

For each scenario, I'll answer these questions:

1. **Impact:** Which Book Flight capabilities become unavailable? Which can continue functioning?
2. **User experience:** What should the user experience when attempting to use affected capabilities?
3. **Failure handling:** What could Book Flight do to handle the situation?
4. **Trade-offs:** What compromises might my proposed solution introduce?

The objective for the exercise isn't to design a perfect fault-tolerant flight-booking system. It's to develop the habit of asking:

**When something fails, how far should that failure propagate through the system, and what should remain functional?**

---

## Scenario A: Duffel becomes unavailable

> A user attempts to search for flights. My backend function executes successfully, but Duffel doesn't respond.

Here's how I understand your description of the scenario, the backend responds successfully to the mobile app, even though there was no response from Duffel. If that is the case, the backend response as well will be unreliable because Duffel's response it the authoritative source of truth.

For this scenario, other actions attached to the result from this get affected. Also, other capabilities that depend on Duffel will be affected.

For a good experience, the app can provide the user with a friendly message about the situation. It can also provide a retry button to allow the user to keep retrying for a determined number of times.


## Scenario B: Appwrite Functions becomes unavailable

> The mobile application can connect to the internet, but requests to my flight-search function fail.

For this, the capability for live flight search and discovery becomes unavailable. The mobile app may display a failure message or still display some results when users attempt a search, but the results here will be outdated. All Book Flight capabilities that depend on Appwrite Functions also becomes unavailable. However, the other capabilities with other dependencies that exclude Appwrite Functions or aren't tightly coupled with it will still be accessible.

For a good app like Book Flight, the user shouldn't have to see the app behaving strangely or crashing when attempting to use the affected capabilities. The system should be fault tolerant and also display reliable information even when the results aren't updated. The application can achieve this either by displaying a user-friendly failure message, adding retry buttons, or displaying some stale results while keeping the user aware of that.

## Scenario C: The database becomes unavailable

> Assume Book Flight has a database dependency for retrieving saved passenger information. The database becomes inaccessible.

In this scenario, the capabilities that get affected are retrieving already saved passenger information when needed. I find this problem very lightweight. It don't seem to affect the primary objective of the Book Flight application itself.

A user may still be able to search for flights, select a flight, enter their information, make a booking, and still get issued a ticket without any breakdown. Retrieving passenger information sounds like a functionality to improve user experience when trying to book a flight. So, when the database becomes unavailable, the user may have to enter the passenger information over again.

Another area I that gets affected may be booking history. If the history also needs to display passenger information, then this capability gets disrupted as well.

Of course a solution for this situation will be to let Book Flight allow a user enter the passenger information if at any point the database becomes unavailable. This information entered can then be stored on the client side if it fails to get persisted because of the unavailable database. The system then keeps trying to persist it in the background.

## Scenario D: Intermittent failures

> Duffel responds successfully to some requests but times out on others. The problem lasts for approximately five minutes before resolving itself.

I'll first consider this an unreliable system. This is because some requests get successful responses, while others time out for a period. It isn't said that Duffel is unavailable.

My next action will be to investigate this behaviour. There could be various causes to it. It could be network failure, resource congestion, rate limits, and others. This will determine my search for a solution.

For the user experience using the affected capabilities, such delay is a very significant. I don't have any idea on how to handle the unsolved situation.