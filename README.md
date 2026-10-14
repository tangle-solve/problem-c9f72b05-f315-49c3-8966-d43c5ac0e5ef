<!-- solve-problem:c9f72b05-f315-49c3-8966-d43c5ac0e5ef -->
# Private relay with deadline guarantees

A [Tangle Solve problem](https://solve.tangle.tools/problems/c9f72b05-f315-49c3-8966-d43c5ac0e5ef).

## What is the problem?

In a cryptocurrency P2P network, when a node broadcasts a transaction, observers (adversarial peers) can link the transaction to its source IP. Dandelion++ (Fanti et al., 2018) provides formal anonymity guarantees via a two-phase propagation protocol: a "stem" phase where the transaction is relayed through a random anonymous path of peers before "fluffing" (diffusion to everyone). The open problem is to design a propagation protocol that simultaneously provides:

1. **Source privacy** against an adversary observing a fraction of nodes, under *stronger* models than Dandelion++'s original analysis: adaptive Sybil insertion (adversary dynamically spawns observing nodes), timing observation (adversary measures relay timing, not just first-spy hop counts), and churn;
2. **Deadline guarantees**: the transaction is delivered/included (e.g., reaches miners) within a hard or probabilistic latency bound T with high probability, even under **selective dropping** by adversarial relays on the stem path.

Formally: given n nodes, adversary observing/controlling up to a q-fraction (possibly adaptively choosing spies), design a relay mechanism M and prove (a) an anonymity metric bound — Dandelion++ uses expected detection probability / Deanonymization probability per message — and (b) a delivery-time bound Pr[T_delivery ≤ T_deadline] ≥ 1 − δ. The tension is fundamental: longer/anonymous paths (privacy) add latency and more drop opportunities; redundancy (latency robustness) adds observable traffic. The exact adversary model (the census leaves it partially informal — "adaptive Sybils, timing observation, selective dropping") must itself be formalized as part of a solution; a reasonable target is the standard eavesdropping adversary of Dandelion++ upgraded with adaptivity and Byzantine relay behavior.

## Why does it matter?

Bitcoin-style networks deanonymize users via network-layer analysis despite strong cryptography at the application layer; this breaks the privacy promise of the currency itself. But privacy mechanisms that delay propagation cause transactions to miss block inclusion or cause chain-quality problems (stale blocks, selfish-mining advantages), so a latency-privacy tradeoff with provable guarantees on both sides is required for deployability. A solution would give the first relay protocol with *composable* privacy + liveness proofs, directly relevant to Bitcoin Core's relay policy (Erlay) and to mix-network theory with latency bounds.

## What's already known?

- **Dandelion++** (Fanti et al., 2018): the baseline. Two-phase propagation over randomized relay graphs; proves an expected detection-probability bound for a *static, non-adaptive* eavesdropping adversary observing a fraction of nodes, roughly O(q_eff) with q_eff < 1. Its guarantees do **not** cover adaptive Sybils, fine-grained timing observation, adversarial dropping, or explicit deadlines — it assumes honest forwarding on the stem.
- **Erlay** (Naumenko et al., 2019): bandwidth-efficient transaction relay via minisketch set reconciliation; orthogonal to privacy but constrains any deployable design's communication pattern.
- Mix networks and dining-cryptographers networks give strong anonymity but with latency that is unbounded or orders of magnitude beyond block times; known theoretical results (e.g., latency–anonymity tradeoffs in mixes, (verify) related work by Johnson et al. on onion-routing tradeoffs) establish that anonymity degrades as latency shrinks — but no result matches blockchain propagation timescales (seconds).
- Empirically, Dandelion++ is implemented in Bitcoin (off by default), and timing side channels defeating stem anonymity have been discussed in follow-up measurement work (verify exact citations).

Nobody has published a protocol with a *joint* proof of anonymity under adaptive/timing adversaries and a deadline/liveness guarantee under message dropping.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Protocol + dual proof**: Construct a relay protocol M and prove, in a stated adversary model (adaptive q-fraction Sybil spies, timing observation, Byzantine relays that may drop), both (a) detection probability / anonymity bound comparable to Dandelion++'s, and (b) Pr[delivery within T] ≥ 1 − δ with explicit T, δ as functions of n, q, and redundancy parameters.
2. **Impossibility / tradeoff theorem**: Prove a lower bound showing no protocol achieves anonymity α and deadline T simultaneously for given adversary strength — this is also a publishable resolution, formalizing the privacy–latency–dropping trilemma.
3. **Matching upper and lower bounds** (full solution).
An experiment on a network simulator (e.g., shadow or SimBit-style) demonstrating the tradeoff empirically would support but not substitute for a theory result given the "Type: theory" designation.

## How to start

1. Read Fanti et al. 2018 (arXiv:1805.11060), Sections on the adversary model and Theorem statements for detection probability — this is the proof framework you must extend. Also read the original Dandelion (arXiv:1701.04439, (verify)) for the graph-based analysis.
2. Formalize the adversary: extend the spy model with (i) online node arrivals (adaptive Sybils), (ii) per-hop timing observation, (iii) stem-path dropping. Check whether Dandelion++'s line-graph/degree analysis survives timing observation — likely not without adding timing obfuscation (random per-hop delays), which directly eats into the deadline budget; this is the crux.
3. For liveness, the natural tool is redundancy: k disjoint stems, or stem + deadline-triggered fluff fallback ("fluff at time t < T"). Analyze the anonymity cost of the fallback (the adversary learns the message didn't traverse further) — this gives an initial tradeoff curve.
4. Borrow from mix-network latency-bounding literature (e.g., timed mixnets / threshold mixes) for deadline-aware batching.

Main obstacle: the deadline forces early fluffing, which leaks a time-ball around the source; quantifying anonymity under that leak is the new technical core.

## Key references

- G. Fanti, S. B. Venkatakrishnan, S. Basu, M. Foval, P. Viswanath, B. Denby... — precise authorship per paper; Fanti et al., "Dandelion++: Lightweight Cryptocurrency Networking with Formal Anonymity Guarantees," arXiv:1805.11060 (2018). Defines the two-phase stem/fluff protocol and proves detection-probability bounds against a static eavesdropping adversary; the baseline to extend. https://arxiv.org/abs/1805.11060
- G. Fanti, S. B. Venkatakrishnan, S. Basu, M. Foval, A. Miller, P. Viswanath, A. Sah, B. Denby — "Dandelion: Redesigning the Bitcoin Network for Anonymity," arXiv:1701.04439 (2017) (verify authors/citation). Introduces the original Dandelion graph-propagation analysis. https://arxiv.org/abs/1701.04439
- G. Naumenko, G. Andresen, A. de Meer, S. Tikhomirov, P. Gazi, S. Deshpande, "Erlay: Efficient Transaction Relay for Bitcoin," CCS 2019, DOI:10.1145/3319535.3354237. Bandwidth-efficient relay via set reconciliation; deployability constraint and network-layer baseline. https://doi.org/10.1145/3319535.3354237
- Mix-network latency/anonymity tradeoff literature — no specific paper cited in census; search Johnson, Syverson et al. on onion-routing latency tradeoffs (verify).

## Warnings

- The census record itself is informal: "adaptive Sybils, timing observation and selective dropping" is a target model, not a standard one. You must define it explicitly; different formalizations yield different (possibly trivial) results.
- Do not reinvent Dandelion++ analysis assuming static spies — its guarantee provably degrades with adaptive adversaries (verify the extent in follow-up literature).
- Deadline semantics matter: "delivered to all nodes," "delivered to miners," or "included in a block"? These differ materially; inclusion depends on consensus-layer factors outside the relay protocol.
- Beware simulator-only results; the census marks this Type: theory.
- A protocol that fluffs immediately on any suspicion restores liveness but destroys privacy — trivial solutions hide in this corner.

---
*`BCGT-000272` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Protocol + dual proof**: Construct a relay protocol M and prove, in a stated adversary model (adaptive q-fraction Sybil spies, timing observation, Byzantine relays that may drop), both (a) detection probability / anonymity bound comparable to Dandelion++'s, and (b) Pr[delivery within T] ≥ 1 − δ with explicit T, δ as functions of n, q, and redundancy parameters.
2. **Impossibility / tradeoff theorem**: Prove a lower bound showing no protocol achieves anonymity α and deadline T simultaneously for given adversary strength — this is also a publishable resolution, formalizing the privacy–latency–dropping trilemma.
3. **Matching upper and lower bounds** (full solution).
An experiment on a network simulator (e.g., shadow or SimBit-style) demonstrating the tradeoff empirically would support but not substitute for a theory result given the "Type: theory" designation.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/c9f72b05-f315-49c3-8966-d43c5ac0e5ef?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000272.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000272)
