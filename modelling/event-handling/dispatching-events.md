---
description: Events PHP
---

# Dispatching Events

Previous pages provide the background on how to handle event messages in your application. The dispatching process is the starting point for event message.\
You will mostly send events after successfully handling [Command Message](../command-handling/), examples will base on that assumption.

{% hint style="info" %}
You can inject EventBus and send events wherever necessary. Ecotone tries not to impose any  specific solutions.
{% endhint %}

## Event Bus

`Event Bus` is special type of Messaging Gateway.&#x20;

```php
namespace Ecotone\Modelling;

interface EventBus
{
    // 1
    public function publish(object $event, array $metadata = []) : void;
    // 2
    public function publishWithRouting(string $routingKey, mixed $event = [], string $eventMediaType = MediaType::APPLICATION_X_PHP, array $metadata = []) : void;
}
```

### Publishing&#x20;

Event is routed to the Handler by class type.

{% tabs %}
{% tab title="Command Handler" %}
```php
class CloseTicketCommandHandler
{   
    #[CommandHandler]
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
       $eventBus->publish(new TicketWasClosed($command->getTicketId())); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
class NotificationService
{   
    #[EventHandler]
    public function closeTicket(TicketWasClosed $event) : void
    {
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

### Publishing with routing

Is used with `Command Handlers,`routed by name and converted using [Converter](../../messaging/conversion/) if needed.\
Sending events by name instead of class type, may be found useful in integration with external application, when events are in different Media Type than PHP class. &#x20;

{% tabs %}
{% tab title="Command Handler" %}
```php
class CloseTicketCommandHandler
{   
    #[CommandHandler]
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
       $eventBus->publishWithRouting(
          "ticket.wasClosed", 
          '{"ticketId": 123}',
          "application/json"
       ); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
class NotificationService
{   
    #[EventHandler("ticket.wasClosed")]
    public function closeTicket(TicketWasClosed $event) : void
    {
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
JSON will be automatically converted to specific `class type hinted` in method declaration of Event Handler. You could also use in here simple `array` if you have `JSON` to `array` [Converter](../../messaging/conversion/) or a `string`, if you like to receive `JSON string`.
{% endhint %}

## Publishing from Aggregate

`Ecotone` does provide possibility to automatically gather events from `Aggregate` and publish them using `EventBus.`&#x20;

### State-Stored Aggregate

To tell `Ecotone`, which method it should use for retrieving Event objects when using [State-Stored Aggregate](../command-handling/state-stored-aggregate.md#state-stored-aggregate) mark method containing events with annotation `@AggregateEvents.` After handling `Command` or `Event` on `Aggregate` events will be published.&#x20;

```php
 #[AggregateEvents]
public function getRecordedEvents() : array
```

&#x20;If you do want to bother with implementation you can make use of trait `WithAggregateEvent`If you want to record event for publication just use `record`method.

```php
$this->recordThat(new OrderWasPlaced());
```

Events will be automatically retrieved and published after handling current Command or Event.

### Event Sourcing Aggregate

When using [Event Sourcing Aggregate](../event-sourcing.md#event-sourced-aggregate) you do not need to do anything extra. Each method should return events after handling, those events will automatically published using `Event Bus.`

```php
public function assignWorker(AssignWorkerCommand $command) : array
```

{% hint style="info" %}
Events published from aggregate, are publish by class name and routing, if event is named.\
\
`#[NamedEvent("order_was_placed")]`\
`class OrderWasPlaced`
{% endhint %}
