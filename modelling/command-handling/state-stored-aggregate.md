---
description: Aggregate PHP
---

# State-Stored Aggregate

This chapter will cover the basics on how to implement an [Aggregate](../modelling-1.md#aggregates). For more details on what an Aggregate is read the [DDD and CQRS concepts](../modelling-1.md#ddd-and-cqrs-concepts) page.

### State-Stored Aggregate

An Aggregate is a regular object, which contains state and methods to alter that state. It can be described as Entity, which carry set of behaviours. \
When creating the Aggregate object, you are creating the _Aggregate Root_.&#x20;

```php
 #[Aggregate] // 1
class Product
{
    #[AggregateIdentifier] // 2
    private string $productId;

    private string $name;

    private integer $priceAmount;
    
    private function __construct(string $orderId, string $name, int $priceAmount)
    {
        $this->productId = $orderId;
        $this->name = $name;
        $this->priceAmount = $priceAmount;
    }

    #[CommandHandler]  //3
    public static function register(RegisterProductCommand $command) : self
    {
        return new self(
            $command->getProductId(),
            $command->getName(),
            $command->getPriceAmount()
        );
    }
    
    #[CommandHandler] // 4
    public function changePrice(ChangePriceCommand $command) : void
    {
        $this->priceAmount = $command->getPriceAmount();
    }
}
```

1. `Aggregate` tells Ecotone, that this class should be registered as Aggregate Root.
2.  `AggregateIdentifier` is the external reference point Aggregate.&#x20;

    This field tells Ecotone to which Aggregate a given Command is targeted.\
    You may also you expose identifier over public method by annotating it with attribute&#x20;

    ```
    #[AggregateIdentifierMethod("productId")]
    ```
3. `CommandHandler` defined on static method acts as _factory method_. Given command it should return _new instance_ of specific aggregate, in that case new Product.
4. `CommandHandler` defined on non static class method is place where you would put business logic and state changes
