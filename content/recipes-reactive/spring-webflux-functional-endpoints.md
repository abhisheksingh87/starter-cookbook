+++
categories = ["recipes"]
tags = ["application development", "reactive","spring", "functional endpoints", "reactor","spring webflux"]
summary = "Spring WebFlux Functional Endpoints"
title = "Spring WebFlux Functional Endpoints"
date = 2021-01-06T14:02:27-05:00
weight = 5
+++

## CONTEXT

Spring WebFlux includes WebFlux.fn, a lightweight functional programming model in which functions are used to route and handle requests and contracts are designed for immutability.
It is an alternative to the annotation-based programming model but otherwise runs on the same *reactor-core* foundation.

This recipe walks you through creating functional endpoints.

## Solution

1. In **WebFlux.fn** an HTTP request is handled with a **HandlerFunction:** a function that takes `ServerRequest` and returns a delayed `ServerResponse` (i.e. `Mono<ServerResponse>`). 
   
   HandlerFunction is the equivalent of the **body** of a @RequestMapping method in the annotation-based programming model.

   Incoming requests are routed to a handler function with a **RouterFunction:** a function that takes ServerRequest and returns a delayed HandlerFunction (i.e. Mono<HandlerFunction>). 
   
   RouterFunction is the equivalent of a `@RequestMapping` annotation, but with the major difference that router functions provide not just data, but also behavior.
   `RouterFunctions.route()` provides a router builder that facilitates the creation of routers, as the following example shows:

    ```java
    @Bean
    public RouterFunction<ServerResponse> accountsRoute(AccountHandler accountHandler) {
        return route(GET("/accounts").and(accept(MediaType.APPLICATION_JSON)), accountHandler::getAllAccounts)
    
              .and(route(GET("/accounts/{id}").and(accept(MediaType.APPLICATION_JSON)), accountHandler::findById))
    
              .and(route(POST("/accounts").and(accept(MediaType.APPLICATION_JSON)), accountHandler::save));
    }
    ```

1.  Create a handler class which will expose a reactive `AccountRepository`.
    
    ```java
    public Mono<ServerResponse> getAllAccounts(ServerRequest request) {
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(accountRepository.findAll(), Account.class));
    }
    
    public Mono<ServerResponse> findById(ServerRequest request) {
        return ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .body(accountRepository.findById(request.pathVariable("id"), Account.class));
    }
    ```
    {{% note%}}
    ServerResponse provides access to the HTTP response and can be created using the build method. The builder can set the response code, response headers or a body.
    Handler functions should be grouped together
    {{% /note%}}

1. Test Case: `WebTestClient` is built on these mock request and response objects to provide support for testing WebFlux applications without an HTTP server.

    ```java
    @Test
    public void createAccount() {
            Account account = Account.builder()
                                    .accountNumber("918345")
                                    .routingNumber("234518")
                                    .accountOwner("alex")
                                    .build();
   
            webTestClient.post().uri("/accounts").contentType(MediaType.valueOf(MediaType.APPLICATION_JSON_VALUE))
                    .body(Mono.just(account),Account.class)
                    .exchange()
                    .expectStatus().isCreated()
                    .expectBody()
                    .jsonPath("$.accountId").isNotEmpty()
                    .jsonPath("$.accountOwner").isEqualTo("alex");
        }
    ```
   The source code is available in: [Wells Fargo GitHub](https://)   
