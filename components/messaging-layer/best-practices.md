# Best Practices

To ensure the Messaging Layer’s stability and maintainability in production, the following best practices are derived from your code’s application context and are applicable to similar event-driven systems.

* **Auto Reconnection**:

Network interruptions, relay downtime, or intentional connection closures can disrupt message flow, making automatic reconnection a key requirement. A well-implemented reconnection strategy ensures the system recovers gracefully from failures without overwhelming resources or losing events.

1. For **TS/JS** developers, we implement a custom reconnection mechanism with exponential backoff for the `nostr-tools` library:

```typescript
import { Relay } from "nostr-tools/relay";

// Configuration constants
const RELAY_URL = "wss://dev-relay.dephy.dev";
const RECONNECT_DELAY_INITIAL_MS = 1000;  // Initial reconnect delay (1 second)
const RECONNECT_DELAY_MAX_MS = 30000;     // Maximum reconnect delay (30 seconds)
const RECONNECT_MAX_ATTEMPTS = 10;        // Maximum reconnection attempts (optional)

// Global variables to manage connection state
let relay: Relay | null = null;           // Active relay connection
let reconnectAttempts = 0;                // Track number of reconnection attempts
let reconnectDelay = RECONNECT_DELAY_INITIAL_MS; // Current reconnect delay

/**
 * Establishes connection to the Nostr relay and sets up event subscription.
 * Handles automatic reconnection on failure.
 */
async function connectToRelay() {
  try {
    console.log(`Connecting to relay: ${RELAY_URL}...`);
    
    // Attempt to connect to the relay
    relay = await Relay.connect(RELAY_URL);
    
    // Reset reconnection state on successful connection
    reconnectAttempts = 0;
    reconnectDelay = RECONNECT_DELAY_INITIAL_MS;
    
    console.log("Connected to relay. Subscribing to events...");

    // Subscribe to events matching the specified filter
    relay.subscribe(
      [
        {
          kinds: [1573],  // Event kind (1573 for custom messages)
          since: Math.floor(Date.now() / 1000),  // Timestamp in seconds
          "#s": ["hello_session"],  // 's' tag filter
          "#p": ["receiver_pubkey"],  // 'p' tag filter
        },
      ],
      {
        // Callback for received events
        onevent: (event) => {
          console.log("Received:", event.content);
        },
      }
    );

    /**
     * Handle connection closure.
     * This callback triggers when the relay intentionally closes the connection.
     */
    relay.onclose = () => {
      console.log("Connection closed by relay. Attempting to reconnect...");
      scheduleReconnect();
    };

  } catch (err) {
    // Handle initial connection failures
    console.error("Initial connection failed:", err);
    scheduleReconnect();
  }
}

/**
 * Schedules a reconnection attempt using exponential backoff.
 * Increases delay between attempts up to RECONNECT_DELAY_MAX_MS.
 */
function scheduleReconnect() {
  // Stop reconnecting after maximum attempts
  if (reconnectAttempts >= RECONNECT_MAX_ATTEMPTS) {
    console.error("Maximum reconnection attempts reached. Terminating.");
    return;
  }

  reconnectAttempts++;
  console.log(`Next reconnection attempt in ${reconnectDelay / 1000} seconds... (Attempt ${reconnectAttempts})`);

  // Schedule reconnection with increasing delay
  setTimeout(() => {
    connectToRelay();
    // Exponential backoff: double delay each time (capped at max delay)
    reconnectDelay = Math.min(reconnectDelay * 2, RECONNECT_DELAY_MAX_MS);
  }, reconnectDelay);
}

// Graceful shutdown handler (e.g., Ctrl+C)
process.on("SIGINT", () => {
  console.log("Terminating gracefully...");
  if (relay) {
    relay.close();  // Properly close the connection
  }
  process.exit(0);  // Exit with success code
});

// Start initial connection
connectToRelay();
```

2. For **Rust** developers using the Nostr library (e.g., `nostr-sdk`), reconnection is handled natively by the library so developers don’t need to implement manual reconnection.
3. For **Go** developers, a reconnection mechanism for Nostr relays is not yet standardized in popular libraries as of April 2025. Developers may need to implement a custom solution similar to the TypeScript example above, examples repo at: [https://github.com/dephy-io/ml-pubsub-go-example](https://github.com/dephy-io/ml-pubsub-go-example).

* **Event Consistency**: Always validate message sources and targets to avoid processing unauthorized or irrelevant events. For example, when updating device states, confirm that the message originates from a legitimate admin or user request.
* **Error Handling and Logging**: Equip each operation (subscriptions, payment validation, state changes) with detailed logging and implement graceful error recovery on critical paths. For instance, if a payment fails, revert the state and notify stakeholders promptly.
* **Time Synchronization**: Since the system relies on timestamps for event filtering and ordering, periodically verify time consistency across nodes to prevent event loss or duplicate processing due to drift.
* **Resource Management**: When subscribing to numerous events or managing multiple devices, set reasonable limits on notification queue sizes and subscription exit policies to prevent memory overflow or performance degradation. Your code demonstrates this by configuring maximum notification sizes.
* **Security**: Safeguard keys and payment-related data (e.g., Solana keypairs) and ensure message content is encrypted or signed to prevent tampering or forgery. Nostr’s public key system provides a foundational layer of security here.
* **Testing and Simulation**: Before deployment, test system behavior with simulated events and payment scenarios, especially under high concurrency or network disruptions. This helps identify edge cases and potential issues.

These practices are partially reflected in your code, such as avoiding redundant operations via state checks and using logs for debugging. Adhering to these guidelines can further enhance the system’s reliability and scalability.
