# Build a Hello World (Rust)

## Prerequisites

Before you begin, ensure you have the following installed:

* [Rust](https://www.rust-lang.org/tools/install)

***

## **Step 1: Create a Rust Workspace**

First, create a new directory for the project and navigate into it:

```bash
mkdir hello-world-ml
cd hello-world-ml
```

Next, create the packages inside the workspace:

```bash
cargo new subscriber --vcs none
cargo new publisher --vcs none
```

Finally, manually create a `Cargo.toml` file in the **root directory** for the workspace with the following content:

```toml
[workspace]
members = ["subscriber", "publisher"]

[workspace.dependencies]
nostr-sdk = "0.38.0"  
nostr = "0.38.0"
tokio = { version = "1.38.0", features = ["full"] }
```

This ensures that both subscriber and publisher share the same dependency versions and avoids issues from creating the workspace `Cargo.toml` before the packages.

***

## **Step 2: Write the Subscriber Code**

Edit `subscriber/Cargo.toml` to use workspace dependencies:

```toml
[package]
name = "subscriber"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { workspace = true }
nostr = { workspace = true }
nostr-sdk = { workspace = true }
```

Replace the contents of `subscriber/src/main.rs` with this code to subscribe to "Hello World" events:

```rust
use nostr::{key::Keys, types::{SingleLetterTag, Timestamp}, Kind};
use nostr_sdk::{Client, Filter, RelayPoolNotification};

const MENTION_TAG: SingleLetterTag = SingleLetterTag::lowercase(nostr::Alphabet::P);
const SESSION_TAG: SingleLetterTag = SingleLetterTag::lowercase(nostr::Alphabet::S);

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize the Nostr client
    let keys: Keys = Keys::generate();
    let client_opts = nostr_sdk::Options::default();

    let client = Client::builder()
        .signer(keys.clone())
        .opts(client_opts)
        .build();

    client.add_relay("wss://dev-relay.dephy.dev").await?;
    client.connect().await;

    // Define a filter for "Hello World" events
    let filter = Filter::new()
        .kind(Kind::Custom(1573))
        .since(Timestamp::now())
        .custom_tag(SESSION_TAG, ["hello_session"])
        .custom_tag(MENTION_TAG, ["receiver_pubkey"]);

    // Subscribe to the filter
    client.subscribe(vec![filter], None).await?;

    println!("Subscribed events on wss://dev-relay.dephy.dev");

    // Handle notifications asynchronously
    client
        .handle_notifications(|notification| async {
            match notification {
                RelayPoolNotification::Event { event, .. } => {
                    println!("Received: {}", event.content);
                }
                _ => {} // Ignore other notification types
            }
            Ok(false) // Keep listening (false means don't stop)
        })
        .await?;

    Ok(())
}
```

***

## **Step 3: Write the Publisher Code**

Edit `publisher/Cargo.toml` to use workspace dependencies:

```toml
[package]
name = "publisher"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { workspace = true }
nostr = { workspace = true }
nostr-sdk = { workspace = true }
```

Edit `publisher/src/main.rs` :

```rust
use nostr::{key::Keys, Kind};
use nostr_sdk::{Client, EventBuilder, Options, Tag};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize the Nostr client
    let keys: Keys = Keys::generate(); // Generate a keypair for the publisher
    let client_opts = Options::default();

    let client = Client::builder()
        .signer(keys.clone())
        .opts(client_opts)
        .build();

    client.add_relay("wss://dev-relay.dephy.dev").await?;
    client.connect().await;

    // Create and send a "Hello World" event
    let event = EventBuilder::new(Kind::Custom(1573), "Hello World").tags([
        Tag::parse(["s".to_string(), "hello_session".to_string()]).unwrap(),
        Tag::parse(["p".to_string(), "receiver_pubkey".to_string()]).unwrap(),
    ]);

    client.send_event_builder(event).await?;
    println!("Published 'Hello World' event to wss://dev-relay.dephy.dev");

    Ok(())
}
```

***

## **Step 4: Compile and Run**:

*   Run the `Subscriber`:

    ```bash
    cargo run -p subscriber
    ```

    This starts the subscriber, which will wait for events.
*   Run the `Publisher` (in a new terminal):

    ```bash
    cargo run -p publisher
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

🎉At this point, the basic publish and subscribe functionality is complete, and the command line has output the results we expected!&#x20;

Next, we will enhance this by integrating with the Solana blockchain and the Messaging Layer.Specifically, when publishing a greeting like "Hello World," the publisher will request a Solana airdrop.

***

## Step 5: Update the Workspace Dependencies

```toml
[workspace]
members = ["subscriber", "publisher"]

[dependencies]
tokio = { workspace = true }
nostr = { workspace = true }
nostr-sdk = { workspace = true }
solana-sdk = { workspace = true }
serde = { workspace = true }
serde_json = { workspace = true }
```

***

## Step 6: Update the Publisher

Replace the contents of `publisher/src/main.rs` with this code to publish a message with a Solana public key:

```rust
use nostr::{key::Keys, Kind};
use nostr_sdk::{Client, EventBuilder, Options, Tag};
use serde::Serialize;
use solana_sdk::{signature::Keypair, signer::Signer};

#[derive(Serialize)]
struct Message {
    greeting: String,
    solana_pubkey: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Generate Solana keypair
    let solana_keypair = Keypair::new();
    let solana_pubkey = solana_keypair.pubkey().to_string();

    // Initialize the Nostr client
    let keys: Keys = Keys::generate(); // Generate a keypair for the publisher
    let client_opts = Options::default();

    let client = Client::builder()
        .signer(keys.clone())
        .opts(client_opts)
        .build();

    client.add_relay("wss://dev-relay.dephy.dev").await?;
    client.connect().await;

    // Create message struct
    let message = Message {
        greeting: "Hello World".to_string(),
        solana_pubkey: solana_pubkey.clone(),
    };
    
    // Create and send a "Hello World" event
    let event = EventBuilder::new(Kind::Custom(1573), serde_json::to_string(&message)?).tags([
        Tag::parse(["s".to_string(), "hello_session".to_string()])?,
        Tag::parse(["p".to_string(), "receiver_pubkey".to_string()])?,
    ]);

    client.send_event_builder(event).await?;
    println!("Published 'Hello World' event to wss://dev-relay.dephy.dev");
    println!("Airdrop request for {}", solana_pubkey);

    Ok(())
}
```

***

## Step 7: Update the Subscriber

Edit `subscriber/Cargo.toml` to use workspace dependencies:

```toml
[package]
name = "subscriber"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { workspace = true }
nostr = { workspace = true }
nostr-sdk = { workspace = true }
solana-sdk = { workspace = true }
solana-client = { workspace = true }
serde = { workspace = true }
serde_json = { workspace = true }
```

Replace the contents of `subscriber/src/main.rs` with this code to subscribe to events and request a Solana airdrop:

```rust
use nostr::{key::Keys, types::{SingleLetterTag, Timestamp}, Kind};
use nostr_sdk::{Client, Filter, RelayPoolNotification};
use serde::Deserialize;
use solana_client::rpc_client::RpcClient;
use solana_sdk::pubkey::Pubkey;
use std::str::FromStr;

#[derive(Deserialize)]
struct Message {
    greeting: String,
    solana_pubkey: String,
}

const MENTION_TAG: SingleLetterTag = SingleLetterTag::lowercase(nostr::Alphabet::P);
const SESSION_TAG: SingleLetterTag = SingleLetterTag::lowercase(nostr::Alphabet::S);

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Initialize the Nostr client
    let keys: Keys = Keys::generate();
    let client_opts = nostr_sdk::Options::default();

    let client = Client::builder()
        .signer(keys.clone())
        .opts(client_opts)
        .build();

    client.add_relay("wss://dev-relay.dephy.dev").await?;
    client.connect().await;

    // Define a filter for "Hello World" events
    let filter = Filter::new()
        .kind(Kind::Custom(1573))
        .since(Timestamp::now())
        .custom_tag(SESSION_TAG, ["hello_session"])
        .custom_tag(MENTION_TAG, ["receiver_pubkey"]);

    // Subscribe to the filter
    client.subscribe(vec![filter], None).await?;

    println!("Subscribed events on wss://dev-relay.dephy.dev");

    // Initialize Solana testnet client
    let solana_client = RpcClient::new("https://api.devnet.solana.com".to_string());

    // Handle notifications asynchronously
    client
        .handle_notifications(|notification| async {
            match notification {
                RelayPoolNotification::Event { event, .. } => {
                    println!("Received {}", event.content);
                    // Parse the JSON content
                    if let Ok(message) = serde_json::from_str::<Message>(&event.content) {
                        // Request airdrop to the solana_pubkey
                        if let Ok(pubkey) = Pubkey::from_str(&message.solana_pubkey) {
                            match solana_client.request_airdrop(&pubkey, 1_000_000_000) {
                                Ok(signature) => {
                                    println!(
                                        "Airdrop requested for {}. Signature: {}",
                                        pubkey, signature
                                    );
                                    if let Ok(confirmed) =
                                        solana_client.confirm_transaction(&signature)
                                    {
                                        if confirmed {
                                            println!("Airdrop confirmed for {}", pubkey);
                                        }
                                    }
                                }
                                Err(e) => println!("Airdrop failed: {:?}", e),
                            }
                        } else {
                            println!("Invalid Solana pubkey: {}", message.solana_pubkey);
                        }
                    } else {
                        println!("Failed to parse message content");
                    }
                }
                _ => {} // Ignore other notification types
            }
            Ok(false) // Keep listening (false means don't stop)
        })
        .await?;

    Ok(())
}
```

***

## Step 8: Compile and Run

* Run the `Subscriber` (in one terminal):

```bash
cargo run -p subscriber
```

* Run the `Publisher` (in a new terminal):

```bash
cargo run -p publisher
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

## Next Steps

This example demonstrates a basic integration between `Messaging Layer` and `Solana`. You could extend this further by:

* Deploying a Solana smart contract (program) to handle more complex interactions
* Using Nostr events to trigger contract calls
* Implementing payment verification or other blockchain-based logic
* Adding error handling and retry mechanisms for airdrop requests

This simple integration opens the door to building decentralized applications that combine messaging  capabilities with Solana's high-performance blockchain.
