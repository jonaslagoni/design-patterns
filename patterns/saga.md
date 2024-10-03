---
title: Saga pattern
description: How you orchestrate transactions in a micro service environment
---

# Saga pattern
When placing an order, there are likely multiple services in the backend that needs to "sign-off" that the purchase went through. For example, a step in the process could be to check that the customer has not reached it's credit limit, and if they have, the transaction needs to be rejected in some fashion. 

How those long-running transaction is coordinated across multiple microservices, ensuring data consistency and eventual completion is possible through the Saga pattern.

There are two saga types, choreograph and orchestration based. The main difference between the two is how the transaction is coordinated and who is responsible for what. 


## Choreograph based
Is the decentralized coordination pattern. Each service involved in the transaction is responsible for listening and reacting to events from other services. 

For example; The payment service performs a payment and emits a "Payment Successful" event, which is consumed by the shipping service to trigger shipping. There are no central service that asks the services to perform anything.

Notice how the shipping service is receiving when payment is successful on it's own without a single service handling the transaction.

<table>
<tr>
<th> Payment service </th> 
<th> Shipping service </th>
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
    "PaymentSuccess": {
      "description": "Payment was successfully made",
      "address": "payment.success",
      "messages": {
        "PaymentSuccess": {
          ...
        }
      }
    }
  },
  "operations": {
    "PaymentMade": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/PaymentSuccess"
      }
    }
  }
}
```
</td>

<td>

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Shipping service",
    "version": "1.0.0"
  },
  "channels": {
    "PaymentSuccess": {
      "description": "Payment was successfully made",
      "address": "payment.success",
      "messages": {
        "PaymentSuccess": {
          ...
        }
      }
    },
    "OrderShipped": {
      "description": "Order was successfully shipped",
      "address": "order.sent",
      "messages": {
        "OrderShipped": {
          ...
        }
      }
    }
  },
  "operations": {
    "PaymentMade": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/PaymentSuccess"
      }
    },
    "OrderShipped": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/OrderShipped"
      }
    }
  }
}
```

</td>
</tr>
</table>

## Orchestration based

A Orchestration based Saga is a single service or part of another that ensures that the transaction runs as expected and calls the corresponding services that are part of the transaction. 

For example; The orchestrator calls when an order is received calls the payment service, and finally the shipping service, controlling the coordination of the transactions.

Notice how the Orchestrator are interacting with payment and shipping service and receive and send to both, orchestrating when and what happens. 

<table>
<tr>
<th> Orchestrator </th> 
<th> Payment service </th> 
<th> Shipping service </th>
</tr>
<tr>
<td> 

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Orchestrator",
    "version": "1.0.0"
  },
  "channels": {
    "PayForOrder": {
      "description": "Make the payment for the order",
      "address": "payment.pay",
      "messages": {
        "PaymentPay": {
          ...
        }
      }
    },
    "PaymentSuccess": {
      "description": "Payment was successfully made",
      "address": "payment.success",
      "messages": {
        "PaymentSuccess": {
          ...
        }
      }
    },
    "SendOrder": {
      "description": "Send an order",
      "address": "order.send",
      "messages": {
        "SendOrder": {
          ...
        }
      }
    },
    "OrderShipped": {
      "description": "Order was successfully shipped",
      "address": "order.sent",
      "messages": {
        "OrderShipped": {
          ...
        }
      }
    }
  },
  "operations": {
    "PayForOrder": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/PayForOrder"
      }
    },
    "PaymentMade": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/PaymentSuccess"
      }
    },
    "SendOrder": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/SendOrder"
      }
    },
    "OrderShipped": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/OrderShipped"
      }
    }
  }
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
    "PayForOrder": {
      "description": "Make the payment for the order",
      "address": "payment.pay",
      "messages": {
        "PaymentPay": {
          ...
        }
      }
    },
    "PaymentSuccess": {
      "description": "Payment was successfully made",
      "address": "payment.success",
      "messages": {
        "PaymentSuccess": {
          ...
        }
      }
    }
  },
  "operations": {
    "PayForOrder": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/PayForOrder"
      }
    },
    "PaymentMade": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/PaymentSuccess"
      }
    }
  }
}
```
</td>

<td>

```json
{
  "asyncapi": "3.0.0",
  "info": {
    "title": "Shipping service",
    "version": "1.0.0"
  },
  "channels": {
    "SendOrder": {
      "description": "Send an order",
      "address": "order.send",
      "messages": {
        "SendOrder": {
          ...
        }
      }
    },
    "OrderShipped": {
      "description": "Order was successfully shipped",
      "address": "order.sent",
      "messages": {
        "OrderShipped": {
          ...
        }
      }
    }
  },
  "operations": {
    "SendOrder": {
      "action": "receive",
      "channel": {
        "$ref": "#/channels/SendOrder"
      }
    },
    "OrderShipped": {
      "action": "send",
      "channel": {
        "$ref": "#/channels/OrderShipped"
      }
    }
  }
}
```

</td>
</tr>
</table>

## Final remarks

The main differences can be seen below;

| Aspect | Choreography | Orchestration | 
|----------|:-------------:|:------:|
**Control**	| Decentralized, self-coordinated|Centralized, orchestrator-controlled
**Communication** | Event-driven (publish/subscribe) | Direct service invocation
**Coupling** | Loosely coupled | Loosely coupled, but tightly coupled to the orchestrator
**Flow Complexity** | Complex with multiple services | Centralized and easier to trace
**Failure Handling** | Compensating actions handled individually by services | Orchestrator manages compensations
**Flexibility** | More flexible and scalable in smaller, simpler use cases | Easier to manage in complex workflows

For better specifying the exact fail scenarios and workflows, one might take a look at [Arazzo](https://spec.openapis.org/arazzo/latest.html).

# Resources

- https://microservices.io/patterns/data/saga.html
- https://medium.com/cloud-native-daily/microservices-patterns-part-04-saga-pattern-a7f85d8d4aa3
- https://www.geeksforgeeks.org/saga-design-pattern/