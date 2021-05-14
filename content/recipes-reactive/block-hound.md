+++
categories = ["recipes"]
tags = ["application development", "blocking","reactor","spring webflux"]
summary = "Detect blocking calls in Reactive code"
title = "Detect blocking calls in Reactive code"
date ="2021-05-12T14:02:26-05:00"
weight = 8
+++

## CONTEXT

One of the most common concerns in reactive-programming is to spot blocking calls in reactive code or replace them with a reactive alternative when possible.
It’s really easy to miss some of these blocking operations and end up shipping to production something that shouldn’t be considered production-ready at all: **scheduling blocking calls inside non-blocking threads can lead to unpredictable behaviour, as well**!
To avoid this, Java agent to detect blocking calls from non-blocking threads [**BlockHound**](https://github.com/reactor/BlockHound) to detect blocking calls in reactive code.

### How does it work?

**BlockHound** is _java agent_ that gets loaded by the JVM before invoking the _main_ entry point of your application and relies on the JVM Instrumentation API to orchestrate byte-code manipulation on classes and redefine their behaviour.
Once BlockHound gets bootstrapped, it marks a set of blocking methods that need to be altered (eg.`Thread.sleep()`) and changes their behaviour so they’ll end up throwing an Error in case they get invoked from within a thread that’s marked as _Non-Blocking_.


### Prerequisite

Add **_BlockHound JUnit Platform_** dependency to build.gradle.

```groovy
implementation group: 'io.projectreactor.tools', name: 'blockhound-junit-platform', version: '1.0.6.RELEASE'
```

### Solution

We will create _BlockingService_ class and add few methods such as `blockingIsAllowed`, `blockingIsNotAllowed`
and test whether _BlockHound_ detects blocking calls or not by creating unit test cases.

## Create Service

Create Blocking Service class and add `blockingIsAllowed` and `blockedIsNotAllowed` method like below:

```java
@Service
public class BlockingService {

    public Mono<Integer> blockingIsAllowed() {
        return getBlockingMono().subscribeOn(Schedulers.elastic());
    }

    public Mono<Integer> blockingIsNotAllowed() {
        return getBlockingMono().subscribeOn(Schedulers.parallel());
    }

    private Mono<Integer> getBlockingMono() {
        return Mono.just(1).doOnNext(i -> block());
    }

    private void block() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```
The `block()` method has a `Thread.sleep(1000)` which is blocking call.

## Create Junit Tests

Create Junit Test Class `BlockingServiceTest` and add test case to test
methods created above:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT, classes = GreenfieldReactiveApplication.class)
public class BlockingServiceTest {

    @Autowired
    private BlockingService blockingService;

    @Test
    void blockingInNonBlockingThreadsShouldNotBeAllowed() {
        StepVerifier
                .create(blockingService.blockingIsNotAllowed())
                .expectErrorMatches(e -> {
                    e.printStackTrace();

                    return e instanceof Error &&
                            e.getMessage().contains("Blocking call!");
                })
                .verify();
    }

    @Test
    void blockingInBlockingThreadsShouldBeAllowed() {
        StepVerifier
                .create(blockingService.blockingIsAllowed())
                .expectNext(1)
                .verifyComplete();
    }
}
```

The **first test method** verifies that blocking code is not allowed when
executed from within a Non-Blocking thread belonging to `Schedulers.parallel()` thread
pool.

The **second test method** verifies that blocking code is allowed when executed
from within a regular thread belonging to `Schedulers.elastic()` thread pool

## Run Tests:

Run tests using `gradle clean build`, you will notice tests pass!
You should also see a stack trace like one below showing **BlockHound**
detected a blocking call and turned it into an Error-Throwing call.

```shell
reactor.blockhound.BlockingOperationError: Blocking call! java.lang.Thread.sleep
	at java.lang.Thread.sleep(Thread.java)
	at com.wellsfargo.reactive.starter.greenfieldreactiveapplicationstarter.service.BlockingService.block(BlockingService.java:24)
	at com.wellsfargo.reactive.starter.greenfieldreactiveapplicationstarter.service.BlockingService.lambda$getBlockingMono$0(BlockingService.java:19)
	at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.onNext(FluxPeekFuseable.java:189)
	at reactor.core.publisher.Operators$ScalarSubscription.request(Operators.java:2344)
	at reactor.core.publisher.FluxPeekFuseable$PeekFuseableSubscriber.request(FluxPeekFuseable.java:137)
	at reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.trySchedule(MonoSubscribeOn.java:187)
	at reactor.core.publisher.MonoSubscribeOn$SubscribeOnSubscriber.onSubscribe(MonoSubscribeOn.java:132)
```

_Under the hood_ **BlockHound** sets up a Junit `TestExecutionListener` which invokes
`BlockHound.install()`.

The source code is available at: [WF GitHub](https://github.wellsfargo.com/app-ebst/wf-reactive-project-starter)


