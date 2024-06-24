# Glossaries

Many definitions have multiple words defining the same name for the concepts, this glossary sections helps create a common language for the design patterns.

- Delta events - small events that add to previous events, i.e. similar to time since last changed in game development. | Used in [ECST](./patterns/event-carried-state-transfer.md)
- Fat events - is a self contained event that include most if not all information. | Used in [ECST](./patterns/event-carried-state-transfer.md)
- Idempotent - a function is said to be idempotent if multiple calls with the same information never changes the outcome. It is similar to stateless in application development. | Used in [ECST](./patterns/event-carried-state-transfer.md)
- Exactly-once delivery - a message pattern that ensures that the consumer(group) only gets the message once, regardless of restarts etc. | Used in [ECST](./patterns/event-carried-state-transfer.md)
- PII - personally identifiable information is any information connected to a specific individual that can be used to uncover that individual's identity, such as their social security number, full name, email address or phone number. | Used in [ECST](./patterns/event-carried-state-transfer.md)
- Eventual consistency - [When we build distributed systems, there are times that state is distributed across our architecture (e.g. when we favour availability over consistency, e.g services have a copy of the data they are consuming vs requesting it from another service). This means data across your architecture in theory will be eventually consistent and at times the state will be inconsistent (as data is replicated across your architecture).](https://eda-visuals.boyney.io/visuals/eventual-consistency) | Used in [ECST](./patterns/event-carried-state-transfer.md)
- 