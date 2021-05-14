+++
date ="2021-05-13T14:02:26-05:00"
title = "Open Tracing Custom Header Propagation"
summary = "Open Tracing Custom Header Propagation"
tags = ["application development", "logging", "open telemetry", "sleuth", "spring-cloud"]
weight = 8
+++

## Context
Please refer to [recipe](/best-practices/spring-cloud-sleuth-otel-tracing/) to set up `spring-cloud-sleuth-otel`.

Spring Cloud Sleuth otel provides `spring.sleuth.propagation.type` property to customize the tracing headers based on 
requirements. For OpenTelemetry Sleuth suports `AWS`, `B3`, `JAEGER` , `W3C`, `OT_TRACER`, `W3C` propagation types.

## How To Change the Context Propagation Mechanism:

To provide a custom propagation mechanism set the `spring.sleuth.propagation.type` property to `CUSTOM` and implement
your own bean (`TextMapPropagator` for **OpenTelemetry**).

Create a class like below:

```java
@Component
public class CustomPropagator implements TextMapPropagator {

    @Override
    public List<String> fields() {
        return Arrays.asList("customTraceId", "customSpanId", "customHeader");

    }

    @Override
    public <C> void inject(Context context, C carrier, TextMapSetter<C> setter) {
        SpanContext spanContext = Span.fromContext(context).getSpanContext();
        if (!spanContext.isValid()) {
            return;
        }
        String customHeaders = "host-name-customer-service";
        setter.set(carrier, "customTraceId", spanContext.getTraceId());
        setter.set(carrier, "customSpanId", spanContext.getSpanId());
        setter.set(carrier, "customHeader", customHeaders);
    }

    @Override
    public <C> Context extract(Context context, C carrier, TextMapGetter<C> getter) {
        String traceParent = getter.get(carrier, "customTraceId");
        if (traceParent == null) {
            return Span.getInvalid().storeInContext(context);
        }
        String spanId = getter.get(carrier, "customSpanId");
        return Span.wrap(SpanContext.createFromRemoteParent(traceParent, spanId, TraceFlags.getSampled(),
                TraceState.builder().build())).storeInContext(context);
    }

}
```

## Running Application
Use a rest client to send **GET** request `/customers`
```shell
2021-05-13 11:08:07.123  INFO [customer-service,,] 25944 --- [nio-8080-exec-1] c.w.s.o.starter.trace.CustomPropagator   : custom header:host-name-customer-service
2020-10-21 12:01:16.285  INFO [customer-service,0b6aaf642574edd3,0b6aaf642574edd3,true] 289589 --- [nio-9000-exec-1] CustomerController : retrieve all customers!
```
You will see the portion of the log statement that Sleuth adds is [customer-service,0b6aaf642574edd3,0b6aaf642574edd3,false]
and the traceId, spanId is using Custom Propagation Technique.
* The first part is the application name (spring.application.name=customer-service in **bootstrap.yml**). 
* The second value is the trace id. 
* The third value is the span id. 
