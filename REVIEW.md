# DeCLAI — Adversarial Peer Review (Reviewer 2 Mode)

**Subject:** Trimbitas, "DeCLAI: Decentralized Compute Credit System for Large Language Model Inference"
**Reviewed:** Sections 1–10 + Bibliography + README, repository state 2026-05-24.
**Recommendation:** **Major revision required.** In its current form the paper is not publishable: load-bearing technical claims do not survive scrutiny, Section 9 ("Experimental Results") describes experiments that were not conducted, and the bibliography contains self-admitted fabricated references.

This file is a synthesis of four parallel adversarial reviews. Three unambiguous typo fixes have already been applied in-line (`section_10` Dr. Asante, `Section_9` "an acceptable cost", `Bibliography` em-dash). All substantive changes are *proposed* only.

---

## 0. Top-line verdict

What works:
- Structure is complete (10 sections + bibliography).
- The framing — "credits, not tokens; mutual aid, not markets" — is a coherent and rhetorically attractive position.
- Prose is fluent and the document reads as a polished draft.

What does not work — in order of severity:
1. **Section 9 contains no actual experiments.** Every figure is a round band ("60–80%", "99.9% accuracy") with no methodology, hardware, dataset, code, CI, or σ. §9.9 describes specific pilots ("six-month pilot at a consortium of 15 universities," "2,847 active participants") that have no IRB, no institutional names, no preprint, no survey instrument, and no supporting artefact in the repository. §9.11 promises bootstrap CIs, Cohen's d, and "open source tools" — none appear. This must be retitled "Projected Performance" or replaced with real measurements before submission. In current form it is academic misconduct.
2. **The bibliography self-admits fabricated references.** Lines 462–467 of `Bibliography.md` explicitly label [10] (Santos et al., *Environmental Research Letters*) and [11] (Mensah et al., *Global Health Innovation*) as "Fictional but realistic citation". These appear in the IEEE list with full vol/issue/page metadata as if real, and are cited in §1.3 as evidence. They must be removed, and §1.3 ("Human Stories Behind the Statistics") must be deleted or radically reframed. [12] ("Graduate Student Computing Survey Consortium") is almost certainly also invented.
3. **The technical core does not hold up.** Theorem 4.1's "proof" is hand-waved (reduces to "DLOG exists" without specifying the commitment scheme); the threat model is internally inconsistent (33% Byzantine vs >50% honest majority used interchangeably); "Proof of Inference" is named four times and never defined; the credit-supply ODE has no equilibrium; the liquidity invariant is dimensionally wrong; the BFT pseudocode in §5.5.1 is majority voting renamed.
4. **The cryptographic claims ignore reality.** zk-SNARKs over a full LLM forward pass (§6.2.3), SMC for transformer inference (§6.3.2), and Functional Encryption on model weights (§6.4.1) are presented as deployable primitives. State-of-the-art ZKML handles ~10M-param models with minutes-to-hours of proving; MPC inference reports 10³–10⁵× slowdown and GB-scale bandwidth per query. None of this is acknowledged.
5. **Missing prior art is fatal.** Petals (Borzunov et al., ACL 2023), Hivemind (Ryabinin & Gusev, NeurIPS 2020), SWARM Parallelism (ICML 2023), DiLoCo (DeepMind 2023), Bittensor, Gensyn, Akash — none appear in the bibliography. Petals is the directly competing system and must be discussed.
6. **README marketing numbers are inflated past their source hedges.** §10 says "Our economic modeling suggests… 50–70% could reduce…"; the README presents the same numbers as fact. The "3,150% increase in computational access" is a single-persona arithmetic toy in §10.1.3, not a general impact figure.

---

## 1. Showstoppers (must fix before any submission)

### 1.1 Fabricated experiments (Section 9)
- No simulation framework named or released. The repo contains only Markdown.
- All metrics are unattributed round bands. §9.11.1 promises 95% CIs, bootstrap resampling, Cohen's d, and preregistration; not one such artefact appears in §9.
- Specific named pilots in §9.9.1–9.9.3 ("15-university consortium," "2,847 participants," "Southeast Asia pilot," "1,200+ developers") have no IRB, no consortium identifier, no preprint, no participant list. They almost certainly did not occur.
- §9.11.2: "All simulation code, analysis scripts, and experimental configurations are publicly available under open source licenses" — false; no code exists in the repository.

**Required fix:** Retitle §9 as "Projected Performance and Planned Evaluation"; replace every "**demonstrates** / **validates** / **achieves**" with "we estimate / we project / a reference implementation would target"; remove §9.9 entirely or label it as "illustrative scenarios, not deployments"; remove §9.11 or rewrite as a *future* evaluation protocol; or — preferable — actually run a small experiment (a 4-node Petals-style deployment on rented A10s would suffice to back several claims).

### 1.2 Self-admitted fabricated bibliography entries
- `Bibliography.md` lines 463–465 explicitly label refs [10] and [11] as "Fictional but realistic citation". These appear in the IEEE list as real publications and are cited as evidence.
- [12] ("Graduate Student Computing Survey Consortium, Proc. SIGCSE 2023, pp. 234–240") is not findable in SIGCSE 2023 proceedings; the "Consortium" non-author is a red flag.
- §1.3 ("Human Stories Behind the Statistics") is built on these three citations — named individuals, fabricated affiliations, fabricated evidence base.

**Required fix:** Delete [10], [11], [12]. Delete §1.3 or rewrite explicitly as "illustrative composites" with no fake citations. Remove the "Reference Context" admission appendix (or remove the citations *and* the admission together).

### 1.3 Theorem 4.1 proof is vacuous
- Sec 4.3 lines 68–76 enumerate three "attacks" and dismisses each by appeal to authority. No commitment scheme is constructed. The reduction to DLOG is asserted without a protocol. The third bullet ("alternative means cost more than the credits earned") concedes the attack is feasible and merely claims it is unprofitable — directly contradicted by §4.5's claim that `α_network` floats with demand.
- The □ on line 76 is unearned. Either supply a real construction (and a real reduction) or remove "Theorem 4.1" and present it as a conjectured property.

### 1.4 Threat model incoherent across §6
- §6.1.1 (line 15): pBFT, *f < n/3*, i.e. ≤33% Byzantine.
- §6.1.3 (line 37): "Honest Majority in Consensus … *weaker than traditional BFT* … >50% honest."
- §6.6.2 (line 224): invokes pBFT again (which requires >2/3 honest).
- §7.7.1 Corollary 7.1 invokes "50% annually" for churn.

These are *different* threat models. The paper appears to use the most convenient one in each subsection. Pick one and apply it consistently.

### 1.5 Math errors
- **Liquidity invariant (§4.4 line 101):** "Total credits in circulation never exceed 110% of trailing 30-day consumption." Credits are a *stock*; consumption is a *flow*. Dimensions don't match unless an implicit "per 30 days" is silently attached, in which case the system cannot absorb a >10% consumption spike — fatal for the elasticity claims in §5.4.
- **Equilibrium ODEs (§4.5 lines 112–114):** `dG/dt = k₁·N(t)·(1 − G/G_max)` is a logistic in G with no coupling to C. `dC/dt → ∞` as `C_available → 0`. There is no fixed point that drives G toward C. The "1,000 active participants → self-sustaining" claim cannot be derived from this system.
- **Throughput vs token-rate (§3.5 line 98 vs §5.4.2 line 132):** 50–100 req/s/1000 participants AND ≥10 tok/s/user AND 512-token generations is arithmetically inconsistent.
- **Carbon math (§7.6.2):** "1000 participants × 4 h/day ≈ 1.5 MW continuous" implies 9 kW per participant — ~20× a consumer GPU. The 500,000 tons CO₂ / 100,000 cars figure inherits this error.
- **Proposition 7.1 (line 68):** uses average cost C_i(s_i)/s_i where the relevant economic quantity is marginal cost. As stated the proposition is wrong.
- **Theorem 7.2 (§7.7.1):** "Sustainable iff median participant has positive utility" — this is the *definition* of sustainability the paper has been using, not a theorem. Tautological.
- **Credit conservation (§7.1):** with reputation bonuses up to γ=1.5× (line 84), more credits are generated per GPU-hour than consumed at β=1. Plus decay (line 140). The bookkeeping does not close.
- **Maria's 3,150% (§10.1.3):** 84 credits × 1.5 × 52 weeks = 6,552 conflates "weekly access" with "annual hours" by multiplying. Assumes 100% usage every week.

### 1.6 Crypto claims that ignore cost
- **§6.2.3 ZK proof of inference**: zkLLM / zkML state-of-the-art (2024) handles ~10M-param models with minutes-to-hours of proving time. A 70B forward pass is several orders of magnitude beyond. Not acknowledged.
- **§6.3.2 SMC transformer inference**: CrypTen, MPCFormer, Iron report 10³–10⁵× slowdown and GB/query bandwidth. Claimed to run on residential links under "real-time AI inference" latency. Unphysical.
- **§6.3.1 DP at ε = 0.1 with 2% quality loss**: ε = 0.1 is an extremely tight privacy budget; quality loss claims with no neighboring-dataset definition, no sensitivity bound, and no citation are not credible.
- **§6.4.1 Functional Encryption on weights**: general FE is theoretical; inner-product FE does not compose to a full transformer at any cost. Category error.
- **§6.4.1 ABE for model confidentiality**: ABE controls *decryption*, not *post-decryption exposure*. Contributors must decrypt to compute; plaintext weights end up in GPU memory regardless. Fundamental hole, never addressed unless TEEs are mandatory — and §6.1.3 line 41 lists TEEs as optional.

### 1.7 Verifiable-compute and Sybil holes
- **No defense against cache/replay attacks.** A node returning a cached answer to a popular prompt cannot be distinguished by "timing signatures" (faster hardware legitimately finishes faster).
- **No deterministic-inference discussion.** Batched cuBLAS GEMMs are non-deterministic across GPU architectures; majority voting in §5.5.1 compares bitstrings without specifying tolerance ε. Four honest GPUs of different types vote and disagree.
- **Sybil resistance is never mentioned.** Open membership + BFT + free identities = trivially defeated.
- **Hardware attestation (§6/§4.3 item 4)** assumes TPM/SGX-style remote attestation on consumer GPUs. Consumer NVIDIA/AMD cards have no such primitive.
- **Double-spend across gateways.** Sec 5.2.1 RESERVE happens at one gateway only; cross-gateway atomicity is not specified. Gossip-replicated ledgers (§8.4) are eventually consistent — exactly the wrong consistency model for a credit ledger.

### 1.8 Missing prior art
Direct competitors absent from bibliography (verified via grep across all sections):
- Petals (Borzunov et al., ACL 2023) — closest existing system. **Mandatory.**
- Hivemind (Ryabinin & Gusev, NeurIPS 2020).
- SWARM Parallelism (Ryabinin et al., ICML 2023).
- DiLoCo (Douillard et al., DeepMind 2023).
- Bittensor / TAO, Gensyn, Akash Network, Golem — token-incentivised decentralised compute (paper critiques io.net and Render without naming the more prominent examples).
- Federated learning (McMahan et al., AISTATS 2017) — at least name and differentiate.
- ZKML literature (zkLLM, Mystique, zkCNN) — required if §6.2.3 stays.

Also: the cited llm-d (ref [202]) points to a non-existent GitHub org; the real llm-d is `github.com/llm-d/llm-d` (Red Hat / IBM / Google / CoreWeave) and targets *datacenter* Kubernetes, not consumer GPU pooling — substantive misread.

### 1.9 Architectural showstoppers for consumer-GPU realism
- **5 ms intra-cluster RTT (§3.3)** requires participants <~500 km on good fibre. Residential cable/DSL routinely sees 20–50 ms within a single metro.
- **Asymmetric residential upload (10–50 Mbps).** A 70B model layer's activation transfer is MB/token/layer; over residential upload this is *seconds per token*, contradicting the 10 tok/s SLA in §5.4.2.
- **"Dedicated network paths" (§5.4.2)** over the public internet is fiction; DeCLAI does not own the network.
- **Kubernetes on residential gaming PCs (§8)** is presented without addressing NAT, dynamic IPs, ISP TOS, or thermal/electricity cost to the contributor.

These constraints are the entire ballgame for any volunteer-compute system. They must be addressed honestly — likely by restricting the design to small models or to lab-network contributors, not pretending residential 70B inference works.

---

## 2. Citation problems (selected)

| Cite | Claim | Problem |
|---|---|---|
| [4] NVIDIA H100 white paper | "market price exceeding $25,000" | White paper does not state retail price |
| [5] Chowdhery PaLM (540B) | "7B model costs $50–100k to train" | PaLM paper has no 7B cost; numbers anyway don't reconcile with the cited 1,000–2,000 GPU-hours |
| [6] Chen et al. Codex (HumanEval) | "$50–100k monthly inference" | Codex paper has no deployment-cost figures |
| [7] CRA "Academic Computing Infrastructure Survey, 2023" | "$50–200k dept budgets" | Report does not exist under that title; likely fabricated |
| [9] AWS P4d page | "prohibitively expensive for most users" | AWS's own product page does not characterise itself as prohibitive |
| [10] Santos et al. *Environmental Research Letters* | Maria Santos vignette | **Self-admitted fictional (Bibliography line 464)** |
| [11] Mensah et al. *Global Health Innovation* | Ghana medical research vignette | **Self-admitted fictional. Journal does not exist.** |
| [12] Graduate Student Computing Survey Consortium | "Surveys indicate >70%…" | Likely fabricated; no such SIGCSE paper |
| [13] Floridi AI4People | "declining participation in AI research competitions" | Floridi is an ethics framework paper; does not document this |
| [14] Bommasani foundation models | "shifted toward incremental improvements" | Foundation-models paper does not make this claim |
| [18] Narayanan Megatron-LM | "consumer hardware matches centralised systems" | Megatron-LM uses thousands of A100s; directly contradicts the claim |
| [20] BOINC (Anderson 2004) | SETI@home anecdote | Wrong reference; should be [196] / [60] |
| [202] llm-d at distributedllm/llm-d | "GPU sharing across institutions" | URL is wrong; project is mischaracterised |
| [183] NSF POSE solicitation 21-572 | "open-source sustainability funding" | Real POSE solicitations are 22-572 / 23-556 |
| [185] Andrew Ng *Machine Learning Yearning* | — | URL wrong |
| [206] Titmuss *Gift Relationship* | "voluntary systems achieve higher donation rates" | Titmuss argued quality/equity, not higher rates — overclaim against the source |

**Bibliography hygiene:** duplicates ([17]≈[155] Switch Transformer; [27]≈[65] PipeDream; [35]≈[61] Cohen BitTorrent; [28]≈[72] partially PBFT). JMLR pagination is consistently mis-formatted across [5], [17], [142], [152], [155], [158] — "vol. X, pp. N-N-M-N" is not real JMLR style. README claims "195+ references"; actual count is 210. The combination of duplicates + mis-formatted JMLR + fabricated entries + non-existent journals strongly suggests an LLM-generated bibliography that was never verified end-to-end.

---

## 3. Language and tone (selected — full list in agents' reports)

The register slides between marketing and academic across the paper. Specific patterns to fix:

- **"revolutionary" / "novel" / "sophisticated"** — appear 20+ times. "Sophisticated" appears 9× in §§3–5 alone. Either describe the actual mechanism or delete the adjective.
- **"ensures" / "guarantees"** — used where "is intended to" or "aims to" is honest. Audit every occurrence.
- **"democratize/democratization"** — 12+ occurrences in §§1–2. Assert the thesis once.
- **"validates" / "demonstrates" / "achieves" / "establishes"** — used throughout §9 and §10 in the indicative for things that have not been measured or built. Replace with "we project / we estimate / a reference implementation would target".
- **§1.3 "Human Stories Behind the Statistics"** — title and content are op-ed register; delete or radically reframe.
- **§5 line 256:** "democratically accessibility" → "democratic accessibility" (typo).

Suggested mass-find-and-replace candidates (do *not* apply blindly):
- `revolutionary` → (delete) or `community-shared`
- `novel <noun>` → either name the construction or delete `novel`
- `paradigm shift` → (delete)
- `comprehensive` → `layered` / (delete)
- `enterprise-grade performance guarantees` → numeric SLA or delete

---

## 4. README marketing-vs-source mismatch

| README claim | Source in paper | Mismatch |
|---|---|---|
| "3,150% increase in computational access" | §10.1.3 — single-persona toy (Maria) | Generalised from one fictional vignette |
| "40–60% reduction in health research timelines" | §10.2.3 — "Quantitative modeling indicates" (no model) | README drops the hedge |
| "30–40% reduction in AI research carbon emissions" | §10.3.3 — "could reduce" | README drops "could" |
| "50–70% reduction in AI development costs" | §10.4.3 — "Our economic modeling suggests" | README drops the hedge |
| "$50,000+ H100s" / "$50–100k training runs" | §1.2 — math doesn't close | Inherits §1.2 error |
| "Paper Status: COMPLETE — ready for publication" | §9.10.1 admits simulations cannot capture deployment | Self-contradictory |
| "195+ references" | Bibliography has 210 entries | Undercount |

Either reconcile the README to the section hedges (preferred) or supply the underlying models in §§7/10 with sensitivity analysis. Do not present unhedged percentages as headline impact figures.

---

## 5. Minor / mechanical (selection)

- File-case inconsistency: `Section_*` (1–6, 9) vs `section_*` (7, 8, 10). Mid-paper rename invalidates README links; pick one convention and update both filenames and README in the same commit.
- Code-block bug in §5.2.2 `OptimalClusterSelection`: `composite_score` referenced after the loop ends; selection variable never stored.
- §5.3.1 vs §5.5.1: pipeline-parallel (one node per layer) vs BFT majority voting (multiple nodes per computation) are mutually exclusive as written.
- §5.6.1 API example signature mismatch (`submit_inference(input_tokens, max_length=512)` vs declared `(input_data, generation_parameters)`).
- §5.5.1 break-on-quorum is iteration-order dependent.
- §4.4 ASCII diagram "Feedback Loop" arrow points the wrong way relative to the prose.
- §4.2 says network works at "500+ participants"; §4.5 says "1,000". Pick one.
- §5.4.2 says 200–800 ms p99.9 for 13B; §3.5 says 500 ms–2 s for "priority". Same tier? Reconcile.
- §8 line 50 "backward compatibility guarantees for at least two major versions" — semver explicitly *breaks* compat at major bumps. Internally contradictory.
- §8 cites [75] for "consensus ensuring consistency even during network partitions" — CAP says you cannot have both. Either citation is misapplied or claim is wrong.
- Embedded code: Rust/Python/Go snippets are illustrative pseudocode; some have type errors and undefined symbols. Either label them as pseudocode throughout or fix.

---

## 6. Suggested rewrite scope (concrete plan)

If you intend to keep this as a single monolithic paper:

1. **Demote §9.** Retitle "Projected Performance and Planned Evaluation". Remove all §9.9 pilot narratives. Move §9.11 to an appendix as "Planned Evaluation Protocol". Replace every indicative-mood claim with subjunctive.
2. **Remove fabricated citations [10], [11], [12] and §1.3.** Delete the "Reference Context" disclosure. Rewrite §1.2 with verifiable figures and properly aligned citations (LLaMA-2 paper for training cost; Hoffmann et al. Chinchilla for scaling cost; a real cloud-pricing analysis).
3. **Pick one threat model.** Either f < n/3 Byzantine (then commit to it everywhere) or honest-majority (then drop pBFT references). Define it in §6.1.1 and cite back from §3, §4, §5, §7.
4. **Either build a real Proof of Inference or remove it.** Options: (a) cite Mystique / zkLLM / Mithril and honestly state prover cost; (b) replace with redundant execution + cross-checking, which is what BOINC actually does and what the related-work section already cites; (c) replace with TEE attestation where available + redundancy elsewhere.
5. **Fix the credit equilibrium.** Replace the ODE pair with a proper supply/demand model with one coupling term; derive the stability condition; show what `α_network` must satisfy. Drop "stable at 1,000 participants" unless it falls out of the math.
6. **Add Petals, Hivemind, SWARM, Bittensor to §2** and write a real comparison table (axes: token vs credit, training vs inference, residential vs lab, BFT vs reputation, model size supported).
7. **De-buzzword §8.** Justify each piece of the stack against an actual DeCLAI requirement, or remove. Confront residential constraints (NAT, asymmetric uplink, ISP TOS) head-on.
8. **Reconcile README to section hedges.** Strip unhedged percentages or label them explicitly as "modelled, not measured".
9. **Bibliography pass:** dedupe; fix JMLR pagination; remove [10], [11], [12]; verify the remaining ~200 entries (do this with a real reference manager — Zotero, BibTeX — not by hand or LLM).

Alternatively — and possibly stronger — **split the paper into two**: (1) a vision/position paper that argues the credit-vs-token framing and the prior-art landscape (most of §§1–2, §10 reframed as "potential impact"); (2) a systems paper that picks *one* technical contribution (e.g., "credit-based BFT for volunteer LLM inference") and actually builds and measures it. The current monolith tries to be both and convinces as neither.

---

## 7. Files reviewed

- `Section_1_Introduction.md`
- `Section_2_Related_Work.md`
- `Section_3_System_Architecture.md`
- `Section_4_Credit_Mechanism_Design.md`
- `Section_5_Distributed_Inference_Protocol.md`
- `Section_6_Security_and_Privacy.md`
- `section_7_economic_analysis.md`
- `section_8_implementation_details.md`
- `Section_9_Experimental_Results.md`
- `section_10_Use_Cases.md`
- `Bibliography.md`
- `README.md`

In-line edits applied during this review:
- `Section_9_Experimental_Results.md` L117: "a acceptable" → "an acceptable"
- `section_10_Use_Cases.md` L44: "a epidemiological" → "an epidemiological"
- `Bibliography.md` L29: removed stray 0x14 control byte in "AI4People—An ethical framework"

All other recommendations are *proposed* and have not been applied.
