---
description: Transactions, Asynchronous, Dead Letter Queue PHP DBAL
---

# DBAL Support

## Installation

```php
composer require ecotone/dbal
```

### Module Powered By

Powered by powerful database abstraction layer [Doctrine/Dbal](https://github.com/doctrine/dbal) and [Enqueue](https://php-enqueue.github.io) for asynchronous communication&#x20;

## Configuration

In order to use `Dbal Support` we need to add `ConnectionFactory` to our `Dependency Container.`&#x20;

### Using Database Connection String

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
    Enqueue\Dbal\DbalConnectionFactory:
        class: Enqueue\Dbal\DbalConnectionFactory
        arguments: ["pgsql://user:password@host:5432/db_name"]
```
{% endtab %}

{% tab title="Laravel" %}
```php
# Register Service in Provider

use Enqueue\Dbal\DbalConnectionFactory;

public function register()
{
     $this->app->singleton(DbalConnectionFactory::class, function () {
         return new DbalConnectionFactory('pgsql://user:password@host:5432/db_name');
     });
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We register our `DbalConnectionFactory` under the class name `Enqueue\Dbal\DbalConnectionFactory`. This will help Ecotone resolve it automatically, without any additional configuration.
{% endhint %}

### Using Existing Connection

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
    Enqueue\Dbal\DbalConnectionFactory:
        factory: ['Ecotone\Dbal\DbalConnection', 'create']
        arguments: ["@Doctrine\DBAL\Connection"]
```
{% endtab %}

{% tab title="Laravel" %}
```php
# Register Service in Provider

use Enqueue\Dbal\DbalConnectionFactory;
use Ecotone\Dbal\DbalConnection;

public function register()
{
     $this->app->singleton(DbalConnectionFactory::class, function ($app) {
         return new DbalConnection::create(app("Doctrine\DBAL\Connection"));
     });
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We register our `DbalConnectionFactory` under the class name `Enqueue\Dbal\DbalConnectionFactory`. This will help Ecotone resolve it automatically, without any additional configuration.
{% endhint %}

### Using Manager Registry

If we want to make use of existing connection using `Manager Registry`, we can do it this way

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
    Enqueue\Dbal\DbalConnectionFactory:
        factory: ['Ecotone\Dbal\DbalConnection', 'createForManagerRegistry']
        arguments: ["@doctrine","default"]

Register Manager Registry under DbalConnectionFactory, if you want to make use of auto configuration. 
Otherwise you will need to tell Message Channel, Transactions the name of Connection Factory.
```
{% endtab %}
{% endtabs %}

## Doctrine ORM Support

If you have set up [Manager Registry Connection](dbal-support.md#using-manager-registry), you may enable support for Doctrine ORM Repositories.

```php
    #[ServiceContext]
    public function getDbalConfiguration(): DbalConfiguration
    {
        return DbalConfiguration::createWithDefaults()
            ->withDoctrineORMRepositories(true);
    }
```

## Message Channel

To create Dbal Backed [Message Channel](../modelling/asynchronous-handling.md), we need to create [Service Context](../messaging/service-application-configuration.md).&#x20;

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function orderChannel()
    {
        return DbalBackedMessageChannelBuilder::create("orders");
    }
}
```

Now `orders` channel will be available in our Messaging System.&#x20;

## Transactions

`Ecotone Dbal` comes with support for transactions.  \
To enable transactions on specific endpoint, mark it with `Ecotone\Dbal\DbalTransaction\DbalTransaction` annotation.

```php
    #[CommandHandler]
    #[DbalTransaction] 
    public function sellProduct(SellProduct $command) : void
    {
        // do something with $command
    }
```

By default `Ecotone`enables transactions for all [Asynchronous Endpoints](../tutorial-php-ddd-cqrs-event-sourcing/php-asynchronous-processing.md) and Command Bus. You may use of [`Service Context`](../messaging/service-application-configuration.md) to turn off this configuration. You may also add more connections to be handled.

```php
class DbalConfiguration
{
    #[ServiceContext]
    public function registerTransactions() : array
    {
        return [
            DbalConfiguration::createWithDefaults()
                ->withTransactionOnAsynchronousEndpoints(true)
                ->withTransactionOnCommandBus(true)
                ->withDefaultConnectionReferenceNames([
                    "Enqueue\Dbal\DbalConnectionFactory",
                    "AnotherDbalConnectionFactory"
                ])
        ];
    }

}
```

## Dead Letter

### Enable Dead Letter

Using [Service Context](../messaging/service-application-configuration.md)

```php
#[ServiceContext]
public function dbalConfiguration()
{
    return DbalConfiguration::createWithDefaults()
        ->withDeadLetter(true);
}
```

Dbal comes with full support for Dead Letter. You can [read more about it here](../modelling/asynchronous-handling.md#storing-retrying-failed-messages).

Setting up, Dbal Dead Letter as final error channel:

```php
#[ServiceContext]
public function errorConfiguration() {
    return ErrorHandlerConfiguration::createWithDeadLetterChannel(
        "errorChannel",
        RetryTemplateBuilder::exponentialBackoff(1000, 10)
            ->maxRetryAttempts(3),
        "dbal_dead_letter"
    );
}
```

In above scenario, message after failing 3 times, will be stored in database for future investigation.

## Dead Letter Console Commands

### Help

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:help
```
{% endtab %}

{% tab title="Laravel" %}
```
artisan ecotone:deadletter:help
```
{% endtab %}
{% endtabs %}

### Listing Error Messages

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:list
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:deadletter:list
```
{% endtab %}

{% tab title="Lite" %}
```php
$list = $messagingSystem->runConsoleCommand("ecotone:deadletter:list", []);
```
{% endtab %}
{% endtabs %}

### Show Details About Error Message

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:show {messageId}
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:deadletter:show {messageId}
```
{% endtab %}

{% tab title="Lite" %}
```php
$list = $messagingSystem->runConsoleCommand("ecotone:deadletter:show", ["messageId" => $messageId]);
```
{% endtab %}
{% endtabs %}

### Replay Error Message

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:replay {messageId}
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:deadletter:replay {messageId}
```
{% endtab %}

{% tab title="Lite" %}
```php
$list = $messagingSystem->runConsoleCommand("ecotone:deadletter:replay", ["messageId" => $messageId]);
```
{% endtab %}
{% endtabs %}

### Replay All Messages

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:replayAll
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:deadletter:replayAll
```
{% endtab %}

{% tab title="Lite" %}
```php
$list = $messagingSystem->runConsoleCommand("ecotone:deadletter:replayAll", []);
```
{% endtab %}
{% endtabs %}

### Delete Message

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:deadletter:delete {messageId}
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:deadletter:delete {messageId}
```
{% endtab %}

{% tab title="Lite" %}
```php
$list = $messagingSystem->runConsoleCommand("ecotone:deadletter:delete", ["messageId" => $messageId]);
```
{% endtab %}
{% endtabs %}

###
