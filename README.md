# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. 
In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber 
is defined as an interface. Explain based on your understanding of Observer design patterns, 
do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model 
struct is enough? 
For me it really depends on how flexible we expect the subscribers to be. If every subscriber in BambangShop behaves the same (same payload, same way to process it), a single concrete struct is totally fine. But once we plan to introduce different subscriber flavors—maybe some only care about certain notifications or need a different payload format—we'll want a trait so the Publisher only knows the `update` contract while each struct can implement it however it likes. The trait becomes an investment to keep everything loosely coupled once subscriber variety grows.
2. id in Program and url in Subscriber is intended to be unique. Explain based on your 
understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently 
use is necessary for this case? 
If we stick with a plain `Vec`, we have to manually scan the list on every insert to make sure a given `id` or `url` is still unique. That's linear time and an easy place to make mistakes when things get busy or parallel. `DashMap` gives us key-value semantics with O(1) lookup/insert/delete plus built-in thread safety. So, with `DashMap`, maintaining uniqueness stays cheap and safe even when multiple threads are reading/writing at the same time.
3. When programming using Rust, we are enforced by rigorous compiler constraints to make a 
thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we 
used the DashMap external library for thread safe HashMap. Explain based on your 
understanding of design patterns, do we still need DashMap or we can implement Singleton 
pattern instead? 
I don't think singleton pattern would work for this case. Singleton ensures that a class has just a single instance that can be used globally. It doesn't necessarily ensure a thread-safe program. In fact, this pattern requires a special treatment in a multithreaded environment so that multiple threads won’t create a singleton object several times.

#### Reflection Publisher-2
1. 
In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. 
Model in MVC covers both data storage and business logic. Explain based on your 
understanding of design principles, why we need to separate “Service” and “Repository” from 
a Model? 
I prefer to keep the Model as a plain data holder while the Service owns the business rules and the Repository owns persistence. Splitting them like this keeps each layer with a single reason to change, makes it easier to mock dependencies in tests, and also protects us from leaking database details into the business logic (or vice versa). In other words, Services can evolve alongside use-cases, Repositories can evolve alongside storage changes, and Models stay lean.
2. What happens if we only use the Model? Explain your imagination on how the interactions 
between each model (Program, Subscriber, Notification) affect the code complexity for 
each model? 
If we stuff everything into the models, `Program`, `Subscriber`, and `Notification` all start knowing too much about each other. Program would need to handle subscription bookkeeping, firing HTTP calls, and maybe even storing data, while Subscriber ends up doing validation and Program-specific flows. The tangled interaction means every change (say, new notification type) forces edits across multiple models, increases chance of circular dependencies (couplers! which we studied in previous module), and makes unit testing painful because there's no seam to mock behavior.
3. Have you explored more about Postman? Tell us how this tool helps you to test your current 
work. You might want to also list which features in Postman you are interested in or feel like it 
is helpful to help your Group Project or any of your future software engineering projects.
Postman is very helpful. I keep the BambangShop collection with environment vars for base URLs so I can switch between local and deployed instances quickly, then use the Collection Runner to replay the subscribe/publish/unsubscribe flow after each change. I’m also starting to rely on the built-in test scripts to assert status codes and response bodies automatically, which feels handy for any future group project regression suite.

#### Reflection Publisher-3
1. 
Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and 
Pull model (subscribers pull data from publisher). In this tutorial case, which variation of 
Observer Pattern that we use?
We use the push model because the publisher actively ships each notification payload right after a product event happens. Notification isn't sent when subscribers poll or log in first.
2. What are the advantages and disadvantages of using the other variation of Observer Pattern 
for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull) 
If we change to use pull model, subscribers would be responsible for polling the publisher’s API to fetch the latest product changes. That approach reduces the load on the publisher (no need to fan out HTTP requests), but we lose the real-time feel and instead add latency and wasted network calls when nothing changes. It also puts more burden on subscriber implementations to schedule polling jobs and reconcile state conflicts.
3. Explain what will happen to the program if we decide to not use multi-threading in the 
notification process. 
Without multi-threading each notification would be sent sequentially, the publisher would block on one HTTP call before moving to the next subscriber. That means a slow or unreachable subscriber stalls everyone behind it, the original product API call stays open longer, and users subscribed to fast endpoints still get their updates late. It's especially painful if we’re broadcasting flash promotions or low-stock alerts.
