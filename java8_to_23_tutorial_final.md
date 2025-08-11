# Java 8 to Java 23: Complete Feature Guide

## Table of Contents
- [Java 8 Features](#java-8-features)
- [Java 9 Features](#java-9-features)
- [Java 10 Features](#java-10-features)
- [Java 11 Features](#java-11-features)
- [Java 12 Features](#java-12-features)
- [Java 13 Features](#java-13-features)
- [Java 14 Features](#java-14-features)
- [Java 15 Features](#java-15-features)
- [Java 16 Features](#java-16-features)
- [Java 17 Features](#java-17-features)
- [Java 18 Features](#java-18-features)
- [Java 19 Features](#java-19-features)
- [Java 20 Features](#java-20-features)
- [Java 21 Features](#java-21-features)
- [Java 22 Features](#java-22-features)
- [Java 23 Features](#java-23-features)

---

## Java 8 Features

### 1. Lambda Expressions

Lambda expressions are a fundamental feature in Java 8 that enables functional programming by allowing you to treat functionality as a method argument. They provide a clear and concise way to represent one method interface using an expression. Lambda expressions are particularly useful for implementing functional interfaces (interfaces with a single abstract method) and can significantly reduce boilerplate code.

The syntax of lambda expressions follows the pattern `(parameters) -> expression` or `(parameters) -> { statements; }`. This feature makes Java code more readable and expressive, especially when working with collections and functional interfaces. Lambda expressions are the foundation for many other Java 8 features like Stream API and method references.

```java
// Traditional approach
Runnable oldWay = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda expression
Runnable newWay = () -> System.out.println("Hello World");

// With parameters
List<String> names = Arrays.asList("John", "Jane", "Bob");
names.forEach(name -> System.out.println(name));
```

### 2. Stream API

The Stream API is one of the most powerful additions to Java 8, providing a functional approach to processing collections of objects. Streams allow you to process data in a declarative way, similar to SQL statements. They support functional-style operations on streams of elements, such as map-reduce transformations on collections.

Streams are not data structures but rather wrappers around data sources like collections, arrays, or I/O channels. They provide a high-level abstraction for processing sequences of data in parallel or sequentially. The Stream API promotes a more functional programming style and can lead to more readable and maintainable code.

#### Filter Operation
The filter operation allows you to select elements from a stream based on a predicate (boolean-valued function). It's used to create a new stream containing only elements that match the specified condition.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob", "Alice");
List<String> filteredNames = names.stream()
    .filter(name -> name.startsWith("J"))
    .collect(Collectors.toList());
// Result: ["John", "Jane"]
```

#### Map Operation
The map operation transforms each element of the stream by applying a function to it. It's used to convert elements from one form to another, creating a new stream with the transformed elements.

```java
List<String> names = Arrays.asList("john", "jane", "bob");
List<String> upperCaseNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Result: ["JOHN", "JANE", "BOB"]
```

#### Collect Operation
The collect operation is a terminal operation that transforms the elements of the stream into a different kind of result, such as a List, Set, or Map. It uses collectors to specify how the elements should be accumulated.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");
String joinedNames = names.stream()
    .collect(Collectors.joining(", "));
// Result: "John, Jane, Bob"
```

#### Parallel Streams
Parallel streams allow you to process elements in parallel, leveraging multiple CPU cores for better performance on large datasets. They're created using the `parallelStream()` method or by calling `parallel()` on an existing stream.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
int sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();
// Result: 30 (2+4+6+8+10)
```

### 3. Method References

Method references provide a way to refer to methods without executing them. They're a shorthand notation of lambda expressions to call a method. Method references make code more readable and concise when the lambda expression is just calling an existing method.

There are four types of method references: reference to a static method, reference to an instance method of a particular object, reference to an instance method of an arbitrary object, and reference to a constructor. They use the `::` operator to separate the class or instance from the method name.

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

// Lambda expression
names.forEach(name -> System.out.println(name));

// Method reference
names.forEach(System.out::println);

// Static method reference
List<Integer> numbers = Arrays.asList("1", "2", "3").stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
```

### 4. Functional Interfaces

Functional interfaces are interfaces that have exactly one abstract method. They serve as the target types for lambda expressions and method references. Java 8 introduced the `@FunctionalInterface` annotation to mark interfaces as functional interfaces and ensure they maintain the single abstract method constraint.

The java.util.function package provides several built-in functional interfaces like Predicate, Function, Consumer, and Supplier. These interfaces cover most common use cases and help standardize functional programming patterns in Java applications.

#### Predicate Interface
Predicate represents a boolean-valued function of one argument. It's commonly used for filtering operations and contains methods like `test()`, `and()`, `or()`, and `negate()`.

```java
Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();

List<String> strings = Arrays.asList("", "hello", "", "world");
List<String> nonEmptyStrings = strings.stream()
    .filter(isNotEmpty)
    .collect(Collectors.toList());
```

#### Function Interface
Function represents a function that accepts one argument and produces a result. It contains methods like `apply()`, `compose()`, and `andThen()` for function composition.

```java
Function<String, Integer> stringLength = String::length;
Function<Integer, Integer> doubleValue = x -> x * 2;

List<String> words = Arrays.asList("hello", "world");
List<Integer> doubledLengths = words.stream()
    .map(stringLength.andThen(doubleValue))
    .collect(Collectors.toList());
```

#### Consumer Interface
Consumer represents an operation that accepts a single input argument and returns no result. It's used for operations that perform side effects like printing or modifying objects.

```java
Consumer<String> printer = System.out::println;
Consumer<String> upperCasePrinter = s -> System.out.println(s.toUpperCase());

List<String> names = Arrays.asList("john", "jane");
names.forEach(printer.andThen(upperCasePrinter));
```

#### Supplier Interface
Supplier represents a supplier of results. It doesn't take any arguments but produces a value. It's often used for lazy evaluation and factory methods.

```java
Supplier<String> randomString = () -> UUID.randomUUID().toString();
Supplier<List<String>> emptyList = ArrayList::new;

String id = randomString.get();
List<String> list = emptyList.get();
```

### 5. Optional Class

Optional is a container class that may or may not contain a non-null value. It was introduced to reduce the number of NullPointerExceptions in Java applications. Optional provides methods to check for the presence of values and to handle absent values gracefully.

Optional encourages better programming practices by making null-checks explicit and providing functional-style methods for handling potentially absent values. It's particularly useful in Stream operations and as return types for methods that might not always return a value.

```java
Optional<String> optional = Optional.of("Hello");
Optional<String> emptyOptional = Optional.empty();

// Check if value is present
if (optional.isPresent()) {
    System.out.println(optional.get());
}

// Functional approach
optional.ifPresent(System.out::println);
String result = optional.orElse("Default Value");
```

### 6. Default Methods in Interfaces

Default methods allow interfaces to have method implementations without breaking existing implementations. They enable interface evolution by allowing new methods to be added to interfaces without forcing all implementing classes to provide implementations.

Default methods use the `default` keyword and provide a default implementation that implementing classes can choose to override. This feature was crucial for adding new methods to existing interfaces like Collection and List in Java 8 without breaking backward compatibility.

```java
interface Vehicle {
    default void start() {
        System.out.println("Vehicle is starting...");
    }
    
    void stop(); // Abstract method
}

class Car implements Vehicle {
    @Override
    public void stop() {
        System.out.println("Car is stopping...");
    }
    // start() method is inherited from interface
}
```

### 7. Date and Time API (java.time)

The new Date and Time API in Java 8 addresses the shortcomings of the old java.util.Date and java.util.Calendar classes. The new API is immutable, thread-safe, and follows the ISO-8601 standard for date and time representation.

The API is designed around the concept of different temporal types like LocalDate, LocalTime, LocalDateTime, ZonedDateTime, and Instant. Each class serves a specific purpose and provides methods for date/time manipulation, formatting, and parsing.

#### LocalDate, LocalTime, LocalDateTime
These classes represent date and time without timezone information. LocalDate represents just the date, LocalTime represents just the time, and LocalDateTime combines both.

```java
LocalDate today = LocalDate.now();
LocalTime currentTime = LocalTime.now();
LocalDateTime now = LocalDateTime.now();

LocalDate specificDate = LocalDate.of(2023, Month.DECEMBER, 25);
LocalDateTime meeting = LocalDateTime.of(2023, 12, 25, 14, 30);
```

#### ZonedDateTime and Instant
ZonedDateTime represents date and time with timezone information, while Instant represents a specific moment in time (Unix timestamp equivalent).

```java
ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of("America/New_York"));
Instant timestamp = Instant.now();

ZonedDateTime utc = ZonedDateTime.now(ZoneOffset.UTC);
Instant fromEpoch = Instant.ofEpochSecond(1640995200);
```

---

## Java 9 Features

### 1. Module System (Project Jigsaw)

The Module System is the most significant feature in Java 9, introducing a new level of abstraction above packages. Modules help organize code better, provide stronger encapsulation, and enable more reliable configuration. They address issues like classpath hell and provide better security by explicitly declaring dependencies and exports.

Modules are defined using module-info.java files that specify module dependencies, exported packages, and service requirements. This system allows for creating smaller, more focused applications and better dependency management. The JDK itself has been modularized, allowing for smaller runtime images.

```java
// module-info.java
module com.example.myapp {
    requires java.base;
    requires java.logging;
    
    exports com.example.myapp.api;
    
    provides com.example.myapp.spi.ServiceProvider
        with com.example.myapp.impl.ServiceProviderImpl;
}
```

### 2. JShell (REPL)

JShell is Java's first official Read-Eval-Print Loop (REPL) tool that allows interactive Java programming. It enables developers to quickly test Java code snippets, experiment with APIs, and prototype solutions without creating full Java applications.

JShell is particularly useful for learning Java, testing code snippets, and exploring new APIs. It supports tab completion, command history, and various commands for managing the REPL environment. This tool significantly improves the developer experience for quick experimentation and learning.

```bash
jshell> int x = 10
x ==> 10

jshell> String greeting = "Hello, World!"
greeting ==> "Hello, World!"

jshell> System.out.println(greeting)
Hello, World!
```

### 3. Improved Stream API

Java 9 enhanced the Stream API with new methods that provide more functionality and convenience. These additions make stream processing more powerful and expressive, allowing for more complex data processing scenarios.

The new methods include takeWhile(), dropWhile(), ofNullable(), and iterate() with a predicate. These methods fill gaps in the original Stream API and provide more control over stream processing.

#### takeWhile() and dropWhile()
These methods provide conditional processing of stream elements. takeWhile() takes elements while a condition is true, and dropWhile() skips elements while a condition is true.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

List<Integer> taken = numbers.stream()
    .takeWhile(n -> n < 5)
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4]

List<Integer> dropped = numbers.stream()
    .dropWhile(n -> n < 5)
    .collect(Collectors.toList());
// Result: [5, 6, 7, 8]
```

#### Stream.ofNullable()
This method creates a stream with a single element if the element is not null, or an empty stream if it is null. It's useful for handling potentially null values in stream operations.

```java
String nullableString = null;
String validString = "Hello";

long count1 = Stream.ofNullable(nullableString).count(); // 0
long count2 = Stream.ofNullable(validString).count();    // 1

List<String> result = Stream.of("a", null, "b")
    .flatMap(Stream::ofNullable)
    .collect(Collectors.toList());
// Result: ["a", "b"]
```

### 4. Private Methods in Interfaces

Java 9 allows private methods in interfaces, which helps in code reusability within interfaces. Private methods can be used to share code between default methods without exposing implementation details to implementing classes.

This feature enables better code organization within interfaces and reduces code duplication. Private methods can be static or instance methods, providing flexibility in implementation design.

```java
interface Calculator {
    default int add(int a, int b) {
        return calculate(a, b, Integer::sum);
    }
    
    default int multiply(int a, int b) {
        return calculate(a, b, (x, y) -> x * y);
    }
    
    private int calculate(int a, int b, BinaryOperator<Integer> operation) {
        // Common logic shared between default methods
        return operation.apply(a, b);
    }
}
```

### 5. Factory Methods for Collections

Java 9 introduced convenient factory methods for creating immutable collections. These methods provide a clean and concise way to create List, Set, and Map instances without the need for builders or utility methods.

The factory methods create immutable collections by default, which promotes better programming practices and thread safety. They're particularly useful for creating small collections with known elements at compile time.

```java
// List factory methods
List<String> immutableList = List.of("apple", "banana", "cherry");

// Set factory methods
Set<Integer> immutableSet = Set.of(1, 2, 3, 4, 5);

// Map factory methods
Map<String, Integer> immutableMap = Map.of(
    "one", 1,
    "two", 2,
    "three", 3
);
```

---

## Java 10 Features

### 1. Local Variable Type Inference (var keyword)

The `var` keyword allows local variable type inference, reducing verbosity in variable declarations. The compiler infers the type based on the initializer expression. This feature is only available for local variables, not for method parameters, return types, or fields.

Type inference makes code more readable and maintainable, especially when dealing with complex generic types. However, it should be used judiciously to maintain code readability and clarity.

```java
// Traditional way
List<String> names = new ArrayList<String>();
Map<String, List<Integer>> complexMap = new HashMap<String, List<Integer>>();

// With var
var names = new ArrayList<String>();
var complexMap = new HashMap<String, List<Integer>>();
var stream = names.stream().filter(s -> s.length() > 3);
```

### 2. Application Class-Data Sharing (CDS)

Application Class-Data Sharing extends the existing Class-Data Sharing feature to allow application classes to be placed in the shared archive. This improves startup time and reduces memory footprint by sharing common class metadata across Java processes.

CDS creates a shared archive of class metadata that can be memory-mapped at runtime, reducing the overhead of loading and parsing class files. This is particularly beneficial for applications with large classpaths or when running multiple JVM instances.

```bash
# Create CDS archive
java -Xshare:dump

# Use CDS archive
java -Xshare:on MyApplication
```

### 3. Garbage Collector Interface

Java 10 introduced a clean garbage collector interface that improves the source code isolation of different garbage collectors. This makes it easier to add new garbage collectors and maintain existing ones.

The interface provides a better abstraction for GC implementations and enables easier experimentation with new garbage collection algorithms. This change primarily benefits JVM developers and doesn't directly impact application developers.

### 4. Heap Allocation on Alternative Memory Devices

This feature enables the HotSpot VM to allocate the Java object heap on alternative memory devices, such as non-volatile dual in-line memory modules (NV-DIMMs). This allows for larger heap sizes and different memory characteristics.

The feature is particularly useful for applications that require large amounts of memory or want to take advantage of persistent memory technologies. It provides better memory utilization options for specialized hardware configurations.

---

## Java 11 Features

### 1. HTTP Client API

Java 11 standardized the HTTP Client API that was introduced as an incubator feature in Java 9. This new API provides a modern, efficient way to make HTTP requests, supporting both synchronous and asynchronous operations.

The HTTP Client API supports HTTP/1.1 and HTTP/2 protocols, provides builder patterns for request construction, and integrates well with the Stream API and CompletableFuture for reactive programming. It's a significant improvement over the legacy HttpURLConnection.

#### Synchronous HTTP Requests
Synchronous requests block the calling thread until the response is received. They're simpler to use but can impact application performance if not used carefully.

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.github.com/users/octocat"))
    .GET()
    .build();

HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```

#### Asynchronous HTTP Requests
Asynchronous requests return immediately with a CompletableFuture, allowing for non-blocking operations and better resource utilization.

```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.github.com/users/octocat"))
    .GET()
    .build();

CompletableFuture<HttpResponse<String>> future = client.sendAsync(request, 
    HttpResponse.BodyHandlers.ofString());

future.thenApply(HttpResponse::body)
      .thenAccept(System.out::println);
```

### 2. String Methods

Java 11 introduced several new utility methods to the String class that make common string operations more convenient and expressive. These methods reduce the need for external libraries and provide more readable code.

The new methods include isBlank(), lines(), strip(), stripLeading(), stripTrailing(), and repeat(). These methods handle common string manipulation tasks and integrate well with the Stream API.

```java
String text = "  Hello World  ";
String multiline = "Line 1\nLine 2\nLine 3";

// New string methods
boolean isEmpty = text.isBlank();           // false
String stripped = text.strip();            // "Hello World"
String repeated = "Hi".repeat(3);          // "HiHiHi"

// Process lines as stream
multiline.lines()
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

### 3. Files Methods

Java 11 added new convenience methods to the Files class for reading and writing files. These methods simplify common file operations and provide more concise ways to work with file content.

The new methods include readString() and writeString(), which handle the common task of reading/writing entire files as strings. They automatically handle character encoding and resource management.

```java
// Write string to file
Path path = Paths.get("example.txt");
Files.writeString(path, "Hello, World!");

// Read string from file
String content = Files.readString(path);
System.out.println(content);

// Write with specific charset
Files.writeString(path, "Hello, 世界!", StandardCharsets.UTF_8);
```

### 4. Local Variable Type Inference for Lambda Parameters

Java 11 extended the var keyword to lambda parameters, allowing for more consistent usage of type inference. This is particularly useful when you need to add annotations to lambda parameters.

While this might seem like a minor enhancement, it provides consistency with local variable type inference and enables the use of annotations on lambda parameters without explicitly specifying types.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Using var in lambda parameters
names.stream()
    .map((@NonNull var name) -> name.toUpperCase())
    .forEach(System.out::println);

// Consistent with local variable inference
BiFunction<String, String, String> concatenator = 
    (var first, var second) -> first + second;
```

### 5. Nest-Based Access Control

Java 11 introduced nest-based access control, which formalizes the concept of nested classes at the JVM level. This feature allows nested classes to access each other's private members without the need for synthetic bridge methods.

This change improves performance by eliminating synthetic methods and provides better reflection support for nested classes. It also makes the relationship between nested classes more explicit at the bytecode level.

---

## Java 12 Features

### 1. Switch Expressions (Preview)

Java 12 introduced switch expressions as a preview feature, providing a more concise and expressive way to write switch statements. Switch expressions can return values and use arrow syntax for cleaner code.

Switch expressions eliminate the need for break statements in many cases and reduce the possibility of fall-through bugs. They support both traditional colon syntax and new arrow syntax, making them more flexible and expressive.

```java
// Traditional switch statement
String result;
switch (dayOfWeek) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        result = "Weekday";
        break;
    case SATURDAY:
    case SUNDAY:
        result = "Weekend";
        break;
    default:
        result = "Unknown";
}

// Switch expression (Java 12)
String result = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Unknown";
};
```

### 2. Compact Number Formatting

Java 12 introduced the CompactNumberFormat class for formatting numbers in a compact, human-readable form. This is particularly useful for displaying large numbers in user interfaces where space is limited.

The compact format represents numbers using abbreviations like "1K" for 1,000 or "1M" for 1,000,000. It supports different styles (short and long) and locales, making it suitable for internationalized applications.

```java
NumberFormat shortFormat = NumberFormat.getCompactNumberInstance(
    Locale.US, NumberFormat.Style.SHORT);
NumberFormat longFormat = NumberFormat.getCompactNumberInstance(
    Locale.US, NumberFormat.Style.LONG);

System.out.println(shortFormat.format(1500));     // "2K"
System.out.println(longFormat.format(1500000));   // "2 million"
System.out.println(shortFormat.format(1234567));  // "1M"
```

---

## Java 13 Features

### 1. Text Blocks (Preview)

Text blocks, introduced as a preview feature in Java 13, provide a cleaner way to write multi-line strings. They eliminate the need for concatenation and escape sequences when dealing with formatted text like JSON, XML, or SQL queries.

Text blocks use triple quotes (""") as delimiters and preserve the formatting of the enclosed text. They automatically handle common indentation and provide better readability for multi-line string literals.

```java
// Traditional string concatenation
String json = "{\n" +
              "    \"name\": \"John\",\n" +
              "    \"age\": 30,\n" +
              "    \"city\": \"New York\"\n" +
              "}";

// Text block (Java 13)
String json = """
              {
                  "name": "John",
                  "age": 30,
                  "city": "New York"
              }
              """;

String sql = """
             SELECT id, name, email
             FROM users
             WHERE age > 18
             ORDER BY name
             """;
```

### 2. Switch Expressions Enhancements

Java 13 enhanced switch expressions from Java 12 by introducing the `yield` statement. The yield statement allows switch expressions to return values from multi-statement blocks, providing more flexibility in complex switch logic.

The yield statement is used when you need to execute multiple statements within a switch case and then return a value. This makes switch expressions more powerful and suitable for complex scenarios.

```java
String result = switch (grade) {
    case A, B -> "Excellent";
    case C -> "Good";
    case D -> {
        System.out.println("Needs improvement");
        yield "Poor";
    }
    case F -> {
        System.out.println("Failed");
        yield "Fail";
    }
    default -> "Invalid grade";
};
```

### 3. Dynamic CDS Archive

Java 13 enhanced Class Data Sharing (CDS) by allowing the creation of dynamic archives. This feature enables the archiving of classes that are loaded during application execution, improving startup time for subsequent runs.

Dynamic CDS archives capture the classes loaded by an application and store them in a format that can be quickly loaded by future application instances. This is particularly beneficial for applications with dynamic class loading patterns.

---

## Java 14 Features

### 1. Switch Expressions (Standard)

Java 14 standardized switch expressions, making them a permanent part of the language. This feature provides a more concise and expressive way to write switch statements that can return values.

Switch expressions support both arrow syntax and traditional colon syntax, with the yield statement for returning values from multi-statement blocks. They eliminate common bugs associated with fall-through behavior and make code more readable.

```java
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY -> 7;
    case THURSDAY, SATURDAY -> 8;
    case WEDNESDAY -> 9;
};

// With yield for complex logic
String season = switch (month) {
    case DECEMBER, JANUARY, FEBRUARY -> {
        System.out.println("Winter season");
        yield "Winter";
    }
    case MARCH, APRIL, MAY -> "Spring";
    case JUNE, JULY, AUGUST -> "Summer";
    case SEPTEMBER, OCTOBER, NOVEMBER -> "Fall";
};
```

### 2. Pattern Matching for instanceof (Preview)

Java 14 introduced pattern matching for instanceof as a preview feature. This enhancement eliminates the need for explicit casting after an instanceof check, making code more concise and less error-prone.

Pattern matching combines the instanceof check and variable declaration in a single expression. If the instanceof check succeeds, the variable is automatically cast to the tested type and can be used within the scope.

```java
// Traditional approach
if (obj instanceof String) {
    String str = (String) obj;
    System.out.println(str.toUpperCase());
}

// Pattern matching for instanceof (Java 14)
if (obj instanceof String str) {
    System.out.println(str.toUpperCase());
}

// In method with multiple checks
public void processObject(Object obj) {
    if (obj instanceof String str && str.length() > 5) {
        System.out.println("Long string: " + str);
    } else if (obj instanceof Integer num && num > 0) {
        System.out.println("Positive number: " + num);
    }
}
```

### 3. Helpful NullPointerExceptions

Java 14 enhanced NullPointerExceptions to provide more detailed information about which variable was null. This improvement significantly helps in debugging by pinpointing exactly what caused the NPE.

The enhanced NPE messages show the method, filename, and line number, along with a detailed description of which part of the expression was null. This makes debugging much more efficient, especially in complex expressions.

```java
// Previous NPE message would be generic
// New detailed message shows exactly what was null
Person person = null;
String name = person.getAddress().getStreet(); // Enhanced NPE shows person is null

List<String> list = Arrays.asList("a", null, "c");
int length = list.get(1).length(); // Enhanced NPE shows list.get(1) is null
```

### 4. Records (Preview)

Records were introduced as a preview feature in Java 14. They provide a compact syntax for declaring classes that are primarily used to store data. Records automatically generate constructors, accessors, equals(), hashCode(), and toString() methods.

Records are immutable by default and focus on modeling data rather than behavior. They reduce boilerplate code significantly and make data-carrying classes more concise and readable. Records are particularly useful for DTOs, value objects, and simple data containers.

```java
// Traditional class
public class PersonOld {
    private final String name;
    private final int age;
    
    public PersonOld(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String name() { return name; }
    public int age() { return age; }
    
    // equals, hashCode, toString methods...
}

// Record (Java 14)
public record Person(String name, int age) {
    // All methods automatically generated
}

// Usage
Person person = new Person("John", 30);
System.out.println(person.name()); // John
System.out.println(person);        // Person[name=John, age=30]
```

---

## Java 15 Features

### 1. Text Blocks (Standard)

Java 15 standardized text blocks, making them a permanent feature of the language. Text blocks provide a cleaner way to write multi-line strings without escape sequences and concatenation.

Text blocks automatically handle indentation and line endings, making them perfect for embedding JSON, XML, SQL, or other formatted text directly in Java code. They improve code readability and maintainability significantly.

```java
String html = """
              <html>
                  <head>
                      <title>My Page</title>
                  </head>
                  <body>
                      <h1>Welcome!</h1>
                      <p>This is a text block example.</p>
                  </body>
              </html>
              """;

String query = """
               SELECT p.name, p.email, a.street, a.city
               FROM person p
               JOIN address a ON p.id = a.person_id
               WHERE p.age > ?
               ORDER BY p.name
               """;
```

### 2. Sealed Classes (Preview)

Sealed classes were introduced as a preview feature in Java 15. They allow you to restrict which classes can extend or implement them, providing more control over inheritance hierarchies.

Sealed classes are declared with the `sealed` keyword and specify permitted subclasses using the `permits` clause. This feature enables more precise modeling of domain concepts and better pattern matching support.

```java
// Sealed class definition
public sealed class Shape permits Circle, Rectangle, Triangle {
    // Common shape behavior
}

// Permitted subclasses
public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
}

public non-sealed class Rectangle extends Shape {
    // Can be further extended
}

public sealed class Triangle extends Shape permits EquilateralTriangle {
    // Can have its own permitted subclasses
}
```

### 3. Hidden Classes

Hidden classes are a low-level feature for frameworks that generate classes at runtime. They cannot be discovered by other classes and have a limited lifecycle, making them suitable for dynamic bytecode generation scenarios.

Hidden classes provide better memory management for dynamically generated classes and improve security by preventing unwanted access to framework-generated classes. This feature primarily benefits frameworks like Spring, Hibernate, and bytecode manipulation libraries.

### 4. ZGC and Shenandoah GC Improvements

Java 15 made significant improvements to both ZGC (Z Garbage Collector) and Shenandoah garbage collectors. These improvements focus on reducing latency and improving concurrent performance.

Both garbage collectors are designed for low-latency applications and can handle large heap sizes with minimal pause times. They're particularly beneficial for applications that require consistent response times and handle large amounts of data.

---

## Java 16 Features

### 1. Pattern Matching for instanceof (Standard)

Java 16 standardized pattern matching for instanceof, making it a permanent feature. This enhancement eliminates the need for explicit casting after instanceof checks and reduces boilerplate code.

Pattern matching makes code more readable and less error-prone by combining the type check and cast in a single expression. The pattern variable is only in scope when the instanceof check succeeds, providing better type safety.

```java
// Simplified object processing
public String formatObject(Object obj) {
    if (obj instanceof String str) {
        return "String: " + str.toUpperCase();
    } else if (obj instanceof Integer num) {
        return "Number: " + (num * 2);
    } else if (obj instanceof List<?> list) {
        return "List with " + list.size() + " items";
    }
    return "Unknown type";
}

// Pattern matching in complex conditions
if (obj instanceof String str && str.length() > 10) {
    System.out.println("Long string: " + str.substring(0, 10) + "...");
}
```

### 2. Records (Standard)

Java 16 standardized records, making them a permanent part of the language. Records provide a concise way to create immutable data classes with automatically generated constructors, accessors, and utility methods.

Records are ideal for modeling simple data aggregates and eliminate much of the boilerplate code associated with traditional data classes. They integrate well with pattern matching and other modern Java features.

```java
// Simple record definition
public record Point(int x, int y) {}

// Record with custom methods
public record Person(String name, int age) {
    // Custom constructor with validation
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
    }
    
    // Additional methods
    public boolean isAdult() {
        return age >= 18;
    }
}

// Usage
Point origin = new Point(0, 0);
Person adult = new Person("John", 25);
System.out.println(adult.isAdult()); // true
```

### 3. Sealed Classes Improvements

Java 16 enhanced sealed classes with better syntax and more flexible inheritance rules. Sealed classes allow you to control which classes can extend them, enabling more precise domain modeling.

The improvements include better integration with records and pattern matching, making sealed classes more useful for algebraic data types and domain modeling scenarios.

```java
// Sealed interface with record implementations
public sealed interface Result<T> permits Success, Error {
}

public record Success<T>(T value) implements Result<T> {}
public record Error<T>(String message) implements Result<T> {}

// Pattern matching with sealed classes
public static <T> String processResult(Result<T> result) {
    return switch (result) {
        case Success<T> success -> "Got: " + success.value();
        case Error<T> error -> "Error: " + error.message();
    };
}
```

### 4. Stream API Enhancements

Java 16 added new methods to the Stream API to make stream processing more convenient and expressive. The new methods include toList() for collecting streams to lists more concisely.

The toList() method provides a simpler way to collect stream elements into a list without using Collectors.toList(), making stream operations more readable and concise.

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Before Java 16
List<String> longNames = names.stream()
    .filter(name -> name.length() > 4)
    .collect(Collectors.toList());

// Java 16 - more concise
List<String> longNames = names.stream()
    .filter(name -> name.length() > 4)
    .toList();

List<Integer> squares = Stream.iterate(1, i -> i <= 10, i -> i + 1)
    .map(i -> i * i)
    .toList();
```

---

## Java 17 Features

### 1. Sealed Classes (Standard)

Java 17 standardized sealed classes, making them a permanent feature. Sealed classes restrict which classes can extend or implement them, providing better control over inheritance hierarchies and enabling more precise domain modeling.

Sealed classes work well with pattern matching and records to create algebraic data types. They help create more maintainable code by making inheritance relationships explicit and controlled.

```java
// Sealed class hierarchy for geometric shapes
public sealed class Shape permits Circle, Rectangle, Triangle {
    public abstract double area();
}

public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public final class Rectangle extends Shape {
    private final double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double area() {
        return width * height;
    }
}

// Pattern matching with sealed classes
public static String describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius " + c.radius;
        case Rectangle r -> "Rectangle " + r.width + "x" + r.height;
        case Triangle t -> "Triangle with area " + t.area();
    };
}
```

### 2. Context-Specific Deserialization Filters

Java 17 enhanced security by providing context-specific deserialization filters. These filters allow applications to validate incoming serialized data more precisely, helping prevent deserialization attacks.

The filters can be configured globally or per ObjectInputStream, providing flexible security policies. They help protect applications from malicious serialized data by allowing or rejecting classes based on various criteria.

```java
// Global filter configuration
ObjectInputFilter globalFilter = ObjectInputFilter.Config.createFilter(
    "java.base/*;!*");
ObjectInputFilter.Config.setSerialFilter(globalFilter);

// Custom filter for specific use cases
ObjectInputFilter customFilter = filterInfo -> {
    Class<?> clazz = filterInfo.serialClass();
    if (clazz != null) {
        if (clazz.getName().startsWith("com.trusted.")) {
            return ObjectInputFilter.Status.ALLOWED;
        }
        return ObjectInputFilter.Status.REJECTED;
    }
    return ObjectInputFilter.Status.UNDECIDED;
};
```

### 3. Enhanced Pseudo-Random Number Generators

Java 17 introduced a new API for pseudo-random number generators that provides better performance, more algorithms, and improved flexibility. The new API is designed around interfaces that allow for easy algorithm switching.

The RandomGenerator interface provides a common abstraction for different PRNG algorithms, making it easier to choose the best algorithm for specific use cases. This includes cryptographically secure generators and high-performance generators for simulations.

```java
// Using different random number generators
RandomGenerator generator = RandomGenerator.of("Xoshiro256PlusPlus");
int randomInt = generator.nextInt(1, 101); // Random between 1-100

// Stream of random numbers
IntStream randomNumbers = generator.ints(10, 1, 101);
randomNumbers.forEach(System.out::println);

// For cryptographic purposes
RandomGenerator secure = RandomGenerator.of("SecureRandom");
byte[] randomBytes = new byte[32];
secure.nextBytes(randomBytes);
```

### 4. Deprecate the Applet API for Removal

Java 17 deprecated the Applet API for removal in a future version. This reflects the reality that web browsers no longer support Java applets, making the API obsolete.

This deprecation encourages developers to migrate to modern web technologies and Java deployment models. Applications using applets should be redesigned using contemporary web frameworks and technologies.

---

## Java 18 Features

### 1. Simple Web Server

Java 18 introduced a simple HTTP server for prototyping, testing, and debugging purposes. This lightweight server is designed for development scenarios and should not be used in production environments.

The simple web server can serve static files and provides basic HTTP functionality out of the box. It's particularly useful for testing web applications, serving documentation, or creating quick prototypes without setting up a full web server.

```java
// Start simple web server programmatically
HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
server.createContext("/", new SimpleFileServer.createFileHandler(
    Path.of("./public")));
server.start();

// Command line usage
// jwebserver -p 8080 -d /path/to/directory
```

### 2. Code Snippets in Java API Documentation

Java 18 introduced the `@snippet` tag for including code examples in Javadoc documentation. This feature allows developers to embed external source files or inline code snippets with syntax highlighting and validation.

The @snippet tag improves documentation quality by ensuring code examples are syntactically correct and up-to-date. It supports various options for highlighting specific parts of the code and including external files.

```java
/**
 * Calculates the factorial of a number.
 * 
 * {@snippet :
 *   int result = factorial(5);
 *   System.out.println(result); // Output: 120
 * }
 * 
 * @param n the number to calculate factorial for
 * @return the factorial of n
 */
public static int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
```

### 3. UTF-8 by Default

Java 18 made UTF-8 the default charset for the standard Java APIs when no charset is explicitly specified. This change provides consistency across different operating systems and locales.

Previously, the default charset was platform-dependent, which could lead to inconsistent behavior across different environments. Making UTF-8 the default improves portability and reduces charset-related issues.

```java
// These operations now use UTF-8 by default
Files.readString(path);           // UTF-8 encoding
Files.writeString(path, content); // UTF-8 encoding
new String(bytes);                // UTF-8 decoding
```

### 4. Internet Address Resolution SPI

Java 18 introduced a Service Provider Interface (SPI) for internet address resolution. This allows developers to replace the built-in address resolution mechanism with custom implementations.

The SPI enables better integration with custom naming services, improved IPv6 support, and platform-specific optimizations. It's particularly useful for applications that need specialized address resolution behavior.

---

## Java 19 Features

### 1. Virtual Threads (Preview)

Virtual threads are lightweight threads that dramatically reduce the effort of writing, maintaining, and debugging high-throughput concurrent applications. They are implemented as user-mode threads scheduled by the JVM rather than the operating system.

Virtual threads enable applications to handle millions of concurrent tasks with minimal memory overhead. They're particularly beneficial for I/O-intensive applications and server-side applications that handle many concurrent requests.

```java
// Creating virtual threads
Thread virtualThread = Thread.startVirtualThread(() -> {
    System.out.println("Running in virtual thread: " + 
        Thread.currentThread().isVirtual());
});

// Using Executors with virtual threads
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 1_000_000; i++) {
        executor.submit(() -> {
            // Simulate I/O operation
            Thread.sleep(1000);
            return "Task completed";
        });
    }
}
```

### 2. Structured Concurrency (Incubator)

Structured concurrency treats groups of related tasks running in different threads as a single unit of work. This simplifies error handling, cancellation, and resource management in concurrent applications.

Structured concurrency ensures that subtasks complete before their parent task completes, providing better error propagation and resource cleanup. It makes concurrent code more reliable and easier to understand.

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser(userId));
    Future<String> order = scope.fork(() -> fetchOrder(orderId));
    
    scope.join();           // Wait for both tasks
    scope.throwIfFailed();  // Propagate any failures
    
    // Both tasks completed successfully
    return processUserOrder(user.resultNow(), order.resultNow());
}
```

### 3. Pattern Matching for switch (Preview)

Java 19 enhanced pattern matching for switch expressions and statements as a preview feature. This allows more sophisticated pattern matching beyond simple type checks.

Pattern matching in switch statements enables more expressive and concise code, particularly when working with sealed classes and records. It supports guard conditions and nested patterns for complex matching scenarios.

```java
// Pattern matching with switch
public String processValue(Object value) {
    return switch (value) {
        case String s when s.length() > 10 -> "Long string: " + s;
        case String s -> "String: " + s;
        case Integer i when i > 0 -> "Positive: " + i;
        case Integer i -> "Non-positive: " + i;
        case List<?> list when list.isEmpty() -> "Empty list";
        case List<?> list -> "List with " + list.size() + " items";
        case null -> "null value";
        default -> "Unknown type";
    };
}
```

### 4. Record Patterns (Preview)

Record patterns were introduced as a preview feature in Java 19, allowing pattern matching to destructure record values. This enables more concise and readable code when working with records.

Record patterns can be nested and combined with other patterns, providing powerful destructuring capabilities. They work particularly well with sealed classes and switch expressions.

```java
// Record definitions
public record Point(int x, int y) {}
public record Circle(Point center, int radius) {}

// Record pattern matching
public String describe(Object shape) {
    return switch (shape) {
        case Circle(Point(var x, var y), var r) when r > 10 -> 
            "Large circle at (" + x + "," + y + ") with radius " + r;
        case Circle(var center, var radius) -> 
            "Circle at " + center + " with radius " + radius;
        case Point(var x, var y) -> 
            "Point at (" + x + "," + y + ")";
        default -> "Unknown shape";
    };
}
```

---

## Java 20 Features

### 1. Scoped Values (Incubator)

Scoped values provide a way to share immutable data within and across threads, without the overhead and complexity of ThreadLocal variables. They're designed for use cases where data needs to be passed down through a call stack or across thread boundaries.

Scoped values offer better performance and memory characteristics than ThreadLocal variables and integrate well with virtual threads and structured concurrency. They provide a cleaner way to handle context information in concurrent applications.

```java
// Define scoped value
private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

// Use scoped value
ScopedValue.where(USER_ID, "user123")
    .run(() -> {
        processRequest();  // Can access USER_ID throughout call stack
    });

// Access scoped value
private void processRequest() {
    String userId = USER_ID.get(); // "user123"
    // Process request with user context
}
```

### 2. Record Patterns and Array Patterns (Preview)

Java 20 enhanced pattern matching with more sophisticated record patterns and introduced array patterns. These features enable more powerful destructuring and pattern matching capabilities.

Enhanced record patterns support nested destructuring and work better with sealed classes. Array patterns allow matching against array structures, providing more comprehensive pattern matching support.

```java
// Enhanced record patterns with nested matching
public record Person(String name, Address address) {}
public record Address(String street, String city, String zipCode) {}

public String processAddress(Person person) {
    return switch (person) {
        case Person(var name, Address(var street, "New York", var zip)) -> 
            name + " lives in NYC at " + street + " " + zip;
        case Person(var name, Address(var street, var city, _)) -> 
            name + " lives in " + city + " on " + street;
    };
}

// Array patterns (preview)
public void processArray(int[] arr) {
    switch (arr) {
        case [1, 2, 3] -> System.out.println("Exact match");
        case [var first, var second, ...] -> 
            System.out.println("Starts with " + first + ", " + second);
        case [] -> System.out.println("Empty array");
    }
}
```

### 3. Virtual Threads Improvements

Java 20 continued to refine virtual threads based on feedback from the preview in Java 19. The improvements include better debugging support, performance optimizations, and enhanced integration with existing APIs.

Virtual threads in Java 20 provide better observability tools and improved performance characteristics. They continue to be the foundation for building highly concurrent applications with minimal resource usage.

```java
// Improved virtual thread creation and management
ThreadFactory factory = Thread.ofVirtual().name("worker-", 1).factory();

try (ExecutorService executor = Executors.newThreadPerTaskExecutor(factory)) {
    List<Future<String>> futures = new ArrayList<>();
    
    for (int i = 0; i < 100_000; i++) {
        futures.add(executor.submit(() -> {
            // Simulate work with I/O
            Thread.sleep(Duration.ofSeconds(1));
            return "Task " + Thread.currentThread().getName() + " completed";
        }));
    }
    
    // Process results
    futures.forEach(future -> {
        try {
            System.out.println(future.get());
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
}
```

---

## Java 21 Features

### 1. Virtual Threads (Standard)

Java 21 standardized virtual threads, making them a permanent part of the platform. Virtual threads are lightweight threads that enable applications to handle millions of concurrent tasks with minimal memory overhead.

Virtual threads are particularly beneficial for I/O-intensive applications and server-side applications. They integrate seamlessly with existing thread-based APIs while providing dramatically better scalability and resource utilization.

```java
// Creating virtual threads
Thread vt1 = Thread.startVirtualThread(() -> {
    // Long-running task
    processData();
});

Thread vt2 = Thread.ofVirtual()
    .name("data-processor")
    .start(() -> processMoreData());

// Virtual thread executor
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<String>> results = IntStream.range(0, 1_000_000)
        .mapToObj(i -> executor.submit(() -> {
            Thread.sleep(1000); // Simulated I/O
            return "Result " + i;
        }))
        .toList();
    
    // Process million tasks with minimal memory usage
    results.forEach(future -> {
        try {
            String result = future.get();
            // Handle result
        } catch (Exception e) {
            // Handle error
        }
    });
}
```

### 2. Sequenced Collections

Java 21 introduced sequenced collections, which are collections with a defined encounter order. This feature fills gaps in the collections framework by providing consistent methods for accessing first and last elements.

Sequenced collections provide reversed views and consistent APIs across different collection types. They make it easier to work with ordered collections and provide better abstraction for sequence-based operations.

```java
// SequencedList methods
List<String> list = new ArrayList<>(List.of("a", "b", "c", "d"));

String first = list.getFirst();    // "a"
String last = list.getLast();      // "d"

list.addFirst("z");                // [z, a, b, c, d]
list.addLast("e");                 // [z, a, b, c, d, e]

// Reversed view
List<String> reversed = list.reversed(); // [e, d, c, b, a, z]

// SequencedSet methods  
LinkedHashSet<Integer> set = new LinkedHashSet<>(Set.of(1, 2, 3, 4));
Integer firstNum = set.getFirst();  // 1
Integer lastNum = set.getLast();    // 4

SequencedSet<Integer> reversedSet = set.reversed();
```

### 3. Record Patterns (Standard)

Java 21 standardized record patterns, allowing destructuring of record values in pattern matching contexts. This feature makes working with records more convenient and expressive.

Record patterns can be nested and used in switch expressions, instanceof checks, and other pattern matching contexts. They provide a clean way to extract data from complex record structures.

```java
// Record definitions
public record Point(double x, double y) {}
public record Circle(Point center, double radius) {}
public record Rectangle(Point topLeft, Point bottomRight) {}

// Pattern matching with records
public double calculateArea(Object shape) {
    return switch (shape) {
        case Circle(Point(var x, var y), var radius) -> 
            Math.PI * radius * radius;
        case Rectangle(Point(var x1, var y1), Point(var x2, var y2)) -> 
            Math.abs(x2 - x1) * Math.abs(y2 - y1);
        case Point p -> 0.0; // Point has no area
        default -> throw new IllegalArgumentException("Unknown shape");
    };
}

// Nested record patterns
public record Address(String street, String city, String country) {}
public record Person(String name, int age, Address address) {}

public String getCountry(Person person) {
    return switch (person) {
        case Person(_, _, Address(_, _, var country)) -> country;
    };
}
```

### 4. Pattern Matching for switch (Standard)

Java 21 finalized pattern matching for switch expressions and statements. This enables more sophisticated matching beyond simple equality checks, including type patterns, guard conditions, and record destructuring.

Pattern matching in switch statements makes code more expressive and reduces the need for multiple if-else statements. It integrates well with sealed classes and records for comprehensive pattern matching scenarios.

```java
// Type patterns with guards
public String classifyNumber(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Negative integer: " + i;
        case Integer i when i == 0 -> "Zero";
        case Integer i -> "Positive integer: " + i;
        case Double d when d.isNaN() -> "Not a number";
        case Double d -> "Double value: " + d;
        case String s when s.isEmpty() -> "Empty string";
        case String s -> "String: " + s;
        case null -> "Null value";
        default -> "Unknown type";
    };
}

// Sealed class pattern matching
public sealed interface Result<T> permits Success, Failure {}
public record Success<T>(T value) implements Result<T> {}
public record Failure<T>(String error) implements Result<T> {}

public <T> String handleResult(Result<T> result) {
    return switch (result) {
        case Success<T>(var value) -> "Success: " + value;
        case Failure<T>(var error) -> "Failed: " + error;
    };
}
```

### 5. String Templates (Preview)

Java 21 introduced string templates as a preview feature, providing a safer and more convenient way to compose strings from literals and expressions. String templates help prevent injection attacks and make string composition more readable.

String templates use template processors to validate and transform template expressions. They provide better type safety and security compared to traditional string concatenation or formatting methods.

```java
// String template examples
String name = "World";
int count = 42;

// STR processor for simple interpolation
String message = STR."Hello, \{name}! Count: \{count}";
// Result: "Hello, World! Count: 42"

// RAW processor returns StringTemplate object
StringTemplate template = RAW."Hello, \{name}!";

// Custom processor for JSON
String json = JSON."""
    {
        "name": "\{name}",
        "count": \{count},
        "timestamp": "\{Instant.now()}"
    }
    """;
```

---

## Java 22 Features

### 1. Unnamed Variables and Patterns

Java 22 introduced unnamed variables and patterns using the underscore (_) symbol. This feature allows developers to indicate that certain variables or patterns are intentionally unused, improving code readability and clarity.

Unnamed patterns are particularly useful in switch expressions, catch blocks, and lambda expressions where certain values need to be matched but not used. They make the intent clear and reduce the noise of unused variable warnings.

```java
// Unnamed variables in destructuring
public record Point(int x, int y, int z) {}

public int getX(Point point) {
    return switch (point) {
        case Point(var x, _, _) -> x; // Only care about x coordinate
    };
}

// Unnamed patterns in catch blocks
try {
    riskyOperation();
} catch (IOException _) {
    // Don't need exception details, just handle I/O errors
    handleIOError();
} catch (SQLException _) {
    // Don't need exception details, just handle SQL errors  
    handleSQLError();
}

// Unnamed variables in lambda expressions
stream.forEach(_ -> System.out.println("Processing item"));

// Multiple assignment with unnamed variables
var results = getCoordinates(); // returns array [x, y, z]
var (x, _, z) = results; // Only interested in x and z
```

### 2. Foreign Function & Memory API (Preview)

The Foreign Function & Memory API allows Java programs to interoperate with code and data outside of the Java runtime. This API enables calling native functions and accessing native memory directly from Java.

This API is designed to replace JNI (Java Native Interface) with a more modern, safer, and more efficient approach. It provides better performance and type safety when working with native libraries and memory.

```java
// Access native memory
try (Arena arena = Arena.ofConfined()) {
    // Allocate native memory
    MemorySegment segment = arena.allocate(100);
    
    // Write to native memory
    segment.set(JAVA_INT, 0, 42);
    segment.set(JAVA_LONG, 4, 12345L);
    
    // Read from native memory
    int value = segment.get(JAVA_INT, 0); // 42
    long longValue = segment.get(JAVA_LONG, 4); // 12345L
}

// Call native functions
Linker linker = Linker.nativeLinker();
SymbolLookup stdlib = linker.defaultLookup();

// Find native function (e.g., strlen)
MethodHandle strlen = linker.downcallHandle(
    stdlib.find("strlen").orElseThrow(),
    FunctionDescriptor.of(JAVA_LONG, ADDRESS)
);
```

### 3. Structured Concurrency (Preview)

Java 22 continued to refine structured concurrency, which treats groups of related tasks as a single unit of work. This approach simplifies error handling and resource management in concurrent applications.

Structured concurrency ensures that all subtasks complete before the parent task completes, providing better error propagation and cancellation semantics. It makes concurrent code more reliable and easier to reason about.

```java
// Structured concurrency example
public OrderSummary getOrderSummary(String userId, String orderId) throws Exception {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        
        // Fork concurrent tasks
        Supplier<User> userTask = scope.fork(() -> userService.getUser(userId));
        Supplier<Order> orderTask = scope.fork(() -> orderService.getOrder(orderId));
        Supplier<List<Item>> itemsTask = scope.fork(() -> itemService.getItems(orderId));
        
        // Wait for all tasks to complete
        scope.join();
        scope.throwIfFailed();
        
        // All tasks succeeded, build summary
        return new OrderSummary(
            userTask.get(),
            orderTask.get(), 
            itemsTask.get()
        );
    }
    // All tasks are automatically cancelled if scope exits early
}

// Custom task scope with timeout
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    scope.fork(() -> longRunningTask1());
    scope.fork(() -> longRunningTask2());
    
    // Wait with timeout
    scope.joinUntil(Instant.now().plusSeconds(30));
    scope.throwIfFailed();
}
```

### 4. Class-File API (Preview)

The Class-File API provides a standard way to parse, generate, and transform Java class files. This API offers a modern replacement for libraries like ASM and provides better integration with the JVM.

The Class-File API enables tools and frameworks to work with bytecode in a more standardized and efficient way. It supports both reading existing class files and generating new ones programmatically.

```java
// Parse a class file
ClassModel classModel = ClassFile.of().parse(classFileBytes);

// Examine class structure
System.out.println("Class name: " + classModel.thisClass().name());
System.out.println("Super class: " + classModel.superclass());

// Transform class (add method)
byte[] transformedBytes = ClassFile.of().transform(classModel, builder -> {
    builder.withMethod("newMethod", 
                      MethodTypeDesc.of(CD_void), 
                      ACC_PUBLIC,
                      methodBuilder -> {
                          methodBuilder.withCode(codeBuilder -> {
                              codeBuilder.return_();
                          });
                      });
});

// Generate new class
byte[] generatedClass = ClassFile.of().build(
    ClassDesc.of("com.example.GeneratedClass"),
    classBuilder -> {
        classBuilder.withSuperclass(CD_Object)
                   .withMethod(INIT_NAME, 
                              MethodTypeDesc.of(CD_void),
                              ACC_PUBLIC,
                              methodBuilder -> {
                                  methodBuilder.withCode(codeBuilder -> {
                                      codeBuilder.aload(0)
                                               .invokespecial(CD_Object, 
                                                            INIT_NAME, 
                                                            MethodTypeDesc.of(CD_void))
                                               .return_();
                                  });
                              });
    });
```

---

## Java 23 Features

### 1. Primitive Types in Patterns (Preview)

Java 23 introduced support for primitive types in patterns, extending pattern matching capabilities beyond reference types. This allows pattern matching to work directly with primitive values, making the feature more complete and useful.

Primitive patterns enable more comprehensive pattern matching and reduce the need for boxing/unboxing operations. They work seamlessly with existing pattern matching features and provide better performance for primitive-heavy applications.

```java
// Primitive patterns in switch expressions
public String classifyNumber(int value) {
    return switch (value) {
        case 0 -> "Zero";
        case 1, 2, 3, 4, 5 -> "Small positive";
        case int i when i > 0 && i <= 100 -> "Medium positive: " + i;
        case int i when i > 100 -> "Large positive: " + i;
        case int i when i < 0 -> "Negative: " + i;
    };
}

// Pattern matching with primitive types in records
public record Temperature(double celsius) {
    public String classify() {
        return switch (celsius) {
            case double temp when temp < 0.0 -> "Freezing: " + temp + "°C";
            case double temp when temp >= 0.0 && temp < 20.0 -> "Cold: " + temp + "°C";
            case double temp when temp >= 20.0 && temp < 30.0 -> "Warm: " + temp + "°C";
            case double temp when temp >= 30.0 -> "Hot: " + temp + "°C";
        };
    }
}

// Primitive destructuring in record patterns
public record Point3D(int x, int y, int z) {}

public String describePoint(Point3D point) {
    return switch (point) {
        case Point3D(0, 0, 0) -> "Origin";
        case Point3D(int x, 0, 0) -> "On X-axis at " + x;
        case Point3D(0, int y, 0) -> "On Y-axis at " + y;
        case Point3D(0, 0, int z) -> "On Z-axis at " + z;
        case Point3D(int x, int y, int z) when x == y && y == z -> 
            "Diagonal point at " + x;
        case Point3D(int x, int y, int z) -> 
            "Point at (" + x + ", " + y + ", " + z + ")";
    };
}
```

### 2. Module Import Declarations (Preview)

Java 23 introduced module import declarations as a preview feature, allowing classes to import all public classes from a module without specifying individual packages. This feature simplifies module usage and reduces boilerplate import statements.

Module imports provide a convenient way to access all public APIs from a module while maintaining proper module boundaries. They're particularly useful when working with large modules or when a class needs to use many classes from the same module.

```java
// Module import declaration
import module java.base;
import module java.logging;

public class ModuleImportExample {
    // Can use classes from java.base without individual imports
    public void example() {
        String text = "Hello"; // java.lang.String
        List<String> list = new ArrayList<>(); // java.util.List, ArrayList
        Optional<String> optional = Optional.of(text); // java.util.Optional
        
        // Classes from java.logging module
        Logger logger = Logger.getLogger("example");
        logger.info("Module import example");
        
        // Stream operations without individual imports
        list.stream()
            .filter(s -> s.length() > 3)
            .map(String::toUpperCase)
            .forEach(System.out::println);
    }
}

// Traditional approach would require:
// import java.util.*;
// import java.util.logging.*;
// import java.util.stream.*;
// etc.
```

### 3. Vector API (Sixth Incubator)

The Vector API continued its development in Java 23 as the sixth incubator version. This API provides a way to express vector computations that compile to optimal vector hardware instructions on supported platforms.

The Vector API enables developers to write high-performance numerical computations that can take advantage of SIMD (Single Instruction, Multiple Data) instructions available on modern processors. This is particularly beneficial for machine learning, scientific computing, and image processing applications.

```java
import jdk.incubator.vector.*;

public class VectorExample {
    static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_256;
    
    // Vector addition example
    public void vectorAdd(float[] a, float[] b, float[] result) {
        int i = 0;
        int upperBound = SPECIES.loopBound(a.length);
        
        // Process vectors
        for (; i < upperBound; i += SPECIES.length()) {
            FloatVector va = FloatVector.fromArray(SPECIES, a, i);
            FloatVector vb = FloatVector.fromArray(SPECIES, b, i);
            FloatVector vc = va.add(vb);
            vc.intoArray(result, i);
        }
        
        // Handle remaining elements
        for (; i < a.length; i++) {
            result[i] = a[i] + b[i];
        }
    }
    
    // Vector dot product
    public float dotProduct(float[] a, float[] b) {
        float result = 0.0f;
        int i = 0;
        int upperBound = SPECIES.loopBound(a.length);
        
        FloatVector sum = FloatVector.zero(SPECIES);
        
        for (; i < upperBound; i += SPECIES.length()) {
            FloatVector va = FloatVector.fromArray(SPECIES, a, i);
            FloatVector vb = FloatVector.fromArray(SPECIES, b, i);
            sum = va.fma(vb, sum); // Fused multiply-add
        }
        
        result = sum.reduceLanes(VectorOperators.ADD);
        
        // Handle remaining elements
        for (; i < a.length; i++) {
            result += a[i] * b[i];
        }
        
        return result;
    }
}
```

### 4. Structured Concurrency (Third Preview)

Java 23 continued refining structured concurrency based on feedback from previous previews. The third preview includes API improvements, better integration with virtual threads, and enhanced error handling capabilities.

Structured concurrency in Java 23 provides more flexible task management and better support for complex concurrent scenarios. It maintains the core principle of treating related concurrent tasks as a single unit of work while offering more control over task lifecycle.

```java
// Enhanced structured concurrency with custom policies
public class DataProcessor {
    
    // Process data with timeout and partial success handling
    public ProcessingResult processDataSources(List<String> sources) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            
            // Fork tasks for each data source
            List<Supplier<String>> tasks = sources.stream()
                .map(source -> scope.fork(() -> processDataSource(source)))
                .toList();
            
            // Wait for first success or timeout
            scope.joinUntil(Instant.now().plusSeconds(30));
            
            // Return first successful result
            return new ProcessingResult(scope.result(), scope.resultState());
        }
    }
    
    // Collect all results with custom error handling
    public AggregatedResult aggregateResults(List<DataSource> sources) throws Exception {
        try (var scope = new StructuredTaskScope<String>() {
            @Override
            protected void handleComplete(Supplier<String> task) {
                // Custom completion handling
                try {
                    String result = task.get();
                    addSuccessfulResult(result);
                } catch (Exception e) {
                    addFailedResult(e);
                }
            }
        }) {
            
            // Submit all tasks
            sources.forEach(source -> 
                scope.fork(() -> processSource(source)));
            
            // Wait for all tasks
            scope.join();
            
            // Return aggregated results
            return buildAggregatedResult();
        }
    }
    
    // Parallel processing with resource limits
    public void processWithResourceLimit(List<Task> tasks) throws Exception {
        // Limit concurrent tasks to available processors
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            Semaphore semaphore = new Semaphore(Runtime.getRuntime().availableProcessors());
            
            List<Supplier<Void>> futures = tasks.stream()
                .map(task -> scope.fork(() -> {
                    semaphore.acquire();
                    try {
                        return processTask(task);
                    } finally {
                        semaphore.release();
                    }
                }))
                .toList();
            
            scope.join();
            scope.throwIfFailed();
        }
    }
}

// Integration with virtual threads
public class VirtualThreadStructuredExample {
    
    public void processRequestsConcurrently(List<Request> requests) throws Exception {
        ThreadFactory virtualThreadFactory = Thread.ofVirtual()
            .name("request-processor-", 1)
            .factory();
            
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            // Process each request in a virtual thread
            List<Supplier<Response>> responses = requests.stream()
                .map(request -> scope.fork(() -> {
                    Thread currentThread = Thread.currentThread();
                    assert currentThread.isVirtual();
                    return handleRequest(request);
                }))
                .toList();
            
            scope.join();
            scope.throwIfFailed();
            
            // Collect all successful responses
            List<Response> results = responses.stream()
                .map(Supplier::get)
                .toList();
                
            processResponses(results);
        }
    }
}
```

### 5. Scoped Values (Preview)

Java 23 continued developing scoped values as a preview feature, providing improvements based on usage feedback. Scoped values offer a better alternative to ThreadLocal variables for sharing immutable data across method calls and threads.

Scoped values provide better performance characteristics than ThreadLocal and integrate well with virtual threads and structured concurrency. They enable clean context propagation without the memory leaks and lifecycle issues associated with ThreadLocal.

```java
public class ScopedValueExample {
    
    // Define scoped values
    private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    private static final ScopedValue<SecurityContext> SECURITY_CONTEXT = ScopedValue.newInstance();
    
    // Web request handler example
    public void handleRequest(HttpRequest request) {
        String userId = extractUserId(request);
        String requestId = generateRequestId();
        SecurityContext security = createSecurityContext(request);
        
        // Establish scoped values for request processing
        ScopedValue.where(USER_ID, userId)
                  .where(REQUEST_ID, requestId)
                  .where(SECURITY_CONTEXT, security)
                  .run(() -> {
                      processRequest(request);
                  });
    }
    
    // Service method that uses scoped values
    private void processRequest(HttpRequest request) {
        // Access scoped values throughout the call stack
        String currentUser = USER_ID.get();
        String currentRequest = REQUEST_ID.get();
        
        log("Processing request " + currentRequest + " for user " + currentUser);
        
        // Call other methods that can access the same scoped values
        validateRequest();
        executeBusinessLogic();
        auditRequest();
    }
    
    // Deep in the call stack, still access scoped values
    private void executeBusinessLogic() {
        String userId = USER_ID.get();
        SecurityContext context = SECURITY_CONTEXT.get();
        
        if (context.hasPermission("ADMIN")) {
            performAdminOperation(userId);
        } else {
            performUserOperation(userId);
        }
    }
    
    // Nested scoped value usage
    public void processWithSubContext() {
        ScopedValue.where(USER_ID, "mainUser")
                  .run(() -> {
                      // Can access USER_ID here
                      String mainUser = USER_ID.get(); // "mainUser"
                      
                      // Create nested scope with different user
                      ScopedValue.where(USER_ID, "subUser")
                                .run(() -> {
                                    String subUser = USER_ID.get(); // "subUser"
                                    performSubOperation();
                                });
                      
                      // Back to main user context
                      String backToMain = USER_ID.get(); // "mainUser"
                  });
    }
    
    // Integration with virtual threads
    public void processVirtualThreads(List<Task> tasks) {
        String contextId = generateContextId();
        
        ScopedValue.where(REQUEST_ID, contextId)
                  .run(() -> {
                      try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
                          List<Future<Void>> futures = tasks.stream()
                              .map(task -> executor.submit(() -> {
                                  // Scoped values are inherited by virtual threads
                                  String inherited = REQUEST_ID.get(); // Same as contextId
                                  processTask(task, inherited);
                                  return null;
                              }))
                              .toList();
                          
                          // Wait for all tasks
                          futures.forEach(future -> {
                              try {
                                  future.get();
                              } catch (Exception e) {
                                  handleError(e);
                              }
                          });
                      }
                  });
    }
}
```

### 6. Flexible Constructor Bodies (Preview)

Java 23 introduced flexible constructor bodies as a preview feature, allowing statements to appear before explicit constructor invocations (`this()` or `super()`). This provides more flexibility in constructor design and enables better validation and preparation before calling parent constructors.

This feature allows for argument validation, logging, and other preparatory work before invoking superclass constructors. It makes constructors more flexible while maintaining type safety and initialization order guarantees.

```java
public class FlexibleConstructorExample {
    
    // Parent class
    static class Shape {
        protected final String name;
        protected final double area;
        
        public Shape(String name, double area) {
            if (area < 0) {
                throw new IllegalArgumentException("Area cannot be negative");
            }
            this.name = name;
            this.area = area;
        }
    }
    
    // Child class with flexible constructor
    static class Circle extends Shape {
        private final double radius;
        
        public Circle(double radius) {
            // Statements before super() call
            if (radius <= 0) {
                System.err.println("Invalid radius: " + radius);
                throw new IllegalArgumentException("Radius must be positive");
            }
            
            // Log constructor invocation
            System.out.println("Creating circle with radius: " + radius);
            
            // Calculate area before calling super
            double calculatedArea = Math.PI * radius * radius;
            
            // Now call super constructor
            super("Circle", calculatedArea);
            
            // Initialize remaining fields
            this.radius = radius;
        }
    }
    
    // Complex validation example
    static class Rectangle extends Shape {
        private final double width;
        private final double height;
        
        public Rectangle(double width, double height) {
            // Multiple validation steps before super()
            System.out.println("Validating rectangle dimensions...");
            
            if (width <= 0 || height <= 0) {
                String message = String.format(
                    "Invalid dimensions: width=%.2f, height=%.2f", 
                    width, height);
                System.err.println(message);
                throw new IllegalArgumentException(message);
            }
            
            // Check for reasonable size limits
            double maxDimension = Math.max(width, height);
            if (maxDimension > 10000) {
                System.out.println("Warning: Very large rectangle created");
            }
            
            // Prepare data for super constructor
            String shapeName = String.format("Rectangle[%.1fx%.1f]", width, height);
            double area = width * height;
            
            super(shapeName, area);
            
            this.width = width;
            this.height = height;
            
            System.out.println("Rectangle created successfully");
        }
    }
    
    // Constructor with this() invocation
    static class ConfigurableCircle extends Circle {
        private final String color;
        private final boolean filled;
        
        // Primary constructor
        public ConfigurableCircle(double radius, String color, boolean filled) {
            // Validation before calling super
            if (color == null || color.trim().isEmpty()) {
                throw new IllegalArgumentException("Color cannot be null or empty");
            }
            
            System.out.println("Creating " + color + " circle");
            super(radius);
            
            this.color = color.trim().toLowerCase();
            this.filled = filled;
        }
        
        // Convenience constructor with this() call
        public ConfigurableCircle(double radius, String color) {
            // Preparation before this() call
            System.out.println("Using default fill setting");
            boolean defaultFilled = radius > 5.0; // Large circles filled by default
            
            this(radius, color, defaultFilled);
        }
        
        // Another convenience constructor
        public ConfigurableCircle(double radius) {
            // Default color logic before this() call
            String defaultColor;
            if (radius < 1.0) {
                defaultColor = "red";
            } else if (radius < 5.0) {
                defaultColor = "blue";
            } else {
                defaultColor = "green";
            }
            
            System.out.println("Auto-selecting color: " + defaultColor);
            this(radius, defaultColor);
        }
    }
}
```

---

## Summary

Java has evolved significantly from Java 8 to Java 23, introducing numerous features that enhance developer productivity, application performance, and code maintainability. Key themes across these versions include:

**Functional Programming**: Lambda expressions, Stream API, and functional interfaces transformed Java into a more functional language, enabling more expressive and concise code.

**Pattern Matching**: From simple instanceof patterns to sophisticated record and sealed class patterns, Java has embraced pattern matching as a core language feature.

**Concurrency Evolution**: Virtual threads and structured concurrency represent a fundamental shift in how Java handles concurrent programming, enabling massive scalability with simpler mental models.

**Modern Language Features**: Records, sealed classes, text blocks, and switch expressions make Java code more expressive and reduce boilerplate significantly.

**Performance and Memory**: Features like compact number formatting, improved garbage collectors, and the Vector API focus on performance optimization and efficient resource usage.

**Developer Experience**: Tools like JShell, improved error messages, and better API design enhance the overall developer experience.

This evolution shows Java's commitment to modernization while maintaining backward compatibility, making it suitable for both legacy applications and cutting-edge development scenarios.