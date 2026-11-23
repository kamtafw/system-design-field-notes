# STAGE 1.02 — Functional and Non-Functional Requirements

The overall assessment is **I understand the basic distinction between functional and non-functional requirements, but I haven't yet developed the precision needed to turn them into useful engineering requirements.**

## Part A: Functional requirements

My six requirements are reasonable, and they describe genuine capabilities that Book Flight should provide.

However, there are two issues.

First, some requirements are written as feature names rather than sufficiently defined requirements.

Second, I haven't consistently addressed all three questions: what the system does, who needs it, and what constitutes successful execution, in the case of each requirement.

Examining them individually,

### FR-01: Allow users to search for flights

My explanation:

> The user should be able to search for flights using flight information like origin, destination, and travel date. A successful execution of this capability returns available flights offers for the user that meets their search criteria.

My explanation is good. It identifies:

- The actor: the user.
- The capability: searching for flights.
- The input: origin, destination, and travel date.
- The expected outcome: matching flight offers.

However, there's an assumption to challenge.

Does a successful flight search necessarily return available flight offers?

Consider these situations:

- No flights match the search criteria.
- The provider returns an empty result.
- The provider is temporarily unavailable.
- The search is valid, but no offers are available for the requested date.

A successful execution of a search operation doesn't necessarily mean that matching flights exist.

This will refine my requirement to:

**FR-01: The system shall allow users to search for flights using specified travel criteria and return matching flight offers or an appropriate result indicating that no offers were found.**

This distinguishes between successfully executing an operation and successfully obtaining a particular business outcome. That's an important distinction in requirements engineering.

### FR-02: Display available flights

My explanation:

> The system should display available flights. An acceptance criteria for this capability means the user can identify the available flights on the screen.

My explanation is correct, but underdeveloped.

I've indicated the expected outcome, but what does it mean for a user to identify available flights? What information must the system display?

I don't need to specify every field to be displayed at this stage, but I should recognize that displaying a flight isn't simply rendering a list. The information displayed must enable the user to make an informed selection.

This will refine my requirement to:

**FR-02: The system shall present flight offers in a format that allows users to compare relevant flight itinerary, airline, and pricing information.**

The exact information requirements can be defined separately.

### FR-03: Allow users to select a flight

My explanation:

> The user should be able to select a flight they want to book or view its details.

My explanation is good, with a small concern. My use of language is vague and can be misinterpreted.

I've combined two related but different capabilities:

1. Selecting a flight.
2. Viewing flight details.

These capabilities may be part of the same user journey, but they don't necessarily have to be the same requirement.

A user might view the details of several flights without selecting any of them.

These can be separated as:

**FR-03: The system shall allow users to select a flight from the displayed search results for further booking action.**

**FR-04: The system shall allow users to view detailed itinerary and offer information before proceeding with a booking.**

Viewing flight details here is misinterpreted as just viewing the itinerary and other flight information on the displayed search results.

It's very important that requirements be sufficiently distinct that each capability working can be independently determined.

### FR-04: Seat selection

My explanation:

> The system maintains a track of the flight the user selected and has provided the user seats for selection. The system also let the user know when the flight has no such service.

My explanation carries good reasoning. I recognized the conditional nature of seat selection.

I'd polish the requirement as:

**FR-05: The system shall allow users to retrieve and select available seats for supported flight offers and communicate when seat selection is available.**

It's important that requirements describe what the system actually promises, not what I assume happens behind the scenes.

### FR-05: Collect passenger information

My explanation:

> The system should always be able to collect a passenger information. The system also needs to confirm if this requirement is satisfied.

This is my weakest FR and needs improvement.

"always be able to collect" is too absolute. The system may have conditional requirements based on passenger type, airline, or destination.

I can reformulate it as:

**FR-06: The system shall allow users to provide and validate the passenger information required for the selected itinerary and passenger types.**

That gives a much clearer expectation.

### FR-06: Allow users to pay

My explanation:

> A user should be able to pay for their flight right on the application. The system also indicates to the user the status of their payment without them having to interpret any technical message.

My explanation is good. I went beyond the payment action and considered the user's understanding of the outcome, which is valuable.

However, there are functional behaviours that needs to be defined.

This will formulate the initial requirement as:

**FR-07: The system shall allow users to initiate payment for a booking and communicate the resulting payment status in a way that supports appropriate subsequent booking actions.**

### Recommendation for Part A

My requirements are a good starting point.

A useful FRs list for Book Flight could include:

| ID    | Functional requirement                                                                   |
| ----- | ---------------------------------------------------------------------------------------- |
| FR-01 | Users can search for flights using specified travel criteria.                            |
| FR-02 | Users can view and compare returned flight offers.                                       |
| FR-03 | Users can view detailed itinerary and offer information.                                 |
| FR-04 | Users can select a flight offer for booking.                                             |
| FR-05 | Users can retrieve and select seats where supported.                                     |
| FR-06 | Users can provide and validate required passenger information.                           |
| FR-07 | Users can initiate payment and view payment status.                                      |
| FR-08 | Users can retrieve their booking details and booking status.                             |
| FR-09 | Users can receive booking confirmation following successful booking and ticket issuance. |

A key takeaway from Part A: **A feature name is not yet a requirement. A useful requirement explains the expected system behaviour and outcome.**

## Part B: Non-functional requirements

This is where I need to be challenged more.

My selection of quality attributes are reasonable:

1. Reliability
2. Usability
3. Privacy
4. Extensibility
5. Backward compatibility

But, the explanations don't consistently describe measurable NFRs.

I'll identify the right general ones, but some of the actual requirements belong to different categories or are too vague to evaluate.

### NFR-01: Reliability

My requirement:

> 99.9% of search results should always match user search queries.

I've identified something important, but classified it incorrectly.

What I described is closer to **functional correctness** or **result accuracy** than reliability.

To distinguish them, suppose a user searches for flights from Lagos to London.

The system returns:

- Lagos to Accra
- Abuja to London
- Lagos to London

The system has returned results, but two of them don't satisfy the user's search criteria.

That's a correctness problem.

Imagine the system sometimes fails to return results because the flight provider is unavailable.

That's a reliability problem.

A better reliability requirement might be:

**NFR-01: The flight-search capability shall maintain 99.9% monthly availability, _assuming an agreed definition of availability and measurement boundary_.**

99.9% availability means approx. 43.8 minutes of downtime per 30-day month.

I could introduce the correctness requirement:

**NFR-02: All returned flight offers must satisfy the user's specified origin, destination, and travel-date criteria.**

Understanding the quality attribute matters so much before assigning a metric to it.

### NFR-02: Usability

My requirement:

> Users should be able to book a flight without creating an account.

This is a perfectly reasonable product decision. But is it really a usability requirement? Not exactly.

It's primarily an interaction or product constraint describing guest checkout. It may improve usability by reducing friction, but it doesn't measure usability itself.

If I consider two applications:

- App A allows guest checkout but has a confusing, difficult-to-navigate booking process.
- App B requires account creation but has an extremely clear booking process.

App A supports guest checkout, but that doesn't automatically make it more usable in every respect.

A more direct usability requirement would be:

**NFR: A first-time user should be able to complete a standard flight-booking process without assistance, with clear feedback at each stage.**

I can even evaluate this NFR through usability testing.

For example:

- Give 10 first-time users a standard booking task.
- Measure how many complete it without assistance.
- Measure completion time.
- Record where users encounter difficulties.

My original guest-checkout idea can remain as a separate functional requirement:

**FR: The system shall allow users to complete a booking without creating an account.**

The important thing is to distinguish the capability from the quality attribute it may support.

### NFR-03: Privacy

My requirement:

> Passenger information and preferences from a booking session must completely cleaned out after the booking session has been completed or cancelled out.

This is a thoughtful attempt at data minimization, but the word **completely** should be challenged.

Imagine a user books a flight.

The booking system may need to retain passenger information to:

- Retrieve the booking.
- Support changes or cancellations.
- Provide customer assistance.
- Meet applicable legal or regulatory obligations.
- Maintain required financial records.

Deleting everything immediately after booking would makes some of these responsibilities impossible.

Here are different types of data and the possible retention approach:

| Data                           | Possible retention approach                                                   |
| ------------------------------ | ----------------------------------------------------------------------------- |
| Temporary search input         | May be cleared when the session ends.                                         |
| Incomplete booking information | Retain only as long as necessary for the intended purpose.                    |
| Completed booking information  | Retain according to operational and legal requirements.                       |
| Payment card details           | Avoid storing raw card details; use the payment provider's secure mechanisms. |
| Saved passenger preferences    | Retain with user consent and provide deletion controls.                       |

A stronger privacy requirement could be:

**NFR: The system shall collect and retain only the passenger information necessary for its stated purposes, protect that information from unauthorized access, and apply defined retention and deletion policies.**

How can I evaluate this?

- Review what information is collected.
- Verify access controls.
- Inspect retention policies.
- Test deletion workflows.
- Confirm that sensitive information isn't unnecessarily exposed in logs or analytics.

One additional point: Privacy isn't just about deleting information. It also concerns what I collect, why I collect it, who can access it, and how it is used.

### NFR-04: Extensibility

My requirement:

> The application should be able to accept new features without requiring changes throughout the entire mobile application or disfiguring the existing experience.

This is one of my strongest answers. I identified an important architectural quality: the ability to evolve the system without causing widespread changes.

However, "accept new features" is broad.

Extensibility can refer to different things:

- Adding new functionality.
- Integrating new providers.
- Supporting new passenger types.
- Adding new payment methods.
- Introducing new flight-related services.

Narrowing the explanation to something meaningful for Book Flight,

**NFR: The system should support adding new functionalities without requiring substantial changes throughout the entire mobile application or disfiguring the existing experience.**

How can I evaluate it?

I introduce a new functionality. I examine:

- How many existing components need modification?
- Can the existing features work with the new provider?

This is a good example of a requirement that can influence architectural boundaries and abstraction decisions.

I shouldn't interpret extensibility as a demand to anticipate every possible feature. That often produces unnecessary abstractions.

### NFR-05: Backward compatibility

My requirement:

> The application with older versions of providers keep working without breaking the user experience.

This is a useful quality attribute, but my wording needs clarification.

Which older versions?

There are at least different compatibility concerns here:

#### Provider API compatibility

Suppose Duffel introduces a new version of its API.

Book Flight should ideally continue operating during a migration period without suddenly breaking because the provider changed an endpoint or response format.

#### Mobile client compatibility

Suppose a user is still using an older version of the Book Flight app.

The backend introduces a new response field or changes its response structure. Should that older app continue working?

All these are different compatibility problems.

A more specific requirement might be:

**NFR: Backward-incompatible changes to the Book Flight mobile API shall not be introduced without an appropriate migration strategy that protects supported existing client versions.**

This can be evaluated by testing older supported app versions against the new backend.

Backward compatibility isn't necessarily an independent quality attribute in every architecture. It's often considered under compatibility, maintainability, and interoperability.

## Part C: Architectural trade-offs

This exercise moves me beyond classifying requirements into reasoning about architectural decisions.

### Scenario 1: Performance vs freshness

My response:

> For this case, I won't cache flight search results. And even if I do, I will set a very short TTL with a maximum of probably about 2 minutes.

I show a good concern about freshness.

Flight offers are volatile. Prices and availability can change between the moment a user searches and the moment they attempt to book.

However, my initial decision not to cache them needs to be challenged.

I treat caching as though it's an all-or-nothing decision. It isn't.

Breaking the information into categories,

| Information                  | Caching suitability            | Reason                                                       |
| ---------------------------- | ------------------------------ | ------------------------------------------------------------ |
| Airport names and codes      | High                           | These don't change frequently.                               |
| Airline names and logos      | High                           | Generally stable information.                                |
| Airport terminal information | Conditional                    | Can change, but often doesn't change frequently.             |
| Flight search offers         | Conditional                    | Useful for short-term performance, but potentially volatile. |
| Flight prices                | Conditional                    | Must be refreshed or validated before booking.               |
| Seat availability            | Low for long-lived caching     | Availability can change as other passengers make selections. |
| Booking status               | Depends on the source of truth | Must reflect the authoritative booking state.                |

My two-minute TTL is a reasonable starting hypothesis, but shouldn't be established as a requirement without understanding the provider's offer-expiration behaviour and the consequences of stale data.

A preferred approach could be: Cache relatively stable reference data aggressively, consider short-lived caching for search results where justified, and require authoritative provider validation before booking.

This illustrates an important principle:

**Caching can improve performance, but cached information must not be mistaken for authoritative current state.**

There's also an important question I haven't considered:

What problem am I trying to solve with caching?

If Duffel responds in 800 milliseconds, caching flight offers may introduce complexity without providing much benefit.

If Duffel takes 8 seconds, I have a stronger reason to consider caching or other performance strategies.

**Architecture should respond to an identified problem, not an assumption that a particular technique is beneficial.**

#### Scenario 2: Availability vs dependency reliability

My response:

> Book Flight shouldn't stop working entirely when Duffel is unavailable. Only capabilities directly tied to the Duffel service gets affected and may stop working when Duffel becomes unavailable.

I gave answer. I showed understanding of the principle of avoiding unnecessary system-wide failure when an individual dependency becomes unavailable.

To make my answer even more concrete, assume Duffel is unavailable.

| Book Flight capability                    | Expected behaviour                                                                         |
| ----------------------------------------- | ------------------------------------------------------------------------------------------ |
| Search for new flights                    | Unavailable                                                                                |
| Retrieve previously saved booking details | May remain available if the required application services and data stores are operational. |
| View saved passenger preferences          | May remain available if independently stored and accessible.                               |
| View historical bookings                  | May remain available.                                                                      |
| Change an existing booking                | Depends on whether the operation requires Duffel.                                          |
| Complete a new booking                    | Should not proceed using an unvalidated or expired offer.                                  |
| Access application settings               | Should remain available if independent of Duffel.                                          |

I should develop an additional distinction.

**A dependency being unavailable doesn't automatically mean every operation that uses it must fail immediately.**

For example, if Duffel is unavailable, the app might still display previously retrieved search results.

Although, it must clearly communicate that those results may not represent current offers. It must not give the user the impression that the displayed offer is guaranteed to be bookable.

This is an example of graceful degradation: the system loses some functionality but preserves what it can safely provide.

#### Scenario 3: Security vs convenience

I wrote:

> I'm finding it a bit difficult to understand and answer this question. It's a quite interesting question and I'll love to here the reasoning on it.

Let's work through it.

Imagine a user has completed a flight booking. Three weeks later, the user returns to Book Flight to book another flight.

I want to make the process easier by remembering their passenger information.

What information should Book Flight remember?

There isn't one universal answer. The decision depends on the value of retaining information, the sensitivity of the information, the user's expectations, and the security implications.

Let's examine three categories.

**Category 1: Low-sensitivity convenience data.**

Examples:

- Preferred currency
- Preferred language
- Frequently used departure airport
- Travel preferences

These are generally reasonable to retain with appropriate user controls.

**Category 2: Personal passenger information.**

Examples:

- Full name
- Date of birth
- Nationality
- Contact information

These are more sensitive data. Remembering them can significantly improve the booking experience, especially for frequent travelers. However, I need to consider:

- Does the user explicitly want this information saved?
- Is it stored securely?
- Can another person using the same device access it?
- Can the user edit or delete it?
- Is the information actually necessary to retain?

**Category 3: Highly sensitive information.**

Examples:

- Passport information
- Travel document numbers
- Identity document images
- Payment credentials

These data deserve considerably more caution.

For example, I would avoid storing card details in Book Flight. I'd use the payment provider's tokenization or secure saved-payment mechanism where supported.

For travel documents, I'd first consider whether persistent storage is genuinely necessary. If it is, I'd introduce stronger access controls, encryption, retention limits, and appropriate user consent.

#### Now consider the actual architectural trade-off

Suppose Book Flight offers two experiences.

**Option A: Don't remember passenger information.**

Every booking now requires users to enter their information again.

Advantages:

- Less sensitive data retained.
- Smaller exposure if the system is compromised.
- Simpler data-retention responsibilities.

Disadvantages:

- Repetitive booking experience.
- More user effort.
- Greater likelihood of input mistakes.

**Option B: Remember passenger information.**

Users can save passenger profiles and reuse them for future bookings.

Advantages:

- Faster repeat bookings
- Less repetitive input.
- Potentially fewer data-entry mistakes.

Disadvantages:

- Greater responsibility for protecting personal information.
- More complicated access control.
- Additional data-retention and deletion requirements.
- Increased consequences if an account is compromised.

Neither of these options is automatically correct.

The architectural decision depends on what the product values and what risks it is willing to accept.

## Important takeaway lessons

### Lesson 1: A feature isn't yet a functional requirement

"Payment" is a feature.

"The system shall allow users to initiate payment for a booking and communicate the resulting payment status" is a FR.

The difference is specificity and expected behaviour.

### Lesson 2: Quality attributes need meaningful measurements

"Reliable," "fast," "secure," and "scalable" are useful quality attributes, but they aren't sufficient requirements by themselves.

I need to establish what the qualities mean in the context of the system.

### Lesson 3: Requirements influence each other

Performance can conflict with freshness.

Convenience can conflict with privacy.

Availability can conflict with the need to validate current information.

All these conflicts are not necessarily problems to eliminate. They're decisions to reason about (and make trade-offs on).

### Lesson 4: Architectural decisions should follow requirements

I shouldn't decide to use caching because caching is a popular architectural technique.

I shouldn't just introduce microservices because I expect the system to grow.

I shouldn't add an abstraction simply because I anticipate future providers.

First, I should understand the problem, its consequences, and the constraints. Then, I can evaluate possible solutions.

## Here's a distinction

| Functional requirements                         | Non-functional requirements                                              |
| ----------------------------------------------- | ------------------------------------------------------------------------ |
| What should the system do?                      | How well should the system do it?                                        |
| Describe capabilities and behaviours.           | Describe quality attributes and constraints.                             |
| Often expressed through features and use cases. | Often expressed through measurable quality targets.                      |
| Example: Users can search for flights.          | Example: 95% of searches complete within 5 seconds under an agreed load. |
