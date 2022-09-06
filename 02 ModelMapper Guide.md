
Guide to ModelMapper
=============

Setting Up
----------

If you're a Maven user just add the `modelmapper` library as a dependency:

```
<dependency>
  <groupId>org.modelmapper</groupId>
  <artifactId>modelmapper</artifactId>
  <version>3.0.0</version>
</dependency>

```

Otherwise you can [download](http://modelmapper.org/downloads) the latest ModelMapper jar and add it to your classpath.

Mapping
-------

Let's try mapping some objects. Consider the following source and destination [object models](https://github.com/jhalterman/modelmapper/tree/master/examples/src/main/java/org/modelmapper/gettingstarted):

#### Source model

```
// Assume getters and setters on each class
class Order {
  Customer customer;
  Address billingAddress;
}

class Customer {
  Name name;
}

class Name {
  String firstName;
  String lastName;
}

class Address {
  String street;
  String city;
}

```

#### Destination Model

```
// Assume getters and setters
class OrderDTO {
  String customerFirstName;
  String customerLastName;
  String billingStreet;
  String billingCity;
}

```

We can use ModelMapper to implicitly map an `order` instance to a new `OrderDTO`:

```
ModelMapper modelMapper = new ModelMapper();
OrderDTO orderDTO = modelMapper.map(order, OrderDTO.class);

```

And we can test that properties are mapped as expected:

```
assertEquals(order.getCustomer().getName().getFirstName(), orderDTO.getCustomerFirstName());
assertEquals(order.getCustomer().getName().getLastName(), orderDTO.getCustomerLastName());
assertEquals(order.getBillingAddress().getStreet(), orderDTO.getBillingStreet());
assertEquals(order.getBillingAddress().getCity(), orderDTO.getBillingCity());

```

How It Works
------------

When the `map` method is called, the *source* and *destination* types are analyzed to determine which properties implicitly match according to a [matching strategy](http://modelmapper.org/user-manual/configuration/#matching-strategies) and other [configuration](http://modelmapper.org/user-manual/configuration). Data is then mapped according to these matches.

Even when the *source* and *destination* objects and their properties are different, as in the example above, ModelMapper will do its best to determine reasonable matches between properties according to the configured [matching strategy](http://modelmapper.org/user-manual/configuration/#matching-strategies).

Handling Mismatches
-------------------

While ModelMapper will do its best to implicitly match source and destination properties for you, sometimes you may need to explicitly define mappings between properties.

ModelMapper supports a variety of mapping approaches, allowing you to use any mix of methods and field references. Let's map `Order.billingAddress.street` to `OrderDTO.billingStreet` and map `Order.billingAddress.city` to `OrderDTO.billingCity`:

```
modelMapper.typeMap(Order.class, OrderDTO.class).addMappings(mapper -> {
  mapper.map(src -> src.getBillingAddress().getStreet(),
      Destination::setBillingStreet);
  mapper.map(src -> src.getBillingAddress().getCity(),
      Destination::setBillingCity);
});

```

Conventional Configuration
--------------------------

As an alternative to manually mapping `Order.billingAddress.street` to `OrderDTO.billingStreet`, we can configure different conventions to be used when ModelMapper attempts to match these properties. The Loose [matching strategy](http://modelmapper.org/user-manual/configuration/#matching-strategies), which more loosely matches property names, will work in this case:

```
modelMapper.getConfiguration().setMatchingStrategy(MatchingStrategies.LOOSE);

```

Additional [matching strategies](http://modelmapper.org/user-manual/configuration/#matching-strategies) and other conventions can also be [configured](http://modelmapper.org/user-manual/configuration) to control ModelMapper's property matching process.

Validating Matches
------------------

Aside from writing tests to verify that your objects are being mapped as expected, ModelMapper has built-in [validation](http://modelmapper.org/user-manual/validation) that will let you know if any destination properties are unmatched.

First we have to let ModelMapper know about the types we want to validate. We can do this by creating a `TypeMap`:

```
modelMapper.createTypeMap(Order.class, OrderDTO.class);

```

Or we can add mappings from a `PropertyMap` which will automatically create a `TypeMap` if one doesn't already exist for the *source* and *destination* types:

```
modelMapper.addMappings(new OrderMap());

```

Then we can validate our mappings:

```
modelMapper.validate();

```

If any `TypeMap` contains a destination property that is unmatched to a source property a `ValidationException` will be thrown.



Configuration
=============

ModelMapper uses a set of conventions and configuration to determine which source and destination properties match each other. Available configuration, along with default values, is described below:

| Setting | Description | Default Value |
| --- | --- | --- |
| Ambiguity ignored | Determines whether destination properties that match more than one source property should be ignored | false |
| Access level | Determines which methods and fields are eligible for matching based on accessibility | public |
| Collections merge | Determines whether the destination items should be replaced or merged while source and destination have different size | true |
| Field matching | Indicates whether fields are eligible for matching | disabled |
| Naming convention | Determines which methods and fields are eligible for matching based on name | JavaBeans |
| Full type matching | Determines wether [ConditionalConverters](http://modelmapper.org/javadoc/org/modelmapper/spi/ConditionalConverter.html) must define a full match in order to be applied | false |
| Implicit matching | Determines whether the implicit mapping (mapping the models intelligently) should be enabled (see [Matching Process](http://modelmapper.org/user-manual/how-it-works/#matching-process)) | true |
| Name transformer | Transforms eligible property and class names prior to tokenization | JavaBeans |
| Name tokenizer | Tokenizes source and destination property names prior to matching | Camel Case |
| [Matching strategy](http://modelmapper.org/user-manual/configuration/#matching-strategies) | Determines how source and destination tokens are matched | Standard |
| Prefer nested properties | Determines if the implicit mapping should map the nested properties, we strongly recommend to disable this option while you are mapping a model contains circular reference | true |
| Skip null | Determines whether a property should be skipped or not when the property value is null | false |

You can read about how this configuration is used during the [matching process](http://modelmapper.org/user-manual/how-it-works/#matching-process).

Default Configuration
---------------------

Default configuration uses the *Standard* matching strategy to match only *public* source and destination *methods* that are named according to the *JavaBeans* convention.

Configuration Examples
----------------------

Adjusting configuration for certain matching requirements is simple. This example configures a `ModelMapper` to allow `protected` methods to be matched:

```
modelMapper.getConfiguration()
  .setMethodAccessLevel(AccessLevel.PROTECTED);

```

This example configures a `ModelMapper` to allow private fields to be matched:

```
modelMapper.getConfiguration()
  .setFieldMatchingEnabled(true)
  .setFieldAccessLevel(AccessLevel.PRIVATE);

```

This example configures a `ModelMapper` to use the `Loose` [MatchingStrategies matching strategy]:

```
modelMapper.getConfiguration()
  .setMatchingStrategy(MatchingStrategies.LOOSE);

```

This example configures a `ModelMapper` to allow any source and destination property names to be eligible for matching:

```
modelMapper.getConfiguration()
  .setSourceNamingConvention(NamingConventions.NONE);
  .setDestinationNamingConvention(NamingConventions.NONE);

```

This example configures a `ModelMapper` to use the `Underscore` name tokenizer for source and destination properties:

```
modelMapper.getConfiguration()
  .setSourceNameTokenizer(NameTokenizers.UNDERSCORE)
  .setDestinationNameTokenizer(NameTokenizers.UNDERSCORE)

```

Available Conventions
---------------------

ModelMapper includes several pre-defined conventions for handling different property matching requirements:

| Convention | Description |
| --- | --- |
| `NamingConventions.NONE` | Represents no naming convention, which applies to all property names |
| `NamingConventions.JAVABEANS_ACCESSOR` | Finds eligible accessors according to JavaBeans convention |
| `NamingConventions.JAVABEANS_MUTATOR` | Finds eligible mutators according to JavaBeans convention |
| `NameTransformers.JAVABEANS_ACCESSOR` | Transforms accessor names according to JavaBeans convention |
| `NameTransformers.JAVABEANS_MUTATOR` | Transforms mutators names according to JavaBeans convention |
| `NameTokenizers.CAMEL_CASE` | Tokenizes property and class names according to Camel Case convention |
| `NameTokenizers.UNDERSCORE` | Tokenizes property and class names by underscores |
| `MatchingStrategies.STANDARD` | Intelligently matches source and destination properties |
| `MatchingStrategies.LOOSE` | Loosely matches source and destination properties |
| `MatchingStrategies.STRICT` | Strictly matches source and destination properties |

Matching Strategies
-------------------

Matching strategies are used during the [matching process](http://modelmapper.org/user-manual/how-it-works/#matching-process) to match source and destination properties to each other. Below is a description of each strategy.

### Standard

The Standard matching strategy allows for source properties to be intelligently matched to destination properties, requiring that *all* destination properties be matched and all source property names have at least one token matched. The following rules apply:

-   Tokens can be matched in *any* order
-   *All* destination property name tokens must be matched
-   *All* source property names must have at least *one* token matched

The standard matching strategy is configured by default, and while it is not exact, it is ideal to use in most scenarios.

### Loose

The Loose matching strategy allows for source properties to be loosely matched to destination properties by requiring that *only* the last destination property in a hierarchy be matched. The following rules apply:

-   Tokens can be matched in *any* order
-   The last destination property name must have *all* tokens matched
-   The last source property name must have at least *one* token matched

The loose matching strategy is ideal to use for source and destination object models with property hierarchies that are very dissimilar. It may result in a higher level of ambiguous matches being detected, but for well-known object models it can be a quick alternative to defining [mappings](http://modelmapper.org/user-manual/property-mapping).

### Strict

The strict matching strategy allows for source properties to be strictly matched to destination properties. This strategy allows for complete matching accuracy, ensuring that no mismatches or ambiguity occurs. But it requires that property name tokens on the source and destination side match each other precisely. The following rules apply:

-   Tokens are matched in *strict* order
-   *All* destination property name tokens must be matched
-   *All* source property names must have *all* tokens matched

The strict matching strategy is ideal to use when you want to ensure that no ambiguity or unexpected mapping occurs without having to inspect a `TypeMap`. The drawback is that the strictness may result in some destination properties remaining unmatched.


API Overview
============

The ModelMapper API consists of a few principal types:

-   **ModelMapper**
    -   The class you instantiate to perform object mapping, configure matching, load PropertyMaps and register Mappers
    -   Contains Configuration and TypeMaps
-   **PropertyMap**
    -   The class you extend to define mappings between source and destination properties for a specific pair of types
-   **TypeMap**
    -   The interface you use to perform configuration, introspection and mapping for a specific pair of types
    -   Contains property mappings that are added from a PropertyMap
    -   Created by a ModelMapper
-   **Converter**
    -   The interface you implement to perform custom conversion between two types or property hierarchies
    -   Added to a ModelMapper, set against a TypeMap, or used in a [mapping](http://modelmapper.org/user-manual/property-mapping/#converters).
-   **Provider**
    -   The interface you implement to provide instances of destination types.
    -   Set against a `TypeMap` or used in a [mapping](http://modelmapper.org/user-manual/property-mapping/#providers).
-   **Condition**
    -   The interface you implement to conditionally create a mapping.
    -   Used in a [mapping](http://modelmapper.org/user-manual/property-mapping/#conditional-mapping).

Also see the [Property Mapping](http://modelmapper.org/user-manual/property-mapping/) section of the User's Guide for an overview of the Mapping API.



Integrations
============

ModelMapper was designed to be easily extensible via the [API](http://modelmapper.org/user-manual/api-overview/) and [SPI](http://modelmapper.org/user-manual/spi-overview/) and to integrate well with existing technologies. Several integrations are described below.

General
-------

-   [OSGi](http://modelmapper.org/user-manual/osgi-integration/)

Provider Integrations
---------------------

[Providers](http://modelmapper.org/user-manual/providers) allow you to provide you own instance of destination objects prior to mapping. ModelMapper has several 3rd party integrations that allow for external libraries to provide destination objects:

-   [Spring](http://modelmapper.org/user-manual/spring-integration)
-   [Guice](http://modelmapper.org/user-manual/guice-integration)
-   [Dagger](http://modelmapper.org/user-manual/dagger-integration)

Value Reader Integrations
-------------------------

Value Readers allow you to read and map values from different types of source object, aside from typical JavaBeans. ModelMapper has several 3rd party integrations that allow for the mapping of values from various types of source objects:

-   [jOOQ](http://modelmapper.org/user-manual/jooq-integration)
-   [Jackson](http://modelmapper.org/user-manual/jackson-integration)
-   [Gson](http://modelmapper.org/user-manual/gson-integration)

Native Integrations
-------------------

Certain libraries natively integrate with ModelMapper without any additional dependencies. These include:

-   [JDBI](http://modelmapper.org/user-manual/jdbi-integration)
