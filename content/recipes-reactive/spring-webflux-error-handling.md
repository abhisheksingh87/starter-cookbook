+++
categories = ["recipes"]
tags = ["reactive","spring", "reactor","spring webflux"]
summary = "Spring WebFlux Exception Handling"
title = "9. Spring WebFlux Exception Handling"
date = 2021-01-06T14:02:27-05:00
weight = 1
+++

## CONTEXT
This recipe walks you though how to do error handling in a spring webflux application.

## SOLUTION
In Reactive Streams, errors are terminal events. As soon as an error occurs, it stops the sequence and gets propagated down the chain of operators to the last step, 
the defined **Subscriber** and its **onError** method.

Such errors should still be dealt with at the application level.
For instance, you might display an error notification in a UI or send a meaningful error payload in a REST endpoint. 
For this reason, the subscriber’s **onError** method should always be defined.

> If not defined, onError throws an **UnsupportedOperationException**.

One of the most important things to note is _any error in a reactive sequence is a terminal event_. Even if error-handling operator is used, it does
not let the original sequence to continue. Rather, it converts the **onError* signal into the start of new sequence (fallback)

### Error Handling Operators
You may be familiar with several ways of dealing with exceptions in a **try-catch** block. Most notably, these include the following:

* Catch and return a static default value.
* Catch and execute an alternative path with a fallback method.
* Catch and dynamically compute a fallback value.
* Catch, wrap to a BusinessException, and re-throw.
* Catch, log an error-specific message, and re-throw. 
  
All of these have equivalents in Reactor, in the form of error-handling operators

* **Static Fallback Value:**
  The equivalent of “Catch and return a static default value” is **onErrorReturn**. The following example shows how to use it:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return findById(request.pathVariable("id"))
      .onErrorReturn("No Customer Found")
      .flatMap(s -> ServerResponse.ok()
      .contentType(MediaType.APPLICATION_JSON)
      .syncBody(s));
}
```
* **Fallback Method:**
  If you want more than a single default value and you have an alternative (safer) way of processing your data, you can use onErrorResume. 
  This would be the equivalent of “Catch and execute an alternative path with a fallback method”. For Example,

The following example shows how to use it:
```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return findById(request.pathVariable("id"))
      .flatMap(s -> ServerResponse.ok()
      .contentType(MediaType.TEXT_PLAIN)
      .syncBody(s))
      .onErrorResume(e -> findByIdFallback()
                          .flatMap(s ->; ServerResponse.ok()
                          .contentType(MediaType.APPLICATION_JSON)
                          .syncBody(s)));
}
```

* **Dynamic Fallback Value:**

If you do not have an alternative way of processing your data, you might want to compute a fallback value
out of the exception you have received. This would be equivalent of "Catch and dynamically compute a fallback value".

The following example shows how to use it:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return findById(request.pathVariable("id"))
      .flatMap(s -> ServerResponse.ok()
      .contentType(MediaType.APPLICATION_JSON)
          .syncBody(s))
      .onErrorResume(e -> Mono.just("Error " + e.getMessage())
                          .flatMap(s -> ServerResponse.ok()
                          .contentType(MediaType.APPLICATION_JSON)
                          .syncBody(s)));
}
``` 

* **Catch and Rethrow:**
The final option using _onErrorResume()_ is to catch, wrap, and re-throw an error e.g. InvalidAccountException:

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    return ServerResponse.ok()
      .body(findById(request.pathVariable("id"))
      .onErrorResume(e -> Mono.error(new InvalidAccountException(
                                    HttpStatus.BAD_REQUEST, 
                                    "Account is invalid", e))), Account.class);
}
```

## Handling Errors at a Global Level

All the above examples provide ways to handle errors at functional level. 
