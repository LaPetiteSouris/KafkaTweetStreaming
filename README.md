# Stream-processing data service

Modeling a simple light way service with fan-out capacity to distributed
work load over Kafka. This is suitable for fast stream-processing of a large
amount of incoming data.

Inspired by this post: </br>

  https://medium.com/@stephane.maarek/how-to-use-apache-kafka-to-transform-a-batch-pipeline-into-a-real-time-one-831b48a6ad85

I made up a small simplistic architecture to stream data from REST API to other services thanks to Kafka for
fast stream-processing. The architecture proposed by the author of the article aboved.

Here the REST API is a simple ExpressJS API and the streaming job is also
a simple Java application which utilizes Kafka Stream

![screenshot](https://user-images.githubusercontent.com/6369285/40370279-32d63c88-5de0-11e8-9dc1-a5c64e942361.jpg)

## How does it work ?

* `POST /message`: Put `{content: "mycontent is ..."}` into the post message
and the content will be relayed to a kafka topic (default is `fwd`). User will get 200 with ACK of message reception.

* The message is later processed in near real time
thanks to Kafka and notably Kafka Stream app (written in Java) to process the messsage.
In this simple application, any message which contains bad keywords will be filtered out. The
filtered message is then streamed back to Kafka topics.
* Another application
using Kafka Sink with DB connection will be in charge of writing all filtered messages
back to database for analyzing purpose.

## How to try it ?

1. You will need Kafka installed and running. For simplicity, I used Confluent package (https://docs.confluent.io/current/cli/command-reference/confluent-start.html) . It is prepacked with a single-node Kafka cluster ready for launching.

2. in `node-fanout-message`, install all required dependencies with `npm install`, then launch the simplified ExpressJS server with `node server.js`. Make sure that Kafka configuration is correct, else the server won't be able to communicate with Kafa. If you use Confluent, then it is pre-configured with default values.

3. Build and launch the java application in `kafka_stream_java`, also notice the Kafka default configuration.

4. Start sending JSON message with POST to the API endpoint. (default at `localhost:3000/message`). 

5. You should be able to see valid message (without keyword `sex, porn, tits`) on Kafka topic named `forward_valid` and invalid message on `forward_invalid` topic.

## Todo

For now Kafka Sink is undone. It should subscribe to `forward_invalid` and `forward_valid` topic and writes respective data to the SQL database.

