---
title: Event Source
description: 
---

# Event Source
In a nutshell it is the principle of having a stream of events, each describing the change that is happening. This principle is closely related to [fine grained events](event-carried-state-transfer.md#fine-grained). This is because that if you lose the current accumulated state, say the state of the customer address, if it's event sourced, you should easily be able to recover the customer state using the original source. Event sourcing is usually paired with a broker or a data lake that stores the events for a long time enabling you to recover correctly.


Further resources:

- https://solace.com/event-driven-architecture-patterns/#cqrs
- https://ibm-cloud-architecture.github.io/refarch-eda/patterns/cqrs/
