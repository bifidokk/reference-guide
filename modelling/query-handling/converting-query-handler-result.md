---
description: Query Result PHP
---

# Converting Query Handler Result

If you have registered [Converter](../../messaging/conversion/) for specific Media Type, then you can tell `Ecotone` to convert result of any [Gateway](../../messaging/messaging-concepts/messaging-gateway.md) to specific format. This is especially useful, when we are dealing with `QueryBus`, when we want to return the result to the caller of the request. \
In order to do this, we need to make use of `Metadata`and `replyContentType` header.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request) : Response
   {
      return new Response(
         $this->queryBus->sendWithMetadata(
            new GetTicketStatusQuery($request->get("ticketId")),
            expectedReturnedMediaType: "application/json"
         );
      )    
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
$queryBus = $messagingSystem->getQueryBus();

// result will be in json
$result = $queryBus->sendWithMetadata(
             new GetTicketStatusQuery($ticketId),
             expectedReturnedMediaType: "application/json"
          );
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
class GetTicketStatusQueryHandler
{   
    #[QueryHandler]
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
        return ["ticketId" => $query->getTicketId(), "status" => "inProgress"];
    }   
}
```
{% endtab %}
{% endtabs %}
