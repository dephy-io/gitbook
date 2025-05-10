# RPC Call

## Connecting to the Messaging Layer

To interact with the Messaging Layer, you have two options: connect to DePHY’s public relay URL or deploy a local instance. Choose based on your needs—public for simplicity, local for development or control.

### 1. Use DePHY’s Relay URL

PRC Endpoints can be found here:

{% content-ref url="rpc-endpoints.md" %}
[rpc-endpoints.md](rpc-endpoints.md)
{% endcontent-ref %}

### 2. Local Deployment

* **Prerequisites**:
  * Docker: Ensure [Docker](https://docs.docker.com/get-docker/) is installed and running on your system to manage the messaging network services.
* **Steps**:
  1.  Clone the repository:

      ```bash
      git clone https://github.com/dephy-io/dephy-messaging-network-self-hosted.git
      cd dephy-messaging-network-self-hosted/dephy-messaging-network
      ```
  2.  Launch the messaging layer using Docker Compose:

      ```bash
      docker compose up --pull always
      ```

This deploys a self-hosted Nostr-based messaging network locally, ready to handle event subscriptions and publications for your DePHY applications. By default, it will be accessible at `wss://localhost:8080`.

## Recommended Messaging Layer Client

Below are recommended libraries for interacting with the Nostr protocol across multiple programming languages, with hyperlinks to their official repositories or documentation.

### **Rust**

* [nostr-sdk](https://github.com/rust-nostr/nostr/tree/master/crates/nostr-sdk)
* [nostr](https://github.com/rust-nostr/nostr/tree/master/crates/nostr)

### **JavaScript/TypeScript**

* [nostr-tools](https://github.com/nbd-wtf/nostr-tools)
* [nostr-relay](https://github.com/fiatjaf/nostr-relay)

### **Python**

* [pynostr](https://github.com/fernandoporazi/pynostr)
* [nostr-py](https://github.com/monty888/nostr-py)

### **Go**

* [go-nostr](https://github.com/nbd-wtf/go-nostr)

### **Kotlin**

* [nostr-kotlin](https://github.com/derkan/nostr-kotlin)

### **Swift**

* [NostrKit](https://github.com/lzich/NostrKit)
