#Publisher/Subscriber example with Spring Cloud Stream
This example is composed of two services, the publisher and the subscriber. The former publishes
the current date and time to a stream every second, while the later receives the messages and
prints them on the console. In the example the publisher is called **Source** while the 
subscriber is called **Sink**. 

It is used [RabbitMQ](https://www.rabbitmq.com) as the message broker. It handles a queue 
that is created with the first *Sink*. Note that if the *Source* is created before any *Sink*
 the messages will be lost until the first *Sink* is created. However, once a *Sink* is created
 the queue is persistent. That is, even if there aren't any *Sinks* messages will be stored
 by the broker.
 
##Project Structure
There are two modules one for the publisher called *Source* and another for the subscriber
called *Sink*. These modules are independent application that can be build and run independently.

They can be run from the IDE as we usually do. Note that you can edit the configuration for 
running them and add VM options.

It is also possible and probably advisable to compile them from outside using maven. Go to 
a console window. Navigate to <project_directory>/sink or <project_directory>/source and 
execute `./mvnw clear package `
. It will create a jar file in a directory called *target*

##Running the example
Run the services in the following order:
1. Start the RabbitMQ broker
    * `./rabbitmq-server -detached ` to run the broker
    * `./rabbitmqctl status` to see its status
    * `./rabbitmqctl stop` to stop it
    * You can monitor the broker in the following address: `http://localhost:15672/`
2. Start one or more *Sink* services
    * `java -jar sink/target/sink-0.0.1-SNAPSHOT.jar` to run the first one with default parameters
    * `java -Dserver.port=8082 -jar sink/target/sink-0.0.1-SNAPSHOT.jar` to run a second one
    in port 8082 and with the same *consumer group*
    * `java -Dserver.port=8083 -Dspring.cloud.stream.bindings.input.group=other -jar sink/target/sink-0.0.1-SNAPSHOT.jar
` to run a third one in a different port and different consumer group
3. Start one single source
    * `java -jar source/target/source-0.0.1-SNAPSHOT.jar
    `
    
##Observe the behaviour
Assuming that the *broker* and the *source* are stated:

* When the first *sink* is started  (as described above) it begins to consume all the messages
* When the second *sink* is started (as described above) it consumes approximately half the messages.
The other half is consumed by the first *sink*. This is because both of them are in the same *consumer
group*
* When the third *sink* is started (as described above) the other *sinks* don't change their
behaviour and the third *sink* gets all the messages
* When at least a *sink* is created and then all are stopped the messages are enqueued by the 
broker. It the a *sink* is run it consumes all the past non consumed messages.
* If the broker is stopped while it had messages stored in the queue they aren't lost. When it
is run again the *sink* receives those messages. Obviously the messages sent by the *source*
while the *broker* was down are lost

##Credits
* This example is based (almost copied) from: https://github.com/spring-cloud/spring-cloud-stream-samples






