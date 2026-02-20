# Phase 6 Handover Implementation Plan

**Status:** Draft
**Target:** Strict separation of concerns with graceful degradation.

## 1. Objective

Transition `cl-hive` from a monolithic plugin into a **Coordination Layer** that orchestrates the fleet by consuming services from specialized plugins (`cl-hive-comms`, `cl-hive-archon`).

Crucially, `cl-hive` must retain "Dual-Stack" capability:
1.  **Monolith Mode (Fallback):** If specialized plugins are missing, use internal modules for Transport (Nostr) and Identity (Local Keys).
2.  **Coordinated Mode (Target):** If plugins are detected, disable internal modules and delegate Transport/Identity via RPC.

---

## 2. Architecture

### Tiered Functionality Matrix

| Tier | Installed Plugins | Mode | Capabilities |
| :--- | :--- | :--- | :--- |
| **Stealth / Basic** | **`cl-hive` Only** | **Local / P2P** | • **Routing & Settlements:** Fully functional.<br>• **Gossip:** Mesh propagation (Bolt8).<br>• **Privacy:** Maximum (Dark Fleet).<br>• **Transport:** Internal Nostr (optional/fallback). |
| **Public / Basic** | **`cl-hive` + `comms`** | **Networked** | • **Global Reach:** `comms` handles Nostr.<br>• **Advisor Access:** Via `comms` marketplace.<br>• **Identity:** Nostr-only (via `comms`). |
| **Full Member** | **Full Stack** | **Sovereign** | • **Governance:** Voting via `archon`.<br>• **Identity:** DID via `archon`.<br>• **Security:** Keys isolated in `archon` vault. |

---

## 3. Implementation Steps

### Phase A: Specialized Plugin Enhancements

Before `cl-hive` can offload duties, the specialized plugins must expose the necessary "service" RPCs.

#### 1. `cl-hive-comms` (The Body)
Needs to allow the Brain (`cl-hive`) to speak and listen.

*   **NEW RPC:** `hive-comms-send-dm(recipient_pubkey, plaintext)`
    *   Allows `cl-hive` to send protocol messages (Handshakes, Intents) using `comms`'s connection and identity.
*   **NEW RPC:** `hive-comms-publish-event(event_json)`
    *   Allows `cl-hive` to broadcast Gossip and Marketplace offers.
*   **NEW FEATURE:** Inbound Message Forwarding
    *   `cl-hive-comms` needs a mechanism to push decrypted DMs (Intents, Handshakes) to `cl-hive`.
    *   *Mechanism:* `cl-hive` registers an RPC `hive-inject-packet`. `cl-hive-comms` calls this when it receives a relevant DM.

#### 2. `cl-hive-archon` (The Passport)
Needs to provide signing services so `cl-hive` doesn't need the private key.

*   **NEW RPC:** `hive-archon-sign-message(message_hex)` (or similar)
    *   Allows `cl-hive` to sign gossip/payloads with the DID key.
*   **VERIFY:** Ensure `hive-archon-status` returns the full public DID Identity for `cl-hive` to use in headers.

---

### Phase B: `cl-hive` Refactoring (The Brain)

#### 1. Transport Adapter Pattern
Refactor `modules/nostr_transport.py` to support swappable backends.

*   **Abstract Base Class:** `TransportInterface`
*   **Implementation A:** `InternalNostrTransport` (The current code).
*   **Implementation B:** `ExternalCommsTransport` (The Bridge).
    *   `send_dm()` -> calls `plugin.rpc.call("hive-comms-send-dm", ...)`
    *   `publish()` -> calls `plugin.rpc.call("hive-comms-publish-event", ...)`

#### 2. Identity Adapter Pattern
Refactor `modules/did_credentials.py` to support remote identity.

*   **Implementation A:** `LocalIdentity` (Current logic, generates/stores keys in `cl_hive.db`).
*   **Implementation B:** `RemoteArchonIdentity` (The Bridge).
    *   `get_pubkey()` -> calls `hive-archon-status`.
    *   `sign()` -> calls `hive-archon-sign-message`.

#### 3. Inbound Ingestion RPC
*   **NEW RPC:** `hive-inject-packet(payload)`
    *   Exposed by `cl-hive` ONLY when running in Coordinated Mode.
    *   Accepts decrypted payloads from `cl-hive-comms`.
    *   Feeds them into the existing `_dispatch_hive_message` logic.

---

### Phase C: Orchestration & Initialization

Update `cl-hive.py` `init()` logic to wire the components.

1.  **Detection:** Call `_detect_phase6_optional_plugins`.
2.  **Selection:**
    *   If `cl-hive-comms` is active:
        *   Instantiate `ExternalCommsTransport`.
        *   Register `hive-inject-packet`.
        *   **Suppress** registration of conflicting user RPCs (`hive-client-*`) to avoid CLN startup errors (let `comms` handle the user-facing client commands).
    *   Else:
        *   Instantiate `InternalNostrTransport`.
        *   Register all RPCs as normal.
3.  **Identity:**
    *   If `cl-hive-archon` is active:
        *   Instantiate `RemoteArchonIdentity`.
    *   Else:
        *   Instantiate `LocalIdentity`.

---

## 4. Verification Checklist

1.  **Standalone Test:** Run `cl-hive` alone. Verify it generates keys, connects to relays, and gossips.
2.  **Full Stack Test:** Run all 3 plugins.
    *   Verify `cl-hive` does **not** open a WebSocket.
    *   Verify `cl-hive` logs "Using External Transport (cl-hive-comms)".
    *   Trigger a gossip broadcast -> Verify it goes out via `cl-hive-comms`.
    *   Receive a DM -> Verify `cl-hive-comms` decrypts it and pushes to `cl-hive` via `hive-inject-packet`.
