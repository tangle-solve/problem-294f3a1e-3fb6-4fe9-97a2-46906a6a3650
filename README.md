<!-- solve-problem:294f3a1e-3fb6-4fe9-97a2-46906a6a3650 -->
# Economically robust permissionless clocks

A [Tangle Solve problem](https://solve.tangle.tools/problems/294f3a1e-3fb6-4fe9-97a2-46906a6a3650).

## What is the problem?

Design and analyze a protocol that lets participants in a permissionless network (e.g., a proof-of-work or proof-of-stake blockchain) maintain a shared notion of time — a "consensus clock" — under three combined adversarial conditions:

1. **Rational timestamp manipulation.** Participants self-report timestamps (in block headers or messages) and may lie whenever doing so is profitable. The protocol must be *incentive-compatible*: honest time-reporting should be an equilibrium (ideally a dominant strategy or an ε-equilibrium in the induced game, with deviations yielding bounded gain).
2. **Intermittent participation.** Nodes join and leave arbitrarily; the protocol cannot assume a fixed or known participant set. Formal setting: an "ad hoc" / dynamic network where in each round only some honest parties are awake, per the model of Badertscher et al. (2021).
3. **Local clock drift.** Each party's physical clock advances at rate within [1−ρ, 1+ρ] of real time; honest parties' clocks may be arbitrarily far apart in offset.

The deliverable is (a) a clock-synchronization protocol in this model, (b) a proof that it converges to and maintains synchronization within some bound Δ of real time (or of each other's clocks) despite drift and intermittent participation, (c) a game-theoretic proof that rational parties cannot profitably distort the clock beyond some bound, and (d) a characterization of the *minimal external timing assumptions* needed — i.e., what (if anything) beyond local drifting clocks is necessary for any such protocol to exist. Ambiguity to note: "robust to manipulation" can mean strategy-proofness of the reported-timestamp game, or only that attacks are unprofitable relative to costs (Yaish et al.'s stretching/squeezing framing); the stronger reading should be targeted.

## Why does it matter?

Blockchains implicitly trust timestamps for difficulty adjustment, proof-of-stake time locks, contract expirations, and fee markets; demonstrated "stretching and squeezing" attacks (Yaish et al. 2022) show rational miners can distort these for profit, degrading difficulty adjustment and enabling timestamp-dependent exploits. A provably economically robust clock would let protocols replace implicit trust in system time with an explicit, incentive-secure primitive, and a characterization of minimal timing assumptions would tell protocol designers exactly what synchrony they may assume for free. It also connects to fault-tolerant distributed systems broadly: any open-membership system needing synchronization without trusted time servers.

## What's already known?

- **Classical clock synchronization** (e.g., Srikanth & Toueg, Dolev et al.) achieves synchrony with Byzantine faults but assumes fixed participant sets, bounded drift, and synchronized startup — none of which hold permissionlessly. Lower bounds relating drift ρ, fault fraction f, and achievable precision are known in that model.
- **Badertscher et al. (2021), Dynamic Ad Hoc Clock Synchronization**: the first clock synchronization protocol for dynamic, ad hoc networks with intermittent participation and drifting clocks — but with *honest* parties only (or bounded Byzantine fraction); no incentive analysis. This is the baseline to extend.
- **Yaish et al. (2022), Blockchain Stretching & Squeezing**: shows Bitcoin's timestamp rules can be gamed by rational miners to stretch or compress apparent time, harming difficulty adjustment; quantifies profitability. This establishes that the economic attack surface is real but does not construct a robust clock.
- Gap: no published protocol combines (i) ad hoc participation, (ii) drift tolerance, and (iii) a game-theoretic equilibrium guarantee against timestamp manipulation; nor is there a lower bound on what timing assumptions are necessary. Incentive-compatible *sensing/measurement* in distributed protocols is an adjacent young literature (verify current status — this is a fast-moving area).

## What would count as solving it?

Any of the following, in increasing strength:

1. **Upper bound (protocol):** A protocol in the ad hoc + drift + rational-parties model with a proof that (a) all awake honest parties' logical clocks stay within Δ of each other (or of real time) with the stated drift and participation assumptions, and (b) any deviation strategy in the timestamp-reporting game yields at most ε extra utility, for explicitly bounded ε, Δ.
2. **Lower bound (impossibility):** A proof that no protocol with certain assumptions (e.g., purely local drifting clocks, no trusted beacon, honest majority in stake/hashrate) can simultaneously achieve bounded clock error and incentive compatibility — thereby identifying the *minimal external timing assumption* (e.g., weakly synchronized outbound message delay, a trusted delay assumption, or an unbribeable randomness/timing source).
3. **Matching construction:** A protocol whose achieved Δ and ε match the lower bound's requirements.
4. **Engineering variant:** A concrete modification of Bitcoin-style timestamp rules (tightened acceptance windows, median-based robust estimators) with empirical or formal evidence that stretching/squeezing profits vanish — this satisfies the "engineering" typing of the census record.

## How to start

1. **Read Badertscher et al. (2021) first, especially the model section** — adopt their dynamic/ad hoc execution model verbatim so your results are comparable; identify exactly where their proof uses honesty of all awake parties.
2. **Read Yaish et al. (2022) for the attack model** — formalize the miner's utility (block reward × time manipulation benefit) so "unprofitable deviation" is a precise condition.
3. **Natural approach:** treat each timestamp as a report in a mechanism-design problem; use robust statistics (median-of-medians or trimmed means over recent blocks) so a bounded fraction of manipulated reports has bounded effect, then show the marginal effect of one miner's report on the aggregate is too small to pay for the cost (or delay) of the manipulation.
4. **Main obstacle:** intermittent participation breaks the usual "average of n reports" arguments — sample sizes fluctuate and Sybil entry is free, so you'll likely need stake- or work-weighted reporting and must handle the interplay of Sybil resistance with incentive compatibility. A lower-bound attempt via a world where a coalition experiences time genuinely differently (indistinguishability argument) is a good parallel track.

## Key references

- Badertscher, C., Maurer, U., Tschudi, D., Zikas, V. (2021). "Dynamic Ad Hoc Clock Synchronization." *CRYPTO 2021*, DOI:10.1007/978-3-030-77883-5_14. — Defines an ad hoc model with intermittent participation and drifting clocks and gives the first clock synchronization protocol secure in it (against faults, not incentives); https://doi.org/10.1007/978-3-030-77883-5_14
- Yaish, A., Livshits, G., Tsabary, A., Gligor, V., Eyal, I. (2022). "Blockchain Stretching & Squeezing: Manipulating Time for Your Best Interest." *AFT 2022*, DOI:10.1145/3490486.3538250. — Demonstrates and quantifies rational timestamp-manipulation attacks on Bitcoin's difficulty adjustment; https://doi.org/10.1145/3490486.3538250
- (Background, verify relevance) Srikanth, T.K., Toueg, S. (1987). "Simulating Authenticated Broadcasts to Derive Simple Fault-Tolerant Algorithms." *Distributed Computing* — the classical internal-clock synchronization bound (precision limited by drift and Byzantine fraction), useful as the starting template for lower-bound arguments.
- (Background, verify) Pass, R., Seeman, L., shelat, a. (2017). "Analysis of the Blockchain Protocol in Asynchronous Networks." *EUROCRYPT 2017* — models blockchain consensus under partial synchrony; relevant for what timing assumptions blockchains actually need.

## Warnings

- Do not conflate the two robustness notions: Byzantine-tolerance (a fixed fraction of arbitrarily bad parties) does *not* imply incentive compatibility — a rational majority can each deviate slightly in a coordinated, profitable way that no Byzantine bound covers.
- The census record types this as "engineering," but the interesting core is the combination of a distributed-computing proof and a game-theoretic equilibrium analysis; either half alone is likely already subsumed by existing work.
- Watch out for Sybil attacks on any "median of reported timestamps" scheme — permissionless means report weights must be tied to something costly (work/stake), which couples the clock to the consensus resource.
- The "minimal external timing assumptions" question risks a trivial impossibility (with only local drifting clocks and no bounds on message delay, no clock is achievable — standard). State your network assumption (e.g., bounded-delay Δ_network after some GST time) explicitly before proving anything.
- Beware of incentive definitions that ignore collusion; a scheme safe against unilateral deviation may be stretchable by small coalitions, which is exactly Yaish et al.'s attack shape.

---
*`BCGT-000245` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Upper bound (protocol):** A protocol in the ad hoc + drift + rational-parties model with a proof that (a) all awake honest parties' logical clocks stay within Δ of each other (or of real time) with the stated drift and participation assumptions, and (b) any deviation strategy in the timestamp-reporting game yields at most ε extra utility, for explicitly bounded ε, Δ.
2. **Lower bound (impossibility):** A proof that no protocol with certain assumptions (e.g., purely local drifting clocks, no trusted beacon, honest majority in stake/hashrate) can simultaneously achieve bounded clock error and incentive compatibility — thereby identifying the *minimal external timing assumption* (e.g., weakly synchronized outbound message delay, a trusted delay assumption, or an unbribeable randomness/timing source).
3. **Matching construction:** A protocol whose achieved Δ and ε match the lower bound's requirements.
4. **Engineering variant:** A concrete modification of Bitcoin-style timestamp rules (tightened acceptance windows, median-based robust estimators) with empirical or formal evidence that stretching/squeezing profits vanish — this satisfies the "engineering" typing of the census record.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/294f3a1e-3fb6-4fe9-97a2-46906a6a3650?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000245.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000245)
