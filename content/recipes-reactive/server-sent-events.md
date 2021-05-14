+++ 
categories = ["recipes"]
tags = ["application development", "sse","reactor","spring webflux"]
summary = "Server-Sent Events in Spring"
title = "Server-Sent Events in Spring"
date = 2021-05-13T14:02:27-05:00 
weight = 9 
+++

## CONTEXT

Server-Sent-Events, or SSE, is an **HTTP** standard that allows a web application to handle a unidirectional event
stream and receive updates whenever server emits data. An SSE is a specification adopted by most browsers to allow
streaming events unidirectionally at any time. The *events* are just a stream of UTF-8 encoded text data that follow the
format defined by the specification. This format consists of a series of key-value elements _(id, retry, data, event,
comments)_ separated by line breaks. Comments are supported as well.

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

### Solution

To achieve this, we can make use of implementations such as the Flux class provided by the Reactor library, or
potentially the `ServerSentEvent` entity, which gives us control over the events metadata.

## Stream Events Using Flux

Create below `streamEvent` method in `CustomerController` like below

```java
@GetMapping("/streamEvents")
public Flux<ServerSentEvent<String>>streamEvents(){
        return Flux.interval(Duration.ofSeconds(1))
        .map(sequence->ServerSentEvent.<String> builder()
        .id(String.valueOf(sequence))
        .event("periodic-event")
        .retry(Duration.ofHours(2))
        .data("SSE - "+LocalTime.now().toString())
        .build());
}
```

ServerSentEvent entity allows us to

* Handle the events metadata which consists of `id`, `event`, `retry`, `comment`, `data`
* There is no need to use `text/event-stream` media type declaration. The data in events metadata can be of type `T`
* The retry value will specific the reconnection time to be used when trying to send the event.

## Consuming the Server-Sent Events with a WebClient

To consumer server-sent events using webclients create a method in `CustomerService` class like below:

```java
public Flux<ServerSentEvent<String>>consumeServerSentEvent(){
        ParameterizedTypeReference<ServerSentEvent<String>>type = new ParameterizedTypeReference<ServerSentEvent<String>>(){};

        return webClientWithTimeout.get()
        .uri("/stream-sse")
        .retrieve()
        .bodyToFlux(type);
}
```

* The **subscribe** method allows us to indicate how we'll proceed when we receive an event successfully, when an error
  occurs, and when the streaming is completed.
* The **retrieve** method is used to retrieve the response and it will throw a `WebClientResponseException` if we
  receive a 4xx or 5xx response.

The source code is available at: [WF GitHub](https://github.wellsfargo.com/app-ebst/wf-reactive-project-starter)


