# ANVIL Donor Mechanism Synthesis — WeKnora + Orca

**Date:** 2026-09-23  
**Status:** RESEARCH-ONLY / NO PRODUCTION AUTHORITY  
**Canonical purpose:** one deduplicated donor-mining doctrine for the ANVIL stack.

## Source binding

### WeKnora
- Fork: `naytewilson/WeKnora`
- Reviewed/main anchor: `1fa915d712288902da7841dc8ff355a56e37a013`
- Upstream: `Tencent/WeKnora`
- Upstream main rechecked 2026-09-23: still `1fa915d712288902da7841dc8ff355a56e37a013`
- License: MIT for the project, with third-party components under their own notices.

### Orca
- Fork: `naytewilson/orca`
- Reviewed fork anchor: `1116c54230244a08dffd642f66d5f5fc98994759`
- Upstream: `stablyai/orca`
- Upstream main rechecked 2026-09-23: `519bde81df438478ff1c6ac010f1339f32a306ee`
- The one-commit drift from the reviewed fork is notebook/editor work and does not alter this source-bound mining review.
- License: MIT.
- Donor-specific source ledger: `naytewilson/anvil#136`.

## Live receiving refs rechecked

- ANVIL main: `e4b0aafe9689646ee78e4b3faf1af145407c0d5e`
- Local Agent Gateway main: `cc9062c8aa482a2eff1cb082867cb8f25e095ff4`
- SIEVE main: `8eb55677d9ac371fc9d4b2b08823ae7cc905e186`
- Paseo Hub main: `115f879f4fc4466e12eca91d41e36707f3f393f0`
- Paseo main: `333844e76b7dc012c3d58109f6a083b96234d7dd`
- Hermes Conduit main: `b76454c38be9e7b7c4f20a8499c4a269c97a9599`

These refs are source-truth anchors for this synthesis, not permanent claims about future state.

## Controller ruling

**Mine donor laws, failure semantics, discriminators, and useful mechanisms. Do not import donor chassis.**

Whole-product adoption is rejected for both donors. Their product boundaries overlap existing ANVIL, SIEVE, Paseo, Hub, Conduit, ContextPlanning, memory, MCP, runtime, and authority surfaces.

The transfer unit is:

`donor failure/contract -> RED discriminator -> ANVIL-native mechanism -> owner-repo verification -> durable receipt`

### Deduplication / synthesis rule

When donors overlap:

1. If one mechanism is strictly stronger inside our authority model, use the stronger one.
2. If they solve different layers of the same problem, compose them into one ANVIL-native contract.
3. If ANVIL already has the stronger implementation, keep ANVIL and import only the donor regression/failure fixture.
4. Never create a second database, control plane, status authority, transport, memory store, queue, machine identity, or presentation authority merely because a donor has one.
5. Optimization claims do not transfer. Reproduce the bottleneck first.
6. Donor implementations are examples, not ceilings. The native hybrid may deliberately exceed both.

# Native hybrid architecture

## H1. Layered Identity Envelope — PRIORITY 0 FOUNDATION

**Synthesis:** Orca durable identity + ANVIL generation fencing + WeKnora compact request-local aliases.

### Donor inputs
Orca:
- durable client/session identity must survive reconnect;
- transport/reconnect incarnation must change when delivery ownership changes;
- repaint/snapshot recovery and execution-stream recovery are separate problems.

WeKnora:
- stable logical `resource://` identity independent of physical storage;
- request-local compact aliases;
- streaming-safe alias reconstruction;
- tool-call decode after stream assembly;
- raw-identifier leak detection;
- fail closed on unknown/foreign refs.

Relevant WeKnora commits:
- `6d14cc3536325c603dfddf61ea8044ce7c1ce9c1`
- `182f67bd86c08840c91670f38290903c42d8348f`

Relevant Orca source:
- `docs/reference/ssh-reconnect-source-recovery.md`
- blob `5dffc50e7dd32cb5c151aac8ed59da924fc92a36`

### ANVIL-native contract

Represent identity as distinct layers, never one overloaded ID:

```
DurableIdentity
  + AuthorityGeneration
  + TransportIncarnation
  + Source/EvidenceGeneration
  + RequestLocalAlias
```

Rules:
- durable identity answers **what is this?**
- authority generation answers **which admitted/owned version may act?**
- transport incarnation answers **which live connection currently owns delivery?**
- source/evidence generation answers **which exact evidence snapshot is being referenced?**
- request-local alias answers only **how cheaply may the model refer to it inside this request?**
- aliases never carry authority;
- reconnect never silently reuses transport ownership;
- a foreign generation/incarnation/reference fails closed.

Likely owners:
- Paseo/Hub for transport incarnation;
- Local Agent Gateway ContextPlanning/EvidenceKit for source and presentation binding;
- SIEVE for model-visible authority;
- Conduit only mirrors verified state.

This is stronger than either donor alone.

## H2. Evolution-Safe Capability Surface — PRIORITY 0 FOUNDATION

**Synthesis:** Orca mixed-version wire law + WeKnora capability-scoped exposure + existing ANVIL fail-closed authority.

### Donor inputs
Orca:
- additive fields remain truly optional;
- absent, null, and present are different epistemic states;
- new semantics/opcodes require capability negotiation;
- host-published semantic changes count as wire changes even if JSON shape does not;
- unknown enum arms degrade without granting affordances;
- test current client -> release host and release client -> current host against a real release baseline.

Source:
- `docs/reference/remote-wire-compatibility.md`
- `tests/e2e/cross-version-wire/*`
- blob `d72f93770c5d036a0429f515aae56b0102eb3c03`

WeKnora:
- tools/modes registered only when capability exists;
- explicit fallback reports what actually ran;
- MCP `tools/list` filtering is rechecked again at `tools/call`;
- fixed configured agent beats caller-selected arbitrary execution authority;
- writes default off.

Relevant commits:
- `92e7c0ab37bb553df950b91680f162127c8d1282`
- `76a19f14c4aec6450421cc3f1256771c4214ff9f`

### ANVIL-native contract

Every remotely published surface should separate:

```
wire_schema_version
semantic_capabilities
authority_scope
availability_state
actual_execution_mode
```

Rules:
- schema compatibility does not imply semantic compatibility;
- discovery narrows exposure, execution re-authorizes;
- unknown capability/enum never widens power;
- silent fallback is forbidden when it changes semantics;
- mixed-version tests cover both directions using a real prior release artifact;
- provider/tool/UI clients render only capabilities established by the host.

Primary owners:
- Paseo protocol / Hub;
- Conduit adapters;
- ANVIL MCP surfaces;
- Local Agent Gateway provider/tool capability contracts.

## H3. Host-Owned Epistemic State Lattice — PRIORITY 0 FOUNDATION

**Synthesis:** Orca execution-host status authority + SIEVE evidence doctrine + WeKnora fail-closed source-reference hygiene.

### Donor inputs
Orca:
- execution host owns canonical agent status;
- clients mirror, never merge/write back;
- precedence resolved once at write time with provenance;
- restored state is unconfirmed, never silently live;
- runtime-scoped terminal identity is not persisted across incarnations;
- explicit live / unverifiable / exited distinctions.

Source:
- `docs/reference/agent-status-store.md`
- blob `97fde3f7b912b260bc758b2b825d64aff059b6f2`

WeKnora:
- canonical source-ref parsing;
- unknown refs fail closed;
- model-supplied labels/titles are rebuilt from authoritative data;
- identifier leaks are observable rather than silently tolerated.

### ANVIL-native contract

Use one execution-host-owned state record plus provenance and an explicit epistemic class:

```
LIVE_VERIFIED
RESTORED_UNCONFIRMED
UNVERIFIABLE
EXITED_OR_REVOKED
```

Rules:
- unavailable/unverifiable is not equivalent to empty, stopped, or current;
- clients may choose presentation policy, not execution truth;
- no client-side state merge may manufacture liveness;
- recovered persisted state must reacquire a live generation before becoming LIVE_VERIFIED;
- source references attached to status must bind canonical IDs, not model/user aliases.

Likely owners:
- Paseo daemon = execution truth;
- Hub = projection/orchestration;
- ANVIL = durable correlation/evidence;
- Conduit = presentation only.

## H4. Recovery Envelope — PRIORITY 1

**Synthesis:** Orca restart/adoption + transport incarnation + WeKnora persistent compaction checkpoints + fail-closed durable generation.

### Donor inputs
Orca:
- versioned readiness;
- bound endpoint distinct from advertised endpoint;
- typed unavailable reasons;
- durable execution scope may outlive service restart;
- verify restart destructiveness before restarting;
- fresh census/adoption when survival is unverifiable.

Source:
- `docs/reference/headless-linux-server.md`
- blob `06ec925ab1cfc269d3580c3ba08840ae2ac6523c`

WeKnora:
- compaction checkpoint survives the turn;
- loader, compaction trigger, and compactor share one token-pricing model;
- do not trim away the overflow that must trigger compaction;
- degraded compaction still persists bounded forward-progress state;
- never persist a truncated durable generated rewrite; bounded continuation may repair, otherwise reject.

Relevant commits:
- `62f7da5eea3c52fd1887bdb0ef114562ce924a98`
- `03d9d66a1ff90022a5855f0e00df216a915bf9b6`
- `653307b5c72d90aa6e43ac2f43bdcf859d14d0ad`

### ANVIL-native contract

Runtime recovery and context recovery are distinct but use the same fencing philosophy.

A recoverable checkpoint/capsule should bind, where applicable:
- durable identity;
- source/evidence generation;
- transport incarnation or runtime generation;
- exact input digests;
- recovery handles;
- epistemic state;
- checkpoint completeness/degraded marker;
- creation time and expiry/revocation state.

Derived summaries/checkpoints remain derived candidates, never pristine evidence.

Restart, reconnect, resume, adoption, compaction, and durable writeback must all either:
- prove their fence still matches, or
- reacquire/rebuild safely.

## H5. Deterministic Model-Context Economy — PRIORITY 1

**Best donor:** WeKnora, extended with ANVIL source/evidence binding.

WeKnora donors:
- `22489ba6633c85830512ea1cd4cf4115967613a2` — stable prompt prefixes, cache markers, hit telemetry.
- `29bec4204a3fe743efdf7c4cc8effd11ba43e45b` — deterministic tool ordering.
- `03d9d66a1ff90022a5855f0e00df216a915bf9b6` — calibrated token estimation.

Current ANVIL-side observation:
- ContextChronicle already fixes block ordering by mutation-frequency class.
- Code census at Local Agent Gateway `cc9062c...` found no established `prompt_cache` / `cache_control` implementation.

### ANVIL-native improvement

Do more than copy cache markers.

Define a canonical **ContextFrameFingerprint** over stable provider-visible material:
- system/policy block digest;
- canonical tool-schema digest;
- semantic capability set;
- provider/model family;
- serialization version;
- stable context-block prefix identity.

Then:
- preserve byte-stable prefixes where semantics are unchanged;
- attach provider-native cache controls only where supported;
- record cache eligibility/hit/miss telemetry;
- invalidate on semantic or authority changes, not incidental map/order churn;
- use calibrated token accounting for load, trigger, and compaction decisions.

This should be proved by byte-stability tests and provider telemetry, not inferred from lower token cost.

## H6. Generation-Fenced Progressive Retrieval — PRIORITY 1 / CONTEXT FABRIC

**Synthesis:** WeKnora progressive disclosure + Orca generation/incarnation-fenced search.

### Donor inputs
WeKnora:
- `search -> read -> cite`;
- `list` for namespace discovery;
- smaller tool schemas;
- explicit actual-mode fallback;
- capability-aware registration.

Commit:
- `92e7c0ab37bb553df950b91680f162127c8d1282`

Orca:
- cursor bound to query + host generation + persisted index incarnation;
- stale cursor is explicit and restarts at page 1;
- no silent unavailable-remote -> local fallback;
- bounded freshness wait, then explicit freshness-truncated result;
- unavailable service is not an empty current index.

Source:
- `docs/reference/agent-session-search-contract.md`
- blob `daaef0fb618c7a44f4d3d05ab8b12e8bf7f77ea2`

### ANVIL-native contract

Model-facing retrieval should expose a small progressive surface while every result carries enough state to prevent temporal/source confusion:

```
requested_mode
actual_mode
source_snapshot_or_generation
index_incarnation
freshness_state
next_cursor
canonical_source_refs
```

Rules:
- search discovers; read expands; cite binds;
- stale cursors and stale generations are explicit values;
- fallback is descriptive, never hidden;
- remote unavailable never silently means local;
- source IDs remain canonical outside the model-local alias layer;
- SIEVE governs what transformed/retrieved material becomes model-visible.

Do not reopen closed SIEVE Production Graduation V1 to do this. Treat it as Context Fabric successor work.

## H7. Capability and Distribution Security Envelope — PRIORITY 2

**Synthesis:** WeKnora narrow endpoint grants + Orca package/skill transaction security + existing ANVIL Skill Registry.

### Donor inputs
WeKnora:
- one-time/rotatable hashed tokens;
- explicit tool and resource scope;
- list-time filtering plus call-time recheck;
- fixed agent;
- writes off by default;
- revocable/expiring resource grants.

Orca:
- deterministic package identity;
- validation-before-mutation;
- path containment;
- interrupted transaction recovery;
- protect locally modified content;
- replay/idempotency;
- short-lived grants.

Source:
- `docs/reference/agent-skill-sharing-threat-model.md`
- blob `efccfd7119b36367b124ca892456a486f83e8dce`

### ANVIL-native contract

Separate reachability, identity, credential, and permission mechanically.

A grant should bind only the minimum needed:
- subject/principal;
- resource or package identity/digest;
- permitted operations/tools;
- generation/incarnation where relevant;
- expiry/revocation;
- provenance.

Package/skill mutation should stage -> validate -> verify identity/provenance -> commit atomically -> emit receipt, with rollback/replay semantics.

Transport possession alone never grants broad authority.

## H8. Memory Relevance Regression — PRIORITY 2

**Keep ANVIL implementation; import WeKnora's failure fixture.**

WeKnora donor:
- `17f893865d4d8337f7e0bf942c6931a8ce7a08c1`

Bug:
- selecting an importance-ranked candidate pool before semantic search made highly relevant low-importance memories unreachable.

Current ANVIL:
- Phase 25/26 semantic recall queries pgvector over durable embedded memories and hybrid RRF fuses lexical + semantic results.
- The exact WeKnora bug is not established in current ANVIL.

Action:
- add a defect-seeded regression proving a semantically exact low-importance memory remains reachable;
- consider an `outside_pool`-style diagnostic to catch future accidental prefilter blindness;
- do not replace ANVIL memory with WeKnora memory.

## H9. Worktree Fingerprint Gate — MEASURE FIRST

**Best donor:** Orca only.

Source:
- `docs/reference/worktree-scan-fingerprint.md`
- blob `fa0e26dac203825a2698e37d75b56670a158d2ca`

Useful mechanism:
- cheap subprocess-free fingerprint of Git admin/worktree state;
- event invalidation + bounded reconciliation;
- fingerprint captured before authoritative Git scan so concurrent mutation remains detectable;
- uncertainty fails open to real Git.

Orca reports a large reduction in its own periodic `git worktree list` use, but that does not prove ANVIL/Paseo has the same bottleneck.

Action:
- instrument current scan rate/latency/wakeup cost first;
- port only if the discriminator is positive.

# Lower-priority donor-specific mining

Useful later, but not allowed to create duplicate subsystems:

WeKnora:
- wiki revision/diff/rollback;
- tombstone + reference-counted artifact reclamation;
- runtime queue operator UX and orphan detection;
- knowledge-base profile aggregation;
- cross-session memory curation UI.

Orca:
- additional daemon/operator UX only where it sharpens existing Paseo/Conduit owners;
- mobile-specific comparisons only after the active Fabric/I5/D12 lineage is stable.

# Explicit rejects

Do not import:
- WeKnora control plane, full RAG/vector-store matrix, sandbox runtime, queue scheduler, auth/RBAC stack, frontend shell, or MCP transport;
- Orca Electron desktop architecture;
- Orca React Native mobile shell;
- Orca Cloud/relay authority plane;
- duplicate orchestration/status databases;
- direct SSH as a second identity for hosts already enrolled through Paseo;
- any donor feature that has no existing ANVIL owner surface.

# Implementation order

## Wave 0 — discriminators before ports

Run bounded owner-repo RED/measurement probes for:
1. bidirectional Paseo/Hub/Conduit version skew;
2. reconnect transport-incarnation fencing;
3. model-visible raw identifier/source leaks;
4. provider prompt-prefix byte stability and cache-marker support;
5. low-importance/high-relevance ANVIL memory recall;
6. worktree-scan subprocess/wakeup cost.

No production mutation is required to establish these gaps.

## Wave 1 — identity + compatibility foundation

If Wave 0 is positive:
- implement the Layered Identity Envelope;
- implement Evolution-Safe Capability Surface tests/contracts;
- preserve existing owner databases and authority boundaries.

## Wave 2 — context economy + recovery

If gaps are proven:
- stable logical source/resource handles with model-local aliases;
- leak detector;
- ContextFrameFingerprint/cache semantics;
- durable/fenced context checkpoint behavior.

## Wave 3 — Context Fabric retrieval

Build generation-fenced progressive retrieval only as a successor Context Fabric lane, without reopening closed SIEVE graduation.

## Wave 4 — security/optimization hardening

- capability/package transaction envelope;
- worktree fingerprint only if measured worthwhile;
- donor UX/revision ideas only after owner-specific gap analysis.

# Transfer acceptance gate

A donor-derived change enters an ANVIL-stack owner repo only when:

1. exact donor SHA/file provenance is bound;
2. exact receiving-repo SHA is reacquired;
3. a current gap or measurable opportunity is established;
4. RED discriminator or measurement exists;
5. implementation uses the existing owner and authority plane;
6. unknown/failure states fail closed;
7. focused tests include the donor failure mode;
8. relevant build/test/lint/runtime proof passes on the owning platform;
9. final diff/staged state is reviewed;
10. exact ref + durable receipt is published;
11. license/notice requirements are preserved for any literal code extraction.

# Canonical continuity

- Unified synthesis: this document.
- Orca donor-specific evidence queue: ANVIL issue #136.
- WeKnora donor branch: `north/weknora-mining-map-20260923`.
- Drive donor receipts remain provenance evidence; they are not separate architectural authorities.

The synthesis, not either donor implementation, is the design target.
