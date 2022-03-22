# Adding new Module

`Modules` are layer which transforms User land code to Ecotone's configuration.\
\
In most of the cases they look for code that is annotated with given Attribute, to register [Messaging Concepts](messaging-concepts/).&#x20;

## Registering new Module

Each Module need to implement `AnnotationModule` and be annotated with `#[ModuleAnnotation]`.

\
**create method:**\
Module is constructed from static `create` method.\
In this method you may use of `AnnotationFinder` to fetch for all the classes that you're interested in.

**canHandle method:**

This tell Ecotone what `extension objects` should be delivered to `prepare method.`\
Those objects are objects created by in user land code by [Service Configuration](service-application-configuration.md).\
\
**getModuleExtensions method:**

This returns extensions defined by Module. It can be list of default ones in case user has not defined any.\
\
**prepare method:**

This registers [Configuration](https://github.com/ecotoneframework/ecotone/blob/master/src/Messaging/Config/Configuration.php) for Ecotone, that can be executed on later stage.

## Phases

To increase application speed, Ecotone works in two phases.

#### **Preparation Phase:**

In preparation Phase Ecotone runs `prepare` method on each of Modules. \
The Configuration that is registered as a result is cached and is reused between executions.

#### Execution Phase:

Execution phase works on preconfigured Configuration by executing given messaging flow.
