# WeKnora -> ANVIL Stack Mining Map

**Date:** 2026-09-23  
**Status:** RESEARCH-ONLY / NO PRODUCTION AUTHORITY  
**Donor fork:** `naytewilson/WeKnora`  
**Donor base:** `1fa915d712288902da7841dc8ff355a56e37a013`  
**Upstream:** `Tencent/WeKnora`  
**Upstream main at census:** `1fa915d712288902da7841dc8ff355a56e37a013`

## Controller ruling

WeKnora is worth mining aggressively as a donor/oracle, but not adopting as a subsystem.

Its product boundary overlaps ANVIL, SIEVE, Paseo Hub, Local Agent Gateway, ContextPlanning, memory, and MCP authority. Whole-product integration would create competing sources of truth and duplicate control planes.

The preferred transfer unit is **behavioral contract + tests + narrowly ported mechanism**, not whole modules.

## Live receiving refs at census

- ANVIL main: `e4b0aafe9689646ee78e4b3faf1af145407c0d5e`
- Local Agent Gateway main: `cc9062c8aa482a2eff1cb082867cb8f25e095ff4`
- SIEVE main: `8eb55677d9ac371fc9d4b2b08823ae7cc905e186`
- Paseo Hub main: `115f879f4fc4466e12eca91d41e36707f3f393f0`
- Paseo main: `333844e76b7dc012c3d58109f6a083b96234d7dd`

## Priority A: port now / prototype against current stack

### A1. Stable resource identity + model-local aliases + leak detection

Donors:
- `6d14cc3536325c603dfddf61ea8044ce7c1ce9c1` — stable `resource://` identity, request-local aliases, streaming decode, revocable grants.
- `182f67bd86c08840c91670f38290903c42d8348f` — canonical source-ref parsing, fail-closed unknown refs, controller-owned document selection, identifier-leak detector.

Receiving seam:
- Local Agent Gateway `EvidenceKit` / `ContextPlanning` / managed-context presentation.
- SIEVE remains the authority deciding what may become model-visible.

Port:
- stable logical handle independent of physical location;
- short request-local aliases for model context;
- streaming-safe alias reconstruction;
- tool-call argument decode after stream assembly;
- raw-identifier leak detector as a production invariant;
- fail closed on unknown or foreign references.

Do not port:
- WeKnora tenant/storage authority;
- public URL behavior;
- any path that bypasses SIEVE presentation authority.

### A2. Durable compaction checkpoints + calibrated token budgeting

Donors:
- `62f7da5eea3c52fd1887bdb0ef114562ce924a98` — compaction checkpoints survive turns.
- `03d9d66a1ff90022a5855f0e00df216a915bf9b6` — history sized by model window, overflow deliberately reaches compaction, estimator self-calibration, degraded checkpoint persistence.

Receiving seam:
- Local Agent Gateway `ContextChronicle` / future context planner and ReCompress path.

Important upstream lesson:
- trimming to the compaction threshold before compaction can silently create a sliding window that never checkpoints.
- loader, trigger, and compactor must use one pricing model.
- degraded compaction must still make forward progress instead of repeating the same oversized failure forever.

ANVIL adaptation:
- any summary/checkpoint is a derived candidate, never pristine evidence;
- preserve exact source bindings/recovery references through SIEVE.

### A3. Provider prompt-cache determinism

Donors:
- `22489ba6633c85830512ea1cd4cf4115967613a2` — byte-stable prompt prefixes + provider cache markers + hit telemetry.
- `29bec4204a3fe743efdf7c4cc8effd11ba43e45b` — deterministic tool ordering; randomized map order had destroyed prefix-cache hits.

Receiving seam:
- Local Agent Gateway ProviderKit/provider request construction.

Current census:
- ContextChronicle already has stable mutation-frequency block ordering.
- Code search at `cc9062c...` found no `prompt_cache` or `cache_control` implementation.

Port:
- canonical tool-schema ordering;
- byte-stability regression fixture;
- provider-specific prompt-cache keys/markers where supported;
- cache-hit/miss telemetry bound to provider/model/request shape.

### A4. Smaller progressive-disclosure retrieval surface

Donor:
- `92e7c0ab37bb553df950b91680f162127c8d1282`.

Useful contract:
- `search_knowledge` -> `read_document` -> cite;
- `list_documents` for namespace discovery;
- capabilities/modes registered only when supported;
- explicit mode fallback instead of pretending a semantic path ran;
- retired tool aliases normalize at registration rather than duplicating permanent surfaces.

Receiving seam:
- Context Fabric model-facing retrieval tools and API Oracle/ANVIL evidence navigation where applicable.

Goal:
fewer overlapping tools, smaller schemas, stronger progressive disclosure, mechanically honest fallbacks.

## Priority B: use as adversarial oracle / regression source

### B1. MCP endpoint scoping

Donor:
- `76a19f14c4aec6450421cc3f1256771c4214ff9f`.

Harvest:
- one-time/rotatable hashed endpoint token;
- explicit tool allowlist;
- explicit resource/KB scope;
- `tools/list` filters AND `tools/call` rechecks;
- fixed configured agent rather than caller-selected arbitrary agent;
- write capabilities off by default;
- per-endpoint rate limit and last-used telemetry.

Do NOT port WeKnora's transport/protocol implementation.
ANVIL's current MCP doctrine is protocol 2026-07-28, stateless Streamable HTTP, no legacy initialize/session semantics. Use WeKnora only as an authorization test oracle.

### B2. Memory relevance regression

Donor:
- `17f893865d4d8337f7e0bf942c6931a8ce7a08c1`.

Upstream bug:
importance-prefiltered candidate pools made relevant low-importance memories unreachable.

Current ANVIL check:
ANVIL Phase-25/26 semantic recall queries pgvector over durable embedded memories and hybrid RRF fuses lexical + semantic results, so the exact WeKnora bug is not presently established.

Harvest:
- add a defect-seeded regression proving a semantically exact low-importance memory remains reachable;
- telemetry equivalent to WeKnora's `outside_pool` can detect future accidental prefilter blindness.

### B3. Fail-closed durable generation

Donor:
- `653307b5c72d90aa6e43ac2f43bdcf859d14d0ad`.

Harvest:
- never write a model-generated durable artifact when completion was truncated;
- bounded continuation may repair it;
- if continuation budget is exhausted, reject writeback.

Candidate receiving surfaces:
ANVIL generated durable docs/receipts and any future self-maintaining knowledge projection.

## Priority C: later inspiration, not immediate code transfer

- Wiki revision/diff/rollback history.
- Tombstone + reference-counted artifact reclamation.
- Runtime queue operator UX, orphan detection, separate worker-pool telemetry.
- Knowledge-base profile aggregation.
- Cross-session memory UI/curation workflow.

These overlap existing ANVIL/Paseo/SIEVE responsibilities and should be mined only after source-specific gap analysis.

## Explicit non-targets

Do not import or adopt wholesale:
- WeKnora control plane;
- its full RAG stack/vector-store matrix;
- its sandbox runtime;
- its queue scheduler;
- its auth/RBAC system;
- its MCP protocol implementation;
- its frontend shell;
- its legacy Python MCP server.

## License boundary

WeKnora declares the project MIT with third-party components under their own licenses/notices. Any literal code extraction must preserve the applicable MIT notice and inspect `THIRD_PARTY_NOTICES.md` / file provenance first.

## Acceptance gate for any transfer

A WeKnora-derived change may enter an ANVIL-stack repo only when:
1. receiving repo source truth is reacquired at an exact SHA;
2. the gap is proven to exist;
3. the mechanism is re-expressed inside existing ANVIL/SIEVE authority boundaries;
4. focused tests include the donor failure mode;
5. no new broad authority or transport path is introduced;
6. diff is reviewed and exact tests pass;
7. provenance records the donor SHA and license status.

