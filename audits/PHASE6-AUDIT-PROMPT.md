# Phase 6 Independent Security Audit Prompt

**Purpose:** This prompt enables an independent AI auditor to conduct a comprehensive security, correctness, and completeness audit of the Phase 6 three-plugin stack (cl-hive-comms, cl-hive-archon, and Docker integration scaffolding in cl-hive).

**Usage:** Provide this prompt to a fresh AI session along with the source files listed below. The auditor should have no prior context about remediations or known issues.

---

## Instructions

You are a senior security auditor specializing in cryptocurrency infrastructure, Lightning Network plugins, and Python security. Conduct a thorough independent audit of the Phase 6 plugin stack described below. Your audit must be systematic, evidence-based, and reference specific file paths and line numbers.

### Audit Scope

Three CLN (Core Lightning) plugins forming a distributed identity, transport, and governance layer for Lightning node fleet management:

| Plugin | Repository Path | Purpose |
|--------|----------------|---------|
| **cl-hive-comms** | `/home/sat/bin/cl-hive-comms/` | Nostr DM + REST transport, advisor management, policy engine, payment tracking |
| **cl-hive-archon** | `/home/sat/bin/cl-hive-archon/` | DID identity provisioning, credential binding, governance polling |
| **Docker scaffolding** | `/home/sat/bin/cl-hive/docker/` | Container integration for optional Phase 6 plugin loading |

### Files to Audit

Read and audit every file listed below. Do NOT skip any file.

**cl-hive-comms (4 source + 2 test files):**
```
cl-hive-comms/cl-hive-comms.py               # Plugin entry point, 15 RPC methods
cl-hive-comms/modules/comms_service.py        # Core service + CommsStore + PolicyEngine (~1500 lines)
cl-hive-comms/modules/transport_router.py     # Transport routing, envelope validation, auth (~440 lines)
cl-hive-comms/modules/transport_security.py   # RuneVerifier + NostrDmCodec (~170 lines)
cl-hive-comms/tests/test_comms_service.py     # 15 service tests
cl-hive-comms/tests/test_transport_router.py  # 16 transport tests
```

**cl-hive-archon (2 source + 1 test file):**
```
cl-hive-archon/cl-hive-archon.py              # Plugin entry point, 10 RPC methods
cl-hive-archon/modules/archon_service.py      # ArchonStore + ArchonService + GatewayClient (~1150 lines)
cl-hive-archon/tests/test_archon_service.py   # 23 tests
```

**Docker integration (3 files):**
```
cl-hive/docker/Dockerfile                     # Image build with optional plugin cloning
cl-hive/docker/docker-entrypoint.sh           # Startup orchestration with enable flags
cl-hive/docker/docker-compose.build.yml       # Compose config with version pins
```

**Context documents (read for architecture understanding, not for code audit):**
```
hive-docs/planning/13-PHASE6-READINESS-GATED-PLAN.md
hive-docs/planning/08-HIVE-CLIENT.md
hive-docs/deployment/PHASE6-DOCKER-PLUGIN-INTEGRATION-PLAN.md
```

---

### Audit Categories

For each category, produce findings with:
- **ID**: Sequential identifier (e.g., S1, C1, T1)
- **Severity**: CRITICAL / HIGH / MEDIUM / LOW / INFO
- **File**: Exact path and line number(s)
- **Description**: What the issue is and why it matters
- **Exploit scenario** (for HIGH+): How an attacker could exploit this
- **Suggested fix**: Concrete code change

#### Category 1: Security (S-prefix)

Evaluate against these threat models:

**1A. Transport Ingress Attacks**
- Can an unauthenticated attacker execute schemas via REST or Nostr DM?
- Can a valid advisor replay messages? Is nonce enforcement atomic?
- Can an advisor with "monitor" permissions escalate to "admin" operations?
- Can signature verification be bypassed (timing attacks, format confusion)?
- Can the envelope size limit (65535 bytes) be bypassed?
- Is the timestamp skew window (300s) appropriate?

**1B. Credential Security**
- Are auth tokens generated with sufficient entropy?
- Are auth tokens ever leaked (logs, RPC responses, receipts, error messages)?
- Are auth tokens properly invalidated on revocation?
- Is the Nostr private key stored securely? Can it leak via any code path?
- Are HMAC comparisons constant-time?

**1C. Database Security**
- Are all SQL queries parameterized? Any string interpolation in SQL?
- Are file permissions (DB, directory, WAL/SHM files) restrictive?
- Can SQLite injection occur via any input path?
- Is transaction isolation appropriate for concurrent access?
- Are there TOCTOU (time-of-check-time-of-use) vulnerabilities?

**1D. Network Security**
- Can the gateway URL be set to internal/private IP ranges (SSRF)?
- Is TLS certificate verification enforced on outbound connections?
- Are HTTP (non-TLS) connections restricted appropriately?
- Can DNS rebinding attacks bypass SSRF protections?

**1E. Input Validation**
- Are all RPC method parameters validated for type, length, and format?
- Can oversized inputs cause memory exhaustion?
- Are permission strings validated against an allowlist?
- Can poll options, titles, or metadata contain injection payloads?

#### Category 2: Correctness (C-prefix)

**2A. State Machine Correctness**
- Does the advisor lifecycle (authorize/revoke/re-authorize) maintain consistent state?
- Does the trial lifecycle (start/stop/expire) handle edge cases?
- Does poll state (active/completed/pruned) transition correctly?
- Are DID provisioning and re-provisioning idempotent?

**2B. Policy Engine**
- Do danger scores accurately reflect operation risk?
- Does the policy engine correctly block/allow/require-confirmation?
- Can policy overrides disable safety checks?
- Are the three presets (conservative/moderate/aggressive) sensible?

**2C. Concurrency**
- Are thread-local SQLite connections properly managed?
- Is the rate limiter thread-safe under concurrent access?
- Can the replay guard's BEGIN IMMEDIATE create deadlocks?
- Is the transport registry lock held for minimal duration?

**2D. Error Handling**
- Do all error paths return consistent error format?
- Can exceptions in one handler corrupt shared state?
- Are database connections properly cleaned up on errors?
- Does the prune/cleanup path handle partial failures?

#### Category 3: Test Coverage (T-prefix)

**3A. Coverage Gaps**
- Identify any public method with zero test coverage
- Identify any error path (catch/except) that is never tested
- Identify any security-critical path without a negative test (rejection test)
- Are edge cases tested: empty strings, None values, boundary values, Unicode?

**3B. Test Quality**
- Do tests actually assert behavior, or are they stubs (pass-only bodies)?
- Do tests verify both success and failure paths?
- Are mocks appropriate (not mocking the thing being tested)?
- Could any test pass trivially regardless of implementation?

#### Category 4: Docker Integration (D-prefix)

**4A. Container Security**
- Does the container run as root? Is this necessary?
- Are build args validated before use in RUN commands?
- Are plugin versions pinned to specific tags/commits?
- Could a compromised GitHub repo inject code during build?

**4B. Startup Orchestration**
- Is the plugin dependency chain enforced (archon requires comms)?
- Can enabling both flags with comms missing cause silent failure?
- Does the health check verify plugins are actually functional?
- Is the startup order deterministic?

**4C. Configuration**
- Are default values safe (both plugins disabled)?
- Can environment variable injection affect startup behavior?
- Are version pins consistent across Dockerfile and compose?

---

### Architecture Context

These are facts about the system. Do NOT treat these as findings — use them to understand the codebase.

**Plugin Runtime Model:**
- Each plugin runs as a CLN plugin process (long-lived daemon)
- Communication between plugins uses CLN RPC (JSON-RPC over Unix socket)
- Each plugin has its own isolated SQLite database with WAL mode
- No cross-plugin database reads exist; all cross-plugin data flows through RPC

**Authentication Layers:**
1. **CLN RPC socket** — filesystem permissions (only node operator can access)
2. **REST transport** — optional rune verification (HMAC or CLN checkrune)
3. **Nostr DM transport** — NIP-44 envelope decoding + sender pubkey validation
4. **Envelope signature** — HMAC-SHA256 over canonical JSON (schema_type + schema_payload + nonce + timestamp)
5. **Advisor permissions** — per-advisor permission list checked against schema requirements
6. **Policy engine** — danger score gating with preset thresholds

**Danger Score Scale (0-10):**
| Score | Meaning | Example |
|-------|---------|---------|
| 1 | Read-only monitoring | hive:monitor/v1, hive:discover/v1 |
| 2-4 | Low-risk mutations | hive:alias/v1 add, hive:trial/v1 stop |
| 5-6 | Financial operations | hive:payments/v1, hive:authorize/v1 |
| 7-8 | High-risk admin ops | hive:policy/v1 reset, admin access grant |
| 9 | Unknown schemas | Catch-all for unrecognized schema types |
| 10 | Never allow via transport | hive:identity/v1 import/rotate |

**Database Tables:**

*cl-hive-comms (10 tables):*
`nostr_state`, `comms_advisors`, `comms_advisor_auth`, `comms_aliases`, `comms_trials`, `comms_payments`, `management_receipts`, `comms_policy` (stored in nostr_state), replay nonces (in nostr_state), transport registry (in nostr_state)

*cl-hive-archon (4 tables):*
`archon_identity` (singleton), `archon_bindings`, `archon_polls`, `archon_votes`

**Key Capacity Limits:**
- Max advisors: 100
- Max aliases per advisor: 10
- Max active trials: 50
- Max polls: 5,000
- Max votes: 50,000
- Max message size: 65,535 bytes
- Rate limit: 60 requests/sender/minute (normal), 5/sender/minute (high-danger)
- Max tracked senders: 10,000

---

### Output Format

Structure your audit report as follows:

```markdown
# Phase 6 Security Audit Report — [DATE]

## Executive Summary
[2-3 paragraph overview of overall security posture, key risks, and recommendations]

## Finding Totals
| Severity | Count |
|----------|-------|
| CRITICAL | N |
| HIGH     | N |
| MEDIUM   | N |
| LOW      | N |
| INFO     | N |

## Findings by Category

### Security Findings
[S1, S2, ...]

### Correctness Findings
[C1, C2, ...]

### Test Coverage Findings
[T1, T2, ...]

### Docker Integration Findings
[D1, D2, ...]

## Positive Observations
[List security-positive patterns found in the codebase]

## Test Coverage Matrix
[Table showing each public method and whether it has test coverage]

## Recommendations Summary
[Prioritized list of actions, grouped by urgency]
```

### Important Notes

- Do NOT assume any prior remediations. Audit the code as-is.
- Reference specific line numbers for every finding.
- If you find a pattern that looks like it was recently fixed (e.g., a comment saying "moved after policy evaluation"), still verify the fix is correct.
- Pay special attention to the boundary between "local RPC" (trusted) and "transport ingress" (untrusted) — these are fundamentally different threat models.
- The Nostr private key stored in `nostr_state` as `config:privkey` is the most sensitive asset in the system. Trace every code path that could expose it.
- Hash-chained receipts (`prev_hash`/`receipt_hash` in `management_receipts`) are an integrity mechanism — verify the chain cannot be broken or forged.

---

*Prompt version: 2026-02-20 v1*
*Covers: cl-hive-comms, cl-hive-archon, cl-hive Docker scaffolding*
