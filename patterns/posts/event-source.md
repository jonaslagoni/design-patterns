---
title: Event Source
description: Event Source or event sourcing in a nutshell is the principle of having a stream of events, each describing the change that is happening.
date: 2024-01-31T12:00:00
---

# Event Source
In a nutshell it is the principle of having a stream of events, each describing the change that is happening, similar to Blockchain, just not as rigid. This principle is closely related to [fine grained events](event-carried-state-transfer.md#fine-grained). This is because that if you lose the current accumulated state, say the state of the customer, if it's event sourced, you should easily be able to recover the customer state using the original source. Event sourcing is usually paired with a broker or a data lake that stores the events for a long time enabling you to recover correctly.

Event Source is in its nature append-only, meaning that events once entered the stream cannot change, only be appended. If the data is not append-only, you allow to modify, delete, or reorder those events it would make it an Mutable Event Log. 

NOTICE: Event Source and [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource), is two different things, the first is a design pattern, the second is a the interface for [server-sent-events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

In AsyncAPI there are no way to describe explicitly you are using event source, except through meta data such as `description` and `summary`. However, one might look like the following:
```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Event source orders",
    "version": "1.0.0"
  },
  "channels": {
    "EventSourceOrders": {
      "description": "Source for orders",
      "address": "/orders",
      "messages": {
        "OrderNew": {
          "$ref": "#/components/messages/OrderNew"
        },
        "OrderPaymentReceived": {
          "$ref": "#/components/messages/OrderPaymentReceived"
        },
        "OrderPreparing": {
          "$ref": "#/components/messages/OrderPreparing"
        },
        "OrderShipped": {
          "$ref": "#/components/messages/OrderShipped"
        }
      }
    }
  },
  "components": {
    ...
  }
}
```

Given the above example, we have a channel that contains all the fine grained events that happened for an order, that contains multiple different messages. This is usually paired with a `discriminator` and correlation id (or topic parameter), so you can easily distinguish between the messages and also between orders. Also notice that there are no final `Order` message, everything is split out into separate messages that explains changes to an order. To get the final order status, we source the information from this channel.

Now all applications will of course be publishing or consuming these messages. For example we might have applications that each separately publish them, say a shipping service, here we simply reuse the channel. 

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Order processing",
    "version": "1.0.0"
  },
  "channels": {
    "EventSourceOrders": {
      "$ref": "./event-source.json#/channels/EventSourceOrders"
    }
  },
  "operations": {
    "OrderShipped": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/EventSourceOrders"
      },
      "messages": [
        {
          "$ref": "#/channels/EventSourceOrders/messages/OrderShipped"
        }
      ]
    }
  }
}
```

### Key Points
- The source is always immutable events, meaning to make a change means producing a new event.
- Because it's immutable events it enables auditing, time traveling (debugging what if), enables easy root cause analysis when something goes down (because everything is traceable).
- Rebuilding status when services go down, nothing is ever lost.


## Further resources:

- https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing
- https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- https://www.eventstore.com/event-sourcing