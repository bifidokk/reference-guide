---
description: Asynchronous PHP RabbitMQ
---

# RabbitMQ Support

## Installation

```php
composer require ecotone/amqp
```

### Module Powered By

[Enqueue](https://github.com/php-enqueue/enqueue-dev) solid and powerful abstraction over asynchronous queues.

## Configuration

In order to use `AMQP Support` we need to add `ConnectionFactory` to our `Dependency Container.`&#x20;

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
# You need to have RabbitMQ instance running on your localhost, or change DSN
    Enqueue\AmqpExt\AmqpConnectionFactory:
        class: Enqueue\AmqpExt\AmqpConnectionFactory
        arguments:
            - "amqp://guest:guest@localhost:5672//"
```
{% endtab %}

{% tab title="Laravel" %}
```php
# Register AMQP Service in Provider

use Enqueue\AmqpExt\AmqpConnectionFactory;

public function register()
{
     $this->app->singleton(AmqpConnectionFactory::class, function () {
         return new AmqpConnectionFactory("amqp+lib://guest:guest@localhost:5672//");
     });
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We register our AmqpConnection under the class name `Enqueue\AmqpExt\AmqpConnectionFactory.` This will help Ecotone resolve it automatically, without any additional configuration.
{% endhint %}

## Message Channel

To create AMQP Backed [Message Channel](../modelling/asynchronous-handling.md) (RabbitMQ Channel), we need to create [Service Context](../messaging/service-application-configuration.md).&#x20;

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function orderChannel()
    {
        return AmqpBackedMessageChannelBuilder::create("orders");
    }
}
```

Now `orders` channel will be available in our Messaging System.&#x20;

## Distributed Publisher and Consumer

To create [distributed publisher or consumer](../modelling/microservices-php.md) provide [Service Context](../messaging/service-application-configuration.md).

### Distributed Publisher

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function distributedPublisher()
    {
        return AmqpDistributedBusConfiguration::createPublisher();
    }
}
```

### Distributed Consumer

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function distributedPublisher()
    {
        return AmqpDistributedBusConfiguration::createConsumer();
    }
}
```

## Custom Message Publisher

### Available actions

```php
interface MessagePublisher
{

    // 1
    public function send(string $data, string $sourceMediaType = MediaType::TEXT_PLAIN) : void;

    // 2
    public function sendWithMetadata(string $data, array $metadata, string $sourceMediaType = MediaType::TEXT_PLAIN) : void;

    // 3
    public function convertAndSend(object|array $data) : void;


    // 4
    public function convertAndSendWithMetadata(object|array $data, array $metadata) : void;
}
```

1. `send` - Send a `string type` via Publisher. It does not need any conversion, you may add additional `Media Type` of `$data`.
2. `sendWithMetadata` - Does the same as `send,` allows for sending additional [Meta data](../tutorial-php-ddd-cqrs-event-sourcing/php-metadata-method-invocation.md#metadata).
3. `convertAndSend` - Allow for sending types, which needs conversion. Allow for sending objects and array, `Ecotone` make use of [Conversion system](../messaging/conversion/conversion.md) to convert `$data`.
4. `convertAndSendWithMetadata` - Does the same as `convertAndSend,` allow for sending additional [Meta data](../tutorial-php-ddd-cqrs-event-sourcing/php-metadata-method-invocation.md#metadata).

### Configuration

If you want to publish Message directly to Exchange, you may use of `Publisher.`

```php
class AMQPConfiguration
{
    #[ServiceContext] 
    public function registerAmqpConfig()
    {
        return 
            AmqpMessagePublisherConfiguration::create(
                MessagePublisher::class, // 1
                "delivery", // 2
                "application/json" // 3
            );
    }
}
```

1. `Reference name` -  Name under which it will be available in Dependency Container.
2. `Exchange name` - Name of exchange where Message should be publisher
3. `Default Conversion [Optional]` - Default type, payload will be converted to.

Publisher is a special type of [Gateway](../messaging/messaging-concepts/messaging-gateway.md), which implements [Publisher interface](amqp-support-rabbitmq.md#available-actions).\
It will be available in your Dependency Container under passed `Reference name.`\
In case interface name `MessagePublisher:class` is used, it will be available using auto-wire.

```php
#[EventHandler] 
public function whenOrderWasPlaced(OrderWasPlaced $event, MessagePublisher $publisher) : void
{
    $publisher->convertAndSendWithMetadata(
        $event,
        [
            "system.executor" => "Johny"
        ]
    );
}
```

### Additional Publisher Configuration

```php
AmqpMessagePublisherConfiguration::create(
    MessagePublisher::class,
    "delivery"
)
    ->withDefaultPersistentDelivery(true) // 1
    ->withDefaultRoutingKey("someKey") // 2
    ->withRoutingKeyFromHeader("routingKey") // 3
    ->withHeaderMapper("application.*") // 4
```

1. `withDefaultPersistentDelivery` - should AMQP messages be `persistent`_._
2. `withDefaultRoutingKey` - default routing key added to AMQP message&#x20;
3. `withRoutingKeyFromHeader` - should routing key be retrieved from header with name
4. `withHeaderMapper` - On default headers are not send with AMQP message. You map provide mapping for headers that should be mapped to `AMQP Message`

## Consumer

To connect consumer directly to a AMQP Queue, we need to provide `Ecotone` with information, how the Queue is configured.&#x20;

```php
class AmqpConfiguration
{
    #[ServiceContext] 
    public function registerAmqpConfig(): array
    {
        return [
            AmqpQueue::createWith("orders"), // 1
            AmqpExchange::createDirectExchange("system"), // 2
            AmqpBinding::createFromNames("system", "orders", "placeOrder"), // 3
            AmqpMessageConsumerConfiguration::create("consumer", "orders") // 4
        ];
    }
}
```

1. `AmqpQueue::createWith(string $name)` - Registers Queue with specific name
2. `AmqpExchange::create*(string $name)` - Registers of given type with specific name
3. `AmqpBinding::createFromName(string $exchangeName, string $queueName, string $routingKey)`- Registering binding between exchange and queue
4. Provides Consumer that will be registered at given name `"consumer"` and will be polling `"orders"` queue

When we do have registered configuration, we can register Consumer for specific queue.

```php
class Consumer
{
    #[MessageConsumer("consumer")]
    public function execute(string $message) : void
    {
        // do something with Message
        // if you have converter registered you can type hint exact type you expect
    }
}
```

## Publisher Transactions

`Ecotone AMQP` comes with support for RabbitMQ Transaction for published messages. \
To enable transactions on specific endpoint, mark it with `Ecotone\Amqp\AmqpTransaction\AmqpTransaction` annotation.

```php
    #[AmqpChannelAdapter(endpointId: "amqp_consumer",queueName: "queue_name") 
    #[AmqpTransaction)
    public function execute(string $message) : void
    {
        // do something with Message
    }
```

If you want to enable/disable for all [Asynchronous Endpoints](../tutorial-php-ddd-cqrs-event-sourcing/php-asynchronous-processing.md) or specific for Command Bus. You may use of `ServiceContext.`&#x20;

{% hint style="info" %}
By default all transactions are enabled
{% endhint %}

```php
class ChannelConfiguration
{
    #[ServiceContext]
    public function registerTransactions() : array
    {
        return [
            AmqpConfiguration::createWithDefaults()
                ->withTransactionOnAsynchronousEndpoints(false)
                ->withTransactionOnCommandBus(false)
        ];
    }

}
```
