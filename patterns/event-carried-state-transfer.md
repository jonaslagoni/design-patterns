---
title: Event Carried State Transfer (ECST)
description: Event Carried State Transfer define different verboseness of the message payloads such as fine grained and snapshots.
---

# Event carried state transfer (ECST)

counter parts: Event-notification

Instead of simply saying `the address of user x has changed`, ECST means events carry related information with it, to become more verbose, so it tells exactly what changed `user x has a new address y and the old address was z`. It is a way of sharing data across systems, so it becomes a question of how much data/information do you want each event to contain? 

It splits out into two at least 3 approaches, fine grained ([Delta events](../glossary.md)), fine grained snapshots and snapshots (fat events).

### Fine grained
Fine grained is the smallest amount of data that can be send using ECST, which simply answers "what has specifically changed?".

- Good when services each has a copy of the data and just need to apply the change them self

Example for `Fine Grained payload` structure:
```json
{
  "addressId": "12366",
  "oldState": {
    "addressLine1": "84 Baker Street"
  },
  "newState": {
    "addressLine1": "48 Baker Street"
  }
}
```

### Fine grained snapshots
Fine grained snapshots include a all the information about the object that changed so you always have access to the complete picture in the event.

- Good if you debug event streams (without the context of applications) to full picture of the state.

Example for `Fine grained snapshots` payload structure:
```json
{
  "addressId": "12366",
  "oldState": {
    "addressContent": {
      "addressPurpose": "billing",
      "addressLine1": "84 Baker Street",
      "addressLine2": "London, UK, E1 7AE",
      "addressTS": 1572134105
    }
  },
  "newState": {
    "addressContent": {
      "addressPurpose": "billing",
      "addressLine1": "48 Baker Street",
      "addressLine2": "London, UK, E1 7AE",
      "addressTS": 1572134205
    }
  }
}
```

### Snapshots
Snapshots take a further step back, and look at the greater object that address belongs to, for example customer, and give you the current snapshot of that state of that object. That way you only need to look at the event itself to know the state of the entity.

Example for `Snapshot` payload structure:
```json
{
  "customerId": "911000100",
  "oldState": {
    "addressContent": {
      "addresses": [
        {
          "addressId": "12366",
          "addressPurpose": "billing",
          "addressLine1": "84 Baker Street",
          "addressLine2": "London, UK, E1 7AE",
          "addressTS": 1572134105
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "44 Infinite Loop",
          "addressLine2": "Cupertino, CA, US, 95014",
          "addressTS": 1452034105
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "500 George Street",
          "addressLine2": "Sydney, NSW, AU, 2000",
          "addressTS": 1452034105
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "15 Karl-Liebknecht-Str",
          "addressLine2": "Berlin, DE, 10318",
          "addressTS": 1452034105
        }
      ]
    }
  },
  "newState": {
    "addressContent": {
      "addresses": [
        {
          "addressId": "12366",
          "addressPurpose": "billing",
          "addressLine1": "48 Baker Street",
          "addressLine2": "London, UK, E1 7AE",
          "addressTS": 1572134205
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "44 Infinite Loop",
          "addressLine2": "Cupertino, CA, US, 95014",
          "addressTS": 1452034105
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "500 George Street",
          "addressLine2": "Sydney, NSW, AU, 2000",
          "addressTS": 1452034105
        },
        {
          "addressId": "123124",
          "addressPurpose": "shipping",
          "addressLine1": "15 Karl-Liebknecht-Str",
          "addressLine2": "Berlin, DE, 10318",
          "addressTS": 1452034105
        }
      ]
    }
  }
}
```

## General

You must make sure that the consumers are [idempotent](../glossary.md) as processing the same event carries message means that the "local" data might change unexpected. Some brokers already have something like this build in if they support [exactly-once delivery](../glossary.md).

> As seen in the examples, the more verbose the event becomes, the more complex it is to extract the data and the more expensive it is to send over the wire

With more verbose events, the more [PII](../glossary.md) comes into play.

You will have [eventual-consistency](../glossary.md) across producer/data owner - and consumers (and even across consumers). "This inconsistency may only last for a few milliseconds but, in a high-traffic system that sends out thousands of ECS messages, the likelihood of an inconsistent read increases rapidly." (https://blogs.mulesoft.com/api-integration/strategy/event-carried-state-messages/)

Example AsyncAPI document for a consumer:
```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Account Service",
    "version": "1.0.0"
  },
  "channels": {
    "addressChanged": {
      "address": "/address",
      "messages": {
        "FineGrained": {
          "$ref": "#/components/messages/FineGrained"
        },
        "FineGrainedSnapshots": {
          "$ref": "#/components/messages/FineGrainedSnapshots"
        },
        "Snapshots": {
          "$ref": "#/components/messages/Snapshots"
        }
      }
    }
  },
  "operations": {
    "addressChanged": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/addressChanged"
      }
    }
  },
  "components": {
    "schemas": {
      "PartialAddress": {
        "type": "object",
        "properties": {
          "addressPurpose": {
            "type": "string"
          },
          "addressLine1": {
            "type": "string"
          },
          "addressLine2": {
            "type": "string"
          },
          "addressTS": {
            "type": "number"
          }
        }
      },
      "FullAddress": {
        "type": "object",
        "required": [
          "addressPurpose",
          "addressLine1",
          "addressLine2",
          "addressTS"
        ],
        "properties": {
          "addressPurpose": {
            "type": "string"
          },
          "addressLine1": {
            "type": "string"
          },
          "addressLine2": {
            "type": "string"
          },
          "addressTS": {
            "type": "number"
          }
        }
      },
      "SnapshotAddress": {
        "type": "object",
        "required": [
          "addresses"
        ],
        "properties": {
          "addresses": {
            "type": "array",
            "items": {
              "type": "object",
              "required": [
                "addressId",
                "addressPurpose",
                "addressLine1",
                "addressLine2",
                "addressTS"
              ],
              "properties": {
                "addressId": {
                  "type": "string"
                },
                "addressPurpose": {
                  "type": "string"
                },
                "addressLine1": {
                  "type": "string"
                },
                "addressLine2": {
                  "type": "string"
                },
                "addressTS": {
                  "type": "number"
                }
              }
            }
          }
        }
      }
    },
    "messages": {
      "FineGrained": {
        "description": "Fine grained ESCT pattern, notice the optional properties for the states",
        "payload": {
          "type": "object",
          "properties": {
            "addressId": {
              "type": "string"
            },
            "oldState": {
              "$ref": "#/components/schemas/PartialAddress"
            },
            "newState": {
              "$ref": "#/components/schemas/PartialAddress"
            }
          }
        }
      },
      "FineGrainedSnapshots": {
        "description": "Fine grained snapshot ESCT pattern, notice the required properties for the states",
        "payload": {
          "type": "object",
          "properties": {
            "addressId": {
              "type": "string"
            },
            "oldState": {
              "$ref": "#/components/schemas/FullAddress"
            },
            "newState": {
              "$ref": "#/components/schemas/FullAddress"
            }
          }
        }
      },
      "Snapshots": {
        "description": "Snapshot ESCT pattern, notice the `customerId` in the root, and `addressId` in the state",
        "payload": {
          "type": "object",
          "properties": {
            "customerId": {
              "type": "string"
            },
            "oldState": {
              "$ref": "#/components/schemas/SnapshotAddress"
            },
            "newState": {
              "$ref": "#/components/schemas/SnapshotAddress"
            }
          }
        }
      }
    }
  }
}
```

Further resources:

- https://medium.com/swlh/event-notification-vs-event-carried-state-transfer-2e4fdf8f6662
- https://solace.com/event-driven-architecture-patterns/
- https://blogs.mulesoft.com/api-integration/strategy/event-carried-state-messages/
- https://itnext.io/the-event-carried-state-transfer-pattern-aae49715bb7f
- https://eda-visuals.boyney.io/visuals/eventual-consistency
