---
title: Locksmith Webhooks
description:
  Locksmith implements websub for Unlock Protocol and allows you to send real
  time updates to registered endpoints.
---

# Webhooks

Locksmith implements [Websub](https://www.w3.org/TR/websub) which allows anyone to receive real time updates from the [Unlock subgraphs](../../tools/subgraph.md). It is a _webhook_ system which many developers will be familiar with with built-in intent verification.

Currently, locksmith support sending updates on new locks and keys. To subscribe, an application will need to send a post request to the hubs located at `/api/hooks/[topic]`. The body needs to match the schema specified in the [Websub w3c spec](https://www.w3.org/TR/websub/#x5-1-subscriber-sends-subscription-request).

## Example

Let's send a subscribe request to receive updates on new locks.

1. We need to send a subscribe request to the hub located at `/api/hooks/:network/locks` where network param should be the ID. For example, to receive updates on new locks created on rinkeby network, the endpoint would be `/api/hooks/4/locks`
2. Make a subscribe request. Here's an example of it in javascript.

```javascript
// Subscribe request to receive updates on new Locks
async function subscribe() {
  const endpoint = "https://locksmith.unlock-protocol.com/api/hooks/4/locks";

  const formData = new FormData();

  formData.set("hub.topic", "https://locksmith-host/api/hooks/4/locks");
  formData.set("hub.callback", "https://your-webhook-url/");
  formData.set("hub.mode", "subscribe");
  formData.set("hub.secret", "unlock-is-best");

  const result = await fetch(endpoint, {
    method: "POST",
    body: formData,
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
  });

  if (!result.ok) {
    // Handle the error
  }
  const text = await result.text();
  return text;
}
```

Once you make a request here with callback URL specified in the Websub W3C specification schema. You will receive an [_intent verification_ request](https://www.w3.org/TR/websub/#x5-3-hub-verifies-intent-of-the-subscriber) on the callback URL. This is an async request which means even if you received a successful response for the subscription request, you are not fully subscribed until your endpoint has confirmed the intent. You won't receive updates if intent confirmation fails for any reason.

To confirm, your endpoint MUST return an HTTP 200 status code, with the `hub.challenge` value in the body. This value will be sent as a query string to your endpoint by the hub. This is done to prevent spam.

Check out this example written in typescript for reference on how to handle callback intent verification and receive updates [https://github.com/unlock-protocol/websub-discord/blob/main/src/middleware.ts](https://github.com/unlock-protocol/websub-discord/blob/main/src/middleware.ts)
