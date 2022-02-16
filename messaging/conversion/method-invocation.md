---
description: Method Invocation PHP
---

# Method Invocation

## Injecting arguments

`Ecotone` inject arguments to invoked method based on `Parameter Converters`.\
Parameter converters tells `Ecotone` how to resolve specific parameter and what kind of argument is it expecting.&#x20;

Suppose that we have Command Handler [Endpoint](../messaging-concepts/message-endpoint/):

```php
#[CommandHandler] 
public function changePrice(
    #[Payload] ChangeProductPriceCommand $command, 
    #[Headers] array $metadata, 
    #[Reference] UserService $userService
) : void
{
    $userId = $metadata["userId"];
    if (!$userService->isAdmin($userId)) {
        throw new \InvalidArgumentException("You need to be administrator in order to register new product");
    }

    $this->price = $command->getPrice();
}
```

Our Command Handler method declaration is built from three parameters. \
`Ecotone` does resolve parameters based on given attribute types. \
\
`Payload` - Does inject payload of the [message](../messaging-concepts/message.md). In our case it will be the command itself\
`Headers` - Does inject all headers as array.\
`Reference`- Does inject service from Dependency Container. If `referenceName`which is name of the service in the container is not given, then it will take the class name as default.

## Default Converters

`Ecotone`, if parameter converters are not passed provides default converters.&#x20;

* First parameter is always `Payload.`&#x20;
* The second parameter, if is `array` then `Headers` converter is taken
* If class type hint is provided for parameter, then `Reference` converter is picked
* Otherwise, if no default converter can be applied exception will be thrown with information about missing parameter.

Our Command Handler can benefit from default converters, so we don't need to use any additional configuration.

```php
#[CommandHandler] 
public function changePrice(ChangeProductPriceCommand $command, array $metadata, UserService $userService) : void
{
    $userId = $metadata["userId"];
    if (!$userService->isAdmin($userId)) {
        throw new \InvalidArgumentException("You need to be administrator in order to register new product");
    }

    $this->price = $command->getPrice();
}
```

## Parameter Converter Types

### Payload Converter

`Payload` converter is responsible for passing payload of the [message](../messaging-concepts/message.md) to given parameter. \
It contains of two attributes:

* `expression` (Optional) - Allow for performing transformations before passing argument to parameter

#### Converting payload

Message's payload is not always the same type as expected in method declaration. \
As in above example, we may expect:&#x20;

```php
ChangeProductPriceCommand $command
```

But the message may contains:

```php
{"payload": '{"productId": 123, "price": 100}', "headers": {"contentType": "application/json"}}
```

The message may contains of special header `contentType`which describes content type of Message as [media type](https://en.wikipedia.org/wiki/Media\_type). Based on this information, if payload of message is not compatible with parameter's type hint, `Ecotone` do the [conversion](conversion.md).

{% hint style="info" %}
Thanks to conversion on the level of endpoint, Ecotone does not expect running `Command Bus` with specific class instance. It may receive anything `xml, json etc` as long as Converter for specific Media Type is registered in the system.
{% endhint %}

#### Expression

Expression does use of great feature of Symfony, called [Expression Language](https://symfony.com/doc/current/components/expression\_language.html). If you have not came across expression language yet, take few minutes to read about it.

```php
Payload(expression: "payload * 2")
```

There are three types of variables available within expression.

* `payload` - which is just payload of currently handled Message
* `headers` - contains of all headers available within Message&#x20;
*   `reference` - which allow for retrieving service from Dependency Container and calling a method on it. The result of the expression will be passed to parameter after optional conversion.

    ```php
    Payload(expression:"reference('calculatingService').multiply(payload, 2)")
    ```

### Headers Converter (Headers)

`Headers` converter is responsible for passing all headers of the [message](../messaging-concepts/message.md) as array to given parameter.&#x20;

### Header Converter (Header)

`Header` converter is responsible for passing specific header from [message](../messaging-concepts/message.md) headers to given parameter. \
It contains attributes:

* `headerName` (Required) - Allow for performing transformations before passing argument to parameter
*   `expression` - Allow for performing transformations before passing argument to parameter, same as in [Payload expression](method-invocation.md#payload-converter-payload)

    &#x20;

### Expression

Besides three types of variables available within expression just as with `Payload Expression`, there is one more variable available `value.` Value holds chosen header by `headerName` attribute.

### Reference (Reference)

`Reference` converter is responsible for retrieving object from Dependency Container and passing it to given parameter. \
It does contain attributes:

* `referenceName` (Optional) - Reference name of given object in Dependency Container. If not passed, then type hint of given parameter will be used.
