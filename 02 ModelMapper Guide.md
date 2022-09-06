
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
