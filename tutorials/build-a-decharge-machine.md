# Build a DeCharge Machine

Before proceeding, ensure you have cloned the [DePHY vending machine examples](https://github.com/dephy-io/dephy-vending_machine-examples) repository:

```bash
git clone https://github.com/dephy-io/dephy-vending_machine-examples.git
```

The DeCharge machine demonstrates a **continuous charging model**, where a device operates over time and charges users incrementally. Its messaging network leverages Nostr for event-driven communication, with Solana handling payments via `lock` and `settle`.

An online demo of these examples is available at: [https://dephy-vending-machine-examples.pages.dev/examples](https://dephy-vending-machine-examples.pages.dev/examples)

## How to Run Locally

1. [Run DePHY messaging network](https://github.com/dephy-io/dephy-messaging-network-self-hosted/tree/main/dephy-messaging-network)
2. Run DePHY vending machine workers by: `docker compose up`
3. [Deploy the Solana program and run the dApp](https://github.com/dephy-io/dephy-vending_machine-examples/blob/main/balance-payment/README.md)

* The `docker compose` setup and the App demo application integrate both **DeCharge** and **Gacha** use cases.
* Ensure all dependencies are installed before running the application.

## Messaging

*   **Message Definition**: Messages are defined as `DephyDechargeMessage` (in `message.rs`), with two variants:

    * `Request`: Initiates a state change (e.g., user requests the machine to start).
    * `Status`: Confirms a state update (e.g., machine is now Working).

    ```rust
    pub enum DephyDechargeMessage {
        Request { 
            to_status: DephyDechargeStatus, 
            reason: DephyDechargeStatusReason, 
            initial_request: EventId, 
            payload: String, 
        },
        Status { 
            status: DephyDechargeStatus, 
            reason: DephyDechargeStatusReason, 
            initial_request: EventId, 
            payload: String, 
        },
    }
    ```
* **Mention Tag (Machine Pubkey)**: The `p` tag identifies the target machine using its Nostr public key. For example, `PublicKey::parse("machine_pubkey")` specifies which device receives the message.
* **Session Tag**: The `s` tag scopes events to a specific session (e.g., "`dephy-decharge-controller`"), ensuring messages are isolated to the current context.
*   **Publishing and Subscription**: The `RelayClient` publishes messages to a Nostr relay and subscribes to events using filters:

    ```rust
    let filter = Filter::new()
        .kind(EVENT_KIND) // Custom kind 1573
        .since(started_at)
        .custom_tag(SESSION_TAG, ["dephy-decharge-controller"])
        .custom_tag(MENTION_TAG, [machine_pubkey.to_hex()]);
    let sub_id = relay_client.subscribe(started_at, [machine_pubkey]).await?;
    ```

    * **Filter**: Retrieves events since `started_at`, scoped to the session and machine pubkey.
    * **Sender**: Typically a user or admin via the dApp or CLI.
    * **Receiver**: The DeCharge node controller handling the machine.
* **Message Handling**: The MessageHandler processes events, updating machine states and coordinating with Solana.

## Building Controller

* **Node**: Listens for user `Request` events, verifies eligibility with `check_eligible`, and updates machine state (e.g., `Available` to `Working`). It notifies the server via `Status` events.
* **Server**: Monitors `Status` events, locks funds with `lock` when the machine starts, and settles the final amount with `settle` when it stops. This split ensures state management and payment processing are separate but coordinated.

## Solana Interaction

* **Controller**: Initiates state change and verifies eligibility

```rust
// Before: Received Request event to start machine
if dephy_balance_payment_sdk::check_eligible(
    &self.solana_rpc_url,
    parsed_payload.namespace_id,
    &parsed_payload.user,
    parsed_payload.nonce,
    PREPAID_AMOUNT,
    &parsed_payload.recover_info,
).await? {
    // After: Send Status event to set machine to Working
    self.client.send_event(mention, &DephyDechargeMessage::Status { ... }).await?;
}
```

*   **Server**: Locks and settles funds

    ```rust
    // When machine starts (Working status)
    // Before: Received Status event indicating machine is Working
    if let Err(e) = dephy_balance_payment_sdk::lock(
        &self.solana_rpc_url,
        &self.solana_keypair_path,
        parsed_payload.namespace_id,
        &parsed_payload.user,
        PREPAID_AMOUNT,
        &parsed_payload.recover_info,
    ).await {
        // After: Send Request event to revert to Available if lock fails
        self.client.send_event(mention, &DephyDechargeMessage::Request { to_status: DephyDechargeStatus::Available, ... }).await?;
    }

    // When machine stops (Available status after 60s)
    // Before: Received Status event indicating machine is Available
    if let Err(e) = dephy_balance_payment_sdk::settle(
        &self.solana_rpc_url,
        &self.solana_keypair_path,
        &parsed_payload.user,
        parsed_payload.nonce,
        TRANSFER_AMOUNT,
    ).await {
        tracing::error!("Settle failed: {:?}", e);
    }
    // After: No event sent; balance is settled
    ```

