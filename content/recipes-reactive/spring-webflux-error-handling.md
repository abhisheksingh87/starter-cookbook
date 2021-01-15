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

### Exception and Errors
**Exceptions** : An exception indicates conditions that an application might want to catch and the application might be able to recover from this condition gracefully.

**Error**: An Error indicates serious problems that an application should not try to catch or recover from. For an example OutOfMemoryError.

The same rules apply to reactive programming world as well, however since everything is a signal (whether it is a successful event or error) people use Errors when they refer to error signals in reactive streams.

_Error signals are first class citizens in reactive world._
>Any error signal in reactive sequence is a terminal event

As soon as an error occurs, it stops the sequence and  it stops the sequence and gets propagated down the chain of operators to the last step, the Subscriber you defined and its onError method.

### Error Handling Operators
You may be familiar with several ways of dealing with exceptions in a **try-catch** block. Most notably, these include the following:

* Catch and return a static default value.
* Catch and execute an alternative path with a fallback method.
* Catch and dynamically compute a fallback value.
* Catch, wrap to a BusinessException, and re-throw.
* Catch, log an error-specific message, and re-throw. 
  
All of these have equivalents in Reactor, in the form of error-handling operators

* Static Fallback Value:
  The equivalent of “Catch and return a static default value” is **onErrorReturn**. The following example shows how to use it:
 
```java
try {
  return findById(54);
}
catch (Throwable error) {
  return "RECOVERED";
}
```

The following example shows the Reactor equivalent:

```java
Flux.just(54)
    .map(this::findById)
    .onErrorReturn("Recovered");
```
* Fallback Method
  If you want more than a single default value and you have an alternative (safer) way of processing your data, you can use onErrorResume. 
  This would be the equivalent of “Catch and execute an alternative path with a fallback method”. For Example,
```java
String name;
try {
  name = externalService("key");
}
catch (Throwable error) {
 name = externalServiceFallBack("key");
}
```
The following example shows the Reactor equivalent:
```java
    Flux.just("key")
        .flatMap(k -> externalService(key)
        .onErrorResume(e -> externalServiceFallBack(key))
        );
```
 
 