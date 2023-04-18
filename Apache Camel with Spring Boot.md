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

**CamelContext**. 
CamelContext is a type of container also We can think of it as Camels runtime system, which keeps everything together.

**Routing Engine**
The routing engine in Camel is responsible for moving messages behind the hood. And we should be aware that this exists and that it handles all of the heavy lifting, ensuring that messages are routed correctly.

**Processor**
The processor is a fundamental Camel notion that represents a node that can use, create, and alter an incoming exchange. Exchanges go from one processor to another during routing; as a result, you can think of a route as a graph with specialised processors as nodes and lines connecting one processor's output to another's input.



# Ref.
https://www.youtube.com/playlist?list=PLK0V_H0fCvPilK2_-WMwahm7HYB0XgJx7
