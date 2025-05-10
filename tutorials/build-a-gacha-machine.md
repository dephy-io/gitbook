# Build a Gacha Machine

Before proceeding, ensure you have cloned the [DePHY vending machine examples](https://github.com/dephy-io/dephy-vending_machine-examples) repository:

```bash
git clone https://github.com/dephy-io/dephy-vending_machine-examples.git
```

The Gacha machine implements a single-charge model, charging a fixed amount for a one-time action (e.g., dispensing an item). It simplifies the messaging and payment logic compared to DeCharge.

An online demo of these examples is available at: [https://dephy-vending-machine-examples.pages.dev/examples](https://dephy-vending-machine-examples.pages.dev/examples)

## How to Run Locally

1. [Run DePHY messaging network](https://github.com/dephy-io/dephy-messaging-network-self-hosted/tree/main/dephy-messaging-network)
2. Run DePHY vending machine workers by: `docker compose up`
3. [Deploy the Solana program and run the dApp](https://github.com/dephy-io/dephy-vending_machine-examples/blob/main/balance-payment/README.md)

* The `docker compose` setup and the App demo application integrate both **DeCharge** and **Gacha** use cases.
* Ensure all dependencies are installed before running the application.

## Messaging

* **Message Definition**: Same as DeCharge (`DephyDechargeMessage`), but typically only processes `Request` to `Working` and `Status` back to `Available`.
* **Mention Tag (Machine Pubkey)**: Identifies the GaCha machine (e.g., `PublicKey::parse("gacha_machine_pubkey")`).
* **Session Tag**: Scopes events (e.g., `"dephy-gacha-controller"`).
*   **Publishing and Subscription**

    ```rust
    let filter = Filter::new()
        .kind(EVENT_KIND)
        .since(Timestamp::now())
        .custom_tag(SESSION_TAG, ["gacha_session"])
        .custom_tag(MENTION_TAG, [gacha_machine_pubkey.to_hex()]);
    let sub_id = relay_client.subscribe(Timestamp::now(), [gacha_machine_pubkey]).await?;
    ```

    * **Sender**: User requesting a dispense.
    * **Receiver**: Gacha node controller.
* **Message Handling**: Processes a single transaction and reverts state.

## Building Controller

* **Node**: Unlike DeCharge, Gacha’s simpler transaction (one payment, one action) doesn’t require a separate server. The node handles both state and payment in a single step using `pay`, making it lightweight.

## Solana Interaction

Uses `pay` for a one-time transaction:

```rust
// Before: Received Request event to start machine
if let Err(e) = dephy_balance_payment_sdk::pay(
    &self.solana_rpc_url,
    &self.solana_keypair_path,
    parsed_payload.namespace_id,
    &parsed_payload.user,
    PAY_AMOUNT,
    &parsed_payload.recover_info,
)
.await
{
    tracing::error!("Failed to pay, error: {:?} skip event: {:?}", e, event);
    return Ok(());
};

self.client
    .send_event(mention, &DephyGachaMessage::Status {
        status: *to_status,
        reason: *reason,
        initial_request: event.id,
        payload: serde_json::to_string(&DephyGachaMessageStatusPayload {
            namespace_id: parsed_payload.namespace_id,
            user: parsed_payload.user.clone(),
            nonce: parsed_payload.nonce,
            recover_info: parsed_payload.recover_info.clone(),
        })?,
    })
    .await?;
```

