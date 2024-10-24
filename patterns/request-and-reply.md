---
title: Request and Reply
description: 
---

# Request and Reply

Request and Reply is one of the most used messaging pattern and offers a two-way communication pattern to exchange messages. The most known form being REST and HTTP where you make synchronous requests over a single channel/path. 

In the event driven world it can quickly become complex as often there are no single channel and can be more dynamic.

In request and reply there are always two parties, the requester, making the request and the replier, in charge of delivering the response to the request according to how it's expected.


<table>
<tr>
<th> Requester </th> 
<th> Replier </th>
</tr>
<tr>
<td> 

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Payment service",
    "description": "The requester",
    "version": "1.0.0"
  },
  "channels": {
    "StartShipping": {
      "address": "shipping/start",
      "messages": {
        "StartShippingRequest": {
          "$ref": "#/components/messages/StartShippingRequest"
        },
        "StartShippingResponse": {
          "$ref": "#/components/messages/StartShippingResponse"
        }
      }
    }
  },
  "operations": {
    "StartShippingRequest": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/StartShipping"
      },
      "messages": [
        {
          "$ref": "#/channels/ping/messages/StartShippingRequest"
        }
      ],
      "reply": {
        "channel": {
          "$ref": "#/channels/StartShipping"
        },
        "messages": [
          {
            "$ref": "#/channels/ping/messages/StartShippingResponse"
          }
        ]
      }
    }
  },
  ...
}
```
</td>

<td>

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Shipping service",
    "description": "The replier",
    "version": "1.0.0"
  },
  "channels": {
    "StartShipping": {
      ...
    }
  },
  "operations": {
    "StartShippingReply": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/StartShipping"
      },
      "messages": [
        {
          "$ref": "#/channels/ping/messages/StartShippingRequest"
        }
      ],
      "reply": {
        "channel": {
          "$ref": "#/channels/StartShipping"
        },
        "messages": [
          {
            "$ref": "#/channels/ping/messages/StartShippingResponse"
          }
        ]
      }
    }
  },
  ...
}
```

</td>
</tr>
</table>

As you can see both take leverage on the same definitions where only the `receive` and `send` action keywords change. You could also use traits for operations and references to simplify the definitions.

## Patterns

Through request and reply there are multiple patterns that are slightly different from the simple request and reply we see in HTTP.

### Dynamic response based on request meta information
One pattern is where the response needs to happen dynamically based on meta information in the request.

Locations for this could be part of the headers, message payload or parameter.

<table>
<tr>
<th> Payload </th> 
<th> Headers </th> 
</tr>
<tr>
<td> 

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Payment service",
    "version": "1.0.0"
  },
  "channels": {
    "StartShippingRequest": {
      "address": "shipping/start",
      "messages": {
        "StartShippingRequest": {
          "$ref": "#/components/messages/StartShippingRequest"
        },
      }
    },
    "StartShippingReply": {
      "address": null,
      "messages": {
        "StartShippingResponse": {
          "$ref": "#/components/messages/StartShippingResponse"
        }
      }
    }
  },
  "operations": {
    "StartShippingRequest": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/StartShippingRequest"
      },
      "reply": {
        "address": {
          "description": "The reply address is dynamically determined
          based on the request header `replyTo`",
          "location": "$message.payload#/replyTo"
         },
        "channel": {
          "$ref": "#/channels/StartShippingReply"
        }
      }
    }
  },
  "components": {
    "messages": {
      "StartShippingRequest": {
        "payload": {
          "type": "object",
          "properties": {
            "replyTo": {
              "type": "string"
            }
          }
        }
      }
    }
  }
  ...
}
```
</td>

<td> 

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Payment service",
    "version": "1.0.0"
  },
  "channels": {
    "StartShippingRequest": {
      "address": "shipping/start",
      "messages": {
        "StartShippingRequest": {
          "$ref": "#/components/messages/StartShippingRequest"
        },
      }
    },
    "StartShippingReply": {
      "address": null,
      "messages": {
        "StartShippingResponse": {
          "$ref": "#/components/messages/StartShippingResponse"
        }
      }
    }
  },
  "operations": {
    "StartShippingRequest": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/StartShippingRequest"
      },
      "reply": {
        "address": {
          "description": "The reply address is dynamically determined
          based on the request header `replyTo`",
          "location": "$message.header#/replyTo"
        },
        "channel": {
          "$ref": "#/channels/StartShippingReply"
        }
      }
    }
  },
  "components": {
    "messages": {
      "StartShippingRequest": {
        "headers": {
          "type": "object",
          "properties": {
            "replyTo": {
              "type": "string"
            }
          }
        }
      }
    }
  }
}
```
</td>
</tr>
</table>


### Multiple messages over the same channel
This pattern is a common one in WebSocket, where multiple different messages flow through the same connection,  

<table>
<tr>
<th> WebSocket </th> 
</tr>
<tr>
<td> 

> Simulating a request and reply scenario, where we expect a specific message results in another over the same connection.

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "WebSocket request/reply example",
    "version": "1.0.0"
  },
  "channels": {
    "rootChannel": {
      "address": "/",
      "messages": {
        "StartShippingRequest": {
          "$ref": "#/components/messages/StartShippingRequest"
        },
        "StartShippingResponse": {
          "$ref": "#/components/messages/StartShippingResponse"
        }
      }
    }
  },
  "operations": {
    "StartShippingRequest": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/rootChannel"
      },
      "messages": [
        {
          "$ref": "/components/messages/StartShippingRequest"
        }
      ],
      "reply": {
        "messages": [
          {
            "$ref": "/components/messages/StartShippingResponse"
          }
        ],
        "channel": {
          "$ref": "#/channels/rootChannel"
        }
      }
    }
  }
}
```
</td>
</tr>
</table>

# Resources

- https://eventstack.tech/posts/asyncapi-v3-request-reply
- https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html
- https://developer.ibm.com/articles/awb-request-response-messaging-pattern-introduction/