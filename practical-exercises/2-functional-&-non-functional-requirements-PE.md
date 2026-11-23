# STAGE 1.02 — Functional and Non-Functional Requirements

## Practical exercise: Define Book Flight's requirements

I will approach this exercise as someone participating in the early design of Book Flight, rather than someone documenting features that have already been implemented.

## Part A — Identify functional requirements

```text
Identify at least 6 functional requirements for Book Flight.

For each one, explain:
1. What capability should the system provide?
2. Who needs that capability?
3. What should happen when the capability is successfully executed?

Don't describe implementation details yet
```

Here are six (6) FRs for Book Flight:

1. Allow user to search for flights
2. Display available flights
3. Allow user select flight
4. Allow passenger select flight seat when seat selection available
5. Collect passenger information
6. Allow the user to pay

These requirements can be explained as:

1. The user should be able to search for flights using flight information like origin, destination, and travel date. A successful execution of this capability returns available flights offers for the user that meets their search criteria.
2. The system should display available flights. An acceptance criteria for this capability means the user can identify the available flights on the screen.
3. The user should be able to select a flight they want to book or view its details. The system having this capability means a user is able to select from the available flights displayed by the system for further action.
4. The system allowing the passenger to select a flight seat when seat selection is available means the system maintains a track of the flight the user selected and has provided the user seats for selection. The system also let the user know when the flight has no such service.
5. The system should always be able to collect a passenger information. The system also needs to confirm if this requirement is satisfied.
6. A user should be able to pay for their flight right on the application. The system also indicates to the user the status of their payment without them having to interpret any technical message.

## Part B — Identify non-functional requirements

```text
Identify at least 5 non-functional requirements.

Try to cover different quality attributes rather than writing five variations of performance requirements.

For each one, explain:
1. What quality or constraint does the requirement address?
2. Why does it matter to Book Flight?
3. How could you determine whether the system satisfies it?

You don't need perfect numerical targets. If you're unsure, propose a target and clearly label it as an assumption.
```

Here are five (5) NFRs for Book Flight:

1. Reliability
2. Usability
3. Privacy
4. Extensibility
5. Backward compatibility

These requirements can be explained as:

1. **Reliability:** 99.9% of search results should always match user search queries.
2. **Usability:** Users should be able to book a flight without creating an account.
3. **Privacy:** Passenger information and preferences from a booking session must completely cleaned out after the booking session has been completed or cancelled out.
4. **Extensibility:** The application should be able to accept new features without requiring changes throughout the entire mobile application or disfiguring the existing experience.
5. **Backward compatibility:** The application with older versions of providers keep working without breaking the user experience.

## Part C — Explore architectural trade-offs

Consider these three scenarios.

```text
SCENARIO 1: Performance vs freshness

Book Flight wants flight search results to load quickly. However, flight prices and availability can change frequently.

Would you cache flight search results? Why or why not? What information might be safe to cache, and what should be validated again before booking?
```

For this case, I won't cache flight search results. And even if I do, I will set a very short TTL with a maximum of probably about 2 minutes. Search results here are very volatile and can change within the moment. I can't mention any information which I'm confident that might be safe to cache, but what I'm certain needs validation before booking are flight availability and prices.

```text
SCENARIO 2: Availability vs dependency reliability

Duffel occasionally becomes unavailable.

Should Book Flight stop working entirely when Duffel is unavailable? Which capabilities could continue functioning, and which should become unavailable?
```

Book Flight shouldn't stop working entirely when Duffel is unavailable. Only capabilities directly tied to the Duffel service gets affected and may stop working when Duffel becomes unavailable. Other capabilities with no direct communication with Duffel continues functioning even when Duffel is unavailable.

```text
SCENARIO 3: Security vs convenience

Book Flight wants to make repeat bookings easier by remembering passenger information.

What information would you allow the application to retain? What security and privacy considerations would influence your decision?
```

I'm finding it a bit difficult to understand and answer this question. It's a quite interesting question and I'll love to here the reasoning on it.
