# DeCLAI: Decentralized Compute Credit System for Large Language Model Inference

[![DOI](https://img.shields.io/badge/DOI-10.13140%2FRG.2.2.21788.19848-blue)](https://doi.org/10.13140/RG.2.2.21788.19848)
[![Preprint](https://img.shields.io/badge/Preprint-Available-brightgreen.svg)](https://doi.org/10.13140/RG.2.2.21788.19848)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Paper Status](https://img.shields.io/badge/Paper-Design%20proposal-yellow.svg)](https://github.com/creativeheadz/DeCLAI)
[![Implementation](https://img.shields.io/badge/Implementation-None%20yet-lightgrey.svg)](https://github.com/creativeheadz/DeCLAI)

> **A design proposal for community-shared GPU sharing for LLM inference, based on a non-monetary credit mechanism.**
>
> Inspired by SETI@home, Folding@home, and the BOINC volunteer-computing tradition, rather than by tokenised compute markets.

## What this repository is

This repository contains a **design-proposal paper** for a decentralised LLM-inference network — DeCLAI. It does **not** contain an implementation, benchmark data, or measurements from a deployed network. Section 9 is a *projected performance and planned evaluation protocol*, not experimental results. Many figures in earlier drafts of this README and the paper were withdrawn or hedged in the 2026-05 revision after an adversarial peer-review pass; see [`REVIEW.md`](REVIEW.md) and [`KNOWN_LIMITATIONS.md`](KNOWN_LIMITATIONS.md) for the audit trail.

### Key design principles
- **Non-monetary**: a credit system where earned credits can only be redeemed by the same principal for inference, not traded, sold, or exchanged for fiat.
- **Volunteer-style participation**: anyone with a consumer GPU can join, subject to the real residential-network constraints discussed in §8.
- **Geographic clustering**: latency budgets shape how inference is partitioned across nodes.
- **No token, no speculation**: no cryptocurrency, no secondary market, no investor narrative.
- **Mutual aid framing**: the social model is closer to volunteer scientific computing or a "blood bank for compute" than to a market.

## What problem the paper is responding to

Frontier-scale AI research now requires GPU resources concentrated in a small number of well-funded organisations. Datacenter accelerators such as the NVIDIA H100 sell for tens of thousands of dollars on the secondary market; cloud GPU pricing puts sustained large-model training and high-throughput inference outside the reach of many academic groups. The paper does **not** claim a precise share of researchers locked out (the "90%" figure cited in earlier drafts was not backed by an identifiable source and has been withdrawn). The argument is qualitative: the structural cost of this concentration is high enough to justify exploring volunteer-style alternatives.

## What the paper proposes

A community-shared inference network in which:
- contributors run a single binary on their GPU machine and earn credits proportional to (validated) work done;
- consumers spend credits to obtain inference; credits cannot be transferred between principals;
- result validation uses redundant execution with floating-point tolerance, not zk-proofs (which are not yet deployable at LLM scale — see §6);
- security is sized for a Byzantine-fraction-below-one-third assumption with explicit acknowledgement of the open Sybil-resistance problem.

## 🏗️ System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Regional      │    │   Regional      │    │   Regional      │
│   Gateway       │◄──►│   Gateway       │◄──►│   Gateway       │
│                 │    │                 │    │                 │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
    ┌─────▼─────┐          ┌─────▼─────┐          ┌─────▼─────┐
    │  Cluster  │          │  Cluster  │          │  Cluster  │
    │Orchestrator│          │Orchestrator│          │Orchestrator│
    └─────┬─────┘          └─────┬─────┘          └─────┬─────┘
          │                      │                      │
   ┌──────▼──────┐        ┌──────▼──────┐        ┌──────▼──────┐
   │ Contributor │        │ Contributor │        │ Contributor │
   │    Nodes    │        │    Nodes    │        │    Nodes    │
   └─────────────┘        └─────────────┘        └─────────────┘
```

## Potential impact — qualitative

The paper sketches several application domains in §10 (academic research, global health surveillance, climate modelling, startup development, research in lower-resourced regions). For each, §10 describes the *mechanism* by which a community-shared inference network could broaden access. Earlier drafts of this README presented a set of headline percentages ("3,150% access increase," "40–60% timeline reduction," "30–40% carbon reduction," "50–70% cost reduction"). These figures were **withdrawn or hedged in the 2026-05 revision** because they reflected toy arithmetic or unsourced projections, not measurements. The revised §10 either shows the derivation explicitly with sensitivity callouts, or uses qualitative language ("could substantially lower," "order-of-magnitude") rather than specific percentages. We do not currently have empirical evidence to support any specific impact number.

## Paper structure

1. **[Introduction](Section_1_Introduction.md)** — the access concentration problem and the design we propose
2. **[Related Work](Section_2_Related_Work.md)** — volunteer computing (BOINC, SETI@home, Folding@home); tokenised compute markets (Bittensor, io.net, Gensyn, Akash); decentralised LLM systems (Petals, Hivemind, SWARM, DiLoCo)
3. **[System Architecture](Section_3_System_Architecture.md)** — three-tier topology and operating envelope under residential constraints
4. **[Credit Mechanism Design](Section_4_Credit_Mechanism_Design.md)** — credit accounting, validation, and an honest discussion of which properties are proven vs conjectured vs left to future work
5. **[Distributed Inference Protocol](Section_5_Distributed_Inference_Protocol.md)** — ε-tolerance redundant execution, model sharding, and validation
6. **[Security and Privacy](Section_6_Security_and_Privacy.md)** — Byzantine threat model with f < n/3; explicit acknowledgement of which cryptographic primitives are research directions vs deployable today
7. **[Economic Analysis](section_7_economic_analysis.md)** — bookkeeping, static balance, and open dynamic-stability questions
8. **[Implementation Details](section_8_implementation_details.md)** — coordination tier vs contributor tier; residential network constraints
9. **[Projected Performance and Planned Evaluation](Section_9_Experimental_Results.md)** — design projections and the evaluation that would have to be run to validate them (no measurements have been collected)
10. **[Use Cases](section_10_Use_Cases.md)** — illustrative scenarios with explicit "not measured" labelling

Supporting:
- **[Bibliography](Bibliography.md)** — IEEE-format citations with a hygiene-notes appendix listing known issues
- **[REVIEW.md](REVIEW.md)** — full adversarial peer-review findings that prompted the 2026-05 revision
- **[KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md)** — concise list of what the paper does and does not currently support

## Status

This is a **design proposal**, not a deployed system. The 2026-05 revision pulled the paper back from claims it did not support. The most useful contributions readers can make are:

- Critique of the design at the level of architecture, threat model, or economic mechanism
- Implementation work — there is currently no reference implementation
- Pointers to prior art that should be in §2 but is not
- Identification of remaining unsupported claims that the 2026-05 revision missed

If you are evaluating this paper for citation, please cite it as a design proposal (technical report) rather than as empirical work.

## Contributing

Contributions are welcome from researchers, engineers, and operators interested in volunteer-style distributed inference. The most useful contributions right now are critique of the design and pointers to relevant prior art — implementation work is not yet started.

## 📄 License

This work is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 📖 Citation

If you find this work useful in your research, please cite:

### BibTeX
```bibtex
@techreport{trimbitas2025declai,
  title={DeCLAI: Decentralized Compute Credit System for Large Language Model Inference},
  author={Trimbitas, Andrei},
  year={2025},
  month={July},
  institution={Independent Research},
  type={Technical Report},
  doi={10.13140/RG.2.2.21788.19848},
  url={https://doi.org/10.13140/RG.2.2.21788.19848},
  note={Available at: \url{https://github.com/creativeheadz/DeCLAI}}
}
```

## 🌐 Connect

- **Author**: [Andrei Trimbitas]([https://linkedin.com/in/andrei-trimbitas](https://www.linkedin.com/in/trimbitasav/))
- **GitHub**: [@creativeheadz](https://github.com/creativeheadz)
- **Email**: a.trimbitas@oldforge.tech

---

*Last substantial revision: 2026-05. The 2026-05 revision applied an adversarial peer-review pass and pulled the paper back from a number of claims it did not support; see [`REVIEW.md`](REVIEW.md) and [`KNOWN_LIMITATIONS.md`](KNOWN_LIMITATIONS.md).*
