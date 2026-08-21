# The ten steps — asks and deliverables

Each step states what it asks and what it produces. The written artifact is what the next step consumes.

---

## Phase I · Outside

### 01 · Outside
**Asks** Who calls this, at what volume, and what can they tolerate — and what is this organisation actually able to operate, host and afford?

**Demand.** Per client class: identity and origin (first-party app, partner system, internal service, machine, scheduled job) · trust zone · volume and shape (read/write mix, average and peak, burst profile) · tolerances, one line each (latency percentile, ordering scoped to a key, durability, accuracy) · devices and networks · geography and regime · adversaries · your own dependencies, which hand you a quota rather than accept one.

**Every client-visible promise is declared here, and nowhere else.** Two of them are routinely deferred and must not be:

- **The views this class reads, each with its own freshness bound** — one bound per view, not one per client. A buyer tolerates a five-minute-old recommendation and a two-second-old basket. Flattening those is how every read ends up on the strict store. The bound is the client's to state; step 03 decides only what it implies.
- **What this class is still handed when a dependency is down** — chosen from six client-visible outcomes in rising severity: correct but slower · correct but less · stale but complete · pending but signalled · absent but signalled · **wrong, which is never permitted.** Say which are accepted, how often, and what each costs them. Step 07 prices and proves these; it does not author them.

Every tolerance names the conditions it holds under. "p99 200 ms" and "p99 200 ms during the nightly partner window" are different requirements, and only the second is testable.

**Supply.** Team capability and size · hosting posture (cloud, on-prem, hybrid, regulated enclave) · contracts already held (CDN, archive tier, managed services) · cost envelope · expected lifecycle (spike · one-off product · prototype becoming platform · platform) · **consumers you cannot rebuild for** — who holds a client library, SDK, schema, image or public API version of this system that you cannot change and redeploy on their behalf. Name them. That one answer buys or refuses most of the delivery machinery at 04 and 09; where it is *nobody*, a second repository, a declinable version, a compatibility mode and a deprecation window are all uncited.

**Deliverable** A client-class table carrying tolerances, the view list with a freshness bound each, and the accepted degraded outcomes — plus an operating-envelope table. These are the **joint-signature artifacts**: everything the business half of the organisation can read, check and correct lives here, and nothing client-visible is decided after this step.

**Exit condition — no figure leaves this step unconsumed.** Beside each volume, name the later step that will divide, bound or decide with it. A figure with no named consumer is a question that should not have been asked: drop it, or find the decision it was standing in for. Checked again at 02, reported at 10.

### 02 · Load
**Asks** What does the stated demand actually amount to?

Requests/second (average and peak, with the peak factor stated) · bytes/day and bytes/year · working-set size · fan-out · growth over the horizon you are designing for.

**Deliverable** The demand arithmetic, shown as division, with every input labelled measured / contractual / assumed. **No technology named yet.**

Every figure elicited at 01 is either divided here or explicitly carried to a named later step. One still unconsumed at 10 is a finding against 01, not against this step.

---

## Phase II · Shape

### 03 · Data
**Asks** What is true, what is derived from it, and what must never break?

Take the step-01 view list and its bounds — they are promises and are not revised here. Resolve each into truth or projection: a view whose bound is looser than the write path is a projection. Then, per data class: which acknowledgement kind writes need (receipt / reservation / completion) · invariants that no view can tell you about · conflict policy · read-your-writes seams · commitment reads (views a client commits against, which cannot be arbitrarily stale).

**Deliverable** A data-class table. This is where most of the design is actually decided.

### 04 · Storage & transport
**Asks** Which store families and channel guarantees do the data classes require — and how many runnable units?

Match store family to access pattern. Declare all six channel properties per edge. Then granularity: runtime units, module seams, repo topology — each split citing a force.

**Deliverable** The labelled graph: every node typed, every edge annotated.

**Repo topology is a third axis, independent of runtime units and module seams.** It is not a scale or team-count question: it is whether a caller can be reached by your commit. One repository until a caller exists that cannot — that is the *published interface* line, and it was answered at 01. Two teams who can still land one commit together have not bought a second repository; one customer-held SDK has. A build system beyond the ecosystem default cites a repository whose correct incremental set can no longer be computed from the directory layout — record the ratio, measured build time against a tolerance stated first. A remote build cache is **a derived store with an invalidation contract**: a non-hermetic build breaks the contract, and a broken one serves confident wrong answers with green dashboards.

**Record the ratio beside the citation.** Per choice: what the citing requirement demands against what the mechanism supplies. Near 1 is sized to the job; tens are headroom and must name the factor — growth, peak, failover; hundreds mean the requirement cannot discriminate the choice, and the choice came from somewhere other than this derivation.

### 05 · Trust
**Asks** Where are the boundaries, and what is enforced at each?

Authn/authz per surface and at which hop · data classification · delegated authority · what is enforced versus merely checked · quota as policy.

**Deliverable** Trust boundaries marked on the graph, with the enforcement point named per invariant.

---

## Phase III · Behaviour

### 06 · Derivation
**Asks** What computes what, and how do partial results compose?

Per processor: stateless / stateful / batch · inputs and outputs · merge semantics when combining across shards, hosts or windows · error bounds if it approximates · model processors (inference) sized like any other.

**Deliverable** Processor table with composition rules. Merge semantics are where correctness quietly dies — combining per-host top-K does not give global top-K.

### 07 · Failure
**Asks** What does each client class still get when a dependency is down?

1. Bring the promises forward from step 01 unchanged — the tolerances and the accepted degraded outcomes are already on the table. Do not re-ask.
2. Prove each promised rung is **reachable on purpose** — a flag, a breaker, a shed rule — and say when it was last exercised. A ladder never exercised is fiction.
3. Mechanics per edge — timeouts, retries with jitter, circuit breakers, bulkheads, fallbacks, dead-letter handling. Every mechanic traces to a rung a client asked for, or it goes.

**Deliverable** A reachability proof per promised rung, and the per-edge failure spec. The ladder is carried forward, not authored here.

### 08 · Runtime & placement
**Asks** How many of what, where?

Instances, regions, placement, data residency, and the **second arithmetic pass**: capacity, given the stores chosen at 04.

**The artifact is decided here, one per runnable unit** — immutable, identified by content or commit, built once and promoted rather than rebuilt per environment. Delivery does not get to invent a runnable unit; if the pipeline emits a deployable the design never named, one of the two is wrong. **Each environment must name what it verifies that production cannot.** Environments are the most reliably uncited mechanism in a system: each is a full copy of the cost, the config drift, the data problem and the access-control surface.

**Deliverable** Capacity arithmetic with headroom stated · the artifact and how it is identified · the environment list with a justification each.

---

## Phase IV · Carry

### 09 · Operations
**Asks** How do we know it is working, how does it change, and what does it cost?

Detection proportioned to consequence — how badly a client is hurt, and where on the ladder that lands. Outcome KPIs in business terms (successful vs abandoned checkouts, not CPU). **Zero-tolerance counts**: promises with no acceptable rate, tracked as counts that must be zero. Release and delivery shape sized by lifecycle. Cost envelope carried from step 01.

**Change is derived here too, and each mechanism cites something:**

- **Version scheme** ← who can decline an upgrade. Semantic versioning requires a declared public API, because the major number is a promise about that boundary. Calendar versioning answers *how old is this*, and fits upgrades driven by support windows, security horizons or regulatory dates. Where the consumer cannot decline, the version is **identity, not promise** — a content hash or commit id.
- **Rollout strategy** ← what must be observed before committing. **Canary** when a graded signal exists and is worth waiting for; its whole value is the abort metric, so a canary without one is a slow deploy at the same price. **Blue-green** when rollback must be *instant*, at the price of a second footprint and state migration solved twice. Rolling replace is the default and cites neither. Name the force, then the strategy.
- **Evolution** ← the change that must land without an outage. **Parallel change** (expand · migrate · contract) for an incompatible interface — *migrate* is the skipped phase, so give it an owner and a date. **Branch by abstraction** inside one runnable unit. **Strangler fig** for replacing a system that already exists.
- **Flags have four lifespans, and conflating them is why flag debt accrues.** *Release* toggles are transient and **their removal is part of the mechanism**. *Experiment* toggles live as long as the experiment. *Permissioning* toggles are an authorisation rule wearing a flag's clothes — enforce them at a surface. **Ops toggles are the degradation ladder's actuators**, and are long-lived by design: a rung nobody can step onto during an incident was a document, not a mechanism.
- **Desired state and drift** ← a drift you must detect. Decompose before adopting: the desired-state repo is a **store, and it is truth** (quasi-static class, needs its own restore path, not rebuildable by replay); the agent's pull is a **channel** owing all six properties, ordering key included; the running system is a **projection** owing a stated staleness bound, commit-to-live, alarmed at half. **Drift is a zero-tolerance count, not a dashboard.** Where nothing but the pipeline can write, there is no drift and the reconciler is uncited.
- **Delivery metrics** — the published set is currently **five**: change lead time · deployment frequency (throughput); change fail rate · deployment rework rate (stability); failed deployment recovery time. Read against a stated tolerance, not an industry percentile.

Configuration is a deploy with no build step and usually no review — versioned, diffable, revertible, alarmed on divergence. **Environment variables are the wrong home for the quasi-static class**: no history, no diff, no reviewer.

**Deliverable** The operations table: what is watched, at what threshold, who is woken, and what it costs · a release procedure naming its rollout strategy, abort metric and last rehearsal · a version scheme naming the consumer it promises to · a flag register with a category and lifespan per entry.

### 10 · Trace
**Asks** Does every mechanism cite a requirement, every requirement map to a mechanism, and how many requirements name the *same* mechanism?

**Deliverable** The traceability map, with a coupling count beside each mechanism. Unmapped requirement → a gap; design for it. Unconsumed figure → a number collected at 01 that nothing here consumes; drop it, or name the decision it should have informed.

**A mechanism citing no client constraint takes the first disposition that applies — a search, not a menu.** ① **Delete** — nothing sits above it. ② **Surface a missing requirement** — tracing it lands on a client constraint that was always true and never written down; the common case, and often a *different* constraint from the one that bought the parent. ③ **Declare self-derived, naming the parent decision** — and the parent now faces the same test. Naming a parent moves the obligation up, it does not discharge it. Citations chain; the recursion ends at a client constraint or the finding sits at the root, and **a five-deep chain on an ungrounded adjective is one finding, not five.** Record the chain's root, depth and the link at which it stops tracing.

**Count the column — the third reading.** Both invariants ask whether a cell is *filled*; neither asks how many requirements name the same mechanism. Above one is a **coupling finding**: those promises fail together. The instrument is a column count on the map you already have. It is **not** a defect to fix by splitting — a second store derived from a coupling number rather than a client constraint is an uncited mechanism, and rule one deletes it. What the count buys: carry the mechanism onto the ladder as *n* entries rather than one (they may sit on different rungs); check whether the promises sharing it have the same signatories, because the strictest one now sets the floor for all of them; and rank the risk register by the count, since the highest is the largest blast radius on the page.

**The cutover, where something already exists.** In a running system an uncited mechanism is Chesterton's fence rather than a razor: report it as *no recorded reason — confirm before removing*, and produce an incremental path rather than a deletion. Named shapes: **strangler fig** across codebases, **branch by abstraction** inside one. Both keep the system releasable throughout, which is what makes them usable.

**Re-run trigger.** A zero-tolerance count that keeps firing is not an ops problem — it is a signal to re-run the derivation from step 03, because the source of truth was modelled wrong.

---

## Who signs which artifact

The ten steps emit a contract and a set of working notes. Say which is which, or the business half argues about topology and the engineering half quietly moves a promise. The boundaries are not the step boundaries.

| Category | Artifacts |
|---|---|
| **The business supplies** | The operating envelope — capability, hosting posture, contracts held, cost envelope, team topology, jurisdictions, lifecycle |
| **Both sign** | Client classes and quotas · every tolerance with its conditions · accepted degraded outcomes · zero-tolerance counts · **the shed order** · cost per request · retention and erasure commitments · published SLOs |
| **Engineering owns** | Primitive typing and the graph · store families · the six channel properties · capacity arithmetic, ratios, headroom · derivation and merge semantics · runtime and placement |

The traceability map is joint **column by column**: *Requirement* both sign, *Mechanism* engineering owns, *Verified by* both sign again. The shed order looks like an implementation detail and is a commercial decision — whoever owns the revenue owns it.

## Amendment — the model is living, not re-derived

Constraints move. The model was not wrong: a store is a **role with constraints**, and a given database is one thing *sufficient* to play it. When constraints change, only the **sufficiency claim** expires. Name the claim that died; do not redo the derivation.

| What changed | Re-check | Explicitly untouched |
|---|---|---|
| New client class | Its tolerances, quota, shed-order position, the surfaces it touches; ladder gains a row | Data classes, invariants, the source of truth |
| A tolerance tightens | The derivation producing that view; possibly the store under it; the alarm set at half the old bound | Client classes, invariants, the cost envelope |
| A volume moves 10× | Demand arithmetic and everything citing it — node counts, **partition counts**, recorded ratios | Tolerances, invariants, acknowledgement semantics |
| An invariant is added | Enforcer, serialisation point for its contended unit, ack semantics on that write path | Everything not touching that invariant |
| Operating envelope shrinks | Store choices, placement, granularity, buy-vs-build; run the subtraction test for real | **Client tolerances** — they do not move because you got poorer; that is a finding to escalate with numbers |

**An amendment to an engineering-owned artifact needs no counter-signature. An amendment to a joint-signature artifact is void until signed again**, because it is a promise with a price. The only change that buys a genuine re-derivation is the re-run trigger above. Write the amendment where the original decision lives, with the date and the claim that expired.
