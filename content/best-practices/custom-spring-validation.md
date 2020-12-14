+++
categories = ["recipes"]
tags = ["practices", "spring boot", "microservice", "custom spring bean validation"]
title = "Custom Spring Bean Validation"
description = "A guide to use custom validation"
date = 2020-12-09
weight = 10
draft = false
authors = ["Rohan Mukesh"]
+++

## Context

While implementing Spring REST endpoints for Spring boot applications, adding validations (inbuilt/custom) becomes inevitable. For most cases the inbuilt validators provided by JSR 380, also known as Bean Validation 2.0 framework would suffice. Some of the inbuilt validators provided are: @NotNull, @NotEmpty, @NotBlank, @Min, @Max, @Size to name a few. There are still instances where the validation need can’t be taken care of by the inbuilt validators provided by JSR 380 and in such cases we need to write custom validators which takes care of providing custom validation logic to the bean attributes.
    
## Use Case:

Let’s assume a use case wherein we need to validate customer location details with custom validation of fields locationId, countryCode and postCode. These three fields should accept *only numeric strings* (ex: “123")

   ```java
    @Getter
    @Setter
    public class CustomerLocation {
        @NumericString(message = "locationId should be numeric")
        private String locationId;
        @NotBlank(message = "city cannot not be empty")
        private String city;
        @NumericString(message = "countryCode should be numeric")
        private String countryCode;
        @NumericString(message = "postCode should be numeric")
        private String postCode;
     }
   ```
In the above example both inbuilt (@NotBlank) and custom (@NumericString) validators are being used. @NotBlank would ensure that the value passed to city attribute is not blank however @NumericString validator would ensure that the value passed to locationId, countryCode & postCode is *a numeric string*

## Setup:
 
Add below dependency to build.gradle. The latest dependency can be checked [here](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-validator%22)
 ```
    compile group: 'org.hibernate', name: 'hibernate-validator', version: '4.2.0.Final'
 ```
If we're using Spring Boot, then we can add only the _spring-boot-starter-web_, which will bring in the hibernate-validator dependency also.

## **Controller:**

Lets see the REST endpoint which validates the incoming request:

   ```java
      @RestController
      public class ValidatorController {
          @PostMapping(value="/v1/validate", produces = "application/json")
          public ResponseEntity<String> validateCustomerLocation(@ValidCustomerLocation @RequestBody CustomerLocation customerLocation){
              return new ResponseEntity<>(HttpStatus.ACCEPTED);
          }
      }
   ```
## Custom validators:

ValidCustomerLocation is a custom validator annotation for which the constraints would be validated by CustomerLocationValidator class as shown below:

   ```java
      @Target({ElementType.FIELD, ElementType.PARAMETER})
      @Retention(RetentionPolicy.RUNTIME)
      @Constraint(validatedBy = {CustomerLocationValidator.class})
      @Documented
      public @interface ValidCustomerLocation {
      String message() default "Invalid customer location";
          Class<?>[] groups() default {};
          Class<? extends Payload>[] payload() default {};
      }
   ```
The implementation of CustomerLocationValidator would override the isValid() method and check if the input is valid or not.

   ```java
      public class CustomerLocationValidator implements ConstraintValidator<ValidCustomerLocation, CustomerLocation> {
      @Autowired
      Validator validator;
    
      @Override
      public boolean isValid(CustomerLocation customerLocation, ConstraintValidatorContext constraintValidatorContext) {
          boolean isValid = true;
          Set<ConstraintViolation<CustomerLocation>> constraintViolations = new HashSet();
          constraintViolations =
                  validator.validate(customerLocation);
          if (!CollectionUtils.isEmpty(constraintViolations)) {
              constraintValidatorContext.disableDefaultConstraintViolation();
              for (ConstraintViolation<CustomerLocation> violation : constraintViolations) {
                  constraintValidatorContext
                          .buildConstraintViolationWithTemplate(violation.getMessageTemplate())
                          .addConstraintViolation();
              }
              isValid = false;
          }
          return isValid;
      }
    }
   ```
Attributes with @NumericString annotation would be validated by NumericStringValidator class as shown below:

 ```java
    @Target({ElementType.FIELD, ElementType.PARAMETER})
    @Retention(RetentionPolicy.RUNTIME)
    @Constraint(validatedBy = {NumericStringValidator.class})
    @Documented
    public @interface NumericString {
        String message() default "String should be numeric";
    
        Class<?>[] groups() default {};
    
        Class<? extends Payload>[] payload() default {};
    }
 ```
The implementation of NumericStringValidator would override the isValid() method and check if the attributes annotated with @NumericString contains only numerals using the regular expression check:

 ```java
    public class NumericStringValidator implements ConstraintValidator<NumericString, String> {
        @Override
        public boolean isValid(String str, ConstraintValidatorContext constraintValidatorContext) {
            if (str.matches("[0-9]+")) return true;
            return false;
        }
    }
```
## Time to test:

Complete code base is present at the below git location:

[https://github.com/rohanmukesh/spring-boot-custom-validator.git](https://github.com/rohanmukesh/spring-custom-validator.git)

Clone the codebase, build and run the CustomvalidatorApplication class. The application runs on default port 8080. Once it is up and running perform the below two tests:

### 1. Valid request:

Endpoint URL: localhost:8080/v1/validate
  ```json    
      {
       "locationId":"123",
       "city": "Frisco",
       "countryCode":"2",
       "postCode":"000000"
     }
 ```
**Response**:

Status: HTTP response code 202 Accepted

### 2. Invalid request:

Endpoint URL: localhost:8080/v1/validate

 ```json
     {
      "locationId":"locationId",
      "countryCode":"countryCode",
      "postCode":"postCode"
     }
 ```   

### Response:

Status: HTTP response code 400 Bad Request
 ```json
    {
     "errorCode": "400 BAD_REQUEST",
     "errorMessage": "Validation Errors",
     "subErrors": [
     {
     "object": "customerLocation",
     "field": "city",
     "rejectedValue": "city",
     "message": "city cannot not be empty"
     },
     {
     "object": "customerLocation",
     "field": "countryCode",
     "rejectedValue": "countryCode",
     "message": "countryCode should be numeric"
     },
     {
     "object": "customerLocation",
     "field": "postCode",
     "rejectedValue": "postCode",
     "message": "postCode should be numeric"
     },
     {
     "object": "customerLocation",
     "field": "locationId",
     "rejectedValue": "locationId",
     "message": "locationId should be numeric"
     }
     ]
    }
```
