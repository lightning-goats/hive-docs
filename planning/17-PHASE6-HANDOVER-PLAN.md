# Phase 6 Handover Implementation Plan

**Status:** Draft
**Target:** Strict microservice architecture with hard dependencies on specialized plugins.

## 1. Objective

Transition `cl-hive` from a standalone monolithic plugin into a strict **Coordination Layer** that orchestrates the fleet by consuming services from specialized plugins (`cl-hive-comms`, `cl-hive-archon`). 

Monolithic operation is being completely purged. `cl-hive` will no longer generate its own keys or manage its own Nostr websocket connections. **Coordinated Mode** is the *only* mode of operation, and `cl-hive` will only enable its functionality if the required companion plugins are detected on the node.

---

## 2. Architecture

### Strict Microservice Model

The fleet operates as a trio of dependent services. `cl-hive` acts as the "Brain" and refuses to start unless the "Body" and "Passport" are present.

*   **`cl-hive` (The Brain):** Pure coordination, routing, mesh propagation logic, and settlement handling. Contains zero native transport or identity generation code.
*   **`cl-hive-comms` (The Body):** Handles all external network communication (Nostr websockets). Provides global reach and client command interfaces.
*   **`cl-hive-archon` (The Passport):** Handles DID identity, sovereign voting, and acts as the secure vault for signing.

---

## 3. Implementation Steps

### Phase A: Specialized Plugin Enhancements

Before `cl-hive` can be stripped of its monolithic code, the specialized plugins must expose the necessary "service" RPCs for the Coordination Layer to consume.

#### 1. `cl-hive-comms` (Transport Service)
Needs to allow `cl-hive` to speak and listen over the network.

*   **NEW RPC:** `hive-comms-send-dm(recipient_pubkey, plaintext)`
    *   Allows `cl-hive` to send protocol messages (Handshakes, Intents) using `comms`'s connection.
*   **NEW RPC:** `hive-comms-publish-event(event_json)`
    *   Allows `cl-hive` to broadcast Gossip and Marketplace offers.
*   **NEW FEATURE:** Inbound Message Forwarding
    *   `cl-hive-comms` needs a mechanism to push decrypted DMs (Intents, Handshakes) to `cl-hive`.
    *   *Mechanism:* `cl-hive` exposes an RPC `hive-inject-packet`. `cl-hive-comms` calls this when it receives a relevant DM.

#### 2. `cl-hive-archon` (Identity Service)
Needs to provide signing and identity services so `cl-hive` operates without private keys.

*   **NEW RPC:** `hive-archon-sign-message(message_hex)`
    *   Allows `cl-hive` to sign gossip/payloads with the DID key securely held in the archon vault.
*   **VERIFY:** Ensure `hive-archon-status` returns the full public DID Identity for `cl-hive` to use in headers and payload construction.

---

### Phase B: Purging the Monolith (`cl-hive` Refactoring)

#### 1. Transport Layer Purge & Bridge
Delete internal connection management in `modules/nostr_transport.py` and replace it entirely with an external bridge.

*   **REMOVE:** `InternalNostrTransport` (websocket connections, relay management, internal event publishing).
*   **IMPLEMENT:** `ExternalCommsTransport` (The Bridge).
    *   `send_dm()` -> strictly routes via `plugin.rpc.call("hive-comms-send-dm", ...)`
    *   `publish()` -> strictly routes via `plugin.rpc.call("hive-comms-publish-event", ...)`

#### 2. Identity Layer Purge & Bridge
Delete local identity generation in `modules/did_credentials.py` and replace it entirely with a remote bridge.

*   **REMOVE:** `LocalIdentity` (generating/storing keys in `cl_hive.db`).
*   **IMPLEMENT:** `RemoteArchonIdentity` (The Bridge).
    *   `get_pubkey()` -> queries `hive-archon-status`.
    *   `sign()` -> requests signature via `hive-archon-sign-message`.

#### 3. Inbound Ingestion RPC
*   **NEW RPC:** `hive-inject-packet(payload)`
    *   Accepts decrypted payloads pushed from `cl-hive-comms`.
    *   Feeds them directly into the existing `_dispatch_hive_message` logic.

---

### Phase C: Orchestration & Initialization

Update `cl-hive.py` `getmanifest` and `init` logic to enforce the strict dependency requirements.

1.  **Dependency Detection:** During initialization, call `_detect_required_plugins` to check for the active presence of `cl-hive-comms` and `cl-hive-archon`.
2.  **Enforcement:**
    *   **If missing either plugin:** `cl-hive` logs a critical warning (`"Required plugins cl-hive-comms and/or cl-hive-archon not found. cl-hive will disable itself."`) and gracefully disables its functionality (either by passing the `disable` flag in the manifest or idling the plugin without registering operational hooks).
    *   **If both plugins are active:** Coordinated mode is enabled.
3.  **Bootstrapping:**
    *   Instantiate `ExternalCommsTransport`.
    *   Instantiate `RemoteArchonIdentity`.
    *   Register `hive-inject-packet`.
    *   Suppress registration of legacy user-facing RPCs (`hive-client-*`) as these are now exclusively handled by `cl-hive-comms`.

---

## 4. Verification Checklist

1.  **Missing Dependency Test (Comms):** Disable `cl-hive-comms`. Start CLN. Verify `cl-hive` detects the missing dependency, logs the appropriate error, and safely disables itself without crashing the node.
2.  **Missing Dependency Test (Archon):** Disable `cl-hive-archon`. Start CLN. Verify `cl-hive` disables itself.
3.  **Full Stack Test:** Run all 3 plugins.
    *   Verify `cl-hive` starts successfully in Coordinated Mode.
    *   Verify `cl-hive` opens **zero** websockets/network connections of its own.
    *   Trigger a gossip broadcast -> Verify `cl-hive` successfully passes the payload to `cl-hive-archon` for signing, and `cl-hive-comms` for broadcasting.
    *   Receive a DM -> Verify `cl-hive-comms` decrypts it and successfully pushes it to `cl-hive` via the `hive-inject-packet` RPC.
