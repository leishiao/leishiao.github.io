# Java Client API Guide



The key classes and interfaces  of RabbitMQ are:

- Channel: represents an AMQP 0-9-1 channel, and provides most of the operations (protocol methods).
- Connection: represents an AMQP 0-9-1 connection
- ConnectionFactory: constructs Connection instances
- Consumer: represents a message consumer
- DefaultConsumer: commonly used base class for consumers
- BasicProperties: message properties (metadata)
- BasicProperties.Builder: builder for BasicProperties

