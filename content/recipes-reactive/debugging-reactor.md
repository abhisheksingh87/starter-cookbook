+++
categories = ["recipes"]
tags = ["application development", "debugging","reactor","spring webflux"]
summary = "Debugging Spring Reactive Applications"
title = "Debugging Spring Reactive Applications"
date ="2021-05-12T14:02:25-05:00"
weight = 10
+++

## CONTEXT

This recipe deals with different approaches to debug reactive applications. The recipe provides different techniques of
adding traceability to the Reactive Pipeline

### Reactive Ecosystem

The Reactive Pipeline can be chronologically divided into three phases:

* Assembly phase
* Subscription phase
* Runtime phase

The **Assembly phase** is when the Project Reactor framework actually builds the pipeline, and the end result of this 
is the creation of the **Publisher object (Mono/Flux)**. This phase is _declarative_ as developer creates the chain
of operators that transform the given publisher, but nothing is **executed** at this point.

The **Subscription phase** is when the Project Reactor framework creates the Subscriber object and the end result is a Subscription object that will then receive the results of the Reactive Pipeline in which the data is fed from the Publisher object.
This phase _triggers_ reactive pipeline to start the data flow and actually produce some results that can be consumed.

The  **Runtime/Execution phase** represents the moment when the pipeline is already publishing data from one end 
(the upstream) to another end (the downstream/subscriber) and it’s where the errors can manifest.
This is the phase where we can _mitigate/resolve_ most of the issues we will encounter when working with Reactive Applications.

Now that we have knowledge of Reactor lifecycle let's deep dive into debugging tools and techniques 

### Detect Blocking Calls:
Please refer to [recipe](/recipes-reactive/block-hound)

### Debugging tools and techniques:

There are few ways we can debug reactor pipeline:

* The `log()`
* The `doOnError()` 
* `onOperatorDebug()`
* The `checkpoint()`.

Let us take a detailed look into each one of them:

## The `log()` operation:

The `log()` method is a quick way to get an easy bird eye’s view of what is going on at one step of your sequence. 
A good practice is to **name** each log call to differentiate them, making them easier to identify which part of the pipeline emitted the data that’s been logged.

The types of operations that are logged by the “log()” method are:

* onNext
* onComplete
* onError
* onSubscribe
* cancel
* request

Each of these operations will be logged every time they occur, and also, it's possible to specify a custom logger as well.
The purpose of this operator is to know what is happening when we subscribe to the publisher.

Example code:

```java
Flux<User> fluxWithLog() {
   return repository
           .findAll()
           .log();
}
```
you will see an output like below:
```shell
09:39:56.503 [main] INFO  reactor.Flux.Zip.1 - onSubscribe(FluxZip.ZipCoordinator)
09:39:56.507 [main] INFO  reactor.Flux.Zip.1 - request(1)
09:39:56.631 [parallel-1] INFO  reactor.Flux.Zip.1 - onNext(Person{username='swhite', firstname='Skyler', lastname='White'})
09:39:56.631 [parallel-1] INFO  reactor.Flux.Zip.1 - request(1)
09:39:56.739 [parallel-1] INFO  reactor.Flux.Zip.1 - onNext(Person{username='jpinkman', firstname='Jesse', lastname='Pinkman'})
09:39:56.739 [parallel-1] INFO  reactor.Flux.Zip.1 - request(2)
09:39:56.832 [parallel-1] INFO  reactor.Flux.Zip.1 - onNext(Person{username='wwhite', firstname='Walter', lastname='White'})
09:39:56.941 [parallel-1] INFO  reactor.Flux.Zip.1 - onNext(Person{username='sgoodman', firstname='Mike', lastname='Goodman'})
09:39:56.942 [parallel-1] INFO  reactor.Flux.Zip.1 - onComplete()
```
## The `doOnError()` method
This operator is intended to be used when the pipeline throws an error and we want to consume that error, most commonly for logging purposes.

## The `onOperatorDebug()` method
This operator is probably the most powerful one we have at our disposition use when debugging reactive issues.
What it does is that **every single operator in the whole application** will log a detailed, in-order stacktrace of what led to a given failure so that we can understand precisely where the issue occurred, 
where the failure was and what operators and in which order they have been called before the error happened.

_Hooks.onOperatorDebug()_ operator enables a **recorder** that is capable of capturing _assembly stack trace_ information when the Publisher is 
created and also some very important information like the operator description, the chain order and so on.

The `onOperatorDebug` hook captures an assembly stack trace which results in **Performance Impact** so it is not recommended
to be used in production but there is [agent based approach](https://github.com/reactor/reactor-core/blob/59aa6a402d94dc9effed3931a028de1492ca88c9/reactor-tools/src/main/java/reactor/tools/agent/ReactorDebugAgent.java)
which is available as a part of **reactor core** for production use.

Example code:

```java
Flux<User> fluxWithLog() {
    Hooks.onOperatorDebug();
	return repository
	    	.findAll()
		    .log(); 
}
```

## The `checkpoint()` operator

Checkpoints operator keeps a check of the items passing through it; there are few variants of this operator as follows. 
It has a few flavours.

* Takes no arguments but forces to capture the assembly stack trace by default. It means we do pay high cost from the performance perspective. 
* Takes a description argument. It does not force the assembly to capture the stack trace.
* Delegate the control of the above arguments to the caller.

The second option is quite popular as it takes a description argument, that will be printed in case error occurred in upstream
of the checkpoint.

> The specialty of this operator is the ability to engage selectively compared to `onOperatorDebug()` Hook.
> Typically `checkpoint` operator is used when you want to find the origin point of an error signal in a selective way
> but with less computational power

Example code:
```java
    Flux<User> fluxWithLog() {
      Hooks.onOperatorDebug();
      return repository
           .findAll()
           .map(user -> {
               if (user.getUsername().equals("wwhite")) {
                  throw new RuntimeException("WrongUser!!");
               } 
               return user;
           })
          .checkpoint("after find all");
}
```

you will see stack trace like below:

```shell
	at java.lang.Thread.run(Thread.java:748)
	Suppressed: java.lang.RuntimeException: WrongUser!!
		at com.wellsfargo.cto.eai.reactor.Part06Request.lambda$fluxWithLog$0(Part06Request.java:47)
		Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException: 
Assembly trace from producer [reactor.core.publisher.FluxMapFuseable] :
	reactor.core.publisher.Flux.map(Flux.java:5936)
	com.wellsfargo.cto.eai.reactor.Part06Request.fluxWithLog(Part06Request.java:45)
Error has been observed at the following site(s):
	|_   Flux.map ⇢ at com.wellsfargo.cto.eai.reactor.Part06Request.fluxWithLog(Part06Request.java:45)
	|_ checkpoint ⇢ after find all
Stack trace:
			at com.wellsfargo.cto.eai.reactor.Part06Request.lambda$fluxWithLog$0(Part06Request.java:47)
			at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:107)
```
