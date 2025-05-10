# Build a Hello World (TypeScript)

### Prerequisites

Before you begin, ensure you have the following installed:

* [Bun](https://bun.sh)

***

### Step 1: Create a Bun Project

First, create a new project directory and initialize it with Bun:

```bash
mkdir hello-world-ml
cd hello-world-ml
bun init -y
```

Next, install the required dependencies:

```bash
bun add nostr-tools @solana/web3.js
```

***

### Step 2: Write the Subscriber Code

Create a file named `subscriber.ts` and add the following code. This script connects to the Nostr relay, subscribes to "Hello World" events (custom kind 1573), and logs any received messages.

```ts
// subscriber.ts

import { Relay } from "nostr-tools/relay";

const RELAY_URL = "wss://dev-relay.dephy.dev";

async function main() {
  const relay = await Relay.connect(RELAY_URL);
  console.log("Subscribed events on", RELAY_URL);

  relay.subscribe(
    [
      {
        kinds: [1573],
        since: Math.floor(Date.now() / 1000),
        "#s": ["hello_session"],
        "#p": ["receiver_pubkey"],
      },
    ],
    {
      onevent: async (event) => {
        console.log("Received:", event.content);
      },
    }
  );
}

main().catch(console.error);

```

***

### Step 3: Write the Publisher Code

Create a file named `publisher.ts` and insert the following code. This script sends a simple "Hello World" event to the relay using a Nostr event with custom tags.

```typescript
// publisher.ts

import { finalizeEvent, generateSecretKey } from "nostr-tools/pure";
import { Relay } from "nostr-tools/relay";

const RELAY_URL = "wss://dev-relay.dephy.dev";

async function main() {
  const relay = await Relay.connect(RELAY_URL);

  const eventTemplate = {
    kind: 1573,
    created_at: Math.floor(Date.now() / 1000),
    tags: [
        ['s', "hello_session"],
        ['p', "receiver_pubkey"],
    ],
    content: "Hello World",
  };

  const sk = generateSecretKey();

  const event = finalizeEvent(eventTemplate, sk);

  await relay.publish(event);
  console.log("Published 'Hello World' event to", RELAY_URL);
}

main().catch(console.error);

```

***

### Step 4: Run the "Hello World" Example

Open two terminals (or tabs):

*   Run the `Subscriber`:

    ```bash
    bun run subscriber.ts
    ```
*   Run the `Publisher` (in a new terminal):

    ```bash
    bun run publisher.ts
    ```
*   Expected Output:

    * `Subscriber` terminal:&#x20;

    ```
    Subscribed events on wss://dev-relay.dephy.dev
    ```

    * `Publisher` terminal:

    {% code fullWidth="false" %}
    ```
    Published 'Hello World' event to wss://dev-relay.dephy.dev
    ```
    {% endcode %}

    * `Subscriber` terminal:

    ```
    Received: Hello World
    ```

***

### Next, Enhance the Functionality with Solana Airdrop

At this point, the basic publish and subscribe functionality is complete. Next, we will enhance this by integrating with the Solana blockchain. Specifically, when publishing a greeting like "Hello World," the publisher will include a Solana public key, and the subscriber will request a Solana airdrop.

***

### Step 5: Update the Publisher

Edit `publisher.ts` to include a structured message and a Solana public key. Replace its contents with the following code:

```typescript
// publisher.ts

import { finalizeEvent, generateSecretKey } from "nostr-tools/pure";
import { Relay } from "nostr-tools/relay";
import { Keypair } from "@solana/web3.js";

const RELAY_URL = "wss://dev-relay.dephy.dev";

interface Message {
  greeting: string;
  solana_pubkey: string;
}

async function main() {
  const relay = await Relay.connect(RELAY_URL);

  // Generate a new Solana keypair and extract its public key
  const solanaKeypair = Keypair.generate();
  const solanaPubkey = solanaKeypair.publicKey.toBase58();

  // Create a Message object
  const message: Message = {
    greeting: "Hello World",
    solana_pubkey: solanaPubkey,
  };

  const eventTemplate = {
    kind: 1573,
    created_at: Math.floor(Date.now() / 1000),
    tags: [
      ["s", "hello_session"],
      ["p", "receiver_pubkey"],
    ],
    content: JSON.stringify(message),
  };

  const sk = generateSecretKey();

  const event = finalizeEvent(eventTemplate, sk);

  await relay.publish(event);
  console.log("Published 'Hello World' event to", RELAY_URL);
  console.log("Airdrop request for", solanaPubkey.toString());
}

main().catch(console.error);

```

***

### Step 6: Update the Subscriber

Edit `subscriber.ts` so that it parses the structured message, extracts the Solana public key, and requests a Solana airdrop. Replace its contents with the following code:

```typescript
// subscriber.ts

import { Relay } from "nostr-tools/relay";
import { Connection, PublicKey } from "@solana/web3.js";

const RELAY_URL = "wss://dev-relay.dephy.dev";

interface Message {
  greeting: string;
  solana_pubkey: string;
}

async function main() {
  const relay = await Relay.connect(RELAY_URL);
  console.log("Subscribed events on", RELAY_URL);
  const solanaConnection = new Connection(
    "https://api.devnet.solana.com",
    "confirmed"
  );

  relay.subscribe(
    [
      {
        kinds: [1573],
        since: Math.floor(Date.now() / 1000),
        "#s": ["hello_session"],
        "#p": ["receiver_pubkey"],
      },
    ],
    {
      onevent: async (event) => {
        console.log("Received:", event.content);

        const message: Message = JSON.parse(event.content);
        if (message.solana_pubkey) {
          const pubkey = new PublicKey(message.solana_pubkey);
          // Request an airdrop of 1 SOL (1 SOL = 1_000_000_000 lamports)
          const signature = await solanaConnection.requestAirdrop(
            pubkey,
            1_000_000_000
          );
          console.log(
            `Airdrop requested for ${pubkey.toBase58()}. Signature: ${signature}`
          );
          // Confirm the transaction
          const confirmation = await solanaConnection.confirmTransaction(
            signature
          );
          if (confirmation.value.err === null) {
            console.log(`Airdrop confirmed for ${pubkey.toBase58()}`);
          } else {
            console.error(
              `Airdrop confirmation failed for ${pubkey.toBase58()}`
            );
          }
        }
      },
    }
  );
}

main().catch(console.error);

```

***

### Step 7: Run the Enhanced Example

Open two terminals (or tabs):

*   Run the `Subscriber`:

    ```bash
    bun run subscriber.ts
    ```
*   Run the `Publisher` (in a new terminal):

    ```bash
    bun run publisher.ts
    ```
*   Expected Output:

    * `Subscriber` terminal:&#x20;

    ```
    Subscribed events on wss://dev-relay.dephy.dev
    ```

    * `Publisher` terminal:

    {% code fullWidth="false" %}
    ```
    Published 'Hello World' event to wss://dev-relay.dephy.dev
    Airdrop request for [ ... ]
    ```
    {% endcode %}

    * `Subscriber` terminal:

    ```
    Received: {"greeting":"Hello World","solana_pubkey":"..."}
    Airdrop requested for ... Signature: ...
    Airdrop confirmed for ...
    ```

***

### Next Steps

This example demonstrates a basic integration between the Messaging Layer and Solana using Bun and TypeScript. You can extend this further by:

* Deploying a Solana smart contract (program) to handle more complex interactions
* Using Nostr events to trigger contract calls
* Implementing payment verification or other blockchain-based logic
* Adding error handling and retry mechanisms for airdrop requests

This simple integration opens the door to building decentralized applications that combine messaging capabilities with Solana’s high-performance blockchain.

Happy coding!
