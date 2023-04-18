# Apache Camel SpringBoot Integration

## Apache Camel Intro
Apache Camel is an open-source enterprise integration framework and allows the users to easily integrate systems that are consuming or producing data. 

Apache Camel helps you to integrate with a variety of other applications. For example, you can integrate with Kafka, ActiveMQ also we can integrate with applications that use JMS furthermore You can make HTTP calls, and you can talk to cloud services like AWS lambda. Apache Camel uses component architecture which makes it lightweight.

There are hundreds of different components which are provided for different databases i.e messages, queues, APIs, and cloud integrations. Apache Camel also supports 200 plus protocols, transports, data formats, and 300 plus converters between these data formats and It provides a domain-specific language (DSL) which can be customized to the needs of application integrations.

![image](https://user-images.githubusercontent.com/20472904/232687061-6616becf-069d-4b8c-a3cf-a84f664bddbd.png)

Apache Camel, we will be having 3 different components i.e **Route, Filter and Process**. Camel supports 3 things i.e Routing, Filtering and processing. It also supports a variety of protocols.

**Route**: It is the process of routing (transferring) the data from system 1 to system 2. File system to file system, File System to MQ, File System to DB. Likewise, MQ to File System, MQ to MQ and MQ to DB and so on.

**Filter**: If you want to remove invalid data or you want to avoid some particular data, we can just use filter. Additionally, you just have to check the condition before transfer.

**Process**: We can do some conversions/modifications or calculations. (XML->json, text->json etc.). Data is sent from one system to another via Camel integration and it also supports various protocols which makes the transfer process easier and less complex.


Some Important Camel Concepts
-----------------------------
<img width="666" alt="image" src="https://user-images.githubusercontent.com/20472904/232687794-dede6305-1db1-4fd2-9e39-ba2850898deb.png">

-   **Message** contains data which is being transferred to a route. Each message has a unique identifier and it's constructed out of a body, headers, and attachments
-   **Exchange** is the container of a message and it is created when a message is received by a consumer during the routing process. Exchange allows different types of interaction between systems -- it can define a one-way message or a request-response message
-   **Endpoint** is a channel through which system can receive or send a message. It can refer to a web service URI, queue URI, file, email address, etc
-   **Component** acts as an endpoint factory. To put it simply, components offer an interface to different technologies using the same approach and syntax. Camel already supports [a lot of components](http://camel.apache.org/components.html) in its DSLs for almost every possible technology, but it also gives the ability for writing custom components
-   **Processor** is a simple Java interface which is used to add custom integration logic to a route. It contains a single *process* method used to preform custom business logic on a message received by a consumer


### 1. Apache Camel :: Hello World
```java
import org.apache.camel.CamelContext;
import org.apache.camel.impl.DefaultCamelContext;

public class ApacheCamelUtil {

    public static void main(String[] args) throws Exception {
        //1.Create CamelContext
        CamelContext camelContext = new DefaultCamelContext();

        //2.Create Routes by extending RouteBuilder 
        camelContext.addRoutes(new ApacheCamelRoutes());
                
        //3.Start the CamelContext
        camelContext.start();
    }
}
```

Create Routes by extending RouteBuilder 
```java
public class ApacheCamelRoutes extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        System.out.println(" Hello, Apache Camel");
    }
}
```

Once you run main method, strat method will call route & prints 
`Hello, Apache Camel`

  
  
  
### 2. FileCopy Example
```java
    public static void fileCopy() throws Exception {
        CamelContext context = new DefaultCamelContext();
        context.addRoutes(new RouteBuilder() {
            @Override
            public void configure() throws Exception {
                from("file:input_box?noop=true")
                .to("file:output_box");
            }
        });

        while (true)
            context.start();
    }

```

  
  
### 3. Producer Consumer Example
```java
    public static void producerConsumer() throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            public void configure() throws Exception {
                from("direct:start").to("seda:end");
            }
        });

        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start", "Hello Everyone");

        ConsumerTemplate consumerTemplate = context.createConsumerTemplate();
        String message = consumerTemplate.receiveBody("seda:end", String.class);
        System.out.println(message);

    }

```



### 4. Producer Consumer with Process
```java
    public static void producerConsumerProcess() throws Exception {
        CamelContext context = new DefaultCamelContext();

        context.addRoutes(new RouteBuilder() {
            public void configure() throws Exception {
                from("direct:start").process(new Processor() {
                    @Override
                    public void process(Exchange exchange) throws Exception {
                        String message = exchange.getIn().getBody(String.class);
                        message = message + "-By Satya \n \n \n *************** \n \n \n";
                        exchange.getOut().setBody(message);
                    }
                }).to("seda:end");
            }
        });
        context.start();

        ProducerTemplate producerTemplate = context.createProducerTemplate();
        producerTemplate.sendBody("direct:start", "Hello Everyone");

        ConsumerTemplate consumerTemplate = context.createConsumerTemplate();
        String message = consumerTemplate.receiveBody("seda:end", String.class);
        System.out.println(message);
    }


```


### 5. ActiveMQ consume message
```java
    public static void consumeMQmessage() throws Exception {
        CamelContext context = new DefaultCamelContext();

        ConnectionFactory connectionfactory = new ActiveMQConnectionFactory();
        context.addComponent (" jms", JmsComponent.jmsComponentAutoAcknowledge(connectionfactory));
        context.addRoutes(new RouteBuilder() {
                    public void configure() throws Exception {
                        from("activemq:queue:my_queue")
                                .to("seda:end");
                    }});
         context.start();

        ConsumerTemplate consumerTemplate = context.createConsumerTemplate();
        String message = consumerTemplate.receiveBody("seda:end", String.class);
        System.out.println(message);
        }

```


# SpringBoot + Apache Camel Integration

I am going to create a rest controller class using camel Java DSL without using spring MVC. If we create a rest controller using camel’s route, we have to extend our rest controller class using **RouteBuilder**. Once we extend the rest controller from RouteBuilder we have to override **configure()** method. We define all our routes in the configuration method like below.

```

```




```

```




```

```




```

```




```

```
















# Ref.
https://www.youtube.com/playlist?list=PLK0V_H0fCvPilK2_-WMwahm7HYB0XgJx7. 

https://medium.com/@naskavinda/building-spring-boot-rest-api-using-apache-camel-and-postgresql-ead210c92503
