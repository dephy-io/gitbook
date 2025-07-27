---
description: 'DePHY DePIN Infra: Data pub-sub.'
---

# Data Pub-Sub

The Messaging Layer is built around a publish-subscribe (Pub-sub) model for data distribution, facilitating the exchange of device states and control commands across distributed nodes. This approach is particularly suited for scenarios requiring real-time responses, such as device management or user-initiated requests.

* **Event-Driven Communication**: The system defines specific event types (e.g., device state changes or user requests) to propagate messages between nodes. Nodes subscribe to relevant event streams and update their local state or trigger actions based on received events, ensuring low-latency and high-throughput messaging.
* **Decentralized Subscriptions**: By leveraging the Nostr protocol, the Messaging Layer allows multiple nodes (e.g., device controllers and servers) to subscribe to the same event source without relying on a centralized server. Subscribers can filter events by criteria like timestamps or device identifiers, processing only pertinent data.
* **Historical Data Synchronization**: The system supports retrieving events from a specified time window, enabling newly joined nodes to quickly sync historical states. This feature is critical for fault tolerance and state recovery during system startups, such as restoring a device’s latest status after a reboot.
* **Flexible Event Tagging**: Custom tags (e.g., session IDs and device public keys) enable precise message routing to target devices or processing logic. This flexibility supports multi-tenant environments or complex event distribution needs.

In practice, this mechanism tracks device availability or working status and broadcasts updates to all relevant parties, maintaining system-wide consistency.

## Hello World!

### Prerequisites

Before you begin, ensure you have the following installed:

* [Rust](https://www.rust-lang.org/tools/install)

### **Step 1: Create a Rust Workspace**

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

### **Step 2: Write the Subscriber Code**

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

### **Step 3: Write the Publisher Code**

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

### **Step 4: Compile and Run**:

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
