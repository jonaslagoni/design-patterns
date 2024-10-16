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

```diff
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
        "StartShipping": {
          "$ref": "#/components/messages/StartShipping"
        },
        "StartShippingReply": {
          "$ref": "#/components/messages/StartShippingReply"
        }
      }
    }
  },
  "operations": {
    "StartShippingRequest": {
+     "action": "send",
      "channel": {
        "$ref": "#/channels/StartShipping"
      },
      "messages": [
        {
          "$ref": "#/channels/ping/messages/StartShipping"
        }
      ],
      "reply": {
        "channel": {
          "$ref": "#/channels/StartShipping"
        },
        "messages": [
          {
            "$ref": "#/channels/ping/messages/StartShippingReply"
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

```diff
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
    "StartShippingResponse": {
+     "action": "receive",
      "channel": {
        "$ref": "#/channels/StartShipping"
      },
      "messages": [
        {
          "$ref": "#/channels/ping/messages/StartShipping"
        }
      ],
      "reply": {
        "channel": {
          "$ref": "#/channels/StartShipping"
        },
        "messages": [
          {
            "$ref": "#/channels/ping/messages/StartShippingReply"
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

```diff
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Payment service",
    "version": "1.0.0",
    "description": "Example with a requester that initiates the request/reply pattern where the reply will happen on whatever is defined in the header `replyTo` of the request."
  },
  "channels": {
    "StartShipping": {
      "address": "shipping/start",
      "messages": {
        "StartShipping": {
          "$ref": "#/components/messages/StartShipping"
        },
      }
    },
    "StartShippingResponse": {
      "address": null,
      "messages": {
        "StartShippingReply": {
          "$ref": "#/components/messages/StartShippingReply"
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
      "reply": {
+       "address": {
+         "description": "The reply address is dynamically determined based on the request header `replyTo`",
+         "location": "$message.header#/replyTo"
        },
        "channel": {
          "$ref": "#/channels/StartShippingResponse"
        }
      }
    }
  },
  ...
}
```


# Resources

- https://eventstack.tech/posts/asyncapi-v3-request-reply
- https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html
- https://developer.ibm.com/articles/awb-request-response-messaging-pattern-introduction/