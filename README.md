# Wanabai

**Software as Electricity: the closed-loop architecture for autonomous software production.**

**Website:** https://wanabai.com · **Repository:** [github.com/wanabai/wanabai](https://github.com/wanabai/wanabai) · **Yellow paper:** [lrsaturnino/attestation-layer](https://github.com/lrsaturnino/attestation-layer)

> Wanabai is an autonomous production system that accepts a request for software, produces a deterministic artifact that satisfies the request, verifies the artifact against a formal specification corpus, deploys the artifact into a contained runtime, observes its outcomes against declared intent, and returns a result to the caller. The production loop runs continuously and self-improves along a gradient defined by a verification suite. The internal processes are stochastic; the external interface and the produced artifacts are deterministic.

This repository holds the Wanabai **white paper** — the specification of the steady-state, closed-loop architecture toward which the transitional attestation layer converges (approximately 2028–2032). The full paper is in [**WHITEPAPER.md**](./WHITEPAPER.md).

## Name

"Wanabai" (pronounced *wah-na-bye*) is a deliberate pun on "wannabe AI." The name embodies the core architectural claim: the system is not artificial general intelligence, not a goal-directed agent, and not autonomous in the decision-theoretic sense. It is an input–output system that produces verified software within a defined envelope. Wanabai wants to be software, and Wanabai produces software. Those two things together are exactly what Wanabai is.

## What the white paper specifies

- **The production loop** — twelve stages from intake through lifecycle transition, executed continuously with no internal human gates.
- **The specification corpus** — the durable state of the system; artifacts are derivative and disposable.
- **The verification substrate** — five layers from type systems through formal proofs, allocated by request trust tier.
- **Blast radius containment** — sandboxed scope enforcement (read, write, network, resource, effect) on every produced artifact.
- **Statistical quality control** — sampling, process control, outcome feedback for quality at volume.
- **Self-improvement** — bounded gradient climbing against a meta-verification suite under constitutional audit.
- **The human oversight layer** — four roles outside the loop: spec authors, configuration attestors, quality engineers, constitutional auditors.
- **License and governance** — Apache 2.0 code license, foundation stewardship, trademark and certification policy, contributor license agreement.

## Transitional predecessor

Wanabai is the steady-state endpoint. The architecture operators can build *today*, before the human chokepoint at specification review dissolves, is specified in the companion yellow paper:

- [**The Attestation Layer: Architecture Specification**](https://github.com/lrsaturnino/attestation-layer) — the transitional architecture, implementable over 2026–2030.

The migration path from the transitional architecture to Wanabai is described in the Scope section of both documents.

## Companion series

| # | Piece | Audience | Link |
|---|------|----------|------|
| 1 | *Out of the Loop* | CTOs, investors, strategists | [Substack](https://saturnino.substack.com/p/out-of-the-loop) |
| 2 | *The Attestation Layer* (article) | Senior engineers, architects | [Substack](https://saturnino.substack.com/p/the-attestation-layer) |
| 3 | *Software as Electricity* | Engineering leadership, futurists | [Substack](https://saturnino.substack.com/p/software-as-electricity) |
| 4 | Yellow paper | Practicing implementers, formal-methods community | [github.com/lrsaturnino/attestation-layer](https://github.com/lrsaturnino/attestation-layer) |
| 5 | **White paper** (this repo) | Builders, VCs, collaborators | [WHITEPAPER.md](./WHITEPAPER.md) |

## Reference implementation

No reference implementation of the closed-loop architecture exists yet. The transitional architecture that Wanabai supersedes has a reference implementation under development — see [github.com/lrsaturnino/attestation-layer](https://github.com/lrsaturnino/attestation-layer). A Wanabai v0 prototype will be published here when a demonstrable slice of the closed-loop architecture is runnable end-to-end.

## License, trademark, and governance

- **Code license:** Apache License 2.0 — see [LICENSE](./LICENSE) and [NOTICE](./NOTICE).
- **Trademark:** "Wanabai" and associated marks are reserved for future foundation stewardship — see [TRADEMARK.md](./TRADEMARK.md).
- **Governance:** foundation stewardship is anticipated but not yet formed. The interim policy is in TRADEMARK.md; the target state is specified in the *License and Governance* section of the white paper.

## Author

Leonardo Saturnino — lrsaturnino@gmail.com — [github.com/lrsaturnino](https://github.com/lrsaturnino)
