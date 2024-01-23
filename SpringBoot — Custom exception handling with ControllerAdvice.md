---
title: SpringBoot — Custom exception handling with @ControllerAdvice
date: 2024-01-23 00:00:00 Z
categories:
- SpringBoot
tags:
- SpringBoot
layout: article
cover: /assets/logo/springboot.png
sharing: true
license: false
aside:
  toc: true
pageview: true
---



Before going further, Let's see how we do validation in spring boot.
![image](https://github.com/smlcodes/OnePageTutorials/assets/20472904/1322ec20-992d-4b88-9035-a23357c78a88)


In Spring Boot, you can perform validation using the Java Bean Validation framework, which is based on the Java Validation API (JSR-380) and the javax.validation package

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```


To perform validation, we need to annotate the fields of the bean/dto class with validation annotations from the javax.validation.constraints package. For example, you can use @NotNull, @Size, @Email, and more to specify validation constraints.

```java
@Data
 @JsonIgnoreProperties(ignoreUnknown = true)
 @NoArgsConstructor
 @AllArgsConstructor
 @Builder
 @XmlRootElement
 @XmlAccessorType(XmlAccessType.FIELD)
 public class EmployeeDto {

 @XmlElement(required = true)
 private Long id;

 @XmlElement(required = true)
 @NotNull
 @Size(min = 4, max = 10)
 private String name;

 @XmlElement(name = "salary", required = true)
 @Digits(integer = 10, fraction = 2)
 private Double salary;

 @XmlElement
 @Size(max = 20)
 private String city;

 @XmlElement(required = true)
 private AccountDto account;
 }

```


To perform validation whenever user submits an employee payload to API call, we need to annotate @Valid annotation against the DTO parameter like below.


```java
@ApiOperation("Create a new Employee or update exist ")
 @PostMapping
 public EmployeeDto save(@RequestBody @Valid EmployeeDto employeeDto) {
   return employeeService.save(employeeDto, id);
 }

```

Ok, let's run & check the validation messages.

Payload:

```
{
  "name": "HYDGHAGHGHAHGAHGAHGAHGHYDGHAGHGHAHGAHGAHGAHG",
  "salary": 25000,
  "city": "HYDGHAGHGHAHGAHGAHGAHG",
  "account": {
    "email": "satya@gmail.com",
    "password": "password",
    "dob": "1990-10-27"
  }

```

Response:

```
{
    "exception": "org.springframework.web.bind.MethodArgumentNotValidException",
    "fingerprint": "036d809a-175d-4db1-ba77-9c5258c1e244",
    "errors": [
        {
            "code": "javax.validation.constraints.Size.message",
            "arguments": {
                "min": 0,
                "max": 20,
                "invalid": "HYDGHAGHGHAHGAHGAHGAHG",
                "property": "city"
            },
            "message": "size must be between 0 and 20"
        }
    ],
    "status": "Bad Request"
}

```

Okay, city allows max of 10 chars, buts it's more than that --- and we got the error.

But, This is NOT READABLE, the end user doesn't want these technical details. Just wants what's the field & what he need to pass.

This we can achieve by implementing Custom exception handling with @ControllerAdvice

Custom exception handling with @ControllerAdvice
================================================

@ControllerAdvice is an annotation in Spring Boot that allows you to define global exception handling for your controllers. You can use it to handle exceptions that occur in multiple controllers or to provide a centralized place for handling exceptions throughout your application.

1. Create a custom exception class. I want to display HTTP Status with all the error messages.

```java
public class ErrorDetails {

 private HttpStatus code;

 private List<String> message;

 }
```


2. Create a CustomExceptionHandler class & annotate with @ControllerAdvice. It contains methods for handling specific exceptions. You can specify the exception types to handle in the @ExceptionHandler annotation on each method.

Iam also extending ResponseEntityExceptionHandler & overriding handleMethodArgumentNotValid(). This is useful in the case of Rest API, because It provides a way to handle exceptions and customize the HTTP response entity that is returned to the client.

```java

@ControllerAdvice
public class CustomExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public final ResponseEntity<Object> handleDBEntityNotFound(EntityNotFoundException ex, WebRequest webRequest) {
        ErrorDetails error = new ErrorDetails();
        error.setCode(HttpStatus.UNPROCESSABLE_ENTITY);
        List<String> message = new ArrayList<>();
        message.add(ex.getMessage());
        error.setMessage(message);
        return new ResponseEntity<Object>(error, error.getCode());
    }

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatus status, WebRequest request) {
        ErrorDetails error = new ErrorDetails();
        error.setCode(HttpStatus.BAD_REQUEST);
        List<String> message = new ArrayList<String>();

        List<String> collect = ex.getBindingResult().getFieldErrors().stream().filter(Objects::nonNull)
                .map(m -> (m.getField() + " " + m.getDefaultMessage())).collect(Collectors.toList());
        message.addAll(collect);
        error.setMessage(message);
        return new ResponseEntity<Object>(error, HttpStatus.BAD_REQUEST);
    }
}

```

That's it now run the application again with the same payload.

```
{
    "code": "BAD_REQUEST",
    "message": [
        "city size must be between 0 and 20",
        "name size must be between 4 and 10"
    ]
}

```

That's all on a high level.
