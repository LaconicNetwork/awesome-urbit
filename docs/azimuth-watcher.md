# Azimuth Watcher

Azimuth is the public-key infrastructure used for Urbit identities, deployed as [smart contracts](https://github.com/urbit/azimuth) on Ethereum. For a deep dive, the official documentation has an [in-depth reference](https://developers.urbit.org/reference/azimuth/azimuth).

It currently [relies on events from Infura](https://developers.urbit.org/reference/azimuth/flow#eth-watcher), as seen in this diagram:

![urbit-infura](/images/roller-agents.png)

Ideally, this core components of the Urbit stack would not rely on centralized entities. Additionally, events are not verifiable, which defeats the purpose of certain applications using them.

The problem of "getting data from Ethereum" is not unique to Urbit and plagues nearly all applications that rely on blockchain data. Users and Dapp developers either run a full archive node (tricky and expensive) or rely on centralized service providers (easy and expensive). This is one reason why Laconic created the [watcher framework](https://github.com/cerc-io/watcher-ts/), which significantly reduces the cost of reading and verifying blockchain data. This framework was used to create the Azimuth Watcher, which provides a GraphQL interface for querying the Azimuth contracts' state.

## Usage

The Azimuth Watcher is open source (see below) but requires good hardware. It is here hosted for your convenience:

- https://azimuth.dev.vdb.to/graphql

### Queries

- range can be maximum 1000 blocks
- includes full history from contract genesis

[Example query from contract genesis](https://azimuth.dev.vdb.to/graphql?query=%7B%0A++azimuthEventsInRange%28fromBlockNumber%3A+%0A6784880%2C+toBlockNumber%3A+%0A6785880%29+%7B%0A++++block+%7B%0A++++++hash%0A++++++timestamp%0A++++%7D%0A++++event+%7B%0A++++++...+on+OwnerChangedEvent+%7B%0A++++++++__typename%0A++++++++owner%0A++++++++point%0A++++++%7D%0A++++++...+on+ActivatedEvent+%7B%0A++++++++__typename%0A++++++++point%0A++++++%7D%0A++++++...+on+SpawnedEvent+%7B%0A++++++++__typename%0A++++++++child%0A++++++%7D%0A++++%7D%0A++++contract%0A++%7D%0A%7D)

```
{
  azimuthEventsInRange(fromBlockNumber: 
6784880, toBlockNumber: 
6785880) {
    block {
      hash
      timestamp
    }
    event {
      ... on OwnerChangedEvent {
        __typename
        owner
        point
      }
      ... on ActivatedEvent {
        __typename
        point
      }
      ... on SpawnedEvent {
        __typename
        child
      }
    }
    contract
  }
}
```

[Recent query](https://azimuth.dev.vdb.to/graphql?query=%7B%0A++azimuthEventsInRange%28fromBlockNumber%3A+%0A18664121%2C+toBlockNumber%3A+%0A18664122%29+%7B%0A++++block+%7B%0A++++++hash%0A++++++timestamp%0A++++%7D%0A++++event+%7B%0A++++++...+on+OwnerChangedEvent+%7B%0A++++++++__typename%0A++++++++owner%0A++++++++point%0A++++++%7D%0A++++++...+on+ActivatedEvent+%7B%0A++++++++__typename%0A++++++++point%0A++++++%7D%0A++++++...+on+SpawnedEvent+%7B%0A++++++++__typename%0A++++++++child%0A++++++%7D%0A++++%7D%0A++++contract%0A++%7D%0A%7D)

```
{
  azimuthEventsInRange(fromBlockNumber: 
18664121, toBlockNumber: 
18664122) {
    block {
      hash
      timestamp
    }
    event {
      ... on OwnerChangedEvent {
        __typename
        owner
        point
      }
      ... on ActivatedEvent {
        __typename
        point
      }
      ... on SpawnedEvent {
        __typename
        child
      }
    }
    contract
  }
}
```

### Websocket Subscriptions

#### With Console

go to: https://azimuth.dev.vdb.to/azimuth/graphql

try:
```
 subscription MySubscription {
    onEvent {
      contract
      event {
        ... on OwnerChangedEvent {
          owner
          point
        }
        __typename
      }
      proof {
        data
      }
    }
  }
```

#### In an app

in, e.g., `azimuth.js`:

```
// Reference: https://github.com/enisdenjo/graphql-ws/tree/v5.12.0#use-the-client
const { createClient } = require('graphql-ws');
const WebSocket = require('ws');

const client = createClient({
  url: 'wss://azimuth.dev.vdb.to/azimuth/graphql',
  webSocketImpl: WebSocket
});

// subscription
(async () => {
  const onNext = (value) => {
    /* handle incoming values */
    console.log('Received new data:', JSON.stringify(value, null, 2));
  };

  let unsubscribe = () => {
    /* complete the subscription */
    console.log('subscription completed')
  };

  const query = `
  subscription MySubscription {
    onEvent {
      contract
      event {
        ... on OwnerChangedEvent {
          owner
          point
        }
        __typename
      }
    }
  }
  `;

  try {
    await new Promise((resolve, reject) => {
      unsubscribe = client.subscribe(
        { query },
        {
          next: onNext,
          error: reject,
          complete: resolve,
        },
      );
    });
  } catch (err) {
    console.error(err);
  }
})();
```

then run:
```
node azimuth.js
```

example responses:
```
Received new data: {
  "data": {
    "onEvent": {
      "contract": "0x223c067F8CF28ae173EE5CafEa60cA44C335fecB",
      "event": {
        "owner": "0xCfB830a6ffBC26e847ec40533e102528F7F9D345",
        "point": "2658108823",
        "__typename": "OwnerChangedEvent"
      }
    }
  }
}
Received new data: {
  "data": {
    "onEvent": {
      "contract": "0x223c067F8CF28ae173EE5CafEa60cA44C335fecB",
      "event": {
        "__typename": "BrokeContinuityEvent"
      }
    }
  }
}
Received new data: {
  "data": {
    "onEvent": {
      "contract": "0x223c067F8CF28ae173EE5CafEa60cA44C335fecB",
      "event": {
        "__typename": "ChangedKeysEvent"
      }
    }
  }
}
```

## DIY

- View the source code [here](https://github.com/cerc-io/azimuth-watcher-ts).
- Use Stack Orchestrator to run the Azimuth Watcher [stack](https://github.com/cerc-io/stack-orchestrator/tree/main/app/data/stacks/azimuth).
