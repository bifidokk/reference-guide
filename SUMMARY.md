# Table of contents

* [Introduction](README.md)
* [Installation](install-php-service-bus.md)
* [Quick Start - DDD CQRS PHP](quick-start-php-ddd-cqrs-event-sourcing/README.md)
  * [CQRS PHP](quick-start-php-ddd-cqrs-event-sourcing/php-cqrs.md)
  * [Event Handling PHP](quick-start-php-ddd-cqrs-event-sourcing/php-event-handling.md)
  * [Asynchronous PHP](quick-start-php-ddd-cqrs-event-sourcing/asynchronous-php.md)
  * [Event Sourcing PHP](quick-start-php-ddd-cqrs-event-sourcing/event-sourcing-php.md)
  * [Microservices PHP](quick-start-php-ddd-cqrs-event-sourcing/microservices-php.md)
* [Tutorial](tutorial-php-ddd-cqrs-event-sourcing/README.md)
  * [Before we start tutorial](tutorial-php-ddd-cqrs-event-sourcing/before-we-start-tutorial.md)
  * [Lesson 1: Messaging Concepts](tutorial-php-ddd-cqrs-event-sourcing/php-messaging-architecture.md)
  * [Lesson 2: Tactical DDD](tutorial-php-ddd-cqrs-event-sourcing/php-domain-driven-design.md)
  * [Lesson 3: Converters](tutorial-php-ddd-cqrs-event-sourcing/php-serialization-deserialization.md)
  * [Lesson 4: Metadata and Method Invocation](tutorial-php-ddd-cqrs-event-sourcing/php-metadata-method-invocation.md)
  * [Lesson 5: Interceptors](tutorial-php-ddd-cqrs-event-sourcing/php-interceptors-middlewares.md)
  * [Lesson 6: Asynchronous Handling](tutorial-php-ddd-cqrs-event-sourcing/php-asynchronous-processing.md)

## Modelling

* [Overview](modelling/modelling-1.md)
* [Command Handling](modelling/command-handling/README.md)
  * [External Command Handlers](modelling/command-handling/external-command-handlers.md)
  * [State-Stored Aggregate](modelling/command-handling/state-stored-aggregate.md)
  * [Event Sourcing Aggregate](modelling/command-handling/event-sourcing-aggregate.md)
  * [Repository](modelling/command-handling/repository.md)
  * [Dispatching Commands](modelling/command-handling/command-dispatching.md)
* [Query Handling](modelling/query-handling/README.md)
  * [Handling Queries](modelling/query-handling/handling-queries.md)
  * [Dispatching Queries](modelling/query-handling/dispatching-queries.md)
  * [Converting Query Handler Result](modelling/query-handling/converting-query-handler-result.md)
* [Event Handling](modelling/event-handling/README.md)
  * [Handling Events](modelling/event-handling/handling-events.md)
  * [Dispatching Events](modelling/event-handling/dispatching-events.md)
* [Interceptors](modelling/interceptors.md)
* [Saga](modelling/saga.md)
* [Asynchronous Handling](modelling/asynchronous-handling.md)
* [Event Sourcing](modelling/event-sourcing.md)
* [Scheduling](modelling/scheduling.md)
* [Microservices PHP](modelling/microservices-php.md)

## Messaging and Ecotone In Depth <a href="#messaging" id="messaging"></a>

* [Overview](messaging/overview.md)
* [Messaging concepts](messaging/messaging-concepts/README.md)
  * [Message](messaging/messaging-concepts/message.md)
  * [Message Channel](messaging/messaging-concepts/message-channel.md)
  * [Message Endpoint](messaging/messaging-concepts/message-endpoint/README.md)
    * [Service Activator](messaging/messaging-concepts/message-endpoint/service-activator.md)
    * [Message Router](messaging/messaging-concepts/message-endpoint/message-routing.md)
    * [Splitter](messaging/messaging-concepts/message-endpoint/splitter.md)
  * [Consumer](messaging/messaging-concepts/consumer.md)
  * [Messaging Gateway](messaging/messaging-concepts/messaging-gateway.md)
* [Method Invocation And Conversion](messaging/conversion/README.md)
  * [Method Invocation](messaging/conversion/method-invocation.md)
  * [Conversion](messaging/conversion/conversion.md)
* [Service (Application) Configuration](messaging/service-application-configuration.md)

## Modules

* [Overview](modules/overview.md)
* [Symfony Bundle](modules/symfony-ddd-cqrs-event-sourcing.md)
* [Laravel](modules/laravel-ddd-cqrs-event-sourcing.md)
* [Ecotone Lite](modules/ecotone-lite.md)
* [JMS Converter](modules/jms-converter.md)
* [RabbitMQ Support](modules/amqp-support-rabbitmq.md)
* [DBAL Support](modules/dbal-support.md)

## Other

* [Contact](other/contact.md)
* [Support ](other/support.md)
* [Prooph Board - Remote Event Storming](partners/inspectio.md)
