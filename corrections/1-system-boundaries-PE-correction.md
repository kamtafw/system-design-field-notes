# STAGE 1

The overall assessment is: **I understand the basic idea of system boundaries, but my understanding of interfaces and dependencies is still developing.**

## Part A: Identify the system boundary

My classification is largely correct.

The product capabilities I mentioned are valid, but there's also a subtle distinction between what my system is responsible for and what it actually performs itself.
Consider payment.
Book Flight is responsible for facilitating a booking payment. But it doesn't actually process the financial transaction. That responsibility belongs to the payment provider.
Book Flight might:

1. Initiate a payment request
2. Send the necessary payment information to the payment provider
3. Receive the payment result
4. Update the booking's payment status
5. Continue the booking workflow

The payment provider handles the actual payment processing.
Similarly, consider notifications.

Book Flight might be responsible for:

- Deciding when a notification should be sent
- Determining its content
- Identifying the intended recipient
- Initiating the delivery request
- Tracking whether delivery succeeded, where supported

The notification provider is responsible for delivering the notification through its infrastructure.

In essence, **outsourcing an operation doesn't necessarily mean outsourcing responsibility for the entire business process.**

Here's a useful way to think about this:

| Activity             | Book Flight                                          | External provider            |
| -------------------- | ---------------------------------------------------- | ---------------------------- |
| Flight search        | Initiates and coordinates search                     | Supplies flight offers       |
| Payment              | Initiates and tracks payment                         | Processes the transaction    |
| Notification         | Determines when and what to send                     | Delivers the notification    |
| Booking confirmation | Determines booking status and generates confirmation | May deliver the confirmation |

I also classified database as internal to the system boundary, and there may be two reasons to consider this.
Imagine Book Flight uses Appwrite as its database provider.

### Perspective 1: Book Flight as the system under consideration

The database gets to be inside the system boundary when the complete product and its internal components are under examination.

### Perspective 2: Book Flight as an application dependent on managed infrastructure

The database service is moved outside the application boundary because it's an externally managed service.

Neither perspective applied in a diagram is automatically incorrect, cos the boundary depends on what is under examination.

**I should refine my understanding of responsibility ownership, particularly when external providers perform operations on my system's behalf.**

**Note:** This is my reasoning. I'm not so clear about the term _responsibility ownership_, and also your explanation for classifying the database. But, this what I get, and I stand to be corrected. The database can be placed within or outside the system boundary depending on what's being examined. I'm allowed to place it within the system boundary if I try to examine every single component my product is able to modify. And I'm not allowed to place it within the system boundary because the database is a service managed by Appwrite which is "outside." This also causes confusion to me about my backend/server-logic which gets to work when deployed on Appwrite Functions.

### System boundaries, databases, and responsibility ownership

My updated understanding is mostly correct. It's not quite a matter of being allowed or not allowed. **A system boundary is determined by what I'm trying to examine, not simply by who manages a component.**

Consider three different perspectives of Book Flight.

#### Perspective A: The Book Flight Product

_I'm examining everything that makes the product work from the user's perspective._

The mobile app, backend logic, database, and external integrations can all be considered parts of the Book Flight system.

The fact that Appwrite manages the database doesn't change the fact that Book Flight depends on it to deliver its functionality.

#### Perspective B: The application software I develop

_I'm examining the software my team actually implements and controls._

My mobile app and backend business logic are inside this boundary. Appwrite's managed database and function execution environment are external services that my software depends on.

#### Perspective C: The deployment infrastructure

_I'm examining where and how the application runs._

Now Appwrite becomes and important part of the picture. It provides infrastructure and managed services on which my app depends.

Notice something interesting: the same database can be inside one system boundary and outside another. Neither diagram is inherently wrong. They answer different questions. _Think of it as looking at same person from different directions._

### What is responsibility ownership?

Think of it as **which entity is accountable for a particular job or outcome.**

Using flight booking.

| Responsibility                                     | Owner                 |
| -------------------------------------------------- | --------------------- |
| Collecting flight search details                   | Book Flight           |
| Deciding when to search for flights                | Book Flight           |
| Providing flight inventory and offers              | Duffel                |
| Managing the booking experience                    | Book Flight           |
| Processing a payment transaction                   | Payment provider      |
| Deciding when a payment should be initiated        | Book Flight           |
| Delivering an email notification                   | Notification provider |
| Deciding what notification should be sent and when | Book Flight           |

The important distinction is: **Using an external service to perform an operation doesn't mean I've transferred ownership of the entire business responsibility to that service.**

For example, a payment provider processes a transaction, but Book Flight remains responsible for managing the booking process and determining what should happen when payment succeeds or fails.

This distinction becomes particularly important when designing systems with external dependencies.

## Part B: Identify its interfaces

I get the general idea here, but my descriptions are mostly focused on the immediate request and not the complete interaction contract.

### 1. User <-> Mobile application

My response:

> They communicate over the app UI. The user passes the search query into the input and is returned a list of available flights matching the query, or some other UI that matches the response state.

I've identified all that's necessary, but I need to think beyond the search input.

The UI is an interface in the architectural sense cos it defines how the user interacts with the app.

But an interface isn't simply a visual element through which information passes. It also establishes expectations about behaviour.

For example:

The user expects:

- Search fields to accept appropriate information
- Invalid inputs to be identified
- Loading states to communicate ongoing operations
- Errors to be communicated meaningfully
- Search results to be presented in an understandable format
- Actions such as selecting a flight to produce predictable outcomes

The app, on the other hand, expects the user to provide information that satisfies the requirements of the search operation.

This makes my statement that: **The mobile app expects a string input from the user** too narrow. A flight search isn't necessarily a single string.

It might involve:

```ts
interface FlightSearchRequest {
	origin: string
	destination: string
	departureDate: string
	adults: number
	children: number
	infants: number
}
```

The app expects a structured input that satisfies the search requirements.

**Note:** I have to say, my statement about what the mobile app expects from the user is wrong. I said it that way cos my thought was very thin, and I was only thinking about what could come directly from the user, i.e. the user can only provide a text to the input field. But, this explanation has broadened my view and understanding so well. I now think of the user <-> mobile app interface as a contract beyond what spans the eye.

My revised understanding is correct. I've moved away from thinking of the interface as merely the input field and the information displayed on the screen.

The interaction contract includes things such as:

- What information the user can provide.
- What information the app promises to display.
- What happens when the input is valid.
- What happens while an operation is processing.
- What happens when the operation fails.
- What actions are available to the user in each state.

The user and the application have expectations of each other.

### 2. Mobile application <-> Backend

My response:

> They communicate over the backend API endpoint. The mobile app sends a request through the endpoint with a search param, and the backend responds with an array of available flights matching the search param.

Again, this is correct in principle. But, it's broader than that.

An API contract actually represents:

| Contract element       | What it establishes                                |
| ---------------------- | -------------------------------------------------- |
| HTTP method            | What operation is being requested                  |
| Endpoint               | Where the operation is exposed                     |
| Request schema         | What information the backend expects               |
| Authentication         | What credentials or permissions are required       |
| Response schema        | What information the mobile application can expect |
| Status codes           | How the outcome of the operation is communicated   |
| Error responses        | How unsuccessful operations are represented        |
| Behavioural guarantees | What the caller can reasonably expect              |

Each of these elements is an expectation met by either entity in the interaction.

The other entity breaks when an expectation is no longer satisfied.

The important lesson is that **an interface establishes expectations that both participating entities must respect.**

**Note:** This interaction went on to strengthen my understanding of an interface. The interaction here is also between two entities, but you'll notice that the contract elements are different from what exist between the user <-> mobile app interaction. But cos it remains a contract, failure from an entity in any interaction to meet the contract element it's responsible for causes an interface contract problem.

My reasoning here is aso developing well.

But, not every failure is a contract violation.

Consider the interaction between the backend function and Duffel.

- My function sends an incorrectly structured request. That's a contract violation.
- Duffel returns a response that doesn't conform to its documented response structure. That's a contract violation.
- Duffel experiences an outage and doesn't respond. That's primarily an availability or operational failure.
- Duffel responds after 30 seconds when my app expects a response within 5 seconds. That may be a violation of a timing guarantee, if the contract explicitly establishes that one.

The distinction matters cos different problems require different solutions.

A malformed request might require fixing my integration logic. An outage might require timeout handling, retries, or graceful degradation.

A good system designer learns to identify the kind of failure before deciding how to handle it.

### 3. Backend <-> Duffel

My response:

> They communicate over the Duffel API through Appwrite. The backend passes the request object, and this object is received by the Duffel API where it's deployed on Appwrite.

My response shows that I'm conflating three things:

1. My backend code.
2. Appwrite Functions.
3. Duffel's API.

Let's make it clear that:

- The mobile app communicates with Appwrite Functions.
- The Appwrite Functions executes the server-side flight-search logic.
- That function communicates with Duffel's API.
- Duffel's credentials remain on the server side.

So, the important correction is: **Duffel's API isn't deployed on Appwrite. My function is deployed on Appwrite and communicates with Duffel.**

I should think of Appwrite as providing the environment in which my backend function executes.

My function is the code responsible for coordinating the operation

Duffel is an external service providing flight inventory.

A simplified representation:

```text
       BOOK FLIGHT SYSTEM

       Mobile Application
               |
               |
               v
       Appwrite Function
       (Your backend logic)
               |
               |
       SYSTEM BOUNDARY
               |
               v
          Duffel API
       (External provider)
```

The interface between Duffel and my backend is the Duffel's API contract.
My function must satisfy the requirements of Duffel's API when making requests.
Duffel must return responses that conform to its documented contract.
My backend then interprets those responses and transforms them into a format expected by my mobile app.
This introduces the **adapter**. The adapter translates between two interfaces or contracts.

Here, the adapter lives in the function, which acts as an integration boundary between my app and the external provider.
It doesn't simply forward arbitrary information. It translates requests, handles provider responses, and manages errors.

The adapter becomes particularly important if Duffel ever gets replaced with another flight inventory provider. Cos ideally, my mobile app shouldn't need to know which provider supplies the flight offers.

I should be able to change the provider integration without unnecessarily changing the mobile app's contract with the backend.

That's one of the benefits of establishing clear interfaces.

**Note:** One major blocker for me was understanding the relationship between the backend code, Appwrite, Appwrite Functions itself, Duffel, and also Duffel API. I feel very much better understanding how they interact. Let me try explaining. The backend code is a remote implementation that gets deployed into Appwrite Functions. The Appwrite Functions is an Appwrite service alongside others like the database. Appwrite Functions provides a working environment for the backend code. Duffel is the flight inventory provider, and the Duffel API is a contract to establish an interaction between Duffel and any other entity. Here, Appwrite uses it for the interaction between Functions, where the server-logic lives and Duffel. Also, this server-logic contains the adapter between the mobile app and Duffel. I know the explanation isn't straight forward, but that's how I have it in my head right now.

My explanation is substantially correct. A small refinement is that:

The backend code isn't inherently remote. It's server-side application logic that becomes remotely accessible when deployed and exposed through an appropriate interface.

Here's the relationship:

```text
                 BOOK FLIGHT
                      |
          +-----------+-----------+
          |                       |
      Mobile App           Server-side logic
                                  |
                         Appwrite Functions
                         (execution environment)
                                  |
                         Flight Search Logic
                                  |
                            Duffel Adapter
                                  |
                             Duffel API
                                  |
                           Duffel Platform
```

Also, the server-side logic contains an adapter that isolates Book Flight's internal flight-search logic from Duffel's API.

The adapter indirectly protects the mobile application from Duffel-specific changes because the mobile app communicates with Book Flight's backend, not Duffel directly.

### 4. Backend <-> Appwrite

My response:

> It's the backend and Appwrite communicating. This communication is through Appwrite Functions. I can't confidently explain how the communication happens or what information is passed between them, or what each entity expects.

My uncertainty here may be justified. Because, in my current architecture, Appwrite Functions are part of the backend. The backend logic is implemented using Appwrite Functions. Appwrite provides the execution environment and associated services. Therefore, drawing and interaction between "the backend" and "Appwrite Functions" could be misleading.

The only time there might be genuine interactions between my function and Appwrite's platform services are:

- My function might interact with Appwrite Databases.
- It might use Appwrite authentication services.
- It might access Appwrite Storage.
- It might use Appwrite SDK capabilities.

Those would be legitimate interactions worth documenting.

In such cases, I'll need to distinguish them from the function's execution environment.

```text
        BOOK FLIGHT

        Mobile App
             |
             v
       Appwrite Functions
       (Backend logic)
          /       \
         /         \
        v           v
   Appwrite      Duffel
   Database      API
```

The exact components depend on which Appwrite services my implementation actually uses.

### 5. User <-> Payment provider

My response:

> The user initiates the payment using the mobile app, and may have no knowledge of a payment provider. I also don't have a clear understanding of this interaction and the systems involved.

An interaction doesn't require both participants to be directly aware of each other.

The user interacts with Book Flight. Book Flight interacts with the payment provider. The payment provider processes the transaction. The user doesn't necessarily need to know which provider is involved.

Consider a hypothetical payment integration:

```text
User
 |
 | Initiates payment
 |
 v
Mobile Application
 |
 | Requests payment initialization
 |
 v
Appwrite Function
 |
 | Sends payment request
 |
 v
Payment Provider
 |
 | Returns payment initialization details
 |
 v
Mobile Application
 |
 | Presents payment interface
 |
 v
User
 |
 | Completes payment
 |
 v
Payment Provider
 |
 | Sends payment confirmation
 |
 v
Appwrite Function
 |
 | Verifies payment result
 |
 v
Booking Database
```

This is simplified, but illustrates the different interactions.

The payment provider may communicate with the backend through a webhook.

A webhook is an HTTP request initiated by one system to notify another system about an event.

For example, the payment provider might send the backend a notification that a transaction has succeeded.

This raises further questions:

- How does my backend know the payment is genuine?
- Does it trust the mobile app when it says payment succeeded?
- What happens if the webhook arrives twice?
- What happens if payment succeeds but my backend is temporarily unavailable?
- How do I associate the payment with the correct booking?

These are questions I'll investigate in later stages.

## Part C: Investigate dependencies

This section demonstrates my level of understanding of the practical implications of system boundaries.

### Scenario 1: Duffel becomes unavailable

My response:

> When Duffel becomes unavailable, this means the system has lost contact with a flight inventory provider, and active services like searching for available flights may not function. The user may only get access to history which have been persisted to the database.

I may have correctly identified that the availability of a critical dependency affects the functionality of my system. But, there's the question: **Does the unavailability of Duffel mean the entire Book Flight system becomes unavailable?**

No. It can only mean functionality dependent on Duffel may become unavailable, while other functionality might continue working.

For example:

| Functionality                             | Expected behaviour                               |
| ----------------------------------------- | ------------------------------------------------ |
| Search for flights                        | Unavailable                                      |
| View previously saved booking information | Potentially available                            |
| View locally cached information           | Available if present                             |
| Access passenger information              | Depends on where it's stored                     |
| Modify an existing booking                | Depends on whether the operation requires Duffel |
| Access application settings               | Should remain available                          |

This should make me understand that an external dependency becoming unavailable should only affect the capabilities that genuinely require it.

This is the beginning of thinking about graceful degradation.

Rather than treating the system as either fully functional or completely unavailable, I begin identifying which capabilities can continue operating independently.

### Scenario 2: Appwrite becomes unavailable

My response:

> This time, every operation with interaction outside the mobile app becomes impossible. The mobile app can only display cached data which is also for a short while. It still functions, but may give a terrible experience to the user. Existing data still lives in the database.

My answer needs more refinement, and two statements are to be challenged.

**First: "Every operation with interaction outside the mobile app becomes impossible."**
This is too broad. My mobile app could potentially interact with other external systems directly if they're unaffected.

Importantly, the failure of Appwrite Functions doesn't necessarily mean every other Appwrite service becomes unavailable.

**Second: "Existing data still lives in the database."**
If Appwrite is unavailable, the database might still physically contain the data, but my app may not be able to access it.

Availability and durability are different properties.

- **Availability:** Can the system access the data when it needs it?
- **Durability:** Will the data survive failures and remain recoverable?

#### Let's make the scenario more precise.

Suppose Appwrite Functions becomes unavailable, but the database remains operational.

What happens?

- The mobile app might still retrieve existing data if it has another available access path.
- New flight searches cannot execute through the unavailable function.
- Passenger information already saved locally may remain accessible.
- Previously saved booking information might still be retrievable through another functioning backend path.

Now suppose the entire Appwrite platform becomes unavailable. The situation is different.

My mobile app may be limited to locally persisted data and functionality that doesn't require the backend.

For clarity, cached data and locally persisted application data aren't necessarily the same thing.

A cache might be temporary and disposable. Local application storage might be deliberately designed to preserve user data.

### Scenario 3: Backend changes its response structure

My response:

> When this happens, changes in the system depends on how coupled it were. For a good system, only the backend and the adapter receiving the backend responses is affected. This means the mobile app and backend contract interface gets changed. The mobile app still expecting the old structure makes it crash very often.

I correctly recognized that coupling influences the impact of changes.

There's a contradiction in my reasoning.

I initially suggest that only the backend and adapter should be affected.

Then I say the mobile app expects the old structure and crashes.

Those two statements describe different outcomes.

#### Case A: A breaking change

Suppose the backend originally returns:

```json
{
	"offers": []
}
```

And it gets changed to:

```json
{
	"data": {
		"offers": []
	}
}
```

My mobile app still expects:

```ts
response.offers
```

The app now receives `undefined` where it expects an array.

And depending on how the response is handled, this could cause rendering errors or other unexpected behaviour.

**The mobile app is affected.**

#### Case B: A backward-compatible change

Suppose I introduce a new optional field:

```json
{
	"offers": [],
	"search_id": "abc123"
}
```

**The existing mobile app ignores the new field and continues working.**

In this case, the backend has changed its response, but the mobile app's existing contract hasn't been broken.

This distinction introduces the concept of backward compatibility.

#### Case C: An adapter isolates the change

Imagine my backend uses a Duffel adapter.

The adapter translates Duffel's response into my app's internal flight-offer model.

Duffel changes its response structure.

I update the adapter to accommodate the new provider response.

**The mobile app doesn't need to know anything about that change.**

This is an example of architectural isolation.

Here are the distinction:

- The Duffel adapter isolates my app from changes to Duffel's interface.
- The mobile app's API contract isolates the mobile client from changes to the backend's internal implementation.

These are two different boundaries.

I was getting close to this idea, but you hadn't separated the two.

**Note:** What caused my error was the picture I had in my head while explaining. I pictured the adapter on the client-side (mobile app); and this means that any change in the backend contract will definitely affect the mobile app. Only that to what extent depends on how effective the adapter is, or how coupled the contract is in the mobile app. I'll use this as opportunity to answer the question: "Can two components communicate through an API and still be tightly coupled?" I'll say yes. Say a UI makes a call and receives a response. Using the response fields directly in the component may only tighten it, and so any change at all can break the contract. This is where adapters help reduce coupling.
