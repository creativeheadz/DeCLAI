# Known Limitations

This file lists the epistemic status of the DeCLAI paper after the 2026-05 revision. It is intended as a transparency document so that readers, reviewers, and citing authors can quickly see what the paper does and does not currently substantiate.

## What the paper *is*

A **design proposal** for a community-shared LLM-inference network with a non-monetary credit mechanism. The paper specifies:

- a three-tier system architecture (gateways, orchestrators, contributor nodes) targeted at residential GPU machines,
- a credit accounting model with explicit decay and reputation multipliers,
- a redundant-execution validation protocol with floating-point ε-tolerance for cross-architecture non-determinism,
- a Byzantine threat model (f < n/3) and an honest enumeration of which cryptographic primitives (zkML, MPC inference, FE) are research directions versus deployable primitives.

## What the paper *is not*

- **Not an implementation.** There is no source code, no benchmark harness, no measurements. Section 9 is titled "Projected Performance and Planned Evaluation Protocol" and contains projections, not experimental results. Claims of pilot deployments in earlier drafts were removed or relabelled as illustrative scenarios in the 2026-05 revision.
- **Not a proof of sustainability.** The credit system's static balance condition is stated and the bookkeeping closes, but dynamic stability of the controller (`α_network` in §4.5) under realistic demand is an open question that the paper explicitly does not claim to resolve.
- **Not a proof of incentive compatibility.** Earlier drafts asserted truth-telling about computational capacity as a dominant strategy under the credit mechanism alone. This is incorrect; participants have an obvious incentive to over-report capability. Truth-telling becomes a best response only under specific assumptions about audit sampling rates and penalty sizes (Conjecture 7.1).
- **Not a security proof for "Proof of Inference."** Earlier drafts named a "Proof of Inference" protocol four times without defining it and asserted a Theorem (4.1) whose proof reduced to "DLOG is hard". The revised §4 replaces this with a property statement based on redundant execution and challenge-response audits — mechanisms that *are* deployable — and explicitly defers the construction of a zero-knowledge proof of LLM inference to future work (zkLLM-class systems handle ~10^7 parameters with minutes-to-hours of proving time; LLM-scale zk-proofs are not yet feasible).

## Specific claims withdrawn or hedged in the 2026-05 revision

The list below is not exhaustive; the full audit trail is in [`REVIEW.md`](REVIEW.md).

### From the README (formerly headline marketing numbers)

- **"3,150% increase in computational access"** — toy multiplication of weekly credits by 52 weeks at 100% utilisation. Replaced in §10.1 with an explicit derivation, a sensitivity callout, and order-of-magnitude qualitative language.
- **"40–60% reduction in health research timelines"** — no underlying model. Hedged in §10.2.
- **"30–40% reduction in AI research carbon emissions"** — no underlying model; depends on counterfactual that is not specified. Hedged in §10.3 and §7.6.
- **"50–70% reduction in startup AI costs"** — no underlying model. Hedged in §10.4.
- **"90% of researchers lack adequate computational resources"** — could not be traced to an identifiable source. Withdrawn in §1.
- **"195+ references"** — actual count is over 220 after the §2 prior-art additions; replaced with a non-numeric Bibliography link.
- **"Paper Status: COMPLETE — ready for publication"** — replaced with "Design proposal".

### From §1 (Introduction)

- **§1.3 "Human Stories Behind the Statistics"** — built on citations [10] and [11] which the original bibliography explicitly disclosed as "fictional but realistic citations" (an academic integrity issue). The subsection has been rewritten as "Illustrative Scenarios" with explicit "composite scenarios constructed for illustration" labelling and no fabricated citations.
- **§1.2 dollar figures** — H100 unit cost, 7B training cost, and university budget figures all hedged with sources and citation discipline; the inconsistent "$50–100k to train a 7B model in 1,000–2,000 GPU-hours" claim (which only works at \$25–\$50/GPU-hour, far above market) has been corrected to reference LLaMA-2's actual ~184k A100-hours and a wider cost band.
- **§1.5** "consumer hardware achieves performance comparable to centralised high-end systems [18]" — Megatron-LM (the cited work) does *not* support this; the rewrite makes the more modest claim that pipeline-parallel inference across heterogeneous nodes is feasible at reduced throughput.

### From §2 (Related Work)

- **llm-d misattribution** — the cited GitHub URL (`distributedllm/llm-d`) was wrong; the real llm-d (`llm-d/llm-d`, Red Hat / IBM / Google / CoreWeave) targets *datacenter Kubernetes inference*, not consumer-GPU volunteer pooling. Corrected.
- **Missing direct prior art** — the original Related Work did not discuss Petals, Hivemind, SWARM Parallelism, DiLoCo, Bittensor, Gensyn, or Akash. The revised §2 adds a "Decentralized LLM Systems" subsection and a comparison table; bibliography entries [211]–[218] cover the additions.
- **Titmuss overclaim** — the prose previously asserted that voluntary blood-donation systems achieve *higher donation rates*. Titmuss's argument was about quality and equity, not donation rate; the prose has been corrected.

### From §3 (System Architecture)

- **"5 ms intra-cluster RTT"** — realistic for short-range fibre, not for residential cable/DSL within a metro (typically 20–50 ms). Replaced with the realistic range.
- **"50–100 inference requests per second per thousand active participants"** and **"≥10 tokens per second per user"** — jointly inconsistent for 70B models at 512-token generations. The throughput numbers have been removed pending measurement.
- **"70% of peak performance even when 30% of network participants become unavailable"** — no model, no simulation. Removed.
- **"Throughput characteristics scale superlinearly with network size"** — this would be extraordinary and is not supported. Removed.
- **"Credit generation and consumption naturally balance as network size approaches infinity"** — overclaim. Replaced with a discussion of the rate controller's role.

### From §4 (Credit Mechanism Design)

- **Theorem 4.1** — proof was vacuous. Replaced with **Property 4.1 (informal)** based on redundant execution and challenge-response audits, with explicit acknowledgement that a rigorous statement requires assumptions on cluster composition, cost model, and audit sampling rate that are left to future work.
- **Liquidity invariant** (§4.4) — was dimensionally inconsistent (stock vs flow). Rewritten with explicit integration over the trailing window and labelled as a design target rather than a guarantee.
- **Equilibrium ODE system** (§4.5) — had no fixed point coupling G and C. Replaced with a single bookkeeping ODE plus an explicit rate-controller variable `α_network`, a static balance condition, and explicit acknowledgement that dynamic stability is an open question.
- **"Networks exceeding 1,000 active participants achieve stable, self-sustaining operation"** — derived from nothing. Removed.

### From §6 (Security and Privacy)

- **Inconsistent threat model** — §6.1.1 (f < n/3 Byzantine), §6.1.3 (>50% honest majority), §6.6.2 (pBFT again) used different assumptions in adjacent subsections. Unified to f < n/3 with explicit Threat-Model Assumptions block.
- **zk-SNARKs for LLM inference** — presented as deployable. Demoted to a "Future Verification Layer" with explicit acknowledgement that current state-of-the-art ZKML handles ~10^7-parameter models with minutes-to-hours of proving time, and that the deployed mechanism is redundant execution with ε-tolerance.
- **Differential privacy at ε = 0.1 with <2% quality loss** — fabricated precision. Removed; DP scoped to telemetry/reputation only.
- **MPC transformer inference at residential latency** — presented as deployable. Demoted to "Research Direction" with explicit 10^3–10^5× slowdown and GB/query bandwidth disclosure.
- **Functional Encryption / ABE for model confidentiality** — does not solve the "decrypt-to-compute" problem (contributors must decrypt to compute, so weights end up in GPU memory). Now stated explicitly; confidentiality framed as partial.

### From §7 (Economic Analysis)

- **η formula** — was a product of FLOPS and Memory (awkward units). Replaced with `min(FLOPS_ratio, Memory_ratio)` and an explicit "self-declared, requires validation" caveat.
- **Conservation equation** — did not account for reputation multipliers or credit decay. Corrected.
- **Theorem 7.1 (Incentive Compatibility)** — sketched proof was wrong; truth-telling is not a dominant strategy under the credit mechanism alone. Demoted to **Conjecture 7.1** with explicit (audit-probability, penalty) conditions.
- **Proposition 7.1** — used average cost; corrected to marginal cost.
- **Theorem 7.2 (Long-term Sustainability)** — was tautological. Restated as **Definition 7.1 (Operational Sustainability)**, an empirical property.
- **Corollary 7.1 "50% annual churn robustness"** — no derivation. Removed.
- **§7.6.2 carbon math** — was approximately 20× off (1.5 MW from 1,000 contributors implied 9 kW per participant; consumer GPUs draw 200–500 W). Corrected. The "500,000 tons CO₂ / 100,000 cars" figure was derived from the wrong base and has been replaced with a discussion of counterfactual dependence.
- **§7.7.2 recession resistance** — argument ran the wrong way (in a downturn, participants are more, not less, likely to shut down idle hardware). Corrected.

### From §9 (formerly "Experimental Results")

- **Entire section retitled** to "Projected Performance and Planned Evaluation Protocol" with a top-level disclaimer that no empirical measurements have been collected.
- **All indicative-mood claims** ("achieves", "demonstrates", "validates") replaced with subjunctive ("would target", "we project").
- **§9.9 "Real-World Deployment Validation"** — described pilots that did not happen ("15-university consortium", "2,847 participants", "Southeast Asia deployment", "$50,000–$100,000 commercial-cloud-equivalent"). Removed or relabelled as hypothetical scenarios.
- **§9.11 "Statistical Validation"** — promised bootstrap CIs, Cohen's d, preregistration that were never computed. Rewritten as planned evaluation protocol.
- **§9.12** "establishes DeCLAI as a viable and transformative approach" — softened to "outlines a viable design whose empirical validation remains future work".

### From §10 (Use Cases)

- **All headline percentages** — see "From the README" above; same numbers, same treatment.
- **Personas (Maria, Kwame Asante, Sarah Chen, etc.)** — kept as illustrative scenarios but explicitly labelled as composite, not interviewed.
- **"60% of annual research budget"** (Asante) and **"40% of seed funding"** (Chen) — kept as narrative figures inside illustrative scenarios.

### From the Bibliography

- **[10], [11], [12]** — withdrawn (fabricated illustrative citations and an unverifiable "Graduate Student Computing Survey Consortium" entry).
- **"Reference Context" appendix** entries for §1.3 — note added clarifying that the §1.3 scenarios no longer carry citations.
- **AI4People formatting** — stray 0x14 control byte removed from entry [13].
- **JMLR pagination** — flagged in Bibliography Hygiene Notes appendix; not all instances corrected pending a full re-verification pass.
- **llm-d URL** — corrected.
- **Crypto refs added** — [219]–[224] for zkLLM, Mystique, CrypTen, MPCFormer, Iron, and Gentry's FHE thesis, supporting the honest acknowledgements in §6.
- **Duplicate entries** — flagged in the hygiene notes but not consolidated, to avoid renumbering downstream in-text citations.

## What still needs work

Items that the 2026-05 revision identified but did not fully address:

1. **Reference implementation.** Nothing in the repository implements the design. A 2–4-node Petals-style proof of concept on rented A10/A100 hardware would let several §9 projections move from "projected" to "measured".
2. **Empirical participant survey.** The illustrative scenarios in §1.3 and §10 substitute for empirical evidence about what researchers actually do under GPU rationing. A small participant survey at one or two universities would replace several hedged claims with real numbers.
3. **Bibliography full re-verification.** The hygiene notes appendix flags known duplicates, JMLR pagination issues, and entries whose titles could not be re-verified. A full Zotero/BibTeX pass would tighten this further.
4. **Sybil resistance under open membership.** Acknowledged as an open problem; no full solution is offered, and the design still depends on reputation bootstrap plus optional hardware attestation. A more rigorous treatment, possibly drawing on stake-based admission or social-graph constraints, would strengthen §4 and §6.
5. **Cross-region credit ledger.** §8.4 describes a two-phase settlement protocol but does not specify it in detail. A full pseudocode and correctness sketch (including partition behaviour and double-spend prevention) would tighten §8.
6. **Quantified deterministic-inference tolerance ε.** §5.5.1 introduces ε-tolerance comparison but does not specify how ε is chosen for a given model. Empirical work on cross-GPU numerical agreement under low-temperature sampling would let this become concrete.

## Citation

If you cite this paper, please cite it as a *design proposal* (technical report), not as empirical work. The DOI on ResearchGate refers to the original 2025 preprint; the 2026-05 revision is the version in this repository.
