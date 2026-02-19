# Lightning Hive Documentation

Canonical documentation for the [cl-hive](https://github.com/lightning-goats/cl-hive) Lightning coordination suite.

## Navigation

### Architecture & Design

| Document | Description |
|----------|-------------|
| [Architecture](ARCHITECTURE.md) | Complete protocol specification |
| [Hive System Overview](planning/15-HIVE-SYSTEM-OVERVIEW.md) | High-level explanation of the suite, plugin boundaries, core flows |
| [Genesis](GENESIS.md) | Origin story and design philosophy |
| [The Hive Article](THE_HIVE_ARTICLE.md) | Long-form writeup |

### Design Documents

| Document | Description |
|----------|-------------|
| [Cooperative Fee Coordination](design/cooperative-fee-coordination.md) | Pheromone-based fee coordination design |
| [No Node Left Behind](design/no-node-left-behind.md) | Fleet health monitoring and NNLB protocol |
| [VPN Transport](design/VPN_HIVE_TRANSPORT.md) | WireGuard VPN transport layer |
| [Liquidity Integration](design/LIQUIDITY_INTEGRATION.md) | cl-revenue-ops integration design |
| [CL Revenue Ops Integration](design/CL_REVENUE_OPS_INTEGRATION.md) | Bridge and circuit breaker design |
| [AI Advisor Database](design/AI_ADVISOR_DATABASE.md) | Advisor mode database design |
| [Fee Distribution Process](fee-distribution-process.md) | Settlement and fee distribution flow |

### Planning Documents

See [Planning Index](planning/00-INDEX.md) for the full dependency-ordered document index.

Key documents:

| # | Document | Status |
|---|----------|--------|
| 01 | [Reputation Schema](planning/01-REPUTATION-SCHEMA.md) | Phase 1 Implemented |
| 02 | [Fleet Management](planning/02-FLEET-MANAGEMENT.md) | Draft |
| 03 | [Cashu Task Escrow](planning/03-CASHU-TASK-ESCROW.md) | Draft |
| 04 | [Hive Marketplace](planning/04-HIVE-MARKETPLACE.md) | Draft |
| 05 | [Nostr Marketplace](planning/05-NOSTR-MARKETPLACE.md) | Draft |
| 06 | [Hive Settlements](planning/06-HIVE-SETTLEMENTS.md) | Draft |
| 07 | [Hive Liquidity](planning/07-HIVE-LIQUIDITY.md) | Draft |
| 08 | [Hive Client](planning/08-HIVE-CLIENT.md) | Draft |
| 09 | [Archon Integration](planning/09-ARCHON-INTEGRATION.md) | Draft |
| 10 | [Node Provisioning](planning/10-NODE-PROVISIONING.md) | Draft |
| 11 | [Implementation Plan (Phase 1-3)](planning/11-IMPLEMENTATION-PLAN.md) | Phase 2 Complete |
| 12 | [Implementation Plan (Phase 4-6)](planning/12-IMPLEMENTATION-PLAN-PHASE4-6.md) | Draft |
| 13 | [Phase 6 Readiness-Gated Plan](planning/13-PHASE6-READINESS-GATED-PLAN.md) | Planning-only |
| 15 | [Hive System Overview](planning/15-HIVE-SYSTEM-OVERVIEW.md) | Living overview |

### Plugin Architecture (Phase 6)

| Document | Description |
|----------|-------------|
| [cl-hive](plugins/cl-hive.md) | Core coordination plugin |
| [cl-hive-comms](plugins/cl-hive-comms.md) | Nostr + REST transport plugin |
| [cl-hive-archon](plugins/cl-hive-archon.md) | DID + VC identity plugin |

### Protocol Specs

| Document | Description |
|----------|-------------|
| [Communication Hardening](specs/HIVE_COMMUNICATION_PROTOCOL_HARDENING_PLAN.md) | Protocol hardening plan |
| [Inter-Hive Relations](specs/INTER_HIVE_RELATIONS.md) | Multi-hive federation spec |
| [Payment-Based Protocol](specs/PAYMENT_BASED_HIVE_PROTOCOL.md) | Payment-gated protocol design |
| [Phase 9 Proposal](specs/PHASE9_PROPOSAL.md) | Phase 9 overview |
| [Phase 9.1 Protocol](specs/PHASE9_1_PROTOCOL_SPEC.md) | Protocol specification |
| [Phase 9.2 Logic](specs/PHASE9_2_LOGIC_SPEC.md) | Logic specification |
| [Phase 9.3 Economics](specs/PHASE9_3_ECONOMICS_SPEC.md) | Economics specification |

### Security

| Document | Description |
|----------|-------------|
| [Threat Model](security/THREAT_MODEL.md) | Security threat analysis |
| [Security Review](SECURITY_REVIEW.md) | Security review findings |
| [Attack Surface](attack-surface.md) | Attack surface analysis |
| [Red Team Plan](red-team-plan.md) | Red team testing plan |
| [MCP Server Hardening](MCP_HIVE_SERVER_REVIEW_AND_HARDENING_PLAN.md) | MCP server security review |

### Deployment

| Document | Description |
|----------|-------------|
| [Docker Plugin Integration](deployment/PHASE6-DOCKER-PLUGIN-INTEGRATION-PLAN.md) | Docker deployment plan for Phase 6 |
| [Manual Install (Non-Docker)](deployment/PHASE6-MANUAL-INSTALL-NON-DOCKER.md) | Manual installation guide |

### Operations

| Document | Description |
|----------|-------------|
| [Joining the Hive](JOINING_THE_HIVE.md) | How to join an existing hive fleet |
| [AI Advisor Setup](AI_ADVISOR_SETUP.md) | AI advisor configuration |
| [Advisor Intelligence](ADVISOR_INTELLIGENCE_INTEGRATION.md) | Advisor intelligence integration |
| [MCP Server](MCP_SERVER.md) | MCP server setup and tool reference |

### Testing

| Document | Description |
|----------|-------------|
| [Testing Guide](testing/README.md) | Testing overview |
| [Testing Plan](testing/TESTING_PLAN.md) | Comprehensive test plan |
| [Simulation Report](testing/SIMULATION_REPORT.md) | Simulation test results |
| [Polar Setup](testing/polar.md) | Polar network testing setup |

### Research

| Document | Description |
|----------|-------------|
| [Swarm Intelligence Research](research/SWARM_INTELLIGENCE_RESEARCH_2025.md) | Academic research survey |

## Code Repository

- [cl-hive](https://github.com/lightning-goats/cl-hive) — Coordination layer plugin
- [cl-revenue-ops](https://github.com/lightning-goats/cl_revenue_ops) — Execution layer plugin
