# Build a LLM Proxy

> This project is a fork of the [DePHY vending machine examples GitHub repository](https://github.com/dephy-io/dephy-vending_machine-examples), available at [https://github.com/dephy-io/dephy-deepseek\_proxy](https://github.com/dephy-io/dephy-deepseek_proxy).

## How to Run

1. [Run DePHY messaging network](https://github.com/dephy-io/dephy-messaging-network-self-hosted/tree/main/dephy-messaging-network)
2. Run DePHY vending machine workers and backend by: `docker compose up`
3. [Deploy the Solana program and run the dApp](https://github.com/dephy-io/dephy-vending_machine-examples/blob/main/balance-payment/README.md)

* Ensure all dependencies are installed before running the application.

## Messaging

The LLM Proxy controller enables token-based access to a large language model via the DePHY messaging network. A key feature is the `Transaction` event, defined in `DephyDsProxyMessage`, which serves two critical purposes:

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum DephyDsProxyMessage {
    // ... other variants (Request, Status) ...
    Transaction {
        user: String,
        tokens: i64,
    },
}
```

1. **Controller-Issued Transaction**: After a user recharges via Solana (verified with `dephy_balance_payment_sdk::pay`), the controller deducts the payment and emits a `Transaction` event. This grants the user a number of tokens for accessing the large language model, proportional to the recharge amount (e.g., 100 lamports = 100 tokens). Example:
   * **Event**: `{ user: "user_pubkey", tokens: 100 }`
   * **Purpose**: Signals that the user now has tokens available.
2. **Backend-Issued Transaction**: When the user interacts with the LLM through the backend and consumes tokens (e.g., during a chat session), the backend emits a `Transaction` event to reflect the deduction. This updates the user’s token balance. Example:
   * **Event**: `{ user: "user_pubkey", tokens: -10 }`
   * **Purpose**: Indicates 10 tokens were consumed, reducing the user’s balance.

These `Transaction` events are published to the Nostr relay with the **p** tag (e.g., "`dephy_dsproxy-controller`") and **s** tag (e.g., "`machine_pubkey`"), allowing both the controller and backend to track and synchronize token balances efficiently.

## Build Controller

* **Node**: The controller listens for user Request events to recharge tokens, processes payments and grants tokens in a single step using pay via Solana, issuing Transaction events.
* **Backend**: The backend subscribes to `Transaction` events to index token balances. When LLM usage consumes tokens, the backend publishes `Transaction` events with **negative values**, which the controller can log or validate, ensuring synchronized token tracking via the messaging layer.

## Solana Integration

Uses `pay` to deduct payment and grant tokens:

```rust
// Before: Received Request event to recharge tokens
if let Err(e) = dephy_balance_payment_sdk::pay(
    &self.solana_rpc_url,
    &self.solana_keypair_path,
    &parsed_payload.user,
    PAY_AMOUNT, // e.g., 100 lamports
    &parsed_payload.recover_info,
).await {
    tracing::error!("Failed to pay, error: {:?}", e);
    return Ok(());
}
// After: Send Transaction event to grant tokens
let tokens = PAY_AMOUNT as i64 * TOKENS_LAMPORTS_RATIO;
self.client.send_event(
    mention,
    &DephyDsProxyMessage::Transaction { 
        user: parsed_payload.user.clone(),
        tokens 
    },
).await?;
```

