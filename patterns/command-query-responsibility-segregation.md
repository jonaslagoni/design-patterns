---
title: Command Query Responsibility Segregation (CQRS)
description: Separation between commands and queries
---

## Command Query Responsibility Segregation (CQRS)

In its most basic form, it's just a separation (segregation) between requests that change data (commands) and requests that query data. For example application 1 handles all the commands and application 2 handles all the queries. Usually because each form has separate performance requirements such as throughput, latency, security, etc.

Because read and write happens in different places, one of the drawbacks is having consistency between queries and commands as they lag behind.

Application in charge of processing commands:
```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Command application",
	  "description": "The application in charge of processing commands",
    "version": "1.0.0"
  },
  "channels": {
    "AddAddress": {
      "address": "/address",
      "messages": {
        "AddressMessage": {
          "$ref": "#/components/messages/AddressMessage"
        }
      }
    }
  },
  "operations": {
    "AddNewAddress": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/AddAddress"
      }
    }
  },
  "components": {
    "schemas": {
      "Address": {
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
      "AddressMessage": {
        "payload": {
          "$ref": "#/components/schemas/Address"
        }
      }
    }
  }
}
```

Application in charge of processing queries:
```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Query application",
	  "description": "The application in charge of processing queries",
    "version": "1.0.0"
  },
  "channels": {
    "GetAddress": {
      "address": "/address",
      "messages": {
        "AddressMessage": {
          "$ref": "#/components/messages/AddressMessage"
        }
      }
    }
  },
  "operations": {
    "QueryAddress": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/GetAddress"
      }
    }
  },
  "components": {
    ...
  }
}
```


Further resources:

- https://solace.com/event-driven-architecture-patterns/#cqrs
- https://ibm-cloud-architecture.github.io/refarch-eda/patterns/cqrs/
