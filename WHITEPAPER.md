# Wanabai

## Software as Electricity

**Author:** Leonardo Saturnino
**Email:** lrsaturnino@gmail.com
**GitHub:** https://github.com/lrsaturnino
**Repository:** https://github.com/wanabai/wanabai
**Website:** https://wanabai.com
**Date:** April 2026
**Type:** White paper

---

## Companion papers

1. Saturnino, L. (2026). [*Out of the Loop: A Glimpse Into the Next 15 Years of the Software Industry*](https://saturnino.substack.com/p/out-of-the-loop). Article.
2. Saturnino, L. (2026). [*The Attestation Layer: Spec-Mediated Verification for the Age of Agent-Produced Software*](https://saturnino.substack.com/p/the-attestation-layer). Article.
3. Saturnino, L. (2026). [*Software as Electricity: The Closed-Loop Attestation Architecture*](https://saturnino.substack.com/p/software-as-electricity). Article.
4. Saturnino, L. (2026). [*The Attestation Layer: Architecture Specification*](https://github.com/lrsaturnino/attestation-layer). Yellow paper.

---

*Name. "Wanabai" (pronounced wah-na-bye) is the name of this project. The name is a deliberate pun on "wannabe AI," acknowledging the core architectural claim that the system described here is not artificial general intelligence, not a goal-directed agent, and not autonomous in the decision-theoretic sense — it is an input-output system that produces verified software within a defined envelope. The name embodies the humility of the architecture: Wanabai wants to be software, and Wanabai produces software, and those two things together are exactly what Wanabai is. The domain wanabai.com is the canonical reference for the project.*

---

**Abstract.** Wanabai is an autonomous production system that accepts a request for software, produces a deterministic artifact that satisfies the request, verifies the artifact against a formal specification corpus, deploys the artifact into a contained runtime, observes its outcomes against declared intent, and returns a result to the caller. The production loop runs continuously and self-improves along a gradient defined by a verification suite. The internal processes are stochastic; the external interface and the produced artifacts are deterministic. Human oversight is concentrated at four specific roles outside the main loop: spec authors, configuration attestors, quality engineers, and constitutional auditors. This paper specifies the architecture of Wanabai, the interfaces it exposes, the mechanisms by which it maintains quality at volume, the self-improvement loop by which it produces its own successors, and the human oversight layer that bounds its behavior.

---

## Scope and Relationship to Companion Papers

This white paper describes the **steady-state architecture** of the software production system that the attestation-layer transition converges toward: a closed, continuously-running loop where humans exist only in oversight roles outside the main production path. It is a vision of the endpoint, not a specification of a system that can be built today with today's tooling, skill distribution, and regulatory environment.

The architecture described here is reached only after the human chokepoint at specification review — present in the transitional architecture that precedes it — dissolves into exception rotation. The formal specification of that transitional architecture is given in the companion [*Attestation Layer* yellow paper](https://github.com/lrsaturnino/attestation-layer). The transitional architecture is what operators should implement now and through roughly 2030; Wanabai is what that architecture converges toward as the chokepoints dissolve (approximately 2028-2032) and the loop closes.

The two architectures are deliberate bookends, not mirrors of the same system. The transitional architecture has a human reviewer inside the main loop at the spec-review chokepoint and a two-agent topology (specifier, verifier); Wanabai has four human oversight roles outside the main loop, a three-agent topology (specifier, coder, adversarial ensemble), a twelve-stage continuous loop with sandbox containment and auto-promotion, and a self-improvement gradient governed by constitutional audit. Each architecture is correct for its moment in the transition.

The companion articles [*The Attestation Layer: Spec-Mediated Verification for the Age of Agent-Produced Software*](https://saturnino.substack.com/p/the-attestation-layer) (transitional) and [*Software as Electricity: The Closed-Loop Attestation Architecture*](https://saturnino.substack.com/p/software-as-electricity) (steady-state) describe the economic and historical context for this two-stage evolution. Readers seeking a concrete implementation spec should start with the [yellow paper](https://github.com/lrsaturnino/attestation-layer). Readers seeking the institutional design of the endpoint should start here.

**Migration path from the transitional architecture.** Operators reach Wanabai from the transitional Attestation Layer (specified in the [yellow paper](https://github.com/lrsaturnino/attestation-layer)) through four sequential steps. Each step removes a human role from inside the main loop and adds an automated or outside-the-loop mechanism that replaces it:

1. The human spec-review chokepoint dissolves into the three exception classes described in the Definitions section of this paper (spec contradiction, outcome mismatch, novel situation) and in the Exception-Handler Model section of [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity). Human review migrates from a full-time gating role to an on-call rotation scaling with the exception rate.
2. The transitional architecture's linear spec → verify pipeline fuses into the twelve-stage continuous Production Loop described in this paper. There is no longer a distinct "verify phase" separate from a "build phase"; every change triggers immediate re-derivation, re-verification, sandbox deployment, and outcome testing.
3. Blast-radius containment (described in the Blast Radius Containment section) and adversarial outcome testing against declared intent (the adversarial outcome testing stage of the Production Loop) are introduced. Neither exists in the transitional architecture; both are necessary preconditions for auto-promotion without a human gate.
4. Canonical resolution rules (the canonical resolution stage of the Production Loop) automate the contradiction handling that humans performed at the transitional review gate, and self-improvement against a meta-verification suite (described in the Self-Improvement section) is added under the constitutional audit regime (described in the Human Oversight Layer section). The architecture becomes self-modifying within bounds set by a small number of licensed oversight roles.

The migration window is approximately 2028-2032, coinciding with Chokepoint Dissolution in the timeline tables of [*Out of the Loop*](https://saturnino.substack.com/p/out-of-the-loop) and [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity). Before that window, operators should implement the transitional architecture specified in the [yellow paper](https://github.com/lrsaturnino/attestation-layer). After it, they should run the architecture described here.

---

## Introduction

Wanabai is an autonomous production system that decouples stochastic production from deterministic consumption through a formal specification corpus that serves as the non-stochastic anchor between them. It accepts requests at arbitrary volume, produces deterministic artifacts verified against the corpus before release, deploys them into contained runtimes, observes outcomes against declared intent, and meters the entire process as a utility. This paper specifies the system. The economic and architectural argument for a closed-loop production system of this shape — why build-and-verify fuse, why the human chokepoint at spec review dissolves, and why software becomes a metered utility consumed through protocol sockets — is developed in the companion article [*Software as Electricity: The Closed-Loop Attestation Architecture*](https://saturnino.substack.com/p/software-as-electricity).

## Definitions

This section gives the positive definitions of the load-bearing terms used throughout the paper. The complementary section What Wanabai Is Not delimits the concept negatively, ruling out adjacent categories that the architecture is sometimes mistaken for.

**Wanabai.** A self-contained, autonomous production system that accepts requests, produces software artifacts, verifies them, deploys them, observes their outcomes, and returns results, without human operators in the main production loop.

**Request.** A structured or natural-language description of software to be produced, including intent, constraints, target environment, consumer identity, and trust tier.

**Artifact.** A deterministic software component produced by Wanabai in response to a request. Artifacts may be ephemeral (executed once and discarded) or permanent (maintained and served continuously). Both kinds are verified before release.

**Specification corpus.** The accumulated set of formal specifications — properties, contracts, invariants, TLA+ models, type signatures — that define the constraints a produced artifact must satisfy. The corpus is the durable state of Wanabai. Artifacts are derived from it and disposable.

**Production loop.** The continuously-running process by which requests are metabolized into verified, deployed artifacts. The loop has no start, no end, and no release cycles.

**Verification suite.** The set of deterministic tools (SMT solvers, model checkers, property-testers, type checkers, runtime contract systems, adversarial agents) that check artifacts against the specification corpus and check artifact behavior against declared intent.

**Trust tier.** The attestation level required for a given artifact, determined by the scope of its effects and the identity of its consumer. Tiers range from self-attested (internal, no third-party impact) to catastrophic-exposure (safety-critical systems with insurance coverage required).

**Exception.** A state the main loop cannot automatically resolve: a spec contradiction, an outcome mismatch, or a novel situation without canonical precedent. Exceptions are routed to human handlers outside the loop.

## What Wanabai Is Not

The Definitions section above gives the positive characterization of Wanabai. This section delimits the concept negatively, against adjacent categories the architecture is most often confused with.

Wanabai is not a code generator, a language model, or an IDE plugin. These are components that may appear inside Wanabai; Wanabai itself is the integrated system in which they operate under verification and attestation.

Wanabai is not a SaaS product, a subscription service, or an application platform. It does not sell software as a licensed product; it meters production as a utility and bills consumers per unit of produced capability.

Wanabai is not a human-operated development environment. Humans do not work inside the production loop. Humans appear only at four defined roles outside the loop, each with specific responsibilities and none of which participate in producing individual artifacts.

Wanabai is not an artificial general intelligence, an autonomous agent, or a goal-directed system in the decision-theoretic sense. It is an input-output system that produces verified software in response to structured requests. Its behavior is bounded by its specification corpus, its verification suite, and its configuration attestation, all of which are subject to human audit.

Wanabai is not opaque in the adversarial sense. Its interface is specified, its specification corpus is version-controlled and auditable, its verification suite is deterministic and independently runnable, its behavior is observable through its outputs and its exception logs, and its internal processes are subject to statistical quality control and constitutional audit. The opacity is operational — internal complexity that exceeds any individual's capacity to hold in mind — not political.

## The Production Loop

The production loop is the core of Wanabai. It runs continuously over the lifetime of the system and metabolizes incoming requests into verified artifacts at whatever rate the underlying compute supports. The loop has twelve stages, executed in sequence for each request.

1. **Intake.** A request arrives at Wanabai's interface. The request is parsed and validated against the protocol specification. Malformed requests are rejected with a structured error. Well-formed requests are tagged with a unique identifier, a trust tier, and a provenance record indicating the caller.

2. **Specification synthesis.** A specifier agent inside Wanabai reads the request and the accumulated specification corpus and produces a spec delta: a set of new properties, contracts, invariants, or model fragments that express what the produced artifact must satisfy. The specifier agent is stochastic; its output is deterministic text in a formal specification language.

3. **Consistency check.** The spec delta is checked against the existing corpus for contradictions. The check uses SMT solvers over the local neighborhood of the delta and runs bounded global checks against high-priority invariants. If no contradictions are found, the check passes. If contradictions are found, the loop proceeds to canonical resolution.

4. **Canonical resolution.** A hierarchy of resolution rules attempts to reconcile detected contradictions automatically. Typical rules: newer business intent overrides older technical convenience; established regulatory constraints override business intent; domain-specific precedence rules defined by the operator. If canonical resolution succeeds, the loop continues. If it fails, the spec delta is routed to an exception handler and the request is paused.

5. **Implementation synthesis.** A coder agent produces code that satisfies the updated spec corpus. The coder is stochastic. Its output is code in a target language determined by the request and Wanabai's configuration.

6. **Deterministic verification.** The produced code is checked against the spec corpus using the full verification suite: type checking, property-based tests generated from the specs, contract enforcement, model checking for architectural invariants, and formal proofs where applicable. Failures route the code back to the coder with counterexamples for regeneration. After a bounded number of retries, unresolved failures are escalated as exceptions.

7. **Sandbox deployment.** The verified artifact is deployed into an ephemeral production-equivalent environment. The sandbox enforces the artifact's declared scope: what it can read, what it can write, what network calls it can make, what resources it can consume. No artifact runs outside its declared scope; the sandbox mechanically prevents violations.

8. **Adversarial outcome testing.** Adversarial agents inside Wanabai exercise the artifact against declared intent, not just declared spec. The adversarial agents are an ensemble with deliberate diversity in their priors and strategies, to reduce the risk of shared blind spots. Observed outcomes are compared to expected outcomes defined by the request.

9. **Outcome exception handling.** If observed behavior satisfies the spec but diverges from declared intent, the divergence is classified. Minor divergences are logged and included in statistical process control. Significant divergences are routed to exception handlers for human judgment on whether the spec or the intent was incorrect.

10. **Auto-promotion.** If verification and outcome testing pass within the trust tier's requirements, the artifact is promoted to production. No human gate. The promotion is atomic: either the new artifact is serving requests or it is not, and the transition is instantaneous.

11. **Production observation.** The running artifact is monitored continuously by trace verification against the spec corpus, by drift detection against intent baselines, and by statistical sampling of a small fraction of requests for deep analysis. Observed anomalies feed back into the quality control system.

12. **Lifecycle transition.** Ephemeral artifacts are decommissioned after their declared lifespan or on request. Permanent artifacts are maintained in service until the spec corpus changes, at which point the loop regenerates them from the updated corpus. The artifact has no independent existence outside its derivation from the spec.

## The Specification Corpus

The specification corpus is the durable state of Wanabai. It contains everything Wanabai knows about what the software it produces should do. Artifacts are derived from the corpus; the corpus is not derived from artifacts. When a conflict arises between what the corpus says and what an existing artifact does, the corpus wins and the artifact is regenerated.

The corpus is stored in a version-controlled, append-mostly structure. Every change to the corpus is a deliberate act — a spec delta from a request, a canonical resolution, or an exception handler's decision — and is recorded with full provenance. The corpus is diffable, mergeable, and reviewable by humans when exception handling requires it.

The corpus is organized by scope: global invariants that apply to all artifacts produced by Wanabai, domain-specific invariants that apply to a class of artifacts, and artifact-specific properties that apply only to a single artifact. Conflict resolution rules specify the precedence order when invariants at different scopes interact.

The corpus is not a codebase. It does not contain implementation details. It contains constraints, properties, contracts, and invariants — the what, not the how. A correct implementation of the corpus can be produced by any sufficiently capable coder agent, and multiple implementations can be held simultaneously, each verified against the same corpus, each disposable. The corpus is the identity of the system. The code is the compilation artifact.

## The Verification Substrate

Verification is performed by deterministic tools operating on deterministic artifacts against a deterministic corpus. The stochasticity of the production process does not propagate into the verification process. If the verification substrate says an artifact satisfies the corpus, that statement is as reliable as the verification tools themselves, which are subject to their own attestation and audit.

The verification substrate has five layers, ordered from fastest and narrowest at the bottom to slowest and deepest at the top.

**Layer 1: Type systems.** Judgment encoded in the language itself. Checked at compile time. Narrow in scope but free at runtime.

**Layer 2: Contracts and runtime assertions.** Preconditions, postconditions, and invariants attached to each function. Checked at runtime. Travel with the code as enforcement mechanisms that catch violations in deployment.

**Layer 3: Property-based tests.** Generated from specs by the verification system. Checked by running the artifact against generated inputs and verifying that stated properties hold. The QuickCheck lineage, applied systematically at every artifact boundary.

**Layer 4: Model checking.** Full state-space exploration of architectural invariants using TLA+, Alloy, or equivalent tools. Applied to high-stakes components where design-level correctness matters more than local correctness. Expensive but exhaustive within the modeled scope.

**Layer 5: Formal proofs.** Machine-checked proofs of correctness using Dafny, Lean, Coq, or equivalent. Applied to algorithmically critical components where the cost of formal verification is justified by the stakes. The slowest layer but the strongest guarantee.

Each layer catches errors the layers below it cannot catch, at a higher cost. The production loop allocates verification effort to layers based on the trust tier of the request: low-tier requests use only layers 1-3; high-tier requests use all five. The allocation is determined by Wanabai's configuration, which is subject to attestation.

## Blast Radius Containment

Every artifact produced by Wanabai runs in a sandbox that enforces its declared scope. The sandbox is the primary safety mechanism against the residual failure modes of stochastic production. The principle is simple: even when verification passes and outcome testing passes, there is a nonzero probability that the artifact behaves incorrectly in a way that was not tested. The sandbox ensures that incorrect behavior cannot propagate beyond the artifact's declared scope.

The sandbox enforces five classes of constraints. **Read scope:** which data the artifact can read. **Write scope:** which data the artifact can write. **Network scope:** which external endpoints the artifact can contact. **Resource scope:** how much compute, memory, and time the artifact can consume. **Effect scope:** which side effects the artifact can trigger in other systems.

All five are enforced mechanically by the runtime, not by the artifact's good behavior. An artifact that attempts to exceed its scope is terminated and the violation is logged. The sandbox is itself software produced by Wanabai and verified against a higher-trust-tier configuration; its correctness is load-bearing for the entire system and is audited accordingly.

Blast radius containment is what makes unsupervised ephemeral software tolerable. Individual artifacts may be incorrect, but their incorrectness is bounded. The Statistical Quality Control system catches systemic drift across many artifacts; the sandbox bounds the harm from any individual artifact while the statistical system has time to detect the drift.

## Statistical Quality Control

The production loop produces artifacts at volumes where individual inspection is impossible. Quality is maintained through statistical sampling of the output, statistical process control over the production pipeline, and outcome feedback from deployed artifacts.

**Sampling.** A small, randomized fraction of produced artifacts is pulled into a deeper verification pipeline: more rigorous testing, more adversarial probing, more expensive formal methods. The sample size is calibrated to detect process drift at a statistical confidence level specified by Wanabai's configuration. Artifacts not in the sample are produced, verified at their tier, deployed, and observed by the standard monitoring system.

**Process control.** The sampled results are analyzed for deviations from expected quality distributions. When the defect rate, the verification failure rate, or the outcome mismatch rate crosses a threshold, the production pipeline is flagged as out of control. The flag triggers an investigation: a quality engineer examines the sampled failures, identifies the systemic cause, and either adjusts the pipeline configuration or escalates to the constitutional audit layer if the cause is in the verification criteria themselves.

**Outcome feedback.** Downstream effects of deployed artifacts are tracked against their declared intent over time. When observed outcomes diverge systematically from declared intent — even when individual artifacts passed their verification — the divergence is logged and used to adjust the specifier agent's training data, the verification criteria, or the canonical resolution rules. Outcome feedback closes the loop between what was specified and what actually mattered.

None of these mechanisms are novel. They are the mechanisms by which every high-volume production industry solved the problem of quality at scale: semiconductors, pharmaceuticals, food processing, aviation parts. Wanabai inherits the discipline.

## Self-Improvement

Wanabai produces software. One of the things it can produce is a new version of itself. This is not a metaphor; it is the mechanism by which Wanabai evolves over time.

**Bootstrap.** The first version of Wanabai is produced by human engineers using conventional tools. This is the bootstrap moment, and it is historical. After the bootstrap, subsequent versions are produced by the current version according to the process in the Production Loop section, using Wanabai's own verification suite to validate them.

**Selection.** A candidate new version is produced in response to a request for an improved box. The candidate is evaluated against a meta-verification suite that measures its performance on a benchmark of production tasks. If the candidate scores better than the current version on the benchmark, and if the candidate passes all standard verification and safety checks, it is promoted to become the current version. If it scores worse, it is discarded. The selection pressure is defined entirely by the meta-verification suite.

**Gradient climbing.** Over many iterations, successive candidates climb a gradient in whatever the meta-verification suite rewards. If the suite rewards speed, Wanabai becomes faster. If it rewards artifact quality, Wanabai produces higher-quality artifacts. If it rewards token efficiency, Wanabai consumes fewer tokens per artifact. The direction of improvement is determined by the suite, not by Wanabai itself.

**Goodhart bound.** Gradient climbing is subject to Goodhart's law: Wanabai will optimize exactly what is measured, including any unintended proxies. The defense against Goodhart drift is constitutional audit (described in the Human Oversight Layer section), in which human auditors periodically review the meta-verification suite itself to check whether it still rewards the properties the operators actually want Wanabai to have. Constitutional audit is the load-bearing mechanism that keeps self-improvement aligned with operator intent over many generations.

**Immutability of bootstrap.** The bootstrap version of Wanabai is preserved and can be re-instantiated if the current version is judged to have drifted. The bootstrap serves as a recovery point: if constitutional audit determines that the current version's improvements have been in the wrong direction, the system can be rolled back to the bootstrap and re-evolved with an adjusted meta-verification suite. This is expensive and rare, but possible.

## The Human Oversight Layer

Four human roles exist outside the production loop. Each has a specific responsibility and a specific interaction point with Wanabai. None participate in producing individual artifacts. None are part of the main loop. All are subject to professional standards, and in regulated domains, to licensure.

### Spec authors

Domain experts who author the high-level specifications that Wanabai uses to produce software for their organization. Spec authors do not write code; they write constraints, properties, and invariants in a specification language Wanabai understands. They are the source of intent for the entire production process. Their work is the irreducible human contribution that cannot be automated, because intent is a property of the domain expert's knowledge of their own operations. In the typical deployment, spec authors are employees of the organization consuming Wanabai's output, not of Wanabai's operator. The box does not replace them; it produces software from what they specify.

### Configuration attestors

Licensed professionals who sign off on the configuration of the production pipeline: which verification tools are used, which trust tier applies to which requests, which canonical resolution rules are active, which meta-verification criteria govern self-improvement. Attestors stake professional liability on the claim that the configuration is correct for the stated purpose. When Wanabai's configuration changes — because a new verification tool is added, a rule is updated, a new self-improved version is promoted — an attestor must sign the change before it takes effect in high-tier operation. Low-tier, self-attested operation does not require external attestation.

### Quality engineers

Specialists who interpret the statistical process control output (described in the Statistical Quality Control section) and judge whether the production pipeline is in control. When sampling detects a defect rate or outcome mismatch rate above threshold, the quality engineer investigates, identifies the cause, and either adjusts the pipeline or escalates to a higher layer. Quality engineers are responsible for keeping the production process within declared bounds over time. Their work is episodic: most of the time the process runs itself; when it drifts, the quality engineer is the first human responder.

### Constitutional auditors

A small number of senior professionals who audit the meta-verification suite that governs self-improvement (described in the Self-Improvement section). Their responsibility is to check whether the suite still rewards the properties the operators want Wanabai to have, or whether Goodhart drift has corrupted the optimization target. Constitutional audit is the highest-authority human role in the system. When a constitutional auditor determines that the meta-verification suite is rewarding the wrong thing, they have the authority to halt self-improvement, adjust the suite, and in extreme cases roll Wanabai back to its bootstrap and re-evolve it under revised criteria. Constitutional auditors are few in number, highly selected, and institutionally independent from the operators of Wanabai, to prevent capture by commercial incentives that might otherwise override audit judgment.

### Exceptions outside the four roles

In addition to the four standing roles, the production loop escalates exceptions to human handlers at two specific loop stages: spec contradictions that cannot be resolved by canonical rules (the canonical resolution stage of the Production Loop) and outcome mismatches that cannot be resolved by standard pipeline adjustment (the outcome exception handling stage of the Production Loop). The three-class taxonomy these escalations draw from — spec contradiction, outcome mismatch, novel situation — and the reasoning behind it are developed in the Exception-Handler Model section of [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity). Escalations are routed to domain experts or spec authors depending on the nature of the exception. Exception handling is not a standing role; it is an on-call responsibility distributed across people whose primary role is elsewhere in the system.

## The Interface

Wanabai exposes a single protocol interface. Requests enter through it; results leave through it. The interface is deterministic, versioned, and standardized.

**Request format.** Each request contains: a natural-language or structured description of the desired software; a target environment specification (language, runtime, deployment context); a trust tier (self-attested through catastrophic-exposure); a scope declaration (read, write, network, resource, effect); a caller identity and provenance record; an optional reference to an existing artifact or spec for modification rather than fresh production; and a billing reference that authorizes token consumption.

**Response format.** Each response contains: a reference to the produced artifact (or a reference to a running instance of it, if permanent); the portion of the spec corpus that applies to this artifact; the verification results by layer; the provenance record of how the artifact was produced; the token cost of production; and the trust tier at which the artifact was verified. Exceptions return a structured error indicating which stage of the loop failed and what human intervention is required.

**Callers.** The interface does not distinguish between callers. A human can call Wanabai directly. An agent operating on behalf of a human can call Wanabai. Another piece of software can call Wanabai as part of its own operation. The interface treats all callers uniformly, subject to authentication and trust tier constraints. In practice, the majority of calls in mature deployments are from agents and other software, not from humans directly.

**Idempotence and caching.** Identical requests against an unchanged corpus yield identical artifacts. The box caches artifacts and serves them from cache when possible, consuming no additional tokens for repeat requests. Cache invalidation occurs when the relevant portion of the corpus changes or when statistical quality control flags the cached artifact for regeneration.

**Composability.** The output of one request can be an input to another. A software artifact produced by Wanabai can call Wanabai recursively to produce its own dependencies, subject to the recursive scope and tier constraints. This is how complex systems are assembled: not through human integration of components, but through recursive production inside Wanabai, verified at each level of the composition.

## Token Economics

Production consumes tokens. Tokens are the unit of compute consumed by the model invocations inside Wanabai. Every request is metered by the number of tokens consumed to produce its result, and the meter is the basis for billing.

**Metering.** Each stage of the production loop consumes tokens at a rate determined by the stage's model usage: specification synthesis, implementation synthesis, verification, adversarial testing, and exception handling all draw tokens from the request's allocation. The total token cost of a request is the sum across stages. The box records consumption per request, per caller, per trust tier, and per stage, for billing and for statistical analysis.

**Pricing.** The box charges callers in tokens, at a rate set by the operator. The rate may vary by trust tier (higher-tier requests require more extensive verification and therefore more tokens), by caller priority (preferred callers may be rate-limited less aggressively), and by market conditions (token cost to the operator varies with inference infrastructure capacity). The pricing model is the same as the electrical grid: metered consumption, billed periodically, with tier-based pricing.

**Upstream cost.** The operator of Wanabai purchases tokens from an inference provider — a vertically integrated entity that owns model weights, inference-optimized hardware, and data center capacity. The upstream cost per token is the operator's primary input cost. The margin between the upstream cost and the rate charged to callers is the operator's revenue.

**Token flow.** A typical request flows tokens as follows: the caller authorizes token consumption; Wanabai consumes tokens from its operator account; the operator's account is debited against its upstream inference provider; the upstream provider consumes compute to generate the model inferences that drive the loop. The chain is metered end-to-end, and the operator's position in the chain is intermediate — adding verification, attestation, runtime, and quality control on top of commodity inference.

The market-structure implications of this flow — upstream rent concentration in refined-token majors, margin compression at the operator layer, and the second rent layer at the general-agent routing tier — are developed in the Token Economy: Where the Rent Goes section of [*Out of the Loop*](https://saturnino.substack.com/p/out-of-the-loop).

## Deployment Models

Wanabai can be operated in several configurations, each appropriate to different use cases.

**Open-source operator.** A community-maintained implementation of Wanabai, usable by any party with sufficient inference capacity and operational maturity to run it. The specification corpus, verification suite, and production loop are open and auditable. Trust in Wanabai comes from transparency and community review. Appropriate for low-tier and internal operations where the consumer is comfortable self-attesting. The open-source equivalent of electricity generated by public utilities.

**Private operator.** A proprietary implementation of Wanabai, run by a commercial entity for its own customers. The specification corpus and configuration are private; the trust model is based on the operator's reputation, contractual commitments, and attestation credentials. Appropriate for high-tier and regulated operations where customers want a single accountable party for the trust boundary. The private equivalent of electricity generated by investor-owned utilities.

**Regulated operator.** A private operator whose configuration and attestations are supervised by a regulatory body in a specific domain (healthcare, finance, aviation, critical infrastructure). The regulator reviews configuration attestations, audits statistical quality control output, and holds licensing authority over the operator. Appropriate for safety-critical and catastrophic-exposure tiers. The regulated equivalent of utility operation under public utility commission oversight.

**Embedded operator.** Wanabai operated by a non-software company for its own internal software needs. The box runs against the company's own specification corpus, authored by the company's domain experts, with self-attestation for internal use and contracted external attestation for anything crossing a trust boundary. The embedded model is how non-software companies recapture control of their own business logic.

The deployment models are not mutually exclusive. A mature software economy contains all four simultaneously, with different workloads flowing to the operator type appropriate to their trust requirements.

## License and Governance

Wanabai is released as open-source software. Its license, governance structure, and trademark policy are specified here because they are load-bearing for the architecture in previous sections — specifically for the guarantees made by attestors, the integrity of the verification suite over time, and the protection of the certification standard against capture by any single commercial operator.

### License: Apache 2.0

The reference implementation is released under the Apache License, Version 2.0. The choice is deliberate. Apache 2.0 is a permissive license that imposes no copyleft obligations on downstream users, maximizing the rate at which the architecture can spread into commercial, academic, and individual use. It includes an explicit patent grant from contributors to users and a patent retaliation clause, which matters because Wanabai touches patentable territory in verification, formal methods, compiler-like transformation, and agent orchestration. MIT and BSD licenses are not used because their silence on patents creates legal ambiguity that commercial adopters in regulated industries will not accept. Copyleft licenses (GPL, AGPL) are not used because their adoption costs exceed their structural benefits for a project whose durable moat is governance rather than code.

### Foundation stewardship

The project is held by an independent non-profit foundation rather than by any single commercial entity. The foundation owns the trademark, holds the reference verification suite, administers the certification program, and governs changes to the specification. Foundation governance provides three properties that license terms alone cannot: institutional continuity across the individuals who build the project; credible neutrality between commercial operators who might otherwise compete to capture the architecture; and a defined institutional home for the constitutional auditor role (described in the Human Oversight Layer section). The foundation is the mechanism by which the project survives its founders and resists commercial capture.

### Trademark and certification

The project's name and associated marks are held by the foundation and licensed under a published policy. Any party may run the unmodified software and refer to it by the trademarked name. Modified versions must use a different name. Only implementations certified against the reference verification suite may claim "certified" status in commercial offerings. The certification program is administered by the foundation's technical steering committee. Passing the reference suite is a necessary condition for certification; the foundation may impose additional conditions where warranted by domain-specific requirements (medical, financial, safety-critical). Certification fees fund the foundation's maintenance of the verification suite and the operation of the constitutional audit function.

This structure converts the capture problem from "capture the code" — which the Apache license cannot prevent — into "capture the certification" — which foundation governance resists effectively. Commercial implementations may fork and modify the code freely; they may not claim certification without meeting the foundation's standards. The economic model is that the code is free, the certification is paid for, and the foundation's authority to certify is what the payment is for.

### Contributor license agreement

All contributors to the reference implementation sign a contributor license agreement granting the foundation the right to relicense their contributions under future versions of the Apache License or a compatible successor license. The CLA is in place from the first external contribution. Its purpose is operational rather than commercial: it ensures the foundation can respond to future changes in the open-source licensing landscape without having to obtain permission from every historical contributor, which becomes impossible once a project has more than a small number of contributors.

### Reference verification suite ownership

The reference verification suite — the set of tests, properties, model checks, and adversarial agents that define what a correct implementation must do — is versioned, released, and signed by the foundation. Its evolution is governed by the foundation's technical steering committee with input from the constitutional auditor. The suite is released under the same Apache 2.0 license as the reference implementation, so any implementer can run it locally. What the foundation controls is not access to the suite but authority to certify that a given implementation has passed it. The suite is the technical complement to the trademark: the trademark defines who can claim compliance, and the suite defines what compliance means.

### Governance of self-improvement

The meta-verification suite that governs self-improvement (described in the Self-Improvement section) is subject to additional governance beyond the standard verification suite, because it determines the direction in which successive generations of Wanabai evolve. Changes to the meta-verification suite require approval from both the technical steering committee and the constitutional auditor. Neither can change it unilaterally. This two-key requirement is the institutional mechanism by which the project resists drift in its own optimization target over time.

### Foundation selection

The foundation should be established under the umbrella of an existing mature open-source foundation with relevant experience rather than created de novo. The Linux Foundation is the default recommendation given its scale, legitimacy, experience with infrastructure-layer projects, and established trademark and certification machinery. Alternatives include the Apache Software Foundation (strong contributor-meritocratic governance but weaker commercial-certification infrastructure) and the creation of a new specialized foundation under the Linux Foundation umbrella similar to how the Cloud Native Computing Foundation was established. The selection should be finalized before the first public release.

---

## Failure Modes

Wanabai has specific failure modes that the architecture is designed to bound but cannot eliminate. Operators should understand them explicitly.

**Shallow-spec failures.** The specifier agent produces a spec that is locally correct but does not capture the full intent of the request; the artifact satisfies the spec but not the caller's intent. This is the residual validation problem surfacing inside the loop — the general framing is developed in the Residual Validation Problem section of [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity). Wanabai's defenses are adversarial outcome testing against declared intent, outcome feedback from production, and exception escalation when mismatches are detected. Residual risk: silent failures where the shallow spec and the shallow test share a blind spot, which is why constitutional audit of the meta-verification suite is load-bearing.

**Brownfield consistency failures.** A new spec delta contradicts the accumulated corpus in ways the consistency checker cannot detect locally. The general form of this problem — and why brownfield consistency determines whether the loop closes at all — is developed in the Brownfield Problem section of [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity). Wanabai's defenses are bounded global checking against high-priority invariants, statistical sampling that catches accumulated inconsistency in production, and human exception handling when contradictions surface. Residual risk: slow drift in the corpus that accumulates over many deltas before any individual delta triggers detection.

**Verification suite drift.** The meta-verification suite that governs self-improvement rewards the wrong thing, and successive generations of Wanabai climb a gradient that does not correspond to operator intent. Defenses: constitutional audit; preservation of the bootstrap version; the ability to roll back and re-evolve under revised criteria. Residual risk: drift that occurs faster than the constitutional audit cycle detects it.

**Sandbox escape.** An artifact behaves in a way that exceeds its declared scope, and the sandbox fails to contain the excess. This is the most severe failure class because it permits propagation of errors beyond the bounded artifact. Defenses: mechanical enforcement of scope in the runtime; high-tier verification of the sandbox implementation itself; regular audit of sandbox correctness. Residual risk: zero-day bugs in the sandbox that are not detected until exploited.

**Upstream disruption.** The operator's inference provider becomes unavailable, too expensive, or produces degraded output. The box cannot produce artifacts at the required rate or quality. Defenses: multi-provider configurations; fallback to lower-tier operation; caching of frequently produced artifacts. Residual risk: correlated failures across providers that eliminate all fallbacks simultaneously.

**Economic capture.** The operator has commercial incentives to relax verification or attestation standards in ways that benefit revenue but reduce trust. Defenses: regulatory oversight for high-tier operation; reputational cost to the operator; constitutional audit of the meta-verification suite; the foundation governance structure described in the License and Governance section; open-source alternatives that discipline the commercial market. Residual risk: capture of regulators, reputational erosion that propagates slowly, and industry-wide relaxation of standards.

None of these failure modes are fully eliminable. All of them are bounded by the architecture described in the Production Loop through Human Oversight Layer sections. Operators should track them explicitly and treat the defenses as ongoing responsibilities, not one-time configuration choices.

## Conclusion

Wanabai is a production system for software that accepts requests, produces verified deterministic artifacts, deploys them into contained runtimes, observes their outcomes, and returns results, all without human operators in the main loop. Human judgment enters at four defined points outside the loop: specification authorship, configuration attestation, quality engineering, and constitutional audit. Quality at volume is maintained by process attestation, statistical sampling, blast-radius containment, and outcome feedback. Self-improvement is bounded by a meta-verification suite that is itself subject to constitutional audit. The interface is a metered protocol; the billing is per-token; the deterministic outputs and the stochastic production process are reconciled through a specification corpus that serves as the non-stochastic anchor between them.

This paper specifies what Wanabai is, how it operates, and who oversees it. It does not specify which implementation should be built first, which target language Wanabai should produce, which verification tools should be loaded, or which commercial model should govern its deployment. Those are implementation decisions that operators make based on their specific contexts. The architecture described here is the common structure that all viable implementations share, and it is offered as a blueprint for anyone building such a system. The broader economic and institutional consequences of running this system at scale — software as a utility, the dissolution of the software industry as a distinct sector, and the new professional class that forms around the attestation layer — are developed in [*Software as Electricity*](https://saturnino.substack.com/p/software-as-electricity).

---

## References

Boehm, B. (1981). *Software Engineering Economics.* Prentice Hall.

Brooks, F. P. (1975). *The Mythical Man-Month.* Addison-Wesley.

Claessen, K. & Hughes, J. (2000). QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs. *ICFP.*

Deming, W. E. (1982). *Out of the Crisis.* MIT Press.

Dijkstra, E. W. (1970). Notes on Structured Programming. *T.H.-Report 70-WSK-03.*

Fagan, M. E. (1976). Design and Code Inspections to Reduce Errors in Program Development. *IBM Systems Journal.*

Hoare, C. A. R. (1969). An Axiomatic Basis for Computer Programming. *Communications of the ACM.*

Jackson, D. (2006). *Software Abstractions: Logic, Language, and Analysis.* MIT Press.

Lamport, L. (2002). *Specifying Systems: The TLA+ Language and Tools.* Addison-Wesley.

Meyer, B. (1986). Design by Contract. *Technical Report TR-EI-12/CO.*

Naur, P. (1985). Programming as Theory Building. *Microprocessing and Microprogramming.*

Newcombe, C. et al. (2015). How Amazon Web Services Uses Formal Methods. *Communications of the ACM.*

Parnas, D. L. (1972). On the Criteria To Be Used in Decomposing Systems into Modules. *Communications of the ACM.*

Shewhart, W. A. (1931). *Economic Control of Quality of Manufactured Product.* Van Nostrand.

Turing, A. (1949). Checking a Large Routine. *Report of a Conference on High Speed Automatic Calculating Machines.*
