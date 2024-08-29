---
title: Event Source
description: 
---

# Event Source
In a nutshell it is the principle of having a stream of events, each describing the change that is happening. This principle is closely related to [fine grained events](event-carried-state-transfer.md#fine-grained). This is because that if you lose the current accumulated state, say the state of the customer address, if it's event sourced, you should easily be able to recover the customer state using the original source. Event sourcing is usually paired with a broker or a data lake that stores the events for a long time enabling you to recover correctly.

Event Source and [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource), is two different things, the first is a design pattern, the second is a the interface for [server-sent-events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)

Further resources:

- https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing
- https://developer.mozilla.org/en-US/docs/Web/API/EventSource
- https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events