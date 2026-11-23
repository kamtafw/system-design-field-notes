# STAGE 1

## Part A: Identify the system boundary

We'll be examining Book Flight as a complete flight-booking product.

Things that can be said to belong to the system are:

- the mobile app
- the backend
- the database

External systems the system depends on include:

- the flight inventory provider
- the payment provider
- the email or notification service

Actors interacting with the system include:

- Book Flight customer (user)

Responsibilities belonging to the system:

- search for flights
- view available flights
- select flights
- enter passenger information
- pay for a booking
- receive booking confirmation

Responsibilities belonging to external systems:

- provide flight inventory to the system
- make flight payment to receiving end
- send notifications to users

## Part B: Identify its interfaces

1. The user searches for flights through the mobile application.
In this interaction, the entities interacting are the user and mobile app. They communicate over the app UI. The user passes the search query into the input and is returned a list of available flights matching the query, or some other UI that matches the response state. In this interaction, the mobile app expects a string input from the user, and the user expects a page of flights.

2. The mobile application requests flight offers from the backend.
Here, it's the mobile app and the backend interacting. They communicate over the backend API endpoint. The mobile app sends a request through the endpoint with a search param, and the backend responds with an array of available flights matching the search param.

3. The backend retrieves offers from Duffel.
The backend and Duffel - our flight inventory provider - are now interacting. They communicate over the Duffel API through Appwrite. The backend passes the request object, and this object is received by the Duffel API where it's deployed on Appwrite. Duffel then responds with the offers.

4. The backend communicates with Appwrite.
It's the backend and Appwrite communicating. This communication is through Appwrite Functions.I can't confidently explain how the communication happens or what information is passed between them, or what each entity expects.

5. The user eventually initiates a payment.
The user initiates the payment using the mobile app, and may have no knowledge of a payment provider. I also don't have a clear understanding of this interaction and the systems involved.

## Part C: Investigate dependencies

Scenario 1: Duffel becomes unavailable.
When Duffel becomes unavailable, this means the system has lost contact with a flight inventory provider, and active services like searching for available flights may not function. The user may only get access to history which have been persisted to the database.

Scenario 2: Appwrite becomes unavailable
This time, every operation with interaction outside the mobile app becomes impossible. The mobile app can only display cached data which is also for a short while. It still functions, but may give a terrible experience to the user. Existing data still lives in the database.

Scenario 3: Your backend changes the structure of its flight-search response.
When this happens, changes in the system depends on how coupled it were. For a good system, only the backend and the adapter receiving the backend responses is affected. This means the mobile app and backend contract interface gets changed. The mobile app still expecting the old structure makes it crash very often.
