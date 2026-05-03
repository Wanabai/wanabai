# Wanabai 0.1: The Spec-Mediated Attestation Harness

*A yellow paper on the transitional architecture. Companion to the [Wanabai white paper](WHITEPAPER.md).*

## Overview

Wanabai 0.1 is a **modular, plug-in pipeline** for spec-mediated
software production. It composes five independently-developable
modules, each with its own recipe and its own ecosystem of pluggable
agents:

1. **Discovery** *(upstream)* — captures functional and non-functional
   requirements from user intent.
2. **Specification** *(upstream)* — translates requirements into a spec
   corpus and runs full-spec attestation.
3. **Decomposition** *(pre-downstream)* — breaks the spec into phases
   and tasks, attaches per-task attestation.
4. **Implementation** *(downstream)* — produces code via a Ralph loop
   bound to the spec, with per-iteration gating.
5. **Delivery** *(post-downstream)* — deploys, sandboxes, and gates the
   shipped artifact.

The pipeline order is **rigid** (each module's input is the previous
module's output), but **modules are independently usable**: any module
can be skipped if the user supplies its output artifact directly. Bring
your own requirements, your own spec, your own decomposition, your own
implementation, or your own delivery — Wanabai picks up wherever the
upstream artifact is provided.

Inside every module the same machinery applies: agents (each agent =
system prompt + model + tools, defined in the Agent Registry) are pinned
to slots by the module's YAML recipe; gates run and emit promises; a
wrapper consolidates promises into a single derivable verdict. **The
recipe is the glue.** Standardized contracts (Promise/Report, Adapter,
Recipe, Registry) make Wanabai a substrate on which an *ecosystem* of
agents, modules, and recipes can be built — different teams plug in
different specifier strategies, different attestation rigors, different
implementer agents, different decomposition styles, all conforming to
the same module contracts.

## References

This harness specifies orchestration topology and gate contracts. For
canonical content of the specification corpus, verification toolchain,
attestation layers, tier model, and migration path toward steady-state
Wanabai, see the companion yellow paper:

- **YP** — *The Attestation Layer: Architecture Specification*
  (Saturnino, 2026). Cited inline as `[YP §<section>]`.

This document is **Wanabai 0.1** — the harness implementation of the
transitional architecture YP specifies for the 2026–2030 window. The
steady-state architecture (Wanabai) described in the companion white
paper *Wanabai: Software as Electricity* is the trajectory this harness
evolves toward as YP's migration path closes the spec-review chokepoint.

## Diagram

```mermaid
flowchart TD
    UserIntent([User Intent])
    Registry[/Agent Registry/]
    Recipes[/Per-module recipes<br/>composed + locked/]

    subgraph Upstream["Upstream"]
        direction TB
        Discovery["Discovery Module<br/>(elicit requirements)"]
        Spec["Specification Module<br/>(author + attest spec corpus)"]
        Discovery --> ReqArt[/Requirements Artifact/]
        ReqArt --> Spec
        Spec --> SpecCorpus[/Spec Corpus + Full-Spec Attestation/]
    end

    subgraph PreDownstream["Pre-Downstream"]
        direction TB
        Decomp["Decomposition Module<br/>(phases, tasks, per-task attestation)"]
        SpecCorpus --> Decomp
        Decomp --> WBS[/Work Breakdown<br/>+ per-task spec slices/]
    end

    subgraph Downstream["Downstream"]
        direction TB
        Impl["Implementation Module<br/>(Ralph loop per task,<br/>per-iteration gating)"]
        WBS --> Impl
        Impl --> Built[/Implemented Artifact<br/>+ concluded promise(s)/]
    end

    subgraph PostDownstream["Post-Downstream"]
        direction TB
        Delivery["Delivery Module<br/>(deploy, sandbox, approve)"]
        Built --> Delivery
        Delivery --> Shipped([Shipped Artifact<br/>+ delivery promise])
    end

    UserIntent --> Discovery

    %% Plug-in machinery
    Recipes -.pin agents + gates.-> Discovery
    Recipes -.pin agents + gates.-> Spec
    Recipes -.pin agents + gates.-> Decomp
    Recipes -.pin agents + gates.-> Impl
    Recipes -.pin agents + gates.-> Delivery
    Registry -.resolve agent IDs.-> Recipes

    %% Bring-your-own / skip flows
    BYO_Req([BYO requirements]) -.skip Discovery.-> Spec
    BYO_Spec([BYO spec corpus]) -.skip Discovery + Spec.-> Decomp
    BYO_WBS([BYO work breakdown]) -.skip upstream + decomp.-> Impl
    BYO_Built([BYO implemented artifact]) -.skip everything except delivery.-> Delivery

    %% Escalation (design-level fail routes upstream)
    Spec -. "design-level fail" .-> Discovery
    Decomp -. "spec-level fail" .-> Spec
    Impl -. "design-level fail" .-> Spec
```

## Modules

Wanabai 0.1 composes five modules in a rigid pipeline. Each module is
independently developable, has its own recipe, and exposes a stable
input/output contract. The pipeline order is fixed (each module's input
is the previous module's output) but modules are independently usable —
any module may be skipped if the user supplies its output artifact
directly.

### Discovery Module (Upstream)

**Purpose.** Capture functional and non-functional requirements from
the user's natural-language intent. The module performs requirements
engineering — Socratic elicitation, structured discovery, market-
standard requirements practices, or any approach the recipe pins.

**Input.** User intent (natural language).

**Output.** Requirements artifact — functional and non-functional
requirements, traceable to source statements, in a format the
Specification module consumes.

**Slots (recipe-pinned, illustrative):**

- `discoverer` — the elicitation agent.
- `requirements_reviewer` — reviews the requirements artifact (HITL or
  HOTL).

**Gates (recipe-configured).** Typically: requirements completeness,
internal consistency, non-functional coverage, optional HITL review.
Each gate emits a Promise/Report per the standard contract.

**Architectural commitments:**

- Self-contained: takes natural-language intent in, produces a
  structured requirements artifact out.
- The artifact format is owned by the *Discovery Module Spec* — not
  the harness.
- Internal recipe is fully owner-controlled; downstream modules see
  only the output artifact.

**Downstream specification requirement.** A separate *Discovery
Module Spec* must define: the requirements artifact format
(functional + non-functional sections, traceability, conventions);
the canonical slot list; the recommended gate set; and the schema
the Specification module expects as input.

### Specification Module (Upstream)

**Purpose.** Translate requirements into the spec corpus and run
**full-spec** attestation. The spec corpus follows YP's `specs/` layout
per [YP §The Specification Artifact as Theory Container]. Attestation
runs at the corpus level (not per-task) — TLA+/TLC for protocols,
Alloy for structural models, agentic spec-conformance review, optional
brownfield integration check against existing code.

**Input.** Requirements artifact (from Discovery, or BYO).

**Output.** Spec corpus + full-spec attestation results + concluded
promise.

**Slots (recipe-pinned, illustrative):**

- `spec_writer` — produces the SDD (architectural models, contracts,
  optional TLA+/Dafny/Lean) per [YP §Specification Tiers].
- `test_writer` — produces the TDD (properties, contracts) per
  [YP §Tier 1] and [YP §Tier 2].
- `spec_attestor` — runs deterministic full-spec checks (TLC, Alloy,
  internal consistency).
- `spec_reviewer` — HITL or HOTL spec-review chokepoint per
  [YP §The Spec-Mediated Verification Pipeline] and
  [YP §Human Review Points].

**Gates (recipe-configured).** Typically: TLC on TLA+, Alloy
structural check, brownfield integration check (when applicable),
HITL or HOTL spec review.

**Spec-Review Chokepoint (HITL/HOTL).** When the recipe declares a
`human_review` gate the run is HITL; otherwise an agent reviews and
the run is HOTL. Mixed mode (HITL for high-stakes specs, HOTL for
routine ones) is recipe-declared.

**Architectural commitments:**

- Spec corpus follows YP's `specs/` directory layout.
- Full-spec attestation runs once on the corpus, not per-task.
- Brownfield integration is recipe-pinned; if existing code is in
  scope, the recipe pins an integration-check gate.
- Failure routes upstream to Discovery for requirements rework per
  *Escalation Patterns*.

**Downstream specification requirement.** A separate *Specification
Module Spec* must define: the SDD and TDD artifact formats; the
canonical slot list; the recommended gate set per YP tiers; the
brownfield-integration gate contract; and the spec-review protocol
for both HITL and HOTL modes.

### Decomposition Module (Pre-Downstream)

**Purpose.** Break the spec corpus into phases and tasks suitable for
iterative implementation. Per-task spec slices are produced; **per-task
attestation runs here**. Models the project's existing task-management
patterns (the `task-manager` agent style) but recipe-driven and
module-bounded.

**Input.** Spec corpus (from Specification, or BYO).

**Output.** Work breakdown — phases and tasks with dependency graph —
plus per-task spec slices and per-task attestation results.

**Slots (recipe-pinned, illustrative):**

- `decomposer` — breaks the spec into phases and/or tasks.
- `task_validator` — per-task spec validation (slice well-formedness,
  tier coverage).
- additional slots per recipe (e.g., `phase_planner`, `dag_builder`).

**Gates (recipe-configured).** Per-task slice well-formedness;
cross-task dependency graph validation; per-task tier-coverage check
(does each slice have the right attestation tiers attached).

**Architectural commitments:**

- Optional module: small specs may skip Decomposition and pass the
  corpus directly to Implementation as a single-task work breakdown.
- Per-task attestation lives here — Implementation's per-iteration
  gates assume each task slice is already validated.
- Output (work breakdown + slices) is the canonical Implementation
  input format.

**Downstream specification requirement.** A separate *Decomposition
Module Spec* must define: the work-breakdown artifact format (phases,
tasks, dependency graph); the per-task spec-slice format; the
canonical slot list; and the rules for how Decomposition validates
slice well-formedness and tier coverage.

### Implementation Module (Downstream)

**Purpose.** Produce code that satisfies the spec via a Ralph loop.
Operates on one task slice at a time (or the whole spec corpus if
Decomposition was skipped). Per-iteration gating with promise/report
consolidation. **Partial attestation per task** — each task converges
independently and emits its own concluded promise.

**Input.** Spec corpus + (optional) work breakdown with task slices —
or BYO implementation target.

**Output.** Implemented code per task slice + concluded promise(s).

**Slots (recipe-pinned, illustrative):**

- `implementer` — produces code under test; pinned for the run.
- `reviewer` — agentic per-iteration review (vs spec slice).
- `auditor` — agentic per-iteration audit (vs spec + tests).
- `gate_wrapper` — consolidates per-iteration gate promises into a
  single derivable concluded verdict.

**Gates (recipe-configured).** Typically: regression vs prev iter,
regression vs whole codebase, unit tests (TDD properties), agentic
review, agentic audit. See *Gate Mechanics* under Cross-Cutting
Machinery.

**Frame.** For each task, the spec slice + its TDD form locked rails
for the iteration. The Ralph loop cannot mutate them; the implementer
can only produce code that satisfies them. Per YP, `src/` is a
regenerable compilation of `specs/`.

**Ralph Loop.** Iterates the implementer over each task slice until
the wrapper emits "concluded" or the budget is exhausted. Bounded
scope per iteration: the implementer addresses only issues in the
previous iteration's wrapper report.

**Circuit Breaker.** Per-task: max iters / tokens / wall-clock for
that task. Exhaustion → did-not-converge for that task; operator
triages; other tasks may still proceed independently.

**Architectural commitments:**

- Frame (spec slice + TDD) is locked per task; Ralph loop cannot
  mutate.
- All recipe-configured per-iteration gates run regardless of
  individual failures.
- Wrapper's `concluded` is derivable from gate promises (see
  *Promise/Report Contract*).
- Design-level failures detected during implementation route upstream
  to Specification per *Escalation Patterns*.
- Tasks are independent: one task's did-not-converge does not block
  other tasks unless the dependency graph requires it.

**Downstream specification requirement.** A separate *Implementation
Module Spec* must define: the per-task input format (spec slice +
TDD); the per-iteration gate set defaults; the wrapper consolidation
algorithm; the bounded-scope policy for next-iter implementer input;
the circuit-breaker thresholds and tuning; the inter-task dependency
sequencing rules.

### Delivery Module (Post-Downstream)

**Purpose.** Take the implemented artifact through deployment,
sandboxing, and approval. The module's **external contract is stable
across the migration to steady-state Wanabai**; only the internal
recipe evolves. In Wanabai 0.1 the recipe pins thin sandbox + HITL
approval; in steady-state Wanabai the recipe pins blast-radius
containment and adversarial outcome testing per the WHITEPAPER.

**Input.** Implemented artifact + concluded promise(s) from
Implementation.

**Output.** Shipped artifact + delivery promise.

**Slots (recipe-pinned, 0.1 illustrative set):**

- `deployer` — provisions and deploys to sandbox.
- `delivery_reviewer` — HITL approver in 0.1; replaced by an agentic
  adversarial outcome tester in steady-state Wanabai.

**Gates (recipe-configured).** Sandbox-deploy success; smoke-test
pass; (optional in 0.1, mandatory in steady-state) blast-radius
containment check and adversarial outcome test; HITL approval gate
(0.1) or its agentic equivalent (steady-state).

**Architectural commitments:**

- External contract stable across the 2028–2032 migration to
  steady-state Wanabai: input and output formats do not change.
- Internal recipe evolves: 0.1 pins thin deploy + HITL; steady-state
  pins sandboxed adversarial outcome testing per WHITEPAPER.
- Module is optional in 0.1 (operator may handle delivery outside the
  harness); becomes mandatory in steady-state Wanabai.
- 0.1 internals are deliberately minimal — the contract surface is the
  durable artifact, the internals are throw-away by design.
  Over-investment in 0.1 internals is wasted work replaced during the
  migration; under-investment is fine.

**Strategic role.** Delivery is the proof point of Wanabai's modular
thesis. If the 0.1 internals can be replaced by steady-state Wanabai's
sandboxed adversarial testing without changing any other module,
modularity has been demonstrated. If the swap requires touching
Implementation, Decomposition, Specification, or Discovery, the
architecture is not yet real — regardless of how many other modules
ship.

**Downstream specification requirement.** A separate *Delivery Module
Spec* must define: the implemented-artifact input format; the
shipped-artifact + delivery-promise output format; the recipe slot
list for both 0.1 (HITL) and steady-state (HOTL adversarial); the
migration mechanics for swapping the internal recipe without
breaking the external contract.

## Cross-Cutting Machinery

The machinery below is shared by every module. Each module's recipe
configures the slots and gates within these mechanics; the contracts
and behaviors are uniform across modules.

### Agent Registry

The registry defines and addresses every agent the harness can invoke.
Recipes never pin raw models; they pin agent identifiers, which the
registry resolves to a full agent definition.

An **agent** is the bound combination of:

- **System prompt** — the instruction text loaded at the start of every
  invocation; defines the agent's role, rigor, and behavior.
- **Model** — the underlying inference engine.
- **Tools** — the whitelist of capabilities the agent may invoke (file
  I/O, web search, language servers, MCP servers, custom scripts).
- **Adapter binding** — the connection mechanism (see *Adapters* below).

Two agents may share an underlying model but differ in system prompt or
tool whitelist; the registry treats them as distinct identities. This is
how a recipe expresses the *basic injection-check auditor* vs *full
harness-verification auditor* rigor difference — the rigor lives in the
agent definition, not the model.

**Architectural commitments:**

- Every agent is addressable by a stable identifier.
- The registry is the single source of truth for what an agent is.
  Recipes only reference identifiers; the harness only invokes through
  the registry's resolution.
- The registry must surface enough metadata for `recipe validate` to
  enforce diversity rules and for the lock to inline a fully-resolved
  agent definition.
- An agent's identity (system prompt + model + tools + adapter) is
  immutable for the duration of any locked run; mutations to a
  registry entry produce a new, distinct identity.

**Downstream specification requirement.** A separate *Agent Registry
Schema* document must define: the registry's storage format and
distribution model; the identifier grammar (versioning, namespacing,
alias policy); the mandatory and optional fields per agent entry; and
the validation rules the harness applies at lock-resolution time.

### Adapters

The adapter is the concrete invocation mechanism the harness uses to
talk to an agent. The harness is adapter-agnostic at the orchestration
level — adding a new adapter type does not change the recipe schema or
the gate contract.

Conceptual adapter categories (extensible):

- **API** — direct call to a vendor inference endpoint over the
  vendor's protocol (HTTP/SDK).
- **MCP** — Model Context Protocol; the agent is exposed as an MCP
  server and invoked via standard MCP messaging. Useful when tool
  surface or resources are vendor-managed.
- **CLI** — headless command-line invocation. The CLI bundles auth,
  retries, tool wiring, and prompt injection so the harness only needs
  to pipe input/output. Useful for vendor parity and for reproducibility
  across local and CI environments.

Adapter choice is a per-agent registry property. Two slots in the same
recipe may pin agents with different adapters — for example, a
CLI-backed implementer and an API-backed reviewer — without any
harness-level plumbing change.

**Architectural commitments:**

- Adapters translate between the harness's abstract agent invocation
  (request: prompt + context + tool whitelist; response: text + tool
  calls) and the adapter's native protocol.
- Adapters are responsible for normalizing the agent's native output
  into the *Promise/Report Contract* shape consumed by gates.
- Failure modes (timeouts, rate limits, transport errors, malformed
  output) surface as gate-level failures; adapters do not silently
  retry across the harness boundary.
- New adapter types plug in without changes to the recipe schema, the
  registry schema, or the gate contract — the harness depends only on
  the adapter's contract, not its implementation.

**Downstream specification requirement.** A separate *Adapter Contract*
document must define: the harness-side calling convention; the
per-adapter native-output normalization rules; the failure-mode
mapping; and the plug-in interface for adding new adapter types.

### Recipe (composable YAML)

Each module has **its own recipe**, declaring slot-to-agent assignment
and the module's gate set. Slots are scoped per module: a `reviewer`
slot in the Specification module is a different addressable target from
a `reviewer` slot in the Implementation module — they pin to different
agents and answer to different gate sets.

A **pipeline recipe** composes the per-module recipes for a full run.
Operators can:

- Run a full pipeline by composing all five module recipes.
- Run a partial pipeline by composing a subset (e.g., Specification +
  Decomposition only), supplying the missing modules' outputs as BYO
  artifacts.
- Override any module recipe via overlays without touching the others.

Per-module slots are documented in each module section under *Modules*
(illustrative slot lists; the canonical lists live in each *Module
Spec*).

- **Composition:** base recipe + overlays, deep merge, child wins.
  Composition applies both within a module and across modules at the
  pipeline level.
- **Resolution:** the composed pipeline recipe is written to a lock
  artifact at run start; agent IDs are resolved against the registry and
  the resolved definitions (system prompt + model + tools + adapter) are
  inlined into the lock. The run uses the lock, not the source recipes
  or live registry — registry mutations after lock time do not affect
  the run.
- **Override semantics:** overlays may override or null-out slots and
  gates. They cannot delete keys via omission (typo safety).
- **Gate set:** each module's recipe declares its gate set. Tier-to-gate
  mapping follows [YP §Specification Tiers] and [YP §Specification
  Language Selection Guide]. Spec-stage deterministic gates (TLC, Alloy)
  belong to Specification; per-task deterministic gates and agentic
  gates belong to Implementation; etc.
- **HITL spec-review chokepoint:** declared in the Specification
  module's recipe per [YP §The Spec-Mediated Verification Pipeline] and
  [YP §Human Review Points]. **If omitted, an agent handles the
  chokepoint and that module's run is HOTL.**
- **Diversity rules** (lint warnings, surfaced by `recipe validate`):
  - within Implementation: `auditor == implementer`,
    `gate_wrapper == implementer`, `test_writer == implementer`
    (latter sourced from the Specification module's pinned agent).
  - within any module: any verifier slot pinned to the same agent as
    the producer slot it verifies.

**Downstream specification requirement.** A separate *Recipe Schema*
document must define: the per-module recipe file format; the pipeline-
recipe composition format; the deep-merge and conflict-resolution
algorithm; the lock format; the `recipe validate` rule set including
the cross-module diversity-warning policy; and the gate-config schema
for each module.

### Gates

Every module's recipe defines a gate set. Each gate — deterministic or
agentic — emits a binary promise (pass/fail) and, on fail, a report
explaining what didn't pass. All configured gates run regardless of
individual failures; a failing gate does not short-circuit the rest. The
N gate promises (and any failure reports) are consumed by a
`gate_wrapper` agent (one per module that has multi-gate consolidation),
which produces a single final promise per module run and, on "not
concluded," a single consolidated report — that consolidated report,
not the individual gate reports, is what feeds the next attempt
(re-author for upstream modules; bounded next iteration for the
Implementation Ralph loop).

**Agent-determined rigor.** For agentic gates (review, audit, etc.) the
rigor of the check is determined by the agent pinned to that slot, not
by the harness. A `reviewer` slot can be pinned to a basic style-and-typo
agent or to one that performs full spec-conformance verification with
web research. Likewise an `auditor` slot can be pinned to a quick
injection-check agent or to one that runs full harness verification,
fetches latest rules from the web, and re-runs full test suites. The
harness is agnostic to rigor; it requires only that the pinned agent
honor the promise/report contract. See [YP §Specification Language
Selection Guide] and [YP §The Attestation Stack] for the full tier model
and per-tool characteristics.

**Gate taxonomy.** Gates fall into three classes by *what* they check
and *when*:

| Class                         | Stage                  | Examples (mapping to YP tiers)                                                              |
|-------------------------------|------------------------|---------------------------------------------------------------------------------------------|
| Spec-stage deterministic      | upstream (pre-impl)    | TLC on TLA+ specs ([YP §Tier 3]); Alloy structural checks; spec internal-consistency checks |
| Code-stage deterministic      | downstream (per-iter)  | Hypothesis property tests ([YP §Tier 1]); runtime contracts ([YP §Tier 2]); Dafny + Z3 ([YP §Tier 4]); Lean kernel ([YP §Tier 4]); regression suites |
| Agentic                       | upstream or downstream | spec reviewer; SDD-conformance reviewer; auditor (rigor agent-determined)                   |
| Continuous (post-`concluded`) | post-loop              | Trace verification with TLC ([YP §Tier 5]); runtime contracts in production                 |

Continuous gates (the last row) are *not* part of the Ralph loop. They
fire after the loop emits "concluded," validating runtime behavior
against the spec corpus indefinitely. Outside the scope of this
harness; mentioned for completeness.

The table below shows a typical example gate set for the
*Implementation* module's per-iteration gating. Ordering is for log
readability and conventional cost progression, not control flow.
Specification, Decomposition, and Delivery modules each have their own
typical gate sets — see the relevant *Module Spec*.

| Order | Gate                          | Type          | Purpose                                                  |
|-------|-------------------------------|---------------|----------------------------------------------------------|
| 1     | CMD Regression vs prev iter   | deterministic | catch what this iteration broke                          |
| 2     | CMD Regression vs codebase    | deterministic | catch what the impl broke globally                       |
| 3     | CMD Unit Tests                | deterministic | run the TDD suite from the Specification module          |
| 4     | Agt Review vs spec slice      | agentic       | does the implementation match the spec slice?            |
| 5     | Agt Audit                     | agentic       | independent correctness check vs spec slice + TDD        |

### Gate Wrapper

A wrapper agent runs after all of a module's configured gates have
completed. Consumes the N per-gate promises (and any reports from
failed gates) and produces:

- A single final binary *concluded* promise — the signal the module's
  control flow checks for completion (the Implementation module's
  Ralph-loop `while`; the upstream modules' single-pass attestation).
- On "not concluded," a single consolidated report — the guidance
  document fed to the next attempt (next Ralph iteration for
  Implementation; re-author for upstream modules; operator triage for
  Delivery in 0.1).

Outcomes:

- **Concluded:** the wrapper attests the module's outputs satisfy its
  rails. The module exits with the promise alone — no final report is
  produced for that module's run.
- **Not concluded:** the consolidated report becomes the next attempt's
  bounded input.

Per-gate report payloads conform to [YP §Feedback Loops] (shrunk
Hypothesis input, contract violation with input state, TLC error trace,
Z3 counterexample with loop-invariant hints) — these are the canonical
schema variants the wrapper consolidates. Design-level failures (TLC
invariant violations, Dafny unprovable postconditions) escalate to an
upstream module rather than retrying in-loop, per [YP §Feedback Loops]
*"TLC invariant violation → ESCALATE TO HUMAN"* and per *Escalation
Patterns* below.

### Promise/Report Contract

The interop contract every gate and the wrapper must honor. The harness
owns the envelope and the conceptual shape; YP §Feedback Loops owns the
per-tier payload variants.

**Per-gate output.** Each gate produces a structured output that
carries, at minimum:

- A binary verdict (pass or fail).
- The identity of the gate that emitted it.
- On fail: a payload describing what went wrong, in a shape drawn from
  [YP §Feedback Loops]. Canonical variants include property
  violations, contract violations, TLC error traces, Z3
  counterexamples, Lean tactic-level failures, regression failures,
  agentic findings, and human rejections (upstream HITL).

The harness imposes the envelope; payload shapes are tier-specific and
follow YP. New tier-specific payload variants extend the contract
without breaking existing ones.

**Wrapper output.** After all gates run, the wrapper produces one of
two outcomes:

- **Concluded** — a promise carrying *evidence* of every gate's pass
  verdict. The wrapper may emit "concluded" only when every gate
  promise in evidence is pass. This makes the wrapper's verdict
  *derivable*: there is no synthesis discretion on the exit path.
- **Not concluded** — a single consolidated report whose *scope* is
  the next iteration's bounded backlog for the implementer. The
  wrapper has synthesis authority on this path: it may deduplicate,
  prioritize, and merge per-gate findings into actionable items, but
  it must preserve every flagged location at minimum.

**Architectural commitments:**

- The envelope is uniform across all gate types (deterministic, agentic,
  upstream, downstream) — only the payload varies.
- The exit verdict is derivable from gate promises, not synthesized by
  the wrapper.
- The next-iteration scope is the wrapper's only synthesis authority
  and is bounded by the union of failed-gate findings.

**Downstream specification requirement.** A separate *Promise/Report
Schema* document must define: the exact field names, types, and
serialization format for the gate-output envelope and the two
wrapper-output cases; the canonical payload variant catalog mapped to
YP tiers; the rules for adding new payload variants when introducing
new gate tiers; and the validation policy at adapter-side
normalization.

### Circuit Breaker

Bounded execution boundary, applied per-module-run. Most relevant in the
Implementation module's per-task Ralph loop, but every module has a
budget and a failure terminator.

Exit conditions:

- `gate_wrapper` emits "concluded" → module run exits with the promise.
- Budget exhausted (max iterations, tokens, or wall-clock — applied at
  the granularity the module defines: per-task for Implementation,
  per-module-run for upstream/downstream modules) → did-not-converge
  report routed per *Escalation Patterns*.

Budget thresholds and granularity are recipe-pinned per module.

## Escalation Patterns

Routing principle: failures route between **modules** based on what
kind of bug they indicate, not by which gate emitted them. Per
[YP §Feedback Loops], design-level failures escalate to an upstream
module rather than retrying in-loop within the current module.

Three routing classes plus two terminal cases:

- **Within-module failures** — the module's own producer didn't
  satisfy its own rails. Same module retries with bounded scope drawn
  from the wrapper's consolidated report. The clearest case is the
  Implementation module's per-iteration retry of the `implementer`
  agent. Discovery, Specification, Decomposition, and Delivery may
  also retry within their own module budgets when their gates flag
  fixable issues.
- **Upstream-module failures** — the input artifact from an upstream
  module is wrong; the current module cannot proceed without
  re-authoring upstream. The current module's run is abandoned; the
  failure report routes to the upstream module that produced the
  input artifact. Examples: TLC invariant violation in Specification
  routes to Discovery (requirements were wrong); spec-level fail in
  Decomposition routes to Specification (spec corpus was wrong);
  design-level fail in Implementation routes to Specification (spec
  was wrong, code can't fix it).
- **Ambiguous failures** — the failure type may indicate either an
  upstream issue or a within-module issue (canonical example: Dafny
  unprovable postcondition — the postcondition may be too strong or
  the implementation may be wrong). Routing is recipe-pinned per
  module.

Two terminal cases:

- Budget exhausted (per-module circuit breaker fires) — did-not-
  converge report routes to the operator.
- Wrapper malformed or timeout — wrapper itself fails to emit a
  promise; treated as a non-recoverable module-run error.

**Architectural commitments:**

- Routing class is determined by failure *kind*, not by which gate
  fired or which module fired it. A reviewer agent in any module that
  flags an upstream-level issue routes upstream regardless of its own
  module's position.
- Upstream re-authoring abandons the current module run; nothing
  carries forward except the failure report fed to the upstream
  module's recipe.
- Within-module retry is bounded by the wrapper's consolidated report;
  the producer agent may not address issues outside that scope.
- Modules are independently terminable: one module's did-not-converge
  does not implicitly fail other modules unless the dependency graph
  requires it.

**Downstream specification requirement.** A separate *Escalation
Policy* document must define: the operator-pinned routing for
ambiguous failure types per module; the upstream re-authoring
mechanics (which failure-report payloads feed which upstream module's
inputs); the wrapper-failure handling protocol; the per-module budget
exhaustion handling; and the recipe-level grammar for declaring
per-failure-kind routing overrides.

## Downstream Specifications

The yellow paper specifies the architecture and the commitments every
implementation must satisfy. Concrete development proceeds against a
set of downstream specification documents — split into **module specs**
(one per module) and **cross-cutting specs** (one per shared
mechanism). Each is owned separately, each focused on the file format,
schema, or protocol that the yellow paper deliberately leaves abstract.

**Module specs (one per module):**

1. **Discovery Module Spec** — requirements artifact format
   (functional + non-functional, traceability); canonical slot list;
   recommended gate set; input schema for the Specification module.
2. **Specification Module Spec** — SDD and TDD artifact formats;
   canonical slot list; recommended gate set per YP tiers;
   brownfield-integration gate contract; spec-review protocol for
   HITL and HOTL modes.
3. **Decomposition Module Spec** — work-breakdown artifact format
   (phases, tasks, dependency graph); per-task spec-slice format;
   canonical slot list; slice well-formedness and tier-coverage
   validation rules.
4. **Implementation Module Spec** — per-task input format (spec slice
   + TDD); per-iteration gate set defaults; wrapper consolidation
   algorithm; bounded-scope policy for next-iter implementer input;
   circuit-breaker thresholds; inter-task dependency sequencing.
5. **Delivery Module Spec** — implemented-artifact input format;
   shipped-artifact + delivery-promise output format; recipe slot
   list for both 0.1 (HITL) and steady-state (HOTL adversarial);
   migration mechanics for swapping the internal recipe without
   breaking the external contract. **Weight contract over internals**:
   the internals are throw-away during the 2028–2032 migration; the
   contract is the permanent artifact and deserves the bulk of this
   spec's attention.

**Cross-cutting specs (one per shared mechanism):**

6. **Agent Registry Schema** — registry storage and distribution
   model; identifier grammar (versioning, namespacing, alias policy);
   per-entry mandatory and optional fields; lock-resolution validation
   rules.
7. **Adapter Contract** — harness-side calling convention; per-adapter
   native-output normalization; failure-mode mapping; plug-in
   interface for new adapter types.
8. **Recipe Schema** — per-module recipe file format; pipeline-recipe
   composition format; deep-merge algorithm; lock format;
   `recipe validate` rule set including cross-module diversity-warning
   policy; gate-config schema for each module.
9. **Promise/Report Schema** — exact field names, types, and
   serialization for the gate-output envelope and the two wrapper-
   output cases; canonical payload variant catalog mapped to YP
   tiers; rules for extending the variant set; adapter-side
   normalization validation.
10. **Escalation Policy** — operator-pinned routing for ambiguous
    failure types per module; upstream re-authoring mechanics;
    wrapper-failure handling; per-module budget-exhaustion handling;
    recipe-level grammar for per-failure-kind routing overrides.
11. **Budget Accounting** — token, iteration, and wall-clock budget
    formulation per module; cost reporting; partial-budget resumption
    semantics; pipeline-level vs per-module budget allocation policy.

Each downstream spec is authoritative for its file format, schema, and
operational semantics. This paper is authoritative for the
architectural commitments those documents must satisfy.

## Resolved Positions

The harness commits to the following positions:

1. **HITL.** The harness adopts YP's spec-review chokepoint as a
   first-class concept, declared in the Specification module's
   recipe. When the recipe declares a human review gate, the
   Specification module run is HITL; when omitted, it is HOTL with
   agentic review only.
2. **Spec corpus shape.** The Specification module produces a corpus
   in YP's `specs/` directory layout: `specs/properties/`,
   `specs/contracts/`, `specs/models/` (TLA+), `specs/verified/`
   (Dafny/Lean). `src/` is a regenerable compilation of `specs/`
   per YP.
3. **Recipe gate scope.** Each module's recipe declares its own gate
   set. Spec-stage deterministic gates (TLC, Alloy) belong to
   Specification; per-task validators belong to Decomposition; per-
   iteration deterministic and agentic gates belong to Implementation;
   deploy/sandbox/approval gates belong to Delivery.
4. **Architectural position.** The harness is **Wanabai 0.1** — the
   transitional-architecture implementation per YP's 2026–2030
   migration window. The steady-state Wanabai architecture (closed
   loop, full HOTL with exception handling, twelve-stage continuous
   loop) is the trajectory this harness evolves toward as the
   chokepoint dissolves.
5. **Agent abstraction.** Recipes pin slots to agent identifiers, not
   raw models. Agents — the bound combination of system prompt, model,
   tools, and adapter — are defined in the Agent Registry. Connection
   mechanism (API, MCP, headless CLI) is a per-agent registry property;
   the harness is adapter-agnostic at the orchestration level. Lock
   resolution inlines the full agent definition into the lock so
   registry mutations do not affect locked runs.
6. **Modular pipeline.** Wanabai 0.1 composes five modules — Discovery,
   Specification, Decomposition, Implementation, Delivery — each with
   its own recipe and stable input/output contract. Pipeline order is
   rigid (each module's input is the previous module's output) but
   modules are independently usable: any module can be skipped if the
   user supplies its output artifact directly. This is the foundation
   for the Wanabai ecosystem — different teams contribute different
   modules, agents, and recipes against stable cross-module contracts.
7. **Per-module recipes + pipeline composition.** Each module owns its
   own recipe, scoped to its own slot namespace. A pipeline recipe
   composes module recipes for a full run; partial pipelines are
   first-class.
8. **Decomposition placement.** Spec breakdown into phases and tasks,
   plus per-task attestation, lives in the Decomposition module
   (pre-downstream). Full-spec attestation lives in Specification
   (upstream). Implementation receives validated task slices and runs
   per-task Ralph loops. YP is silent on decomposition; this is
   Wanabai's contribution.
9. **Delivery contract stability across migration.** The Delivery
   module's external contract (input: implemented artifact + concluded
   promise; output: shipped artifact + delivery promise) is stable
   across the 2028–2032 migration to steady-state Wanabai. The
   internal recipe evolves: 0.1 = thin deploy + HITL approval;
   steady-state = blast-radius sandbox + adversarial outcome testing
   per the WHITEPAPER. Module replacement is recipe-level, not
   architecture-level.

## Invariants

1. Each module's pinned agent (per slot) is fixed for the duration of
   that module's run — same system prompt, same model, same tools,
   same adapter. The Implementation module's `implementer` is fixed
   per task across all Ralph iterations.
2. Gate failure never triggers an **agent swap**. Within-module failure
   → bounded retry within the module; upstream-module failure →
   escalate per *Escalation Patterns*.
3. All recipe-configured gates run on every module run regardless of
   individual failures. Each gate emits a binary promise (pass/fail)
   and, on fail, a report. The module's `gate_wrapper` consolidates
   the gate promises into a single final concluded promise; on "not
   concluded," it also produces a single consolidated report. The
   next attempt's bounded scope is the wrapper's consolidated report —
   not the raw per-gate reports.
4. Pipeline recipe (composed of per-module recipes) is locked at run
   start and immutable for the run's duration.
5. Once a module produces its output artifact and emits a concluded
   promise, the artifact is frozen for downstream modules. Downstream
   actors may not mutate upstream artifacts; design-level failures
   abandon the current module run and route upstream for re-authoring.
6. Every gate output conforms to the *Promise/Report Contract*. Every
   wrapper's *concluded* verdict is **derivable** — emitted only when
   every gate's verdict in the carried evidence is *pass*. The wrapper
   retains synthesis authority only on the not-concluded path.
7. Module flow order is rigid (Discovery → Specification →
   Decomposition → Implementation → Delivery), but modules are
   independently usable: any module may be skipped if its output
   artifact is supplied directly as BYO input to the next module.
   No module may run before its input artifact exists.
8. The Delivery module's external contract (input: implemented
   artifact + concluded promise; output: shipped artifact + delivery
   promise) is stable across the migration to steady-state Wanabai.
   Internal recipes evolve; the contract does not.
