---
description: PHP Messages
---

# Lesson 1: Messaging Concepts

{% hint style="info" %}
Not having code for _Lesson 1?_&#x20;

`git checkout lesson-1`
{% endhint %}

## Key concepts / background

In this first lesson, we will learn fundamental blocks in messaging architecture and we will start building back-end for Shopping System using CQRS. \


_Ecotone_ from the ground is built around messaging to provide a simple model for building integration solutions while maintaining the separation of concerns that is essential for producing maintainable, testable code. To achieve that, fundamental messaging blocks are implemented using [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com) (EIP). In order to use higher level of abstraction, CQRS and DDD support, we need to briefly get known with those patterns. This will help us understand, how specific parts of _Ecotone_ does work together.&#x20;

{% hint style="info" %}
_`Ecotone` _ is heavily inspired by [_Spring Integration_](https://spring.io/projects/spring-integration)_,_ [_Axon Framework_](https://docs.axoniq.io/reference-guide/) _and_ [_NServiceBus_](https://particular.net/nservicebus)_._
{% endhint %}

### Message

![](../.gitbook/assets/message.jpg)

A message is a generic wrapper for any PHP type (scalar, object, compound) with metadata used by the framework while handling that object.\
The payload can be of any type, and the headers (metadata) hold commonly required information such as ID, timestamp and framework specific information. Headers are also used for passing values to and from connected transports. \
Developers can also store any arbitrary key-value pairs in the headers, to pass needed meta information.

```php
interface Message
{
    public function getPayload();

    public function getHeaders() : MessageHeaders;
}
```

### Message Channel

![](../.gitbook/assets/message-channel-connection.svg)

_Message channel_ abstracts communication between components. It does allow for sending and receiving messages.\
A message channel may follow either point-to-point or publish-subscribe semantics. \
With a point-to-point channel, only one consumer can receive each message sent to the channel. \
Publish-subscribe channels, broadcast each message to all subscribers on the channel.&#x20;

_Pollable channels_ extends Message Channels with capability of buffering Messages within a queue. The advantage of buffering is that it allows for throttling the inbound messages and preventing of message loss.&#x20;

### Message Endpoint

![](../.gitbook/assets/endpoint-1.jpg)

Message Endpoints are consumers and producers of messages. Consumer are not necessary asynchronous, as you may build synchronous flow, compound of multiple endpoints. \
You will not have to implement them directly, as you should not even have to build messages and invoke or receive message directly from the [Message channel](php-messaging-architecture.md#message-channel). Instead you will be able to focus on your specific domain model with with an implementation based on plain PHP objects. By providing declarative configuration, you can "connect” your domain-specific code to the messaging infrastructure provided by `Ecotone`.&#x20;

{% hint style="info" %}
If you are familiar with Symfony Messager/Simplebus, for now you can think of Endpoint as a _Message Handler_, that can be connected to asynchronous or synchronous transport.&#x20;
{% endhint %}

### Messaging Gateway

![](../.gitbook/assets/gateway\_execution.svg)

The Messaging Gateway encapsulates messaging-specific code (The code required to send or receive a [Message](php-messaging-architecture.md#message)) and separates it from the rest of the application code.\
It does build from domain specific objects a [Message](php-messaging-architecture.md#message) that is send via [Message channel](php-messaging-architecture.md#message-channel). \
To not have dependency on the `Ecotone` _API_ — including the gateway class, `Ecotone` provides the Gateway as interface. Framework generates a proxy for any interface and internally invokes the gateway methods. By using dependency injection, you can then expose the interface to your business methods.

{% hint style="info" %}
Command/Query/Event buses are implemented using Messaging Gateway.
{% endhint %}

{% hint style="success" %}
Great, now when we know fundamental blocks of `Ecotone` and _Messaging Architecture_, we can start implementing our Shopping System! \
If you did not understand something, do not worry, we will see how does it apply in practice in next step.&#x20;
{% endhint %}

## To The Code!

Do you remember this command from [Setup part](before-we-start-tutorial.md#setup-for-tutorial)?

```php
bin/console ecotone:quickstart
"Running example...
Hello World
Good job, scenario ran with success!"
```

If yes and this command does return above output, then we are ready to go.

{% tabs %}
{% tab title="Symfony" %}
{% code title="https://github.com/ecotoneframework/quickstart-symfony/blob/lesson-1/src/EcotoneQuickstart.php" %}
```php
Go to "src/EcotoneQuickstart.php"

# This class is autoregistered using Symfony Autowire
```
{% endcode %}
{% endtab %}

{% tab title="Laravel" %}
{% code title="https://github.com/ecotoneframework/quickstart-laravel/blob/lesson-1/app/EcotoneQuickstart.php" %}
```php
Go to "app/EcotoneQuickstart.php"

# This class is autoregistered using Laravel Autowire
```
{% endcode %}
{% endtab %}

{% tab title="Lite" %}
```php
https://github.com/ecotoneframework/quickstart-lite/blob/lesson-1/src/EcotoneQuickstart.php

Go to "src/EcotoneQuickstart.php"

# This class is autoregistered using PHP-DI
```
{% endtab %}
{% endtabs %}

this method will be run, whenever we execute_`ecotone:quickstart`_. \
This class is auto-registered using auto-wire system, both [Symfony](https://symfony.com/doc/current/service\_container/autowiring.html) and [Laravel](https://laravel.com/docs/7.x/container) provides this great feature. For `Lite` clean and easy to use [`PHP-DI`](https://github.com/PHP-DI/PHP-DI) is taken.\


Thanks to that, we will avoid writing configuration files for service registrations during this tutorial. \
And we will be able to fully focus on what can `Ecotone` provides to us.&#x20;

```php
<?php

namespace App;

class EcotoneQuickstart
{
    public function run() : void
    {
        echo "Hello World";
    }
}
```

### Command Handler - Endpoint

We will start by creating `Command Handler`.\
Command Handler is place where we will put our business logic. \
Let's create namespace `App\Domain\Product` and inside `RegisterProductCommand,` command for registering new product:

```php
<?php

namespace App\Domain\Product;

class RegisterProductCommand
{
    private int $productId;

    private int $cost;

    public function __construct(int $productId, int $cost)
    {
        $this->productId = $productId;
        $this->cost = $cost;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }

    public function getCost() : int
    {
        return $this->cost;
    }
}
```

{% hint style="info" %}
```php
private int $productId;
```

Describing types, will help us in later lessons with automatic conversion. Just remember right now, that it's worth to keep the types defined.
{% endhint %}

Let's register a Command Handler now by creating class `App\Domain\Product\ProductService`

```php
<?php

namespace App\Domain\Product;

use Ecotone\Modelling\Attribute\CommandHandler;

class ProductService
{
    private array $registeredProducts = [];
    
    #[CommandHandler]
    public function register(RegisterProductCommand $command) : void
    {
        $this->registeredProducts[$command->getProductId()] = $command->getCost();
    }
}
```

First thing worth noticing is `#[CommandHandler]`. \
This [attribute](https://wiki.php.net/rfc/attributes\_v2) marks our `register` method in `ProductService` as an [Endpoint](php-messaging-architecture.md#message-endpoint), from that moment it can be found by `Ecotone.`

`Ecotone` will read method declaration and base on the first parameter type hint will know that this `CommandHandler` is responsible for handling `RegisterProductCommand.`&#x20;

{% hint style="info" %}
`PHP 8 Introduced neat feature called`[`Attributes`](https://wiki.php.net/rfc/attributes\_v2)`You may now know it under the name Annotations. Annotations were userland solution and Attributes are built in into language feature. Ecotone make use of it to decouple your code from the framework.` \
``\
`Thank you Benjamin Eberlei and Martin Schroder for introducing this feature :)`
{% endhint %}

{% hint style="info" %}
`#[ClassReference]` is a special [Attribute](https://wiki.php.net/rfc/attributes\_v2)  it informs `Ecotone`how this service is registered in `Depedency Container`. As a default it takes the class name, which is compatible with auto-wiring system. If `ProductService` would be registered in `Dependency Container` as `productService`, we would do this:

```php
#[ClassReference("productService")
class ProductService
```
{% endhint %}

### Query Handler - Endpoint

We also need possibility to query our `ProductService` for registered products.\
This is the role of `Query Handlers`. They do query the state and return it to us.  \
Let's starts with `GetProductPriceQuery` _class._ This _query_ will tell us what is the price of specific product.

```php
<?php

namespace App\Domain\Product;

class GetProductPriceQuery
{
    private int $productId;

    public function __construct(int $productId)
    {
        $this->productId = $productId;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }
}
```

We also need Handler for this query. Let's add `Query Handler` __ to the `ProductService`

```php
<?php

namespace App\Domain\Product;

use Ecotone\Modelling\Attribute\CommandHandler;
use Ecotone\Modelling\Attribute\QueryHandler;

class ProductService
{
    private array $registeredProducts = [];

    #[CommandHandler]
    public function register(RegisterProductCommand $command) : void
    {
        $this->registeredProducts[$command->getProductId()] = $command->getCost();
    }

    #[QueryHandler] 
    public function getPrice(GetProductPriceQuery $query) : int
    {
        return $this->registeredProducts[$query->getProductId()];
    }
}
```

{% hint style="info" %}
Some CQRS frameworks expects Handlers be defined as a class, not method. This is somehow limiting and producing a lot of boilerplate. `Ecotone` does allow for full flexibility, if you want to have only one handler per class, so be it, if more just annotate next methods.
{% endhint %}

{% tabs %}
{% tab title="Laravel" %}
```php
# As default auto wire of Laravel creates new service instance each time 
# service is requested from Depedency Container, we need to register 
# ProductService as singleton.

# Go to bootstrap/QuickStartProvider.php and register our ProductService

namespace Bootstrap;

use App\Domain\Product\ProductService;
use Illuminate\Support\ServiceProvider;

class QuickStartProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(ProductService::class, function(){
            return new ProductService();
        });
    }
(...)
```
{% endtab %}

{% tab title="Symfony" %}
```php
Everything is set up by the framework, please continue...
```
{% endtab %}

{% tab title="Lite" %}
```
Everything is set up, please continue...
```
{% endtab %}
{% endtabs %}

### Command and Query Bus - Gateways

It's time to call our endpoints.\
You may remember that [endpoints](php-messaging-architecture.md#message-endpoint) need to be connected using [Message Channels](php-messaging-architecture.md#message-channel) and we did not do anything like this yet. Thankfully `Ecotone` does create synchronous channels for us, so we don't need to bother about it.&#x20;

{% hint style="info" %}
In case of Query And Command Handlers, point to point channels are created automatically.\
You will learn how to easily replace them with asynchronous channels in next lessons.
{% endhint %}

We need to create [`Message`](php-messaging-architecture.md#message) and send it to correct [`Message Channel`](php-messaging-architecture.md#message-channel). \
In order to do it we will use [Messaging Gateway](php-messaging-architecture.md#messaging-gateway). Message Gateways are responsible for creating `Message` from given parameters and send it to the correct `channel`.\
For sending Commands we will use `Command Bus Gateway`.\
For sending Queries we will use `Query Bus Gateway`. \
\
Let's inject and call Query and Command bus into EcotoneQuickstart class.&#x20;

```php
<?php

namespace App;

use App\Domain\Product\GetProductPriceQuery;
use App\Domain\Product\RegisterProductCommand;
use Ecotone\Modelling\CommandBus;
use Ecotone\Modelling\QueryBus;

class EcotoneQuickstart
{
    private CommandBus $commandBus;
    private QueryBus $queryBus;

// 1
    public function __construct(CommandBus $commandBus, QueryBus $queryBus)
    {
        $this->commandBus = $commandBus;
        $this->queryBus = $queryBus;
    }

    public function run() : void
    {
// 2    
        $this->commandBus->send(new RegisterProductCommand(1, 100));
// 3
        echo $this->queryBus->send(new GetProductPriceQuery(1));
    }
}
```

1. `Gateways` are auto registered in Dependency Container and available for auto-wire.  `Ecotone` comes with few set up Gateways. Command and Query buses are available instantly to you. You will be able to extend them or create your own ones, if needed.
2. We are sending command`RegisterProductCommand` to the `CommandHandler` we registered before.&#x20;
3. Same as above, but in that case we are sending query `GetProductPriceQuery` to the `QueryHandler`

If you run our testing command now, you should see the result.&#x20;

```php
bin/console ecotone:quickstart
Running example...
100
Good job, scenario ran with success!
```

### Event Handling

We want to notify, when new product is registered in the system. \
In order to do it, we will make use of `Event Bus Gateway` which can publish events.\
Let's start by creating `ProductWasRegisteredEvent.`

```php
<?php

namespace App\Domain\Product;

class ProductWasRegisteredEvent
{
    private int $productId;

    public function __construct(int $productId)
    {
        $this->productId = $productId;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }
}
```

{% hint style="info" %}
As you can see `Ecotone` does not really care what class Command/Query/Event is. It does not require to implement any interfaces neither prefix or suffix the class name.\
In fact commands, queries and events can be of any type and we will see it in next Lessons.\
In the tutorial however we use Command/Query/Event suffixes to clarify the distinction.
{% endhint %}

Let's inject `EventBus` into our `CommandHandler` in order to publish `ProductWasRegisteredEvent` after product was registered.

```php
use Ecotone\Modelling\EventBus;

 #[CommandHandler]
public function register(RegisterProductCommand $command, EventBus $eventBus) : void
{
    $this->registeredProducts[$command->getProductId()] = $command->getCost();

    $eventBus->publish(new ProductWasRegisteredEvent($command->getProductId()));
}
```

You may wonder, how `EventBus` is injected into the Command Handler's method.\
`Ecotone` does control method invocation for [endpoints](php-messaging-architecture.md#message-endpoint), if you have type hinted for specific class, framework will look in Dependency Container for specific service in order to inject it automatically. It does work similar to auto-wire system. If you want to know more, check chapter about [Method Invocation](../messaging/conversion/method-invocation.md).

Now, when our event is published, whenever new product is registered, we want to listen for it and notify. Let's create new class and annotate method with `EventHandler`.&#x20;

```php
<?php

namespace App\Domain\Product;

use Ecotone\Modelling\Attribute\EventHandler;

class ProductNotifier
{
    #[EventHandler] // 1
    public function notifyAbout(ProductWasRegisteredEvent $event) : void
    {
        echo "Product with id {$event->getProductId()} was registered!\n";
    }
}
```

1. `EventHandler` tells `Ecotone` to handle specific event based on declaration type hint, just like with `CommandHandler.` As `Commands` are `point to point,` which means they are targeting one Handler, `Events` on other side are `publish subscribe`, which means, there may be multiple Handlers for specific event.

{% hint style="info" %}
You may use interfaces or abstract classes for your first parameter type hint. \
You may even type hint using [`Union Types`](https://wiki.php.net/rfc/union\_types\_v2) `(ProductWasRegistered|ProductWasSent).` \
This allows for listening to more than one event within specific method, which in result decrease boilerplate in the business code.

Ecotone will resolve connection between published event and Event Handler.
{% endhint %}

If you run our testing command now, you should see the result.&#x20;

```php
bin/console ecotone:quickstart
Running example...
Product with id 1 was registered!
100
Good job, scenario ran with success!
```

{% hint style="success" %}
Great, we have just finished Lesson 1. \
In this lesson we have learned basic of Messaging and CQRS. \
That was the longest lesson, as it had to introduce new concepts. Incoming lessons will be much shorter :)

\
We are ready for Lesson 2!
{% endhint %}

{% content-ref url="php-domain-driven-design.md" %}
[php-domain-driven-design.md](php-domain-driven-design.md)
{% endcontent-ref %}
