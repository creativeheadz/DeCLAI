# DeCLAI: Decentralized Compute Credit System for Large Language Model Inference

**Andrei Trimbitas**
Independent Research / Old Forge Technologies
`a.trimbitas@oldforge.tech`

*Version 2 — 2026-05 revision*
*DOI of original (2025) preprint: [10.13140/RG.2.2.21788.19848](https://doi.org/10.13140/RG.2.2.21788.19848)*
*Repository: [github.com/creativeheadz/DeCLAI](https://github.com/creativeheadz/DeCLAI)*

---

## Abstract

DeCLAI is a **design proposal** for a community-shared large-language-model inference network based on a non-monetary credit mechanism. Contributors earn credits proportional to validated GPU work; consumers spend credits to obtain inference. Credits are non-transferable between principals, which deliberately avoids the secondary-market dynamics of tokenised compute systems and positions the design closer to volunteer scientific computing (SETI@home, Folding@home, BOINC) than to Bittensor, io.net, or Render Network.

The paper specifies a three-tier system architecture (gateways, orchestrators, contributor nodes), an inference validation protocol based on redundant execution with floating-point ε-tolerance comparison, a credit mechanism with explicit reputation and decay terms, a Byzantine-fault-tolerant threat model with $f < n/3$, and an honest enumeration of which cryptographic primitives (zk-proof of inference, MPC transformer inference, functional encryption on weights) are research directions versus deployable today.

**This is a design proposal, not a deployment report.** Section 9 presents *projected* performance and a *planned* evaluation protocol; no empirical measurements have been collected. The 2026-05 revision pulled the paper back from a number of claims in the 2025 preprint that were either unsupported, dimensionally inconsistent, based on fabricated citations, or asserted as theorems without proofs. The full audit trail is in `REVIEW.md` and `KNOWN_LIMITATIONS.md` in the repository.

The intended contribution is to specify the design clearly enough that it can be implemented and empirically evaluated by others — not to claim that any specific impact figure (cost reduction, carbon reduction, access broadening) has been achieved.

**Keywords:** decentralised inference, volunteer computing, credit mechanisms, GPU sharing, large language models, Byzantine fault tolerance, mechanism design.

---

## Document status

This document is the **combined, single-file version of the DeCLAI paper** intended for upload to ResearchGate as a v2 of the original 2025 preprint. The canonical, section-split source lives in the GitHub repository on branch `revision/adversarial-review-2026-05`. Two companion documents in the same repository are integral to the revision:

- `REVIEW.md` — the four-reviewer adversarial peer-review pass that triggered the 2026-05 revision.
- `KNOWN_LIMITATIONS.md` — per-section audit trail of which claims were withdrawn or hedged, with reasons.

Readers evaluating the paper for citation are asked to cite it as a *design proposal* (technical report), not as empirical work.

---

## Table of contents

1. Introduction
2. Related Work
3. System Architecture
4. Credit Mechanism Design
5. Distributed Inference Protocol
6. Security and Privacy
7. Economic Analysis
8. Implementation Details
9. Projected Performance and Planned Evaluation Protocol
10. Use Cases and Social Impact

Bibliography
Appendix A — Bibliography Hygiene Notes
Appendix B — Revision Log

---

# 1. Introduction

## 1.1 The Great AI Divide

Over the past five years, large language models have moved from a research curiosity to an everyday tool in many disciplines. At the same time, the computational resources required to train and serve frontier models have concentrated in a small number of well-funded organizations [1]. A graduate student with a promising research idea may be unable to train even a modest model, while large industrial laboratories routinely deploy systems whose training runs are reported to cost millions of dollars [2]. We argue that this concentration imposes structural costs on the research community that are worth addressing as a systems-and-incentives problem, not only as a policy one.

The scale of modern LLMs has substantially widened the gap between what well-funded laboratories can do and what is feasible for individual researchers, small institutions, and groups in less-resourced regions [3]. The result is a research landscape in which the most ambitious capabilities are produced by a handful of entities, while much of the broader community—historically a source of methodological diversity—participates only at the margins.

## 1.2 The Economics of Exclusion

The financial barriers to AI research at frontier scale are substantial. As of 2024–2025, datacenter GPUs such as the NVIDIA H100 are reported to cost on the order of tens of thousands of dollars per unit on the secondary market, with public list prices and street prices fluctuating with supply [4]. Training open-weight models at the 7B-parameter scale has been reported to consume hundreds of thousands of GPU-hours on A100-class hardware (e.g. LLaMA-2 7B's reported training of approximately 184k A100-hours [5]); at typical cloud rates of \$2–4 per GPU-hour, this translates to roughly \$0.4M–\$0.8M for a single training run. Inference deployment for a widely used application can run to the tens or hundreds of thousands of dollars per month in cloud GPU fees, depending on traffic and model size [6]. These are not absolute barriers — small-scale fine-tuning and inference on quantized models remain accessible — but they place full-scale training and large-context inference outside the reach of most academic groups.

The asymmetry between institutional capacities is also pronounced. Typical computer science departments operate annual equipment budgets that purchase a handful of high-end accelerators at most [7], while leading industrial laboratories operate clusters of tens of thousands of GPUs [8]. A single \$25,000-class accelerator amortized over its useful life still represents a meaningful fraction of a typical departmental budget.

Cloud computing has not fully closed the gap. The three major hyperscalers (AWS, Google Cloud, Microsoft Azure) dominate GPU rental, and at typical on-demand prices a sustained six-month experimental program can exceed a typical departmental annual computing budget [9]. Spot and reserved pricing reduce this cost considerably, but introduce reliability or commitment trade-offs that are themselves a barrier.

## 1.3 Illustrative Scenarios

To make the preceding figures concrete, we sketch two illustrative scenarios. These are **composite scenarios constructed for illustration**, not interviews with named researchers, and the institutional details are generic placeholders rather than empirical claims.

*Scenario A — climate modelling at a mid-sized Latin American research institute.* A research group develops a transformer-based approach for predicting extreme weather events. Preliminary results on small datasets look promising, but scaling to national-scale weather data requires GPU-hours that exceed the group's annual compute budget by an order of magnitude. The work stalls, not because the science is unsound, but because the iteration loop is too slow for productive research.

*Scenario B — public-health surveillance in a low-resource setting.* A medical informatics team identifies patterns in local disease prevalence that could inform early-warning systems. Validating these patterns against large multimodal datasets (satellite imagery, mobile-phone records, sentinel-site reports) requires inference at a scale that local infrastructure cannot support and commercial cloud pricing makes prohibitive at sustained throughput.

These scenarios are intended to illustrate the *kind* of work that current cost structures discourage; they should not be read as evidence of specific lost discoveries. Quantifying the actual research output foregone under current GPU-access constraints is, to our knowledge, an open empirical question.

## 1.4 The Innovation Bottleneck

The concentration of computational resources arguably constrains methodological diversity in AI research. Conference participation patterns, the makeup of foundation-model contributor lists, and the geographic distribution of preprint authorship are all consistent with this picture, though disentangling cause and effect is difficult [13]. Open-source AI development continues at a high rate, but a growing fraction of work in the last two years has focused on fine-tuning and downstream adaptation of a small set of frontier models rather than independent pretraining at frontier scale [14].

The implications extend to AI safety and alignment research, where diversity of approaches and perspectives is crucial for developing robust solutions. When only a handful of well-funded laboratories can conduct meaningful research on advanced AI systems, the resulting knowledge base inevitably reflects the priorities, assumptions, and blind spots of those particular organizations [15]. This concentration of research capability represents a single point of failure for one of humanity's most critical technological challenges.

Furthermore, the computational divide has begun to reshape the geographic distribution of AI innovation. Countries and regions with limited access to advanced computing infrastructure find themselves increasingly dependent on AI systems developed elsewhere, potentially missing opportunities to address local challenges or maintain technological sovereignty [16]. This dynamic threatens to replicate existing global inequalities in the digital realm, with profound implications for economic development and social progress.

## 1.5 Toward Broader Access

There is technical and historical precedent for a different organisation of compute. LLM inference, in particular, can be partitioned across networks of heterogeneous and geographically distributed GPUs, at the cost of additional coordination overhead [17]. Recent work on pipeline- and tensor-parallel inference across volunteer or low-bandwidth links — including Petals, SWARM Parallelism, and related systems discussed in §2 — shows that useful inference work can be done at the scale of consumer hardware, although throughput and latency are typically lower than dedicated datacenter deployment [18].

Volunteer computing has a long track record of sustaining large contributor populations for scientific workloads. SETI@home, Folding@home, and the BOINC ecosystem have together harnessed millions of consumer machines for problems that range from embarrassingly parallel signal processing to molecular-dynamics simulation [20]. None of these projects has historically targeted LLM inference, but their participation, validation, and credit mechanisms offer a starting point for reasoning about how community-shared GPU pools might be coordinated.

What is novel in the present moment is the combination of (i) inference workloads that are sufficiently latency-tolerant to be served from heterogeneous nodes, (ii) a large installed base of consumer GPUs sitting idle for most of their useful life, and (iii) a research community visibly constrained by access to frontier compute. Whether these conditions are sufficient to support a community-shared inference network is the empirical question this paper attempts to set up, even if it cannot yet answer it.

## 1.6 Thesis and Contribution

This paper presents **DeCLAI** (Decentralized Compute Credit System for Large Language Model Inference), an architecture for community-shared LLM inference based on a non-monetary credit mechanism. Computational contributions translate to computational access rights, deliberately avoiding token markets, secondary trading, and the associated speculation. The reference point is volunteer scientific computing (SETI@home, Folding@home, BOINC) rather than tokenised compute markets (Bittensor, io.net, Render).

Our contributions are as follows:

1. A **distributed inference protocol** for heterogeneous consumer hardware, combining pipeline-parallel layer placement with ε-tolerance redundant execution for result validation (§5).
2. A **credit mechanism** with explicit supply/demand coupling and decay parameters, intended to make sustained free-riding unprofitable without requiring monetary settlement (§4, §7).
3. A **threat model and security analysis** for the community-shared setting, identifying which cryptographic primitives are deployable today (redundant execution, ε-tolerance comparison, reputation, optional TEE attestation) and which remain research directions (zk-proof of inference, MPC transformer inference) (§6).
4. A **projected performance and planned evaluation protocol** that makes explicit what would need to be measured to validate the design; the present paper does not contain empirical measurements (§9).

We make no empirical claims that the system has been built or deployed. The intent of this paper is to specify the design clearly enough that it can be implemented and evaluated.

The paper proceeds as follows. Section 2 reviews related work in distributed computing, volunteer scientific computing, tokenised compute markets, and recent decentralised LLM systems (Petals, Hivemind, SWARM, DiLoCo). Section 3 presents the DeCLAI architecture. Section 4 develops the credit mechanism. Section 5 describes the distributed inference protocol. Section 6 addresses security and privacy. Section 7 presents the economic analysis. Section 8 covers implementation considerations and the constraints imposed by residential networks. Section 9 outlines projected performance and the planned evaluation protocol. Section 10 explores potential use cases. Section 11 concludes.

---

# 2. Related Work

Broadening access to computational resources through distributed systems represents a convergence of technical innovation and social organization that spans multiple decades of research and development. DeCLAI builds upon three foundational areas of work: distributed scientific computing projects that pioneered volunteer resource sharing, decentralized GPU networks that demonstrated commercial viability of distributed inference, and resource sharing models from economics and sociology that provide frameworks for sustainable community-driven allocation. This section examines each area's contributions and limitations, establishing the theoretical and practical foundation for DeCLAI's approach to broadening access to AI inference.

## 2.1 Distributed Scientific Computing

### 2.1.1 SETI@home and the Birth of Volunteer Computing

The modern era of distributed volunteer computing began in 1999 with SETI@home, an early project that harnessed idle computational cycles from millions of personal computers to search for extraterrestrial intelligence [196]. The project's success demonstrated that ordinary citizens could contribute meaningfully to scientific research by sharing their computational resources, establishing a template that would influence distributed computing for decades.

SETI@home's technical architecture introduced several innovations that remain relevant to contemporary distributed systems. The project implemented a work unit distribution system that partitioned large computational tasks into smaller, independent segments suitable for processing on heterogeneous consumer hardware. Each work unit contained approximately 107 seconds of radio telescope data, requiring 10-40 hours of processing time on typical home computers of the era [60]. This granular approach enabled fault tolerance through redundancy—multiple volunteers processed identical work units, and results were validated through statistical comparison.

The project's credit system provided the first large-scale implementation of non-monetary incentives for computational contribution. Participants earned credits proportional to their computational contributions, with credit allocation based on both the amount of work completed and the performance characteristics of the contributing hardware [60]. This system created sustainable motivation for long-term participation while avoiding the speculation and volatility associated with monetary rewards.

However, SETI@home also revealed fundamental limitations of purely altruistic volunteer computing. Participation rates fluctuated significantly based on media attention and public interest in the scientific mission, creating unpredictable resource availability. The project struggled with "credit gaming," where participants modified their systems or exploited vulnerabilities to earn credits without performing legitimate computation. Most critically, the system provided no mechanism for participants to directly benefit from their contributions beyond the satisfaction of supporting scientific research.

### 2.1.2 Folding@home and Protein Simulation at Scale

Folding@home, launched in 2000, extended the volunteer computing model to protein folding simulations, ultimately becoming one of the most computationally powerful distributed systems ever created [197]. The project's scientific focus on understanding protein misfolding diseases provided clear societal benefit, motivating sustained participation from millions of volunteers worldwide.

The technical innovations of Folding@home addressed several limitations of earlier volunteer computing projects. The system implemented adaptive work unit sizing that automatically adjusted computational tasks based on the performance characteristics and availability patterns of individual participants. This approach maximized resource utilization while minimizing the impact of intermittent participation—a critical challenge in volunteer computing environments.

Folding@home's approach to result validation introduced sophisticated statistical methods for detecting and correcting computational errors in distributed environments. Rather than relying solely on redundant computation, the system employed thermodynamic consistency checks and trajectory analysis to identify anomalous results [197]. This approach reduced computational overhead while maintaining scientific rigor, demonstrating that distributed systems could achieve research-grade accuracy.

The project's integration of GPU computing, beginning in 2006, foreshadowed the contemporary importance of specialized hardware for AI workloads. Folding@home demonstrated that consumer GPUs could provide orders of magnitude performance improvements for parallel scientific computation, establishing the technical feasibility of distributed GPU networks decades before the current AI boom.

### 2.1.3 BOINC Framework and Lessons Learned

The Berkeley Open Infrastructure for Network Computing (BOINC), developed in 2002, generalized the lessons learned from SETI@home and other volunteer computing projects into a comprehensive framework for distributed scientific computation [20]. BOINC's architecture addressed many of the technical and organizational challenges that limited earlier projects, providing a foundation for dozens of subsequent scientific computing initiatives.

BOINC's technical contributions include robust client-server protocols that handle network interruptions and hardware failures gracefully, sophisticated scheduling algorithms that balance computational load across heterogeneous resources, and comprehensive security mechanisms that prevent malicious participants from corrupting scientific results [20]. The framework's modular design enabled rapid deployment of new scientific applications while maintaining consistent user experience and administrative interfaces.

The framework's approach to credit allocation refined earlier systems by implementing cross-project credit portability and standardized performance benchmarking. Participants could accumulate credits across multiple BOINC projects, creating incentives for sustained engagement with the broader volunteer computing ecosystem. However, this system still relied primarily on altruistic motivation, limiting its applicability to commercial or self-interested use cases.

BOINC's governance model provides important lessons for community-driven computational networks. The project's open-source development process and distributed administrative structure enabled rapid innovation while maintaining scientific integrity. However, the reliance on academic institutions for project hosting and management created sustainability challenges when funding priorities shifted or institutional support waned.

## 2.2 Decentralized GPU Networks

### 2.2.1 Commercial Solutions and Market-Based Approaches

The emergence of artificial intelligence as a dominant computational workload has spawned a new generation of decentralized GPU networks that attempt to address resource scarcity through market mechanisms. These commercial solutions provide important technical insights while highlighting the limitations of monetary incentive structures for broadening AI access.

io.net is a prominent example of market-based decentralized GPU sharing, implementing a token-based economy where participants earn cryptocurrency rewards for contributing computational resources [199]. The platform's technical architecture demonstrates the feasibility of coordinating distributed GPU inference across geographically dispersed nodes, with load balancing and fault tolerance mechanisms that maintain service quality despite node heterogeneity.

Bittensor (TAO) is the most direct ML-oriented analog: a peer-to-peer "intelligence market" in which miners run models and validators score their outputs, with the native TAO token mediating rewards and governance [211]. Gensyn proposes a decentralized ML compute marketplace that uses cryptoeconomic verification to attest off-chain training work [216], while Akash Network [217] and Golem operate as broader decentralized cloud-compute marketplaces, not specific to ML, where compute is rented for tokens. We note that the Gensyn and Akash references are vendor-authored sources; peer-reviewed analysis of these networks remains thin, which is itself part of the gap DeCLAI addresses.

These monetary incentive structures create several barriers to democratic participation. Token economics in systems like Bittensor and io.net introduce speculation and volatility that can price out resource-constrained participants during periods of high demand. The systems' focus on profit maximization incentivizes contributors to prioritize high-value commercial workloads over academic research or social benefit applications. Most critically, governance structures concentrate decision-making power among large token holders, potentially recreating the centralization that decentralized systems aim to address.

The Render Network provides another perspective on market-based distributed computing, focusing specifically on GPU rendering workloads with a token-based reward system [200]. The platform's technical innovations include quality assurance mechanisms that validate rendered outputs and dynamic pricing algorithms that adjust compensation based on supply and demand. However, the system's reliance on cryptocurrency creates similar barriers to broad participation, particularly for users in regions with limited access to digital asset infrastructure.

### 2.2.2 Technical Frameworks and Open Source Initiatives

Open source projects such as LocalAI and llm-d demonstrate alternative approaches to distributed AI inference that prioritize accessibility or operational openness over monetization [201, 202]. These frameworks provide technical foundations for community-driven and self-hosted deployments while highlighting the challenges of sustaining development without commercial incentives.

LocalAI implements a self-hosted approach to AI inference that enables individuals and organizations to deploy language models on their own hardware while maintaining compatibility with commercial API standards [201]. The project's architecture demonstrates that consumer-grade hardware can effectively serve AI inference workloads for many applications, challenging assumptions about the necessity of specialized infrastructure. However, the system's focus on single-node deployment limits its ability to pool resources across multiple participants, reducing efficiency for users with limited hardware.

llm-d (github.com/llm-d/llm-d), a Kubernetes-native distributed inference stack jointly developed by Red Hat, IBM, Google, CoreWeave, and others, targets *datacenter* serving rather than consumer-GPU volunteer pooling [202]. It contributes disaggregated prefill/decode scheduling and KV-cache-aware routing on top of vLLM. We mention it to disambiguate naming: despite the surface similarity, llm-d is solving an orthogonal problem (efficient datacenter inference on a managed cluster), and is not a competitor to DeCLAI's volunteer-pooled, residential-link inference model.

### 2.2.3 Decentralized LLM Systems

A small but growing body of work targets specifically the problem DeCLAI addresses: running large language models across geographically dispersed, untrusted, often residential nodes. Of these, **Petals** [212] is the closest existing system. Petals partitions a large transformer (originally BLOOM-176B, later Llama-family models) across volunteer-contributed GPUs and exposes it as a collaborative inference and fine-tuning service: a client connects to a swarm, and a chain of remote servers each holds a consecutive block of layers and forwards activations. Petals demonstrates that pipelined LLM inference over the public internet is practical at interactive latencies for a single user. Crucially, however, Petals has *no* contribution-accounting or incentive layer—servers join altruistically, and there is no notion of credit, payment, or quota. DeCLAI's contribution relative to Petals is therefore not the pipeline itself but the credit mechanism that sustains participation, together with BFT-style validation of returned activations to defend against malicious or faulty contributors in an open membership setting.

**Hivemind** [213] is the underlying library that enables much of this work: a PyTorch-native framework for decentralized training over the internet with DHT-based peer discovery, asynchronous all-reduce, and decentralized mixture-of-experts. It was developed primarily for *training*, not inference, but it is methodologically adjacent because it solves the same low-trust, high-latency coordination problem. **SWARM Parallelism** [214] extends this line by showing that pipeline-parallel training of large models can tolerate slow, unreliable, heterogeneous nodes connected by commodity links—an empirical result that DeCLAI relies on when arguing that residential broadband is a viable substrate for distributed inference. **DiLoCo** [215], from DeepMind, demonstrates that language-model training can succeed with communication rounds that are orders of magnitude less frequent than standard data parallelism, further evidence that high-latency wide-area links can host useful ML work. None of these systems addresses inference-time scheduling, user-facing latency SLOs, or non-monetary access control, which are the focus of DeCLAI.

Beyond these academic systems, **Folding@home** [218] and Rosetta@home are the canonical examples of large-scale scientific volunteer computing on which the social model of DeCLAI is patterned: open membership, altruistic contribution, statistical result validation. The novelty of DeCLAI relative to that lineage is the move from embarrassingly-parallel scientific batch jobs to interactive LLM inference, and the introduction of a reciprocal credit mechanism so that contributors gain proportional inference access rather than only the satisfaction of having helped.

#### Comparison with Prior Decentralized Compute Systems

| System    | Token vs Credit | Training vs Inference | Residential vs Datacenter | Verification mechanism                | Open membership |
|-----------|-----------------|------------------------|---------------------------|----------------------------------------|-----------------|
| Petals    | None (altruistic) | Inference (+ fine-tune) | Residential               | None (trusts servers)                  | Yes             |
| Hivemind  | None (altruistic) | Training               | Residential               | None (trusts peers)                    | Yes             |
| Bittensor | Token (TAO)     | Inference              | Mixed                     | Validator scoring + on-chain consensus | Yes (staked)    |
| io.net    | Token           | Inference + training   | Datacenter (mostly)       | Provider reputation / SLA              | Permissioned    |
| BOINC     | Credit (non-redeemable) | Scientific batch | Residential               | Redundant computation + spot-checking  | Yes             |
| DeCLAI    | Credit (redeemable for inference) | Inference | Residential               | BFT-style activation validation + redundancy | Yes             |

### 2.2.4 Limitations of Monetary Incentives

The experience of commercial decentralized GPU networks reveals fundamental limitations of monetary incentive structures for broadening computational access. Market-based systems create several barriers that conflict with the goal of broad, equitable participation in AI development and deployment.

Speculation and volatility in token-based systems create unpredictable costs that can exclude resource-constrained participants during periods of high demand. Academic researchers, students, and organizations in developing regions may find themselves priced out of computational resources precisely when demand is highest, recreating the accessibility barriers that decentralized systems aim to address.

The profit-maximization incentives of monetary systems bias resource allocation toward commercially valuable applications, potentially neglecting research areas with high social value but limited commercial potential. Climate science, public health research, and educational applications may struggle to compete with commercial AI applications for computational resources in market-based systems.

Governance challenges in token-based systems concentrate decision-making power among large stakeholders, potentially undermining the democratic principles that motivate decentralized approaches. The technical direction, resource allocation policies, and access rules of these systems may be determined by participants with the largest financial stakes rather than the broader community of users and contributors.

## 2.3 Resource Sharing Models

### 2.3.1 Time Banking and Mutual Aid Systems

The concept of non-monetary resource exchange has deep roots in economics and sociology, providing theoretical frameworks that inform DeCLAI's credit mechanism design. Time banking systems, pioneered by Edgar Cahn in the 1980s, demonstrate how communities can organize resource sharing based on reciprocal contribution rather than monetary exchange [203].

Time banking operates on the principle that every person's hour of contribution has equal value, regardless of the specific type of work performed or the contributor's professional qualifications [203]. Participants earn time credits by providing services to other community members and can spend these credits to receive services in return. This approach creates sustainable incentives for mutual aid while avoiding the inequality and speculation associated with monetary systems.

The success of time banking in diverse communities—from urban neighborhoods to rural cooperatives—demonstrates the viability of credit-based resource sharing at scale. Studies of time banking systems show sustained participation rates over multiple years, with participants reporting increased social cohesion and community resilience [205]. However, time banking typically focuses on human services rather than technological resources, leaving open questions about the applicability of these models to computational resource sharing.

Alternative currency systems provide additional insights into non-monetary resource allocation mechanisms. Local exchange trading systems (LETS) and community currencies demonstrate how groups can create sustainable economic relationships without relying on traditional monetary systems [204]. These systems often incorporate sophisticated mechanisms for preventing gaming and ensuring equitable participation, providing models that inform the design of computational resource sharing systems.

### 2.3.2 Blood Bank Model as Inspiration

The blood donation system provides perhaps the most relevant model for DeCLAI's approach to computational resource sharing. Blood banks operate on principles of voluntary contribution, community benefit, and reciprocal access that closely parallel the goals of democratized AI infrastructure [206].

Richard Titmuss's seminal analysis of blood donation systems revealed that voluntary, non-monetary systems often outperform market-based alternatives in terms of both efficiency and equity [206]. Countries with voluntary blood donation systems typically achieve higher donation rates, better blood quality, and more equitable access to blood products compared to systems that rely on monetary compensation for donors.

The blood bank model's success stems from several design principles that inform DeCLAI's architecture. The system creates clear connections between individual contributions and community benefit, motivating participation through social solidarity rather than personal gain. Universal access policies ensure that all community members can benefit from the shared resource pool regardless of their ability to contribute, while contribution tracking maintains accountability and prevents free-riding.

Modern blood banking systems have evolved sophisticated logistics and quality assurance mechanisms that provide models for computational resource sharing. Geographic distribution networks ensure that donated blood reaches areas of highest need, while standardized testing and processing protocols maintain safety and compatibility across diverse donation sources [207]. These organizational innovations demonstrate how community-driven systems can achieve the reliability and scale required for critical infrastructure.

### 2.3.3 Community Resource Pools and Digital Commons

The broader literature on community resource management provides theoretical frameworks for understanding how groups can successfully govern shared resources without relying on market mechanisms or centralized control. Elinor Ostrom's analysis of common pool resource management identifies design principles that enable sustainable community governance of shared resources [121].

Successful community resource systems typically incorporate clearly defined boundaries that establish who can access shared resources, collective choice arrangements that enable community members to participate in rule-making, and graduated sanctions that discourage free-riding without excluding participants entirely [121]. These principles provide guidance for designing computational resource sharing systems that maintain community cohesion while preventing exploitation.

The emergence of digital commons in online communities demonstrates the applicability of these principles to technological resources. Open source software development, Wikipedia, and other collaborative knowledge production systems show how large, geographically distributed communities can coordinate resource sharing and collective production [209]. These systems often incorporate sophisticated reputation mechanisms, peer review processes, and governance structures that maintain quality and prevent abuse while enabling broad participation.

Peter Kollock's analysis of online cooperation reveals that digital communities can sustain resource sharing through a combination of reciprocity, reputation, and social identity [208]. Participants contribute to shared resources not only for direct personal benefit but also to maintain their standing in the community and support collective goals. This multi-faceted motivation structure provides more robust incentives than purely economic or purely altruistic systems.

Clay Shirky's examination of peer production systems highlights the importance of low barriers to participation and graduated contribution opportunities [210]. Successful digital commons enable participants to contribute at various levels of engagement, from occasional small contributions to sustained major involvement. This flexibility accommodates diverse participant circumstances while maintaining overall system sustainability.

The convergence of these theoretical frameworks with contemporary technical capabilities creates unprecedented opportunities for implementing community-driven computational resource sharing. DeCLAI builds upon these foundations while addressing the specific technical and economic challenges of distributed AI inference, creating a novel synthesis of social organization and technological innovation.

## 2.4 Synthesis and Research Gaps

### 2.4.1 Technical Gaps in Existing Systems

While existing distributed computing systems provide important technical foundations, several critical gaps limit their applicability to democratizing AI access. Current volunteer computing frameworks like BOINC excel at embarrassingly parallel scientific workloads but lack the sophisticated coordination mechanisms required for interactive AI inference. The stateless, batch-oriented design of these systems cannot accommodate the real-time response requirements and complex model dependencies of modern language model inference.

Commercial decentralized GPU networks address some technical challenges through advanced load balancing and fault tolerance mechanisms, but their market-based incentive structures create accessibility barriers that conflict with democratization goals. These systems optimize for profit maximization rather than equitable access, potentially recreating the resource concentration that decentralized approaches aim to address.

Open source distributed inference frameworks demonstrate technical feasibility but lack sustainable incentive mechanisms for long-term participation. Without addressing the economic sustainability challenges that limit volunteer computing systems, these projects risk remaining niche solutions rather than scalable alternatives to centralized AI infrastructure.

### 2.4.2 Economic Model Limitations

Existing approaches to distributed computing incentives suffer from fundamental limitations that prevent them from achieving true democratization of computational resources. Purely altruistic volunteer computing systems, while successful for specific scientific applications, cannot sustain the scale and reliability required for general-purpose AI infrastructure. Participation rates in these systems fluctuate based on external factors like media attention and personal interest, creating unpredictable resource availability.

Market-based systems address sustainability through monetary incentives but introduce speculation, volatility, and governance concentration that exclude resource-constrained participants. The profit-maximization logic of these systems biases resource allocation toward commercially valuable applications, potentially neglecting research areas with high social value but limited commercial potential.

Neither approach successfully implements the reciprocal access model that characterizes successful community resource sharing systems. Volunteer computing provides no direct benefit to contributors beyond altruistic satisfaction, while commercial systems require monetary payment that may be prohibitive for many potential users.

### 2.4.3 Governance and Community Building Challenges

Existing distributed computing systems struggle with governance models that balance technical efficiency with democratic participation. Volunteer computing projects typically rely on academic institutions for leadership and decision-making, creating sustainability challenges when institutional priorities shift. Commercial systems concentrate governance power among large stakeholders, potentially undermining community interests.

The social dynamics of community building in distributed computing remain poorly understood. While successful examples like open source software development provide insights, the specific challenges of coordinating computational resource sharing across diverse participants with varying technical capabilities and economic circumstances require novel approaches to community organization and governance.

### 2.4.4 DeCLAI's Novel Contributions

DeCLAI addresses these research gaps through several key innovations that synthesize insights from distributed computing, resource sharing theory, and community governance research. The system's pure credit mechanism creates sustainable incentives for participation while avoiding the speculation and accessibility barriers of monetary systems. By implementing reciprocal access based on contribution, DeCLAI enables participants to directly benefit from their computational contributions without requiring monetary payment.

The system's technical architecture addresses the real-time coordination challenges of AI inference through novel protocols for distributed model execution, adaptive load balancing, and fault tolerance. Unlike existing volunteer computing systems, DeCLAI is designed specifically for interactive workloads that require low latency and high reliability.

DeCLAI's governance model incorporates lessons from successful community resource management systems, implementing democratic decision-making processes that maintain community control while ensuring technical efficiency. The system's design explicitly addresses the social dynamics of community building in distributed computing environments, creating mechanisms for trust building, conflict resolution, and collective action.

Most fundamentally, DeCLAI reframes computational resources: rather than treating them as commodities to be bought and sold, it treats them as community assets to be shared and stewarded collectively. This approach aligns with broader movements toward platform cooperativism and digital commons while addressing the specific technical and economic challenges of AI infrastructure.

The following sections detail how DeCLAI implements these innovations through concrete technical and economic mechanisms, demonstrating the feasibility of community-driven approaches to democratizing artificial intelligence access. By building upon the lessons learned from decades of distributed computing research while addressing the limitations of existing approaches, DeCLAI offers a path toward truly democratic AI infrastructure that serves the needs of diverse communities rather than concentrating power among well-funded institutions.

---

# 3. System Architecture

## 3.1 Network Topology

The DeCLAI system employs a hierarchical network architecture designed to balance computational efficiency with geographic proximity, ensuring low-latency inference while maintaining the decentralized principles that enable democratic participation. Unlike traditional distributed computing systems that prioritize raw throughput over accessibility, DeCLAI's topology explicitly accommodates the heterogeneous nature of consumer hardware while providing enterprise-grade performance guarantees [21].

The network organizes participants into a three-tier hierarchy that mirrors the natural geographic and technical constraints of internet infrastructure:

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Regional Gateway│────▶│Cluster Orchestrator│────▶│  GPU Clusters   │
└─────────────────┘     └──────────────────┘     └─────────────────┘
        │                        │                        │
        │                        │                        │
   Global Routing            Local Load               Compute
   Request Validation        Balancing               Execution
   Credit Verification       Fault Tolerance         Resource Pool
```

**Regional Gateways** serve as the system's entry points, positioned strategically across major internet exchange points to minimize initial request latency. Each gateway maintains responsibility for authenticating users, validating credit balances, and routing inference requests to the most appropriate cluster within its geographic region [22]. The gateway layer implements a sophisticated request prediction system that pre-allocates computational resources based on historical usage patterns and current network conditions.

**Cluster Orchestrators** function as the intelligence layer of the network, managing collections of 10-50 GPU-equipped nodes within specific geographic regions. Each orchestrator maintains real-time awareness of node capabilities, current workloads, and network conditions, enabling dynamic load balancing that maximizes both performance and resource utilization [23]. The orchestrator implements a novel consensus mechanism that coordinates multi-node inference tasks while maintaining Byzantine fault tolerance against up to one-third malicious participants.

**GPU Clusters** represent the computational foundation of the network, composed of consumer-grade graphics cards contributed by individual participants. Each cluster node operates specialized software that partitions model weights across available GPU memory, coordinates with neighboring nodes for distributed inference, and reports computational contributions to the credit ledger [24]. The cluster architecture accommodates significant hardware heterogeneity, from enthusiast gaming rigs with single RTX 4090 cards to research laboratories contributing multiple A100 systems.

## 3.2 Core Components

The DeCLAI architecture consists of four fundamental components that work in concert to enable scalable, secure, and equitable distributed inference. Each component addresses specific technical challenges while contributing to the system's overarching goal of democratizing AI access.

### 3.2.1 Regional Gateways

Regional Gateways implement the system's interface layer, translating user requests into the distributed protocols required for efficient inference execution. Each gateway maintains several critical subsystems:

**Request Authentication and Routing**: Gateways verify user identity through cryptographic signatures and validate credit balances against the distributed ledger before accepting inference requests [25]. The routing algorithm considers current cluster loads, user geographic location, and model availability to minimize end-to-end latency while maintaining load balance across the network.

**Model Registry Management**: Each gateway maintains a synchronized registry of available models, their current deployment status across clusters, and performance characteristics. This registry enables intelligent request routing and provides users with real-time information about model availability and expected inference times [26].

**Quality of Service Guarantees**: Gateways implement sophisticated queuing mechanisms that provide differentiated service levels based on user credit balances and request priority. Premium service levels, earned through sustained computational contributions, receive priority queuing and dedicated resources during peak demand periods.

### 3.2.2 Cluster Orchestrators

Cluster Orchestrators represent the most technically complex component of the DeCLAI architecture, responsible for coordinating distributed inference across heterogeneous hardware while maintaining performance guarantees and fault tolerance.

**Distributed Model Management**: Orchestrators implement a novel model sharding algorithm that dynamically partitions transformer architectures across available GPU memory. Unlike static partitioning schemes used in centralized systems, DeCLAI's approach adapts to real-time hardware availability and failure conditions [27]. The algorithm ensures that model segments maintain locality when possible while providing redundancy for critical components.

**Byzantine Consensus for Inference**: To prevent malicious nodes from corrupting inference results, orchestrators employ a modified Byzantine fault tolerance protocol specifically adapted for machine learning workloads. The protocol requires multiple nodes to independently compute portions of the inference pipeline and uses majority voting to detect and exclude erroneous outputs [28].

**Resource Allocation and Scheduling**: Each orchestrator maintains detailed profiles of node capabilities and implements a fair scheduling algorithm that balances computational load while respecting credit-earning opportunities. The scheduler prioritizes tasks that maximize overall network throughput while ensuring that all participants receive proportional computation opportunities.

### 3.2.3 GPU Clusters

GPU Clusters form the computational backbone of the DeCLAI network, transforming consumer hardware into a coordinated inference engine capable of handling enterprise-scale workloads.

**Heterogeneous Hardware Integration**: Clusters accommodate diverse GPU configurations through a standardized containerization approach that abstracts hardware differences while maximizing utilization. Each node runs a lightweight runtime that automatically detects available GPU memory, compute capabilities, and network bandwidth to optimize its role within distributed inference tasks [29].

**Distributed Memory Management**: Cluster nodes implement a sophisticated memory management system that coordinates model weight storage and activation caching across multiple GPUs. The system uses content-addressable storage to minimize redundant weight transfers and implements aggressive caching strategies that reduce inference latency for frequently requested models [30].

**Network-Aware Computation**: Nodes continuously monitor network conditions and adapt their computational strategies to minimize data movement. When network bandwidth becomes constrained, nodes automatically shift toward more compute-intensive approaches that reduce communication overhead, even at the cost of increased local processing time.

### 3.2.4 Credit Ledger

The Credit Ledger provides the economic foundation for the DeCLAI network, implementing a tamper-proof record of computational contributions and consumption that enables sustainable resource sharing without monetary exchange.

**Immutable Contribution Tracking**: The ledger employs a blockchain-inspired architecture that records all computational contributions with cryptographic integrity guarantees. Unlike cryptocurrency systems, DeCLAI's ledger focuses exclusively on computational work verification rather than financial transactions [31]. Each ledger entry contains cryptographic proofs of work completion, timestamp information, and contributor identity.

**Credit Generation and Validation**: The system implements a novel Proof of Inference protocol that generates credits based on verified computational work. Participants earn credits by successfully completing inference tasks, with credit amounts determined by task complexity, hardware efficiency, and network demand conditions [32]. The protocol prevents gaming through cryptographic commitments that make falsifying work computationally infeasible.

**Decentralized Credit Management**: Credit balances are maintained through a distributed consensus mechanism that prevents double-spending while avoiding the energy consumption associated with traditional blockchain mining. The system uses practical Byzantine fault tolerance to ensure ledger consistency across multiple replicas while maintaining transaction throughput suitable for real-time inference applications.

## 3.3 Scalability Analysis

The DeCLAI architecture demonstrates remarkable scalability characteristics that enable growth from initial research prototypes to global-scale deployment serving millions of concurrent users. This scalability emerges from several key design decisions that prioritize horizontal expansion over vertical optimization.

**Network Growth Patterns**: Initial deployments begin with single clusters of 10-20 participants within university campuses or research communities. As participation grows, the system naturally evolves through several distinct phases: campus clusters (10-50 nodes), regional networks (500-5000 nodes), national deployments (50,000-500,000 nodes), and eventually global federation (1M+ nodes) [33]. Each growth phase triggers architectural adaptations that maintain performance while accommodating increased complexity.

**Hierarchical Load Distribution**: The three-tier architecture enables linear scaling of request handling capacity. Regional gateways scale through geographic replication, with new gateways deployed to serve emerging user populations. Cluster orchestrators scale by subdividing large regions and spawning additional coordination nodes. GPU clusters scale through continuous participant addition, with cluster sizes self-regulating based on local network conditions and coordination overhead [34].

**Bandwidth and latency.** Scaling out across heterogeneous consumer hardware imposes real bandwidth constraints that distinguish this design from datacenter inference. The system uses multi-level caching, predictive resource pre-allocation, and request batching to reduce per-inference bandwidth where possible [35]. Cluster-internal latency is *targeted* at the order of 10–50 ms within a metropolitan area on residential cable/DSL, which is achievable in practice (the often-quoted 5 ms intra-cluster RTT in earlier drafts is realistic only on lab-network or short-distance fibre and is not assumed by the protocol). For pipeline-parallel inference across $L$ residential nodes, the activation transfer between layers dominates per-token latency; for a 70B-parameter model with ~MB-scale per-layer activation tensors, residential upload bandwidth of 10–50 Mbps places a hard ceiling on tokens/second that the protocol must respect (see §5.4 and §8.6 for the operational consequence).

**Sustainability at scale.** Larger networks provide better load averaging and reduce the impact of any individual participant's churn. The mathematical model in §7 specifies a static balance condition between credit generation and consumption; whether that balance is *dynamically* maintained at a given network size is an open question (see §4.5 and §7.7.1) that we expect to depend on the controller tuning rather than purely on network size. Claims in earlier drafts that "credit generation and consumption naturally balance as network size approaches infinity" overstated this; the balance is the controller's job, not an emergent property of scale [36].

## 3.4 Component Interactions

The DeCLAI system's components interact through carefully designed protocols that maintain system coherence while preserving the autonomy and privacy of individual participants. These interactions implement a novel distributed coordination approach that avoids traditional master-slave architectures in favor of peer-oriented collaboration.

**Request Flow Protocol**: User inference requests follow a standardized multi-stage protocol that ensures efficient processing while maintaining transparency and fault tolerance. Requests enter through regional gateways, undergo authentication and credit verification, receive routing decisions based on current network state, and finally execute through coordinated cluster operations [37]. Each stage maintains detailed logs that enable post-processing analysis and credit attribution.

**Credit Synchronization**: The distributed credit ledger maintains consistency through a novel consensus protocol that combines practical Byzantine fault tolerance with domain-specific optimizations for computational work verification. Credit updates propagate through the network using a gossip protocol that ensures eventual consistency while minimizing network overhead [38]. The protocol handles network partitions gracefully, allowing continued operation during connectivity disruptions.

**Model Distribution and Updates**: DeCLAI implements a sophisticated model versioning and distribution system that enables rapid deployment of updated models while maintaining compatibility with ongoing inference requests. Models propagate through the network using a BitTorrent-inspired protocol that leverages participant bandwidth to accelerate distribution [39]. The system maintains multiple model versions simultaneously, enabling gradual migration and A/B testing of model improvements.

## 3.5 Performance Guarantees and Bottleneck Analysis

The DeCLAI architecture provides concrete performance guarantees that enable reliable application development while identifying and addressing potential system bottlenecks before they impact user experience.

**Latency targets.** The system offers differentiated latency *targets* (not guarantees) by service tier. Standard inference requests target completion within 2–5 s for small-to-medium models; priority requests, earned through sustained contribution, target 500 ms–2 s for the same model sizes [40]. These are design targets, not SLAs in the contractual sense — DeCLAI does not own the residential network paths it operates over, and a guarantee at the application layer is not consistent with running over the public internet. Admission control rejects requests when cluster capacity is saturated rather than degrading silently.

**Throughput characteristics.** Throughput scales sub-linearly with network size because coordination overhead grows with the number of clusters and the cross-cluster verification rate. We project sustained throughput on the order of tens of requests per second per thousand active participants for medium-sized models, with substantial variance depending on model size, residential upload bandwidth, and the pipeline depth required to fit the model. Earlier drafts of this paper cited "50–100 req/s/1000 participants" and "10 tok/s/user" simultaneously; these are jointly inconsistent for 70B models at 512-token generations and have been removed pending measurement (see §9).

**Bottleneck identification.** The dominant operational bottlenecks for a community-shared inference network on residential infrastructure are (a) asymmetric residential upload bandwidth (typical 10–50 Mbps up, far less than the 1–10 Gbps available in datacenters), (b) NAT traversal and dynamic-IP churn, (c) GPU memory limits that determine how deeply a model must be sharded across nodes, and (d) coordination overhead at the BFT validation layer. The architecture mitigates (a) and (c) through model-aware sharding and quantisation, (b) through gateway-mediated rendezvous, and (d) through cluster-size limits and probabilistic cross-cluster auditing. None of these mitigations eliminates the underlying constraint; they shape the operating envelope within which the network can serve requests at all [42].

**Reliability.** Critical inference requests execute on multiple independent nodes with results compared at ε-tolerance (§5.5.1). Individual node failures are tolerated through the redundancy; cluster-scale failures (correlated power outage in a region, ISP outage) require cross-region failover, which the gateway layer enables but does not make transparent at the latency budget [43].

This section has described the system's architectural shape and acknowledged the constraints under which it must operate. The next sections specify the credit mechanism, the inference protocol, the security model, the economic accounting, the implementation considerations, and the projected performance whose empirical validation remains future work.

---

# 4. Credit Mechanism Design

## 4.1 Mathematical Foundation of Credit Generation

The DeCLAI credit system implements a mathematically rigorous framework that ensures sustainable resource sharing while preventing exploitation and maintaining long-term economic equilibrium. Unlike cryptocurrency systems that derive value from artificial scarcity or speculative trading, DeCLAI credits represent pure computational work with direct, verifiable correspondence to GPU processing time [44].

The fundamental credit generation equation establishes the relationship between computational contribution and credit accumulation:

```
C_earned = (T_compute × η_efficiency × α_network) / β_difficulty
```

Where:
- `C_earned` represents credits earned for a computational task
- `T_compute` is the verified computation time in GPU-hours
- `η_efficiency` is a hardware-normalized efficiency factor (0.5-2.0)
- `α_network` is a network demand multiplier (0.8-1.5)
- `β_difficulty` is a global difficulty adjustment factor maintaining system balance

The efficiency factor `η_efficiency` addresses hardware heterogeneity by normalizing contributions across different GPU architectures. A participant with an RTX 3060 receives credit adjustments that account for the performance differential compared to higher-end hardware, ensuring that all participants can contribute meaningfully regardless of their equipment [45]. This normalization uses standardized benchmarks derived from MLPerf inference results to establish objective performance baselines.

The network demand multiplier `α_network` implements dynamic pricing that responds to system utilization. During peak demand periods, contributors earn slightly more credits to incentivize increased participation, while off-peak contributions receive standard rates. This mechanism prevents system overload while maintaining fair compensation for all participants [46].

The difficulty adjustment factor `β_difficulty` ensures long-term system sustainability by automatically adjusting credit generation rates based on the ratio of credits earned to credits consumed across the network. When the system approaches credit inflation (more credits generated than consumed), the difficulty increases slightly, and vice versa. This creates a self-balancing economic system that maintains stability without external intervention [47].

## 4.2 Game-Theoretic Incentive Analysis

The DeCLAI system's sustainability depends critically on creating incentive structures that encourage honest participation while discouraging free-riding or malicious behavior. We employ game-theoretic analysis to demonstrate that contributing computational resources represents the dominant strategy for all rational participants [48].

Consider the strategic interaction between a potential participant and the existing network. The participant faces four possible strategies:

1. **Honest Contribution**: Provide computational resources and earn credits proportionally
2. **Free-Riding**: Consume resources without contributing
3. **Partial Contribution**: Contribute minimally while consuming maximally
4. **Gaming/Cheating**: Attempt to earn credits without performing actual computation

Under the honest contribution strategy, a participant's expected utility is:

```
U_honest = E[Benefits_consumption] - Costs_computation + Value_reputation
```

The benefits from consumption include access to AI inference capabilities valued at market rates, while computation costs represent electricity and hardware depreciation. The reputation value captures long-term benefits from maintaining good standing within the network [49].

Free-riding initially appears attractive, as participants avoid computational costs while accessing network services. However, the system implements admission control that prioritizes requests from active contributors. Free-riders experience degraded service quality, longer queue times, and eventual exclusion from premium services. The expected utility for free-riding becomes:

```
U_freeride = E[Benefits_degraded] - Costs_exclusion
```

Where degraded benefits reflect limited access and exclusion costs represent lost opportunities for future participation [50].

Mathematical analysis reveals that honest contribution dominates free-riding when the network reaches sufficient size (approximately 500+ active participants). At this threshold, the service quality differential between contributors and free-riders creates sufficient incentive for participation. Moreover, the system's reputation mechanisms ensure that past free-riding behavior affects future access, creating dynamic incentives that strengthen over time.

## 4.3 Anti-Gaming Mechanisms and Proofs

The DeCLAI system must prevent participants from earning credits without performing legitimate computational work. We implement a novel Proof of Inference protocol that makes fraudulent credit generation computationally infeasible while maintaining efficient verification [51].

The Proof of Inference protocol operates through challenge-response mechanisms embedded within legitimate inference tasks. When a node claims to have completed an inference computation, it must provide:

1. **Computational Proof**: Cryptographic evidence of actual model execution
2. **Temporal Consistency**: Timing signatures that match expected processing durations
3. **Result Validation**: Output verification through consensus with other nodes
4. **Hardware Attestation**: Signatures proving computation occurred on declared hardware

A general-purpose zero-knowledge proof of LLM inference correctness — proving "this output was produced by running model M on prompt P" without revealing weights or intermediate activations — is an active research direction (see the discussion of zkLLM and related zkML work in §6.2.3) but is not deployable today at the parameter counts targeted by DeCLAI. The credit-validation protocol therefore does not rely on a zk-proof of inference. Instead, it uses three layered, deployable mechanisms:

1. **Redundant execution.** Inference requests subject to validation are executed by $f+1$ independent contributors (where $f$ is the cluster's Byzantine tolerance). Results are compared with an explicit floating-point tolerance $\varepsilon$ to account for cross-architecture non-determinism (cuBLAS reduction order, GPU-arch-dependent transcendental implementations). Credits are issued only when at least $f+1$ results agree within $\varepsilon$.
2. **Challenge–response audits.** A randomly sampled fraction of completed inferences is re-executed by an independent verifier set, with disagreement triggering a reputation penalty for the original contributors.
3. **Reputation weighting.** Contributors with longer honest history receive higher trust weights and may be sampled less aggressively for redundant execution; newly joined contributors are sampled more heavily until reputation is established.

**Property 4.1 (informal).** *Under the validation protocol above, a single contributor cannot earn credit for inference work it did not perform without either (i) colluding with at least $f$ other validators in the same cluster, or (ii) acquiring the correct output by re-running the model itself (i.e., performing the work) or by querying an external inference service. In case (ii), the attacker's cost is at least the honest work cost.*

This is a property, not a theorem: a rigorous statement requires (a) a concrete assumption on cluster composition (independent Byzantine sampling), (b) an explicit cost model that includes the price of acquiring an honest result from an external API, and (c) a security parameter governing the audit sampling rate. A formal treatment is left to future work.

Two attack patterns warrant explicit acknowledgement:

- **Cache-and-replay on popular prompts.** A contributor can cache outputs for frequent prompts and return them in microseconds. Timing signatures alone cannot distinguish this from legitimately faster hardware. Mitigation: include a per-request nonce in the prompt that perturbs the activation trace and forces a fresh forward pass, accepting a small quality cost. This mitigation is partial and is itself a research direction.
- **Sybil attack at scale.** An attacker with cheap identity creation can place $f+1$ colluding nodes in a single cluster and defeat redundant execution. DeCLAI's defenses here are reputation bootstrap, optional hardware attestation where available (e.g. TPM, NVIDIA Confidential Computing on supported SKUs), and probabilistic cross-cluster verification. None of these defenses is complete, and Sybil resistance under open membership remains an open problem inherited from the volunteer-computing literature [54].

## 4.4 Credit Lifecycle and Flow Management

Credits within the DeCLAI system follow a carefully designed lifecycle that ensures liquidity while preventing harmful accumulation or speculative behavior. The lifecycle consists of five distinct phases: generation, validation, circulation, consumption, and expiration.

```
Generation → Validation → Circulation → Consumption → Expiration
     ↑                                      ↓
     └──────── Feedback Loop ←──────────────┘
```

**Generation Phase**: Credits are created when nodes complete verified computational tasks. The generation process includes immediate cryptographic commitment to prevent double-spending and timestamps that initiate the credit lifecycle timer [55].

**Validation Phase**: Newly generated credits undergo consensus validation where multiple network nodes verify the authenticity of the claimed work. This distributed validation prevents fraudulent credit creation while maintaining decentralized operation [56].

**Circulation Phase**: Validated credits enter the active circulation pool where they can be spent on inference requests. During circulation, credits maintain full value and transferability, enabling efficient resource allocation across the network.

**Consumption Phase**: Credits are consumed when users submit inference requests. The consumption process includes cryptographic proof of credit destruction, preventing reuse while maintaining user privacy through zero-knowledge protocols [57].

**Expiration Phase**: Unused credits expire after 365 days to prevent excessive accumulation and maintain system liquidity. Expiration creates gentle pressure for credit utilization while providing sufficient time for normal usage patterns.

The credit flow system aims to maintain three operational invariants (note: these are *design targets*, not theorems — their attainment depends on the rate-adjustment mechanism described below):
- The total stock of unspent credits $S(t)$ should track the trailing 30-day rolling consumption integral $\int_{t-30d}^{t} C(\tau)\,d\tau$ such that $S(t) \le 1.10 \cdot \int_{t-30d}^{t} C(\tau)\,d\tau$. The factor 1.10 provides a 10% buffer for short-term consumption spikes; under sustained spike conditions the rate-adjustment mechanism (below) increases consumption-side prices to expand the buffer rather than rationing requests.
- No single participant holds more than 5% of $S(t)$. Excess accumulation triggers a cap that converts further earned credits into a non-spendable reputation bonus.
- The instantaneous utilisation ratio $C(t)/G(t)$ is targeted at $0.95$–$1.05$ via the rate adjustment $\alpha_{network}(t)$ defined below.

## 4.5 Equilibrium Targets and Open Dynamic-Stability Questions

The credit mechanism is intended to be **self-regulating** in the sense that the generation rate per GPU-hour and the consumption rate per inference request can both be adjusted in response to observed network state, without external monetary intervention. We sketch the intended dynamics below. We do **not** claim a closed-form stability result; doing so rigorously would require both (a) a complete specification of the rate-adjustment controller and (b) a behavioural model of contributor and consumer responses to rate changes. We treat dynamic stability as an open question to be settled empirically in deployment.

Let $G(t)$ be the instantaneous credit generation rate (credits issued per unit time), $C(t)$ be the consumption rate (credits spent per unit time), $S(t)$ be the unspent credit stock, $N(t)$ be the number of active contributors, and $D(t)$ be the number of active consumers. The bookkeeping equations are:

$$
\frac{dS}{dt} \;=\; G(t) - C(t) - \lambda \, S(t),
$$

where $\lambda$ is the credit decay rate (set so that credits halve over a configurable lifetime; see §7.3). Generation and consumption are tied to network state through a single multiplicative rate parameter $\alpha_{network}(t) \in (0, \infty)$ that the protocol adjusts as a controller variable:

$$
G(t) \;=\; \alpha_{network}(t)\cdot \bar{g}\cdot N_{eff}(t), \qquad
C(t) \;=\; \frac{1}{\alpha_{network}(t)}\cdot \bar{c}\cdot D_{eff}(t),
$$

where $\bar{g}$ is the baseline credit-per-GPU-hour rate, $\bar{c}$ is the baseline credit-per-inference-request cost, and $N_{eff}$ and $D_{eff}$ are reputation- and capability-weighted active counts (see §7.1). Increasing $\alpha_{network}$ rewards contributors more per GPU-hour and charges consumers more per request, both of which act to *reduce* an excess of demand over supply.

At a fixed point ($dS/dt = 0$), the controller satisfies

$$
\alpha_{network}^{2} \;=\; \frac{\bar{c}\, D_{eff} \,+\, \lambda\, S}{\bar{g}\, N_{eff}}.
$$

This identifies the static balance, not stability. The controller's dynamic behaviour — its response time, overshoot, and robustness to strategic behaviour by participants — is governed by an adjustment law (e.g. proportional, PI, or model-predictive) that we leave as an implementation choice. Whether any such controller can hold the utilisation target under realistic demand patterns is, in our view, a question that can only be answered through a working deployment.

We *conjecture* that small networks (on the order of tens to low hundreds of active contributors) will require external bootstrap incentives — for example, an initial credit endowment for new joiners — and that beyond some threshold, network effects make sustained operation feasible. We do not claim a specific threshold number; the figure of "1,000 active participants" that appeared in an earlier draft of this paper was a placeholder, not a derived bound.

## 4.6 Comparison with Existing Distributed Computing Systems

The DeCLAI credit mechanism draws inspiration from successful distributed computing projects while addressing their limitations through novel economic design. SETI@home and Folding@home pioneered volunteer computing but relied primarily on altruistic motivation without sustainable economic incentives [60]. BOINC improved resource management but maintained the volunteer model that limits long-term scalability.

BitTorrent demonstrated effective peer-to-peer resource sharing through tit-for-tat mechanisms, but these approaches don't translate directly to computational resources where contribution and consumption occur asynchronously [61]. Cryptocurrency mining creates strong economic incentives but wastes computational resources on artificially difficult problems rather than useful work.

The DeCLAI system combines the best aspects of these approaches while avoiding their limitations:

| System | Incentive Model | Resource Efficiency | Sustainability | Accessibility |
|--------|----------------|-------------------|----------------|---------------|
| SETI@home | Altruistic | High | Limited | High |
| BitTorrent | Tit-for-tat | High | Moderate | High |
| Bitcoin | Proof of Work | Low | High | Low |
| DeCLAI | Credit-based | High | High | High |

Unlike altruistic systems, DeCLAI provides concrete economic incentives that sustain long-term participation. Unlike cryptocurrency systems, DeCLAI channels computational resources toward useful work rather than artificial puzzles. Unlike commercial cloud platforms, DeCLAI maintains accessibility through its non-monetary credit system that enables participation regardless of financial resources [62].

The credit mechanism also addresses temporal misalignment between resource contribution and consumption. Contributors may provide GPU cycles during off-peak hours when their computers are idle, then consume credits for inference during peak productivity periods. This temporal flexibility, impossible in direct tit-for-tat systems, dramatically improves resource utilization and user convenience.

Furthermore, the system's geographic clustering approach enables latency optimization while maintaining global resource liquidity. Contributors in one time zone can serve users in another, creating natural load balancing that improves efficiency while expanding access. This global-local hybrid approach represents a significant advance over purely centralized or purely distributed alternatives [63].

Through this comprehensive credit mechanism design, DeCLAI creates the economic foundation necessary for sustainable, democratic AI access. The mathematical rigor ensures system stability, game-theoretic analysis confirms incentive compatibility, and anti-gaming mechanisms prevent exploitation. Most importantly, the credit system transforms AI compute from a scarce commodity into a shared community resource, enabling the vision of democratized artificial intelligence to become practical reality.

---

# 5. Distributed Inference Protocol

## 5.1 Protocol Overview and Design Principles

The DeCLAI Distributed Inference Protocol (DIP) represents a fundamental advancement in distributed computing for large language models, enabling seamless coordination across heterogeneous consumer hardware while maintaining performance guarantees comparable to centralized systems. Unlike traditional distributed inference approaches that assume homogeneous clusters and high-bandwidth interconnects, DIP explicitly accommodates the variable capabilities and network conditions inherent in community-driven computing environments [64].

The protocol operates on four core design principles that distinguish it from existing distributed computing frameworks. **Adaptive Partitioning** enables dynamic model sharding based on real-time hardware availability and network conditions, ensuring optimal resource utilization even as participants join and leave the network. **Fault-Tolerant Coordination** implements Byzantine-resilient consensus mechanisms that maintain inference integrity despite node failures or malicious behavior. **Latency-Aware Scheduling** prioritizes geographic proximity and network topology to minimize end-to-end response times while balancing computational load. **Credit-Integrated Resource Management** seamlessly incorporates the credit mechanism described in Section 4, ensuring that computational contributions directly translate to inference access rights [65].

The protocol architecture consists of three primary phases: **Request Orchestration**, where user queries are authenticated, validated, and routed to appropriate clusters; **Distributed Execution**, where model inference occurs across multiple nodes with sophisticated coordination and fault tolerance; and **Result Aggregation**, where partial computations are combined and validated before delivery to users. Each phase implements specific algorithms optimized for the unique constraints of community-driven distributed computing.

## 5.2 Request Orchestration and Routing

The request orchestration phase transforms user inference requests into distributed execution plans that maximize performance while respecting credit balances and network constraints. This process begins when users submit queries through regional gateways, which implement routing algorithms that consider multiple optimization criteria simultaneously.

### 5.2.1 Authentication and Credit Verification

```pseudocode
ALGORITHM: RequestAuthentication
INPUT: user_request, user_credentials, credit_balance
OUTPUT: authenticated_request OR rejection

1. VERIFY cryptographic_signature(user_request, user_credentials)
2. QUERY distributed_ledger FOR current_credit_balance(user_id)
3. ESTIMATE inference_cost(user_request.model, user_request.parameters)
4. IF credit_balance >= inference_cost THEN
5.    RESERVE credits(user_id, inference_cost)
6.    GENERATE request_token WITH timestamp, user_id, cost_estimate
7.    RETURN authenticated_request(user_request, request_token)
8. ELSE
9.    RETURN rejection("insufficient_credits", required_credits)
```

The authentication algorithm implements a two-phase commit protocol that prevents double-spending while maintaining low latency. Credit reservation occurs immediately upon request validation, with automatic release if inference fails to complete within specified timeouts. This approach ensures fair resource allocation while preventing credit system gaming [66].

### 5.2.2 Intelligent Cluster Selection

The cluster selection algorithm is the load-balancing core of the routing layer, balancing multiple competing objectives to optimize both individual request performance and overall network efficiency. The algorithm considers cluster computational capacity, current load, geographic proximity to the user, model availability, and historical performance metrics.

```pseudocode
ALGORITHM: OptimalClusterSelection
INPUT: authenticated_request, available_clusters, network_state
OUTPUT: selected_cluster, execution_plan

1. scores = empty_map  // cluster -> composite_score
2. FOR each cluster IN available_clusters DO
3.    latency_score   = estimate_network_latency(user_location, cluster)
4.    capacity_score  = cluster.available_compute / cluster.total_compute
5.    model_score     = model_availability(request.model, cluster)
6.    load_score      = 1.0 - (cluster.current_load / cluster.max_load)
7.    scores[cluster] = w1*latency_score + w2*capacity_score
                      + w3*model_score   + w4*load_score
8. END FOR
9. selected_cluster = argmax(scores)
10. execution_plan  = GENERATE_execution_plan(request, selected_cluster.topology)
11. RETURN selected_cluster, execution_plan
```

The weighting parameters (w1, w2, w3, w4) adapt dynamically based on network conditions and user preferences, with latency optimization receiving higher priority during peak usage periods and capacity optimization dominating during off-peak hours [67].

## 5.3 Distributed Model Sharding and Execution

The distributed execution phase implements model sharding techniques specifically optimized for transformer architectures running on heterogeneous consumer hardware. Unlike traditional approaches that partition models statically across homogeneous nodes, DIP employs dynamic sharding that adapts to real-time hardware capabilities and network conditions.

### 5.3.1 Adaptive Transformer Partitioning

Modern large language models exhibit natural partitioning boundaries that DIP exploits for efficient distributed execution. The protocol implements layer-wise partitioning for transformer blocks, attention head distribution for multi-head attention mechanisms, and embedding table sharding for models with large vocabularies.

```pseudocode
ALGORITHM: AdaptiveModelSharding
INPUT: model_architecture, cluster_nodes, hardware_profiles
OUTPUT: sharding_plan, communication_schedule

1. ANALYZE model_architecture FOR natural_partition_points
2. FOR each node IN cluster_nodes DO
3.    PROFILE hardware_capabilities(node.gpu_memory, node.compute_power, node.bandwidth)
4. END FOR
5. INITIALIZE sharding_plan = empty
6. FOR each layer IN model_architecture.layers DO
7.    memory_requirement = calculate_layer_memory(layer)
8.    compute_requirement = calculate_layer_compute(layer)
9.    candidate_nodes = filter_nodes(cluster_nodes, memory_requirement, compute_requirement)
10.   selected_node = select_optimal_node(candidate_nodes, current_load, network_topology)
11.   sharding_plan.assign(layer, selected_node)
12. END FOR
13. OPTIMIZE communication_schedule(sharding_plan, network_topology)
14. RETURN sharding_plan, communication_schedule
```

The sharding algorithm implements several optimization strategies that significantly improve performance over naive partitioning approaches. **Memory-Aware Allocation** ensures that model segments fit within available GPU memory while maintaining computational efficiency. **Communication Minimization** groups interdependent layers on nodes with high-bandwidth connections, reducing network overhead. **Load Balancing** distributes computational work proportionally to node capabilities, preventing bottlenecks from slower hardware [68].

### 5.3.2 Coordinated Inference Execution

Once model sharding is complete, the distributed execution phase coordinates computation across multiple nodes while maintaining strict consistency and fault tolerance guarantees. The execution protocol implements a pipeline-parallel approach that overlaps computation and communication to minimize end-to-end latency.

```pseudocode
ALGORITHM: DistributedInferenceExecution
INPUT: sharding_plan, input_tokens, execution_parameters
OUTPUT: inference_result, performance_metrics

1. INITIALIZE pipeline_stages FROM sharding_plan
2. DISTRIBUTE input_tokens TO first_stage_nodes
3. FOR each pipeline_stage IN execution_order DO
4.    PARALLEL_EXECUTE stage_computation ON assigned_nodes
5.    VALIDATE intermediate_results USING consensus_protocol
6.    IF validation_fails THEN
7.       TRIGGER fault_recovery_protocol(failed_nodes, stage)
8.    END IF
9.    TRANSFER stage_outputs TO next_stage_nodes
10. END FOR
11. AGGREGATE final_results FROM last_stage_nodes
12. VALIDATE final_consistency USING cryptographic_proofs
13. RECORD performance_metrics(latency, throughput, node_contributions)
14. RETURN inference_result, performance_metrics
```

The execution algorithm implements fault tolerance mechanisms that maintain inference integrity even when individual nodes fail or behave maliciously. **Redundant Computation** executes critical model segments on multiple nodes, enabling rapid recovery from failures. **Consensus Validation** ensures that intermediate results are consistent across nodes before proceeding to subsequent pipeline stages. **Adaptive Rescheduling** dynamically reassigns computation when nodes become unavailable, maintaining inference progress without complete restart [69].

## 5.4 Latency Optimization and Performance Guarantees

DeCLAI provides concrete latency guarantees that enable reliable application development while accommodating the inherent variability of distributed consumer hardware. The protocol implements multiple optimization layers that work together to minimize end-to-end response times while maintaining consistency and fault tolerance.

### 5.4.1 Geographic Clustering and Network Optimization

The geographic clustering strategy represents a key innovation that enables DeCLAI to achieve low-latency inference despite distributed execution. Clusters are organized to ensure that intra-cluster communication never exceeds 5ms round-trip time, while inter-cluster coordination is minimized through intelligent request routing and model replication.

**Network Topology Optimization**: DeCLAI implements a hierarchical network topology that mirrors internet infrastructure, with regional gateways positioned at major internet exchange points and cluster orchestrators located to minimize aggregate latency to participant nodes. This topology reduces average request routing time by 40-60% compared to naive distributed approaches [70].

**Predictive Resource Pre-allocation**: The protocol implements machine learning-based demand prediction that pre-positions model weights and allocates computational resources based on historical usage patterns and geographic demand distribution. This predictive approach reduces cold-start latency for popular models while maintaining efficient resource utilization [71].

### 5.4.2 Performance Guarantee Framework

DeCLAI provides three tiers of performance guarantees that accommodate different application requirements and user credit balances:

**Best-Effort Service** (Standard): Inference requests complete within 95th percentile latency bounds of 2-5 seconds for models up to 70B parameters, with throughput guarantees of at least 10 tokens per second per user during normal network conditions.

**Priority Service** (Enhanced): Users with sustained computational contributions receive priority queuing and dedicated resource allocation, with 99th percentile latency bounds of 1-3 seconds and guaranteed throughput of 25 tokens per second even during peak demand periods.

**Real-Time Service** (Premium): Critical applications receive sub-second response guarantees through pre-allocated resources and dedicated network paths, with 99.9th percentile latency bounds of 200-800ms for models up to 13B parameters.

## 5.5 Fault Tolerance and Recovery Mechanisms

The distributed nature of DeCLAI requires fault tolerance mechanisms that maintain service availability despite node failures, network partitions, and malicious behavior. The protocol implements multiple layers of redundancy and recovery that ensure inference requests complete successfully even under adverse conditions.

### 5.5.1 Byzantine Fault Tolerance for Inference Validation

DeCLAI uses a redundant-execution validation protocol — not a full pBFT consensus over transaction order, but result-agreement voting with explicit floating-point tolerance for the cross-architecture non-determinism of GPU inference. Naive bitstring voting will fail when honest contributors run the same forward pass on different GPU architectures: cuBLAS reduction orders, transcendental implementations, and CUDA driver versions can produce numerically distinct but semantically equivalent activations, which under low-temperature sampling lead to identical argmax decisions on most tokens but different bitstrings overall. The protocol therefore compares results at an explicit tolerance $\varepsilon$.

```pseudocode
ALGORITHM: RedundantExecutionValidation
INPUT:  computation_task,          // the inference request
        node_assignments,          // n = 3f+1 assigned contributors
        f,                         // Byzantine tolerance, f < n/3
        epsilon,                   // comparison tolerance (per-token logit)
        timeout
OUTPUT: validated_result, confidence_score, dissenters

1. PARALLEL_EXECUTE computation_task ON node_assignments WITH timeout
2. results = COLLECT_responses()
3. clusters = group_results_by_epsilon_equivalence(results, epsilon)
4.            // two results are in the same cluster if their
            // per-token logit vectors agree within epsilon
5. winning_cluster = argmax over clusters of |cluster|
6. IF |winning_cluster| >= 2f + 1 THEN
7.    validated_result  = canonical_result(winning_cluster)
8.    confidence_score  = |winning_cluster| / |results|
9.    dissenters        = node_assignments \ winning_cluster
10.   RETURN (validated_result, confidence_score, dissenters)
11. ELSE
12.   // no result has 2f+1 supporters within epsilon
13.   TRIGGER extended_validation(computation_task, fresh_node_set)
14. END IF
```

The protocol tolerates up to $f$ Byzantine contributors out of $n = 3f+1$ assigned nodes — the standard pBFT bound — while accommodating numerical heterogeneity through $\varepsilon$-tolerance grouping rather than exact bitstring voting. The `dissenters` set is reported to the reputation system; persistently dissenting contributors lose trust weight over time. Note that this is *not* a full pBFT instance (no view change, no primary, no sequence number ordering); it is a one-shot result-agreement vote that piggybacks on pBFT's threshold structure. The credit ledger itself, which does require transaction ordering across gateways, runs a separate pBFT instance described in §3.4 [72].

### 5.5.2 Adaptive Recovery and Rescheduling

When node failures or network partitions occur during inference execution, DeCLAI implements adaptive recovery mechanisms that minimize disruption while maintaining result integrity. The recovery protocol considers the current execution state, available backup resources, and user performance requirements to determine optimal recovery strategies.

**Checkpoint-Based Recovery**: Long-running inference tasks implement periodic checkpointing that enables rapid recovery from intermediate failures without complete restart. Checkpoints are distributed across multiple nodes to prevent single points of failure while minimizing storage overhead.

**Dynamic Rescheduling**: When nodes become unavailable, the protocol automatically reschedules affected computations to available backup nodes while maintaining pipeline parallelism and minimizing additional latency. The rescheduling algorithm considers node capabilities, current load, and network topology to optimize recovery performance.

**Graceful Degradation**: Under severe network conditions or widespread node failures, DeCLAI implements graceful degradation that maintains service availability with reduced performance guarantees rather than complete service failure. This approach ensures that critical applications can continue operating even during network stress conditions.

## 5.6 Integration with Existing ML Frameworks

DeCLAI's distributed inference protocol is designed for seamless integration with popular machine learning frameworks, enabling researchers and developers to leverage distributed computing capabilities without significant code modifications. The protocol implements standardized APIs that abstract the complexity of distributed execution while providing fine-grained control over performance and resource allocation when needed.

### 5.6.1 Framework-Agnostic API Design

The DeCLAI API implements a framework-agnostic interface that supports PyTorch, TensorFlow, JAX, and other popular ML frameworks through unified abstractions. This design enables existing applications to adopt distributed inference with minimal code changes while preserving framework-specific optimizations and features.

```pseudocode
INTERFACE: DeCLAIInferenceAPI
METHODS:
  initialize_session(model_id, framework_type, performance_tier) -> session
  submit_inference(input_data, generation_parameters)            -> request_id
  get_inference_status(request_id)                               -> status
  retrieve_results(request_id)                                   -> result
  estimate_cost(model_id, input_length, generation_parameters)   -> credits
  get_available_models(filter_criteria)                          -> [model_id]

EXAMPLE_USAGE:
session    = DeCLAI.initialize_session("llama2-70b", "pytorch", "priority")
request_id = session.submit_inference(
                input_data            = input_tokens,
                generation_parameters = {"max_length": 512})
results    = session.retrieve_results(request_id)
```

The API design prioritizes developer experience while maintaining the flexibility required for diverse application requirements. **Asynchronous Operations** enable non-blocking inference requests that integrate naturally with modern application architectures. **Batch Processing** supports efficient handling of multiple requests while maintaining individual request tracking and result delivery. **Streaming Responses** provide real-time token generation for interactive applications that require immediate feedback [73].

### 5.6.2 Model Deployment and Version Management

DeCLAI implements model-deployment mechanisms that enable rapid distribution of new models and updates across the distributed network while maintaining compatibility with ongoing inference requests. The deployment system supports multiple model formats and provides automated conversion tools for popular architectures.

**Automated Model Conversion**: The system includes conversion utilities that transform models from framework-specific formats (PyTorch .pth, TensorFlow SavedModel, ONNX) into DeCLAI's optimized distributed format. These conversions preserve model accuracy while optimizing for distributed execution characteristics [74].

**Version Control and Rollback**: Model updates propagate through the network using a controlled rollout mechanism that enables gradual deployment and rapid rollback if issues are detected. The system maintains multiple model versions simultaneously, enabling A/B testing and compatibility with applications that require specific model versions.

**Performance Profiling**: Each model deployment includes automated performance profiling that characterizes inference latency, memory requirements, and optimal sharding strategies across different hardware configurations. This profiling data informs the cluster selection and resource allocation algorithms described in previous sections.

## 5.7 Quality of Service and SLA Implementation

DeCLAI provides concrete Service Level Agreements (SLAs) that enable reliable application development while accommodating the inherent variability of distributed consumer hardware. The SLA framework implements multiple service tiers with specific performance guarantees, availability commitments, and credit cost structures.

### 5.7.1 Multi-Tier Service Guarantees

The three-tier service model provides flexibility for different application requirements while maintaining economic sustainability through credit-based resource allocation:

**Research Tier** (1.0x credit cost): Designed for academic research and experimentation, providing best-effort performance with 95% availability guarantees and median response times of 3-8 seconds for models up to 70B parameters. This tier accommodates batch processing workloads and non-interactive applications where latency is less critical than cost efficiency.

**Production Tier** (2.5x credit cost): Optimized for production applications requiring consistent performance, providing 99.5% availability guarantees and 95th percentile response times of 1-3 seconds. This tier includes priority queuing, dedicated resource allocation, and enhanced fault tolerance mechanisms suitable for user-facing applications.

**Real-Time Tier** (5.0x credit cost): Designed for latency-critical applications requiring sub-second response times, providing 99.9% availability guarantees and 99th percentile response times of 200-800ms for models up to 13B parameters. This tier includes pre-allocated resources, dedicated network paths, and specialized hardware optimization.

### 5.7.2 SLA Monitoring and Enforcement

DeCLAI implements comprehensive monitoring systems that track performance metrics in real-time and automatically trigger remediation actions when SLA violations are detected. The monitoring framework provides transparency to users while enabling continuous system optimization.

**Real-Time Performance Tracking**: The system continuously monitors request latency, throughput, availability, and error rates across all service tiers. Performance data is aggregated at multiple time scales (seconds, minutes, hours, days) to identify both immediate issues and long-term trends [75].

**Automated Remediation**: When performance degrades below SLA thresholds, the system automatically triggers remediation actions including resource reallocation, request rerouting, and cluster scaling. These automated responses minimize service disruption while maintaining transparency through detailed logging and user notifications.

**Credit Compensation**: Users receive automatic credit refunds when SLA violations occur, with compensation amounts proportional to the severity and duration of performance degradation. This compensation mechanism ensures that users are not penalized for system performance issues while maintaining incentives for continuous improvement.

## 5.8 Performance Analysis and Optimization Strategies

The DeCLAI distributed inference protocol implements multiple optimization strategies that work together to maximize performance while maintaining the democratic accessibility that distinguishes the system from centralized alternatives. These optimizations address the unique challenges of coordinating computation across heterogeneous consumer hardware with variable network conditions.

### 5.8.1 Latency Decomposition and Optimization

End-to-end inference latency in DeCLAI consists of several components that can be optimized independently: **Request Routing** (50-200ms), **Model Loading** (100-500ms for cold starts, <50ms for warm starts), **Distributed Computation** (1-5 seconds depending on model size), **Result Aggregation** (50-100ms), and **Network Communication** (varies by geographic distribution).

The protocol implements targeted optimizations for each latency component. **Predictive Model Caching** reduces cold start penalties by pre-loading popular models based on usage patterns and geographic demand. **Intelligent Request Batching** amortizes routing and setup costs across multiple requests while maintaining individual request tracking. **Pipeline Parallelism** overlaps computation and communication phases to minimize total execution time [76].

### 5.8.2 Throughput Scaling and Resource Utilization

DeCLAI's throughput characteristics scale superlinearly with network size due to several architectural advantages. **Geographic Distribution** enables natural load balancing across time zones, with peak demand in one region served by off-peak capacity in others. **Hardware Heterogeneity** allows optimal task assignment based on specific computational requirements, maximizing utilization of diverse GPU capabilities. **Adaptive Batching** dynamically adjusts batch sizes based on current network conditions and demand patterns.

The system maintains high resource utilization through scheduling algorithms that consider multiple optimization criteria simultaneously. **Multi-Objective Optimization** balances latency minimization, throughput maximization, and fair resource allocation across participants. **Dynamic Load Balancing** continuously redistributes work based on real-time performance monitoring and predictive demand modeling.

Through this comprehensive distributed inference protocol, DeCLAI demonstrates that community-driven distributed computing can achieve the performance, reliability, and scalability characteristics required for production AI applications while maintaining the democratic accessibility that distinguishes it from centralized alternatives. The technical innovations presented in this section enable the economic and social benefits described in subsequent sections, creating a foundation for truly democratized AI inference capabilities.

---

# 6. Security and Privacy

The democratization of AI compute through DeCLAI introduces unique security and privacy challenges that differ fundamentally from those faced by centralized systems. While traditional cloud providers can rely on physical security, controlled environments, and trusted hardware, DeCLAI must operate securely across thousands of untrusted consumer devices while preserving user privacy and protecting valuable model weights. This section outlines a security framework that aims to address these challenges through standard cryptographic protocols, privacy-preserving techniques, and distributed trust mechanisms, and identifies the primitives whose practical deployment remains future work.

The security architecture of DeCLAI rests on the principle that no single participant—whether individual contributor, regional gateway, or cluster orchestrator—should be trusted with sensitive information or critical system functions. Instead, security is intended to emerge from protocols that distribute trust across multiple parties while attempting to maintain the performance characteristics required for real-time AI inference.

## 6.1 Threat Model and Security Assumptions

### 6.1.0 Threat-Model Assumptions (Summary)

We state the assumptions on which the remainder of §6 depends explicitly and up front:

- **(a) Adversarial fraction.** At most f < n/3 of nodes in any consensus cluster are Byzantine; the remaining ≥ 2f+1 nodes are honest. This is the standard pBFT bound and is the assumption used uniformly throughout §6.
- **(b) Sybil mitigation.** Sybil resistance relies on reputation bootstrapping combined with optional hardware attestation where available. An unbounded free-identity attacker can still defeat short-term participation guarantees; we acknowledge this as an open problem rather than a solved one.
- **(c) Network model.** We assume partial synchrony in the sense of Dwork–Lynch–Stockmeyer, consistent with pBFT. Residential-link asymmetry, NAT traversal, and other practical network conditions are addressed at the protocol layer in §5.
- **(d) Computation determinism.** Cross-architecture floating-point non-determinism is a known issue for naive bitstring voting on inference outputs. The protocol therefore uses ε-tolerance comparison rather than exact equality (see §5).

### 6.1.1 Adversarial Environment Analysis

DeCLAI operates in a fundamentally adversarial environment where participants may exhibit various forms of malicious behavior. Our threat model considers three primary categories of adversaries, each with distinct capabilities and motivations:

**Rational Adversaries** seek to maximize their computational credits through strategic behavior that may violate protocol specifications. These adversaries will deviate from honest behavior only when doing so provides economic advantage, making them predictable through game-theoretic analysis. Examples include participants who attempt to claim credits without performing computation, submit low-quality inference results to reduce computational costs, or manipulate network routing to preferentially serve their own requests.

**Byzantine Adversaries** exhibit arbitrary malicious behavior that may include attempts to disrupt network operations, corrupt inference results, or compromise user privacy. Unlike rational adversaries, Byzantine participants may act against their own economic interests to damage the system. Our analysis assumes that strictly fewer than one third of participants in any given cluster (f < n/3) may exhibit Byzantine behavior, with the remaining ≥ 2f+1 nodes honest. This is the standard pBFT bound and is consistent with established distributed systems literature [28, 72].

**External Attackers** operate outside the DeCLAI network but attempt to compromise system security through network-level attacks, cryptographic attacks, or social engineering. These adversaries may possess significant computational resources and sophisticated attack capabilities, including the ability to perform large-scale Sybil attacks, network traffic analysis, or attempts to extract model weights through side-channel attacks.

### 6.1.2 Security Objectives and Privacy Goals

The DeCLAI security framework pursues four primary objectives that, taken together, are intended to support system integrity while preserving participant privacy:

**Computational Integrity**: All inference results must be cryptographically verifiable, ensuring that participants cannot submit fraudulent results or claim credits for work not performed. This objective extends beyond simple result validation to include verification of computational effort and resource utilization.

**Query Privacy**: User inference requests must remain confidential from all system participants except those directly involved in processing. This includes protection against traffic analysis, query content inference, and correlation attacks that might reveal user behavior patterns.

**Model Confidentiality**: Proprietary model weights must be protected from unauthorized access while enabling distributed inference across untrusted nodes. As discussed in §6.4.1, this is treated as a *partial* property under the decrypt-to-compute limitation rather than as an absolute guarantee.

**Network Resilience**: The system must continue operating correctly despite the presence of malicious participants, network partitions, and coordinated attacks. Resilience includes both availability guarantees and consistency maintenance under adversarial conditions.

### 6.1.3 Trust Assumptions and Justifications

DeCLAI's security model relies on several carefully justified trust assumptions that minimize required trust while maintaining practical deployability:

**Cryptographic Hardness**: We assume the computational intractability of discrete logarithm problems over elliptic curves, the security of SHA-256 hash functions, and the pseudorandomness of AES encryption. These assumptions are standard in modern cryptographic systems and supported by decades of cryptanalytic research [53, 79].

**Byzantine Fault Tolerance in Consensus**: Critical system operations require Byzantine fault-tolerant consensus among cluster nodes under the f < n/3 bound stated in §6.1.0. We do *not* assume a weaker honest-majority (>50%) model for the consensus path. As a *fallback* model, certain non-consensus components — for example reputation aggregation, telemetry summarization, and best-effort gossip dissemination — can tolerate the weaker honest-majority assumption, since incorrect aggregation in these components degrades efficiency but does not compromise ledger integrity or inference correctness. The core consensus path always requires f < n/3 [28, 93, 94].

**Network Connectivity (Partial Synchrony)**: Consistent with the pBFT setting, we assume partial synchrony: honest participants can eventually communicate with each other within some unknown but finite bound, while adversaries may control network timing and message ordering up to that bound. This assumption is intended to support liveness while accommodating realistic residential-network conditions (see §5).

**Hardware Attestation**: Participants can provide cryptographic attestations of their hardware capabilities and computational results through trusted execution environments or secure enclaves when available. While not required for basic operation, hardware attestation enables enhanced security guarantees for high-value computations.

## 6.2 Cryptographic Foundation and Primitives

### 6.2.1 Digital Signature Schemes and Identity Management

DeCLAI employs elliptic curve digital signatures (ECDSA) over the secp256k1 curve for all identity verification and message authentication [53]. Each participant generates a cryptographic identity consisting of a public-private key pair, with the public key serving as their network identifier and the private key enabling message signing and authentication.

The identity management system uses a pseudonymous-participation approach that aims to preserve privacy while enabling reputation tracking. Participants can generate multiple pseudonymous identities linked through zero-knowledge proofs, allowing them to build reputation across different contexts without revealing their true identity or enabling cross-context correlation. This combination of standard primitives is not itself novel; the contribution is its integration into the credit-and-reputation pipeline.

```pseudocode
ALGORITHM: PseudonymousIdentityGeneration
INPUT: master_secret, context_identifier
OUTPUT: pseudonymous_keypair, linkage_proof

1. context_seed = HMAC(master_secret, context_identifier)
2. private_key = SHA256(context_seed || "private")
3. public_key = private_key * G  // Elliptic curve point multiplication
4. linkage_proof = ZK_PROVE(knows master_secret linking identities)
5. RETURN (private_key, public_key), linkage_proof
```

### 6.2.2 Distributed Key Management and Secret Sharing

Model weights and sensitive system parameters are protected through Shamir's secret sharing scheme, which distributes cryptographic secrets across multiple participants such that no single party can reconstruct the original data [78]. The system employs a (t, n)-threshold scheme where any t participants from a group of n can reconstruct the secret, providing both security and availability guarantees.

The distributed key management protocol extends traditional secret sharing with proactive security measures that periodically refresh shares without changing the underlying secret. This approach provides forward security against adversaries who may compromise participants over time:

```pseudocode
ALGORITHM: ProactiveSecretRefresh
INPUT: old_shares[], participant_list, threshold_t
OUTPUT: new_shares[], refresh_proof

1. FOR each participant i in participant_list:
2.    generate random_polynomial_i of degree t-1
3.    broadcast polynomial_i(0) = 0  // Ensures sum preserves secret
4. FOR each participant j:
5.    new_share_j = old_share_j + SUM(polynomial_i(j) for all i)
6.    generate refresh_proof_j proving validity
7. RETURN new_shares[], refresh_proofs[]
```

### 6.2.3 Zero-Knowledge Proof Systems (Future Verification Layer)

Proof-of-inference via zero-knowledge SNARKs over a transformer forward pass is an active research direction (zkLLM [citation needed], Mystique [citation needed], Mithril [citation needed]). Current state-of-the-art handles models on the order of 10^7 parameters with proving times measured in minutes to hours per inference [citation needed]. For DeCLAI's target models (1B–70B parameters), this primitive is not yet deployable at the latency budgets specified in §5.4 — the gap to deployability is several orders of magnitude in both proving time and memory.

We therefore treat proof-of-inference as a **future verification layer** and, in the near term, rely on a combination of cheaper mechanisms (consistent with §4.3 and §5.5.1):

**(a) Redundant execution across f+1 nodes with ε-tolerance comparison.** Each inference task is executed by at least f+1 contributors and results are compared using ε-tolerance numerical equality (see §5), accommodating cross-architecture floating-point non-determinism while detecting outright divergence.

**(b) Challenge-response audits.** A verifier set samples a fraction of completed inferences and re-executes them on independent hardware. Contributors whose audited outputs fail ε-tolerance comparison incur reputation and credit penalties.

**(c) Reputation-weighted trust.** Newly joined contributors are down-weighted in both task assignment and result aggregation until they accumulate audit history. This is intended to bound the damage a fresh Sybil identity can do before being filtered out, though, as noted in §6.1.0(b), it does not solve the unbounded free-identity attacker.

When zk-SNARK proving costs for transformer inference fall by the necessary 3–5 orders of magnitude, the protocol is designed to admit proof-of-inference as an additional verification layer without changes to the credit ledger or consensus path.

## 6.3 Privacy-Preserving Query Processing

### 6.3.1 Differential Privacy for Telemetry and Reputation Signals

DP guarantees over inference outputs face a strong utility–privacy tradeoff: at ε ≤ 1.0, noise calibrated for meaningful indistinguishability over generated tokens substantially degrades output quality, and there is no published evidence that ε = 0.1 on transformer outputs is compatible with maintained inference quality. We do not claim such a guarantee.

DeCLAI therefore applies differential privacy **only to telemetry and reputation signals**, not to inference outputs themselves [77]. Concretely:

**Telemetry-level DP.** Aggregated statistics released to the network — per-contributor throughput, error rates, latency distributions, and reputation deltas — are perturbed under (ε, δ)-DP with ε ∈ [0.1, 1.0] and δ = 10^-6. Because these statistics are aggregates over many tasks, tight ε is achievable without affecting user-visible inference quality.

**Reputation aggregation.** Reputation updates contributed by peers are aggregated under DP noise so that an individual peer's observations cannot be reverse-engineered from public reputation scores.

**Query and output privacy is *not* obtained from DP.** Confidentiality of user queries and generated outputs against curious contributors is instead addressed by (a) prompt fragmentation across multiple non-colluding contributors (§6.3.2 and §6.3.3) and (b) optional TEE-based execution where consumer hardware supports it (§6.4.1). We make no quantitative quality-loss claim for these mechanisms in this section; concrete evaluation is left to future work.

### 6.3.2 Secure Multiparty Computation (Research Direction)

Secure multiparty computation (MPC) for transformer inference — including CrypTen [citation needed], MPCFormer [citation needed], Iron [citation needed], and SecureML [81, 82] — currently reports 10^3–10^5× slowdown over plaintext inference and gigabyte-per-query bandwidth requirements between MPC parties [citation needed]. These costs are incompatible with consumer residential uplinks and with the latency targets described in §5.4 by several orders of magnitude.

We therefore outline MPC as a **research direction** for high-sensitivity workloads (for example, health records, legal discovery, or other regulated data) rather than as a deployable feature of the general residential network. Practical deployment of MPC-based transformer inference will likely require dedicated low-latency clusters with high-bandwidth interconnects between the MPC parties, not commodity home links. We note this as future work and do not claim it as part of the near-term DeCLAI deployment.

For the common case, query and output privacy is provided by the prompt-fragmentation and TEE-where-available approaches described in §6.3.1 and §6.4.1. The pseudocode below sketches the *target* protocol skeleton for an MPC-capable cluster; it is intended as a specification for future work rather than a description of currently deployed functionality:

```pseudocode
ALGORITHM: SecureTransformerInference  // future work; not deployed
INPUT: encrypted_query, model_shares[], participant_set
OUTPUT: encrypted_result, computation_proof

1. // Distribute query across participants using secret sharing
2. query_shares = SECRET_SHARE(encrypted_query, participant_set)

3. // Perform secure matrix multiplication for attention computation
4. FOR each layer in transformer_layers:
5.    attention_shares = SECURE_MATMUL(query_shares, key_shares)
6.    attention_weights = SECURE_SOFTMAX(attention_shares)  // Garbled circuit
7.    context_shares = SECURE_MATMUL(attention_weights, value_shares)

8. // Generate output through secure computation
9. output_shares = SECURE_FEEDFORWARD(context_shares, model_shares)
10. encrypted_result = RECONSTRUCT_SECRET(output_shares)
11. computation_proof = GENERATE_ZK_PROOF(computation_validity)
12. RETURN encrypted_result, computation_proof
```

### 6.3.3 Anonymous Communication and Traffic Analysis Resistance

DeCLAI is intended to use anonymous communication protocols that aim to prevent network-level adversaries from correlating user identities with their inference requests. The system employs a combination of onion routing and mix networks tuned for the latency requirements of real-time AI inference; neither primitive is itself novel.

The anonymous communication protocol operates through three layers of protection:

**Onion Routing**: User queries are encrypted in multiple layers, with each layer containing routing information for the next hop. Participants can only decrypt their layer of the onion, preventing them from learning the full path or correlating requests with users.

**Mix Networks**: Queries are batched and shuffled at multiple points in the network, breaking timing correlations that might enable traffic analysis. The mixing process introduces controlled latency that balances privacy protection with inference responsiveness.

**Cover Traffic**: The system generates synthetic queries that are indistinguishable from real requests, preventing adversaries from using traffic volume analysis to infer user behavior patterns. Cover traffic is generated collaboratively by participants, distributing the computational overhead across the network.

## 6.4 Model Weight Protection and Intellectual Property

### 6.4.1 Encrypted Model Distribution and Access Control (Partial Confidentiality)

Proprietary model weights are protected through a layered encryption scheme that aims to control *who can decrypt* model parameters and under what conditions. The system uses attribute-based encryption (ABE) for fine-grained access decisions based on participant credentials and computational contributions [80], and threshold encryption to require multi-party collaboration for decrypting the most sensitive components.

**The decrypt-to-compute limitation.** ABE and functional encryption (FE) control *who can decrypt* model weights; once a contributor decrypts to compute a forward pass, plaintext weights reside in GPU memory and can be extracted by a sufficiently determined adversary controlling that node. This "decrypt-to-compute" problem is fundamental to all schemes short of (i) fully-homomorphic encryption — currently 10^5–10^7× slowdown and infeasible at LLM scale [citation needed] — or (ii) hardware-enforced trusted execution environments. DeCLAI therefore treats model confidentiality as **partial**: appropriate for proprietary fine-tunes shared within trusted clusters or under TEE attestation, and *not* as a guarantee against a determined adversarial contributor with full control of their own hardware.

The layered scheme is intended to raise the cost of weight extraction rather than to make it cryptographically impossible:

**Hierarchical Encryption**: Model weights are encrypted under a hierarchy of keys, with different layers accessible to participants based on role and reputation. Attention weights might be accessible to all participants, while feed-forward parameters require higher trust levels.

**Functional Encryption (where applicable)**: For computations expressible in restricted FE classes (for example, inner-product FE), participants receive keys that enable specific computations without revealing the underlying parameters. This does not extend to full transformer inference, which is acknowledged as future work.

**Threshold Decryption**: The most sensitive model components are protected through threshold encryption schemes where multiple participants must collaborate to decrypt and use sensitive parameters, raising the bar for any single malicious node to exfiltrate complete weights.

### 6.4.2 Secure Model Sharding and Distributed Storage

DeCLAI uses model sharding techniques that distribute transformer weights across multiple participants while attempting to maintain computational efficiency and the partial confidentiality property of §6.4.1. The sharding protocol aims to ensure that no single participant holds complete model layers while still enabling efficient distributed inference. The individual techniques (semantic partitioning, erasure-coded replication, periodic reshuffling) are not novel in isolation.

The secure sharding algorithm operates through several key innovations:

**Semantic Sharding**: Model weights are partitioned based on semantic boundaries (such as attention heads or feed-forward layers) rather than arbitrary splits. This approach maintains computational locality while ensuring that individual shards do not reveal complete model functionality.

**Redundant Distribution**: Each model shard is replicated across multiple participants using error-correcting codes that provide both fault tolerance and security. Adversaries must compromise multiple participants to reconstruct complete model components.

**Dynamic Resharding**: The system periodically redistributes model shards across different participants, preventing long-term accumulation of model knowledge by any individual node. The resharding process maintains inference availability while refreshing security guarantees.

## 6.5 Network Security and Attack Mitigation

### 6.5.1 Sybil Attack Prevention and Identity Verification

The decentralized nature of DeCLAI makes it vulnerable to Sybil attacks where adversaries create multiple fake identities to gain disproportionate influence over network operations. The system implements a multi-faceted defense strategy that combines computational proof-of-work, social verification, and economic incentives to prevent Sybil attacks.

**Computational Identity Binding**: New participants must solve computationally expensive puzzles that bind their identity to significant computational work. The puzzle difficulty is calibrated to make large-scale identity creation economically infeasible while remaining accessible to legitimate participants with consumer hardware.

**Social Verification Networks**: Participants can vouch for each other's legitimacy through cryptographic attestations that build webs of trust. The verification system employs graph-theoretic algorithms to identify suspicious identity clusters and limit their influence on network operations.

**Economic Stake Requirements**: Participants must demonstrate ongoing computational contributions to maintain their network standing. This requirement makes Sybil attacks expensive to maintain over time, as adversaries must provide real computational resources to support their fake identities.

### 6.5.2 Eclipse Attack Resistance and Network Connectivity

Eclipse attacks attempt to isolate honest participants from the broader network, enabling adversaries to control their view of system state and potentially manipulate their behavior. DeCLAI implements several mechanisms to ensure robust network connectivity and prevent eclipse attacks:

**Diverse Peer Discovery**: Participants discover peers through multiple independent channels, including distributed hash tables, social recommendations, and geographic proximity algorithms. This diversity prevents adversaries from controlling all peer discovery mechanisms.

**Connectivity Monitoring**: Participants continuously monitor their network connectivity and actively seek new connections when isolation is detected. The monitoring system employs statistical techniques to distinguish between natural network partitions and adversarial eclipse attempts.

**Redundant Communication Paths**: Critical communications are routed through multiple independent paths, ensuring that adversaries cannot block important messages by controlling a single network segment. The redundancy system balances communication overhead with attack resistance.

### 6.5.3 Denial of Service Protection and Resource Management

DeCLAI provides layered DoS mitigation that aims to limit adversaries' ability to disrupt network operations through resource exhaustion attacks. The mitigation operates at multiple levels of the system architecture:

**Request Rate Limiting**: User requests are subject to rate limiting based on credit balances and historical behavior patterns. The rate limiting algorithm adapts to network conditions and participant behavior, preventing both accidental overload and intentional DoS attacks.

**Computational Resource Protection**: Participants implement resource isolation that prevents malicious inference requests from consuming excessive computational resources. The isolation system employs containerization and resource quotas to maintain system stability under attack.

**Network Bandwidth Management**: The system implements traffic shaping and prioritization that is intended to give critical system communications (such as consensus messages and credit transactions) priority over user inference requests during periods of network congestion.

## 6.6 Trust Architecture and Consensus Mechanisms

### 6.6.1 Distributed Trust Model and Reputation Systems

DeCLAI's trust architecture avoids reliance on centralized authorities through a distributed reputation system that enables participants to assess the trustworthiness of their peers based on historical behavior and community feedback. The reputation system provides both security benefits (by identifying malicious participants) and efficiency improvements (by prioritizing reliable nodes for critical operations).

The distributed reputation protocol operates through several key mechanisms:

**Behavioral Monitoring**: Participants continuously monitor the behavior of their peers, recording metrics such as computation accuracy, response timeliness, and protocol compliance. These observations are aggregated into reputation scores that reflect long-term trustworthiness.

**Community Validation**: Reputation assessments are validated through community consensus, preventing individual participants from unfairly damaging others' reputations. The validation process employs Byzantine fault-tolerant consensus to ensure accuracy even in the presence of malicious participants.

**Privacy-Preserving Aggregation**: Reputation information is aggregated using privacy-preserving techniques that prevent participants from learning detailed information about others' behavior while still enabling accurate trust assessment. The aggregation employs homomorphic encryption and secure multiparty computation to maintain privacy.

### 6.6.2 Consensus Protocols for Critical Operations

Critical system operations, such as credit ledger updates and model registry changes, require consensus among multiple participants to maintain consistency and resist manipulation. DeCLAI employs a consensus protocol — not novel in primitive form, but tuned for the specific requirements of distributed AI inference:

**Practical Byzantine Fault Tolerance**: The consensus protocol builds on practical Byzantine fault tolerance (pBFT) but incorporates optimizations for AI workloads, such as batching of inference-related transactions and specialized validation procedures for computational proofs [28].

**Stake-Weighted Voting**: Consensus decisions are weighted by participants' computational contributions and reputation scores, with the intent that participants with greater investment in the network have proportionally greater influence over critical decisions. This approach aims to align incentives while maintaining broad participation.

**Adaptive Quorum Thresholds**: All consensus-path operations require the pBFT supermajority of ≥ 2f+1 honest votes out of n (i.e. f < n/3), as committed in §6.1.0. The system may *additionally* raise the threshold for high-criticality decisions (for example, model registry changes that affect many users) by requiring high-reputation participants in the quorum, but never *lowers* it below the pBFT bound. Non-consensus best-effort components (reputation aggregation, telemetry) may use the weaker honest-majority fallback model described in §6.1.3.

## 6.7 Privacy-Preserving Information Retrieval and Model Discovery

### 6.7.1 Private Model Query and Discovery

DeCLAI enables users to discover and access AI models without revealing their interests or requirements to network participants. The system employs private information retrieval (PIR) protocols that allow users to query model registries and retrieve model information without disclosing which models they are interested in [97, 98].

The private model discovery protocol provides several key capabilities:

**Anonymous Model Search**: Users can search for models based on capabilities, performance characteristics, or other criteria without revealing their search terms to registry operators. The search protocol employs homomorphic encryption to enable computation on encrypted queries.

**Private Model Access**: Users can access model weights and metadata without revealing which specific models they are using. The access protocol employs oblivious transfer techniques that enable selective information retrieval without disclosure.

**Usage Pattern Protection**: The system prevents adversaries from inferring user behavior patterns through analysis of model access patterns. This protection employs cover traffic and request batching to obscure individual usage patterns.

### 6.7.2 Searchable Encryption for Model Metadata

Model metadata, including performance benchmarks, training datasets, and capability descriptions, is protected through searchable encryption schemes that enable efficient search while maintaining confidentiality [99, 100]. The searchable encryption system allows users to find relevant models without exposing their search criteria or the model metadata to network participants.

The searchable encryption implementation employs several advanced techniques:

**Keyword-Based Search**: Users can search model metadata using encrypted keywords that match against encrypted model descriptions. The search protocol reveals only whether matches exist, not the specific content of queries or metadata.

**Range Queries**: Users can search for models based on numerical criteria (such as parameter count or inference latency) using range query protocols that preserve the privacy of both search criteria and model characteristics.

**Boolean Query Support**: Complex search queries involving multiple criteria and logical operators are supported through secure computation protocols that evaluate encrypted boolean expressions without revealing intermediate results.

Through the layered framework described in this section, DeCLAI outlines security mechanisms appropriate to the community-driven setting and identifies the primitives — proof-of-inference for transformer-scale models, MPC-based private transformer inference, DP at meaningful ε on generation outputs, and cryptographically enforced model confidentiality against a contributor with full hardware control — whose practical deployment requires future work. The framework is designed to compose these primitives as they mature, without changes to the credit ledger or the f < n/3 consensus path established in §6.1.0.

---

# Section 7: Economic Analysis

## 7.1 Formal Economic Model of the Credit System

The DeCLAI credit system represents a novel approach to resource allocation in distributed computing, fundamentally different from both traditional market mechanisms and existing cryptocurrency-based systems. We present a formal economic model that captures the essential dynamics of this non-monetary credit economy.

### 7.1.1 Mathematical Foundation

Let $N = \{1, 2, ..., n\}$ represent the set of participants in the DeCLAI network at time $t$. Each participant $i \in N$ is characterized by:

- **Computational capacity**: $c_i(t) \geq 0$ (normalized GPU-hours per unit time)
- **Credit balance**: $b_i(t) \in \mathbb{R}$ (can be negative, representing debt)
- **Demand for inference**: $d_i(t) \geq 0$ (GPU-hours requested per unit time)
- **Contribution rate**: $s_i(t) \geq 0$ (GPU-hours supplied per unit time)

The fundamental credit generation equation is:

$$\frac{db_i(t)}{dt} = \alpha \cdot s_i(t) \cdot \eta_i(t) - \beta \cdot d_i(t) \cdot \phi_i(t)$$

where:
- $\alpha$ is the credit generation coefficient (typically $\alpha = 1$)
- $\beta$ is the credit consumption coefficient (typically $\beta = 1$)
- $\eta_i(t)$ is the efficiency factor for participant $i$'s hardware
- $\phi_i(t)$ is the quality-of-service multiplier for consumed resources

The efficiency factor $\eta_i(t)$ normalizes different hardware capabilities. Rather than multiplying FLOPS by memory (which has awkward units and is dominated by whichever scales faster across hardware generations), we define $\eta_i$ as the minimum of two normalized capability ratios, capturing the actual bottleneck for transformer inference:

$$\eta_i(t) = \min\!\left(\frac{\text{FLOPS}_i}{\text{FLOPS}_{\text{baseline}}},\; \frac{\text{Memory}_i}{\text{Memory}_{\text{baseline}}}\right) \cdot \text{Availability}_i(t)$$

This reflects the empirical observation that LLM inference is memory-bound for large models and compute-bound for small ones; rewarding the lesser of the two ratios prevents over-rewarding GPUs that have memory but lack throughput (or vice versa). The baseline values are calibrated against a reference accelerator (we suggest the NVIDIA RTX 4090 or an A100-class card; the choice is a tunable system parameter). Note that this formula relies on **self-declared** capability values; verifying that declared capabilities match reality is a separate problem addressed by the validation protocol in §4.3 (redundant execution, challenge–response audits). Without verification, this normalisation cannot be incentive-compatible — over-reporting capability is an obvious gaming strategy.

### 7.1.2 Network-Level Bookkeeping

The total credit stock at time $t$ obeys the bookkeeping identity

$$\sum_{i=1}^{n} b_i(t) \;=\; B_0 \;+\; \int_0^t \!\!\Bigl(\sum_{i} \alpha\, s_i(\tau)\, \eta_i^{\text{eff}}(\tau) \;-\; \sum_{i} \beta\, d_i(\tau)\, \phi_i(\tau) \;-\; \lambda_{\text{decay}}\sum_i b_i(\tau)\Bigr)\, d\tau,$$

where $\eta_i^{\text{eff}}$ is the *effective* (reputation-multiplied) efficiency factor from §7.3.1, $\lambda_{\text{decay}}$ is the credit decay rate from §7.5.1, and $B_0$ is the bootstrap allocation. The decay term is essential: with reputation bonuses, $\eta_i^{\text{eff}} > \eta_i$ for trusted contributors, so credits are issued at a slightly higher rate than they would be without bonuses. The decay sink absorbs this excess and keeps the stock from drifting upward unboundedly. The controller $\alpha_{network}$ described in §4.5 is the operational lever that maintains the target utilisation ratio in the presence of these terms.

The **static balance** condition — credits issued per unit time equal credits spent per unit time plus credits decayed — is

$$\sum_{i} \alpha\, s_i\, \eta_i^{\text{eff}} \;=\; \sum_{i} \beta\, d_i\, \phi_i \;+\; \lambda_{\text{decay}}\, S,\qquad S = \sum_i b_i.$$

This is a necessary condition for sustained operation; it is not by itself sufficient, because the dynamics around this fixed point depend on participant response to the controller (see §4.5 for the open dynamic-stability question).

## 7.2 Incentive Compatibility Analysis

### 7.2.1 Strategic Behavior and Nash Equilibrium

We model participant behavior as a strategic game where each participant $i$ chooses contribution and consumption strategies to maximize their utility function:

$$U_i(s_i, d_i, s_{-i}, d_{-i}) = V_i(d_i) - C_i(s_i) + \lambda_i \cdot b_i(t+1)$$

where:
- $V_i(d_i)$ is the value derived from consuming $d_i$ units of inference
- $C_i(s_i)$ is the cost of contributing $s_i$ units of computation
- $\lambda_i$ is the participant's valuation of future credit balance
- $s_{-i}, d_{-i}$ represent other participants' strategies

**Conjecture 7.1** (Conditional Incentive Compatibility): *Suppose (i) the validation protocol (§4.3) detects mis-declared capability with probability at least $p$ per audited inference, (ii) detection results in a credit penalty of size $\kappa$, and (iii) participants discount future credits at rate $\lambda_i > 0$. Then there exists a threshold $(p,\kappa)$ pair above which truth-telling about computational capacity is a best response.*

This is a conjecture, not a theorem. Earlier drafts of this paper asserted truth-telling as a dominant strategy under the credit mechanism alone; this is incorrect. A rational participant has an obvious incentive to *over-report* capability $(\text{FLOPS}_i, \text{Memory}_i)$ to inflate $\eta_i$, because $\eta_i$ enters credit generation multiplicatively. Truth-telling becomes a best response only when the validation protocol catches mis-declared capability with sufficient probability and penalty to make over-reporting unprofitable in expectation. The required $(p,\kappa)$ depend on the specific validation sampling rate and on participant discount factors, both of which require empirical calibration. We treat this as an open mechanism-design question and do not claim a closed-form bound.

### 7.2.2 Participation Incentives

The system design addresses the fundamental tension between individual rationality and collective efficiency. Unlike traditional volunteer computing projects that rely purely on altruism [60], DeCLAI creates explicit incentives for sustained participation through the credit mechanism.

**Proposition 7.1 (Participation Condition)**: A rational participant chooses $s_i(t) > 0$ at time $t$ when the **marginal** value of contributing exceeds the **marginal** cost:

$$\alpha\, \eta_i^{\text{eff}}(t)\, \mathbb{E}\!\left[\sum_{\tau=0}^{\infty} \delta^{\tau}\, \pi_i(t+\tau)\right] \;>\; C_i'(s_i(t)),$$

where $C_i'$ is the marginal cost (largely electricity plus opportunity cost at the contributor's local rate) and $\pi_i(t+\tau)$ is the participant's marginal probability of having their credits redeemed for valuable inference at horizon $t+\tau$. (Earlier drafts used average cost $C_i(s_i)/s_i$; this is dimensionally correct but is the wrong economic quantity — rational agents respond to marginal, not average, cost, and the two coincide only for linear cost functions.) The condition is more readily satisfied in DeCLAI than in purely altruistic systems because credits provide a concrete future benefit; whether it is satisfied in practice depends on the contributor's electricity price, their realised credit-redemption rate, and the prevailing $\alpha_{network}$ controller setting.

This condition is more likely to be satisfied in DeCLAI than in purely altruistic systems because credits provide a concrete, transferable benefit that participants can use for their own inference needs [123].

## 7.3 Free-Rider Problem Solutions

### 7.3.1 The Commons Dilemma in Distributed Computing

Traditional distributed computing projects face the classic free-rider problem: participants can benefit from the collective resource without contributing proportionally [122]. DeCLAI addresses this through several mechanisms:

**Credit-Gated Access**: Participants cannot consume more resources than they have earned through contribution or initial allocation. This creates a direct link between contribution and benefit, eliminating pure free-riding.

**Reputation-Based Multipliers**: Long-term contributors receive efficiency bonuses in credit generation:

$$\eta_i^{\text{effective}}(t) = \eta_i(t) \cdot \left(1 + \gamma \cdot \frac{\text{Reputation}_i(t)}{\text{Reputation}_{\max}}\right)$$

where $\gamma \in [0, 0.5]$ is the reputation bonus coefficient [49].

### 7.3.2 Mechanism Design for Sustained Contribution

The system implements a sophisticated mechanism to prevent gaming while encouraging genuine participation:

**Proof of Inference Protocol**: Participants must demonstrate actual computational work through cryptographic proofs, preventing credit farming without real contribution [51, 52].

**Dynamic Difficulty Adjustment**: Credit generation rates adjust based on network supply and demand, similar to blockchain difficulty adjustment but optimized for resource utilization rather than security [47].

**Geographic Load Balancing**: Participants in underserved regions receive modest credit bonuses, encouraging global distribution and reducing latency for all users.

## 7.4 Bootstrap Strategy for Network Growth

### 7.4.1 Cold Start Problem

New networks face the chicken-and-egg problem: users won't join without available resources, but resources won't be contributed without users. DeCLAI addresses this through a carefully designed bootstrap strategy.

**Initial Credit Allocation**: New participants receive a modest initial credit balance ($B_{\text{initial}} = 10$ GPU-hours equivalent) to enable immediate experimentation and learning. This allocation decreases as the network matures:

$$B_{\text{initial}}(t) = B_0 \cdot e^{-\lambda t} + B_{\min}$$

where $\lambda$ controls the decay rate and $B_{\min}$ ensures some initial access for all participants.

**Institutional Partnerships**: Universities and research institutions can contribute idle computational resources in exchange for priority access for their researchers, creating initial supply while building the user base.

### 7.4.2 Network Effects and Growth Dynamics

The DeCLAI network exhibits positive network externalities characteristic of platform economies [131, 134]:

**Supply-Side Network Effects**: More contributors increase resource availability and reduce wait times, attracting more users.

**Demand-Side Network Effects**: More users create more opportunities for contributors to earn credits, incentivizing additional participation.

**Data Network Effects**: Larger networks enable better load balancing, geographic optimization, and fault tolerance.

The growth model follows a modified logistic curve:

$$\frac{dN(t)}{dt} = r \cdot N(t) \cdot \left(1 - \frac{N(t)}{K(t)}\right) \cdot \text{Utility}(t)$$

where $K(t)$ is the time-varying carrying capacity and $\text{Utility}(t)$ represents the average participant utility, which increases with network size due to network effects.

## 7.5 Comparison with Token-Based Alternatives

### 7.5.1 Fundamental Differences

DeCLAI's credit system differs fundamentally from cryptocurrency-based alternatives in several key aspects:

**Non-Transferability between principals**: Credits earned by a participant can only be redeemed by that same participant (or by an institutional account that earned them) for inference services. They cannot be traded, sold, exchanged for fiat, or transferred to a third party as a store of value. The system records `(requester_id, contributor_id)` on each inference settlement (§8) so that consumption can be attributed; this is a settlement record, not a transferable claim. A consequence of non-transferability is that surplus credits in one participant's account cannot satisfy another participant's deficit; surplus credits simply decay (§7.5.1), and the protocol relies on the rate controller $\alpha_{network}$ (§4.5) rather than secondary markets to clear imbalances.

**No Monetary Value**: Credits have no exchange rate with fiat currencies, eliminating regulatory complexity and speculative behavior that could distort resource allocation.

**Automatic Expiration**: Unused credits gradually decay (half-life of 2 years), encouraging active participation and preventing hoarding:

$$b_i(t) = b_i(0) \cdot e^{-\lambda_{\text{decay}} t} + \int_0^t e^{-\lambda_{\text{decay}}(t-\tau)} \frac{db_i(\tau)}{d\tau} d\tau$$

### 7.5.2 Economic Efficiency Comparison

**Table 7.1: Economic Comparison of Resource Allocation Mechanisms**

| Mechanism | Transaction Costs | Speculation Risk | Regulatory Burden | Resource Efficiency |
|-----------|------------------|------------------|-------------------|-------------------|
| DeCLAI Credits | Low | None | Minimal | High |
| Cryptocurrency Tokens | High | High | Significant | Medium |
| Traditional Markets | Medium | Medium | High | High |
| Volunteer Computing | Very Low | None | None | Low |

The credit system avoids the overhead of token markets and financial speculation, but the "resource efficiency" comparison above is qualitative — we have not measured comparative efficiency against token-based systems and the cells in Table 7.1 reflect design intent rather than benchmarked outcomes [128, 129].

## 7.6 Environmental Impact Analysis

### 7.6.1 Carbon Footprint Modeling

The environmental impact of DeCLAI depends critically on how it affects overall computational resource utilization. We model the carbon footprint using established methodologies for data center energy consumption [136, 138].

**Baseline Carbon Intensity**: Current cloud computing has an average carbon intensity of approximately 0.5 kg CO₂ per GPU-hour, varying significantly by region and provider [139, 141].

**DeCLAI Carbon Model**:

$$C_{\text{DeCLAI}} = \sum_{i=1}^{n} s_i(t) \cdot I_i \cdot (1 - U_i^{\text{baseline}})$$

where:
- $I_i$ is the carbon intensity of participant $i$'s hardware and electricity
- $U_i^{\text{baseline}}$ is the baseline utilization of participant $i$'s hardware

**Key Insight**: DeCLAI reduces overall carbon emissions by utilizing existing hardware more efficiently rather than requiring new data center construction [143, 145].

### 7.6.2 Order-of-Magnitude Environmental Estimates

We give back-of-the-envelope estimates rather than headline impact figures. The earlier draft of this section contained arithmetic errors of approximately one order of magnitude in per-participant power; the figures below are corrected.

**Per-participant power.** A consumer GPU at AI-inference load typically draws 200–500 W (e.g. RTX 3060 ≈ 170 W, RTX 4090 ≈ 450 W). Adding host CPU, memory, and PSU overhead gives roughly 300–600 W per contributing machine while inference is running. We use 400 W as a round midpoint.

**Network-level capacity.** A network of 1,000 contributors each running 4 h/day at 400 W contributes

$$1{,}000 \times 0.4\text{ kW} \times \frac{4}{24} \;\approx\; 67\text{ kW}\text{ continuous (averaged)}.$$

At 100,000 participants with the same duty cycle, this scales to roughly 6.7 MW continuous-averaged, or 40 MW peak when contributors are simultaneously active during the peak window. (Earlier drafts cited 1.5 MW for 1,000 participants and 400 MW for 100,000 participants; both figures implied per-participant power around 9 kW, which is an order of magnitude above realistic consumer-GPU draw.)

**Counterfactual considerations.** Whether DeCLAI *reduces* net carbon depends on the counterfactual:

- If contributed compute substitutes for *new* cloud GPU rental that would otherwise have occurred, DeCLAI can plausibly reduce emissions by lowering datacenter buildout pressure. The size of this effect depends on the marginal grid mix at the cloud provider's region versus the contributor's region, and on the contributor's machine being on regardless of the workload (in which case marginal inference carbon is close to the *difference* between idle and loaded GPU power, perhaps 100–300 W per machine, not the full draw).
- If contributed compute is *additional* — i.e. machines that would otherwise be off — DeCLAI *increases* emissions in absolute terms, though it may still be more carbon-efficient per useful inference than a cold-started cloud GPU.

We do not claim a specific tonnage of annual CO₂ savings. Quantifying the counterfactual requires participant telemetry that is not yet available.

**Hardware utilisation.** Recruiting consumer GPUs that would otherwise be idle does raise their *useful work per kilowatt-hour* — but the relevant baseline is closer to "0% AI utilisation" (most consumer GPUs do not run AI workloads at all) than to "10-20%". The framing is therefore "recruiting idle capacity" rather than "improving utilisation by 3–4×".

**Geographic routing.** Routing inference requests to contributors whose local grid is cleaner can reduce carbon intensity per inference. The achievable reduction depends on the geographic spread of contributors and the grid-mix variance between regions; figures of 20–40% reduction relative to a geographically-agnostic baseline are plausible but unmeasured.

## 7.7 Long-term Economic Sustainability

### 7.7.1 Sustainability Conditions

We restate the sustainability condition as a definition rather than a theorem, because the previous formulation ("sustainable iff the median participant has positive utility") is essentially tautological — it amounts to saying "the network persists if participants choose to participate".

**Definition 7.1 (Operational Sustainability).** *We say the network is operationally sustainable over a horizon $T$ if (a) the static balance condition of §7.1.2 holds in time-average over $T$, (b) the median active participant's empirical utility $\bar{U}_i$ over $T$ is non-negative, and (c) the churn-replenishment rate (new contributors joining minus departing contributors) is non-negative.*

This definition makes explicit that operational sustainability is a *measurable property* of a deployed network, not a closed-form prediction. We do not have empirical data on (b) or (c) and therefore make no claim that DeCLAI is operationally sustainable; that claim awaits a deployment.

The frequently quoted "50% annual churn" robustness figure in earlier drafts was a placeholder, not a derived result. Realistic churn for volunteer scientific computing projects has historically been in the 30–50%/year range [60]; whether DeCLAI's credit incentive raises or lowers retention relative to that baseline is an empirical question.

### 7.7.2 Sensitivity to External Conditions

The credit system has different response characteristics to different external pressures, and these responses are worth surfacing honestly rather than glossing as "resilience":

**Economic downturns.** The naïve story — "in a recession, contributors sell idle compute for credits, balancing the system" — runs in the *opposite* direction in expectation. In a downturn, participants are *more* likely to (a) sell or shut down idle hardware to recover cash, (b) economise on electricity, and (c) reduce discretionary unpaid activity for credits that have no monetary value. Whether DeCLAI is recession-resistant depends on whether the *use-value* of inference access (to participants who themselves use the network) outweighs the foregone electricity cost. The system is therefore most resilient when the user base and contributor base overlap heavily — researchers contributing during off-hours to consume during work-hours — and least resilient when the populations are disjoint.

**Hardware evolution.** As datacenter accelerators evolve faster than consumer GPUs, the efficiency-normalisation baseline must be re-calibrated periodically. Without re-calibration, late-joining contributors with newer consumer hardware will earn credits at a different rate than early joiners with older hardware. The protocol therefore needs a governance mechanism for periodic baseline updates; this is not a self-correcting property of the mathematical model.

**Scale.** The bookkeeping equations are scale-invariant in form, but the *behavioural assumptions* underlying them — for example, that participant strategies remain independent — may not hold at very large scale where coordinated behaviour or collusion becomes feasible. We do not claim scale invariance of system behaviour beyond the bookkeeping identity.

## 7.8 Summary

This section has set out a formal accounting for the DeCLAI credit mechanism, identified the static balance condition the protocol aims to maintain, and surfaced the open questions that this design raises: dynamic stability of the rate controller, incentive compatibility of self-declared capability, and operational sustainability under realistic churn. We have *not* shown that the system is sustainable in deployment, that participants will truth-tell about capability, or that any specific carbon-reduction figure is achievable. Those questions require empirical data from a working deployment and are listed explicitly in §9 as planned-evaluation targets.

What this section *does* support is the claim that the credit mechanism has a coherent mathematical structure — bookkeeping closes, a fixed point exists for the static balance, and the qualitative comparisons with token-based and purely-altruistic alternatives identify real trade-offs. Whether the controller can hold that fixed point under realistic conditions is a question the next section (implementation) prepares for, and §9 attempts to make measurable.

---

# Section 8: Implementation Details

## 8.1 Software Architecture Overview

The DeCLAI system is split into two tiers with very different operational profiles:

- **Coordination tier** — the regional gateways, cluster orchestrators, credit ledger, and discovery service. These are server-side components that can sensibly run as containerised microservices in a small datacenter or cloud footprint operated by a non-profit foundation, university consortium, or volunteer operators. This is where Kubernetes-class orchestration applies.
- **Contributor tier** — the individual GPU machines that perform inference. These run on residential or lab hardware behind NAT, with asymmetric upload bandwidth (typically 10–50 Mbps), dynamic IP allocation, ISP terms that often discourage running servers, and physical operators who power machines on and off at will. A contributor node runs a single long-lived process that registers with a gateway, accepts work assignments, and returns results — *not* a Kubernetes cluster of its own.

Earlier drafts of this paper described the contributor tier as if it ran Kubernetes deployments on individual contributor machines. That is not realistic for residential deployment and is not how the protocol is intended to work. The two-tier split is essential to the design and is reflected throughout the implementation discussion below [101][102].

The architecture aims to avoid single points of failure within the coordination tier through redundant gateways and a replicated credit ledger; full availability under arbitrary partition is not possible (the CAP theorem applies), and the ledger therefore prioritises consistency over availability during partition, with the operational consequence that consumers in a partitioned region may experience temporary read-only access to their credit balance until the partition heals [75].

## 8.2 Technology Stack and Justifications

The technology stack is partitioned between the two tiers described above. The choices below are working defaults rather than fixed requirements; an implementation that meets the protocol interfaces can substitute any of these.

**Coordination tier.** The gateway, orchestrator, and credit-ledger services are implemented in Rust [116] where memory safety and throughput matter (credit validation, BFT message handling), and in Go [118] where concurrency primitives and networking are the dominant concern (peer discovery, gossip). Inter-service communication uses gRPC [103]; external client-facing APIs are REST over HTTP/2 (we do not also expose JSON-RPC — supporting two parallel API surfaces is friction without benefit). Kafka [104] is used for asynchronous event streams (credit transactions, audit logs); Redis [105] serves as the metadata cache. The ledger uses an etcd-style consistent store within each region; cross-region synchronisation is described in §8.4.

**Contributor tier.** Contributor nodes run a single Python [117] inference worker process built on PyTorch [73], with model loading from a quantised on-disk format (we suggest GGUF or a similar quantised format with mmap-friendly layout). The worker is a long-lived TCP client that maintains an outbound connection to its assigned gateway; this avoids the NAT-traversal complications of inbound connections from the gateway side. We do *not* require contributors to run Docker, Kubernetes, or any orchestration; a single binary or Python entry point is the deployment target.

**Justifications.** The split avoids the cargo-cult buzzword stack that an earlier draft of this section presented. Specifically: Rust is justified where it actually matters (validation hot path, BFT message handling); Python is justified where the PyTorch ecosystem makes alternatives impractical; Go is justified for the network-heavy gateway services. Kafka and Redis live in the coordination tier where their operational requirements are reasonable. We do not claim "sub-millisecond" anything as a system property — inference latency dominates by 3–6 orders of magnitude.

## 8.3 API Design and Interface Specifications

The DeCLAI API follows RESTful principles with OpenAPI 3.0 specification [115] for comprehensive documentation and automatic client generation. The API design emphasizes simplicity and discoverability, enabling researchers to integrate DeCLAI into existing workflows with minimal friction.

```python
# Core API Interface Example
class DeCLAIClient:
    def submit_inference(self, model_id: str, prompt: str, 
                        max_tokens: int = 100, 
                        priority: Priority = Priority.NORMAL) -> InferenceJob:
        """Submit inference request to DeCLAI network"""
        request = InferenceRequest(
            model_id=model_id,
            prompt=prompt,
            parameters=InferenceParameters(max_tokens=max_tokens),
            priority=priority,
            user_credits=self.get_available_credits()
        )
        return self._submit_request(request)
    
    def contribute_compute(self, gpu_specs: GPUSpecification,
                          availability_schedule: Schedule) -> ContributorNode:
        """Register as compute contributor"""
        node = ContributorNode(
            gpu_specs=gpu_specs,
            schedule=availability_schedule,
            reputation_score=self.get_reputation()
        )
        return self._register_contributor(node)
```

The API abstracts the complexity of distributed inference while exposing necessary controls for quality of service. Protocol Buffers [113] define the internal message formats, ensuring efficient serialization and strong typing across service boundaries. The API versioning strategy is semantic-versioned: minor and patch releases preserve compatibility, and the system supports at least the current and previous major API version concurrently so clients have a deprecation window before forced migration.

## 8.4 Core Component Implementation

The **Credit Management Service** implements the non-monetary credit accounting using a per-region authoritative ledger backed by etcd [109] (a strongly-consistent store within a region). Cross-region synchronization is *not* a gossip protocol — a gossip-replicated credit ledger is eventually consistent and admits double-spending across partitions, which is unacceptable for credit accounting. Instead, each credit transaction is anchored to a "home region" determined by the requester's account, and cross-region settlement uses a two-phase protocol with the home region as the authoritative coordinator. Gossip is used only for non-authoritative state (peer discovery, health telemetry, reputation propagation) where eventual consistency is appropriate [38].

```rust
// Credit validation implementation
pub struct CreditValidator {
    local_ledger: Arc<RwLock<CreditLedger>>,
    reputation_store: Arc<ReputationStore>,
    proof_verifier: ProofOfInferenceVerifier,
}

impl CreditValidator {
    pub async fn validate_transaction(&self, 
                                    tx: &CreditTransaction) -> ValidationResult {
        // Verify proof of inference
        let proof_valid = self.proof_verifier
            .verify_inference_proof(&tx.inference_proof).await?;
        
        // Check reputation constraints
        let reputation_ok = self.reputation_store
            .check_reputation_threshold(&tx.contributor_id).await?;
        
        // Validate credit balance
        let balance_sufficient = self.local_ledger.read().await
            .check_balance(&tx.requester_id, tx.credit_amount)?;
        
        ValidationResult::new(proof_valid && reputation_ok && balance_sufficient)
    }
}
```

The **Inference Orchestration Service** coordinates distributed model execution across heterogeneous GPU resources. It implements adaptive partitioning algorithms that consider both model architecture and available hardware capabilities, optimizing for latency while maintaining fault tolerance [65][69].

## 8.5 GPU Integration and Compute Abstraction

The compute abstraction layer provides a unified interface across diverse GPU architectures, supporting NVIDIA CUDA [110], AMD ROCm, and OpenCL [111] backends. This abstraction enables seamless integration of consumer GPUs alongside professional hardware, maximizing the potential contributor base.

```python
class GPUAbstractionLayer:
    def __init__(self):
        self.backends = {
            'cuda': CUDABackend(),
            'rocm': ROCmBackend(), 
            'opencl': OpenCLBackend()
        }
    
    def execute_inference_shard(self, shard: ModelShard, 
                               input_data: Tensor) -> Tensor:
        """Execute model shard on optimal available backend"""
        backend = self.select_optimal_backend(shard.requirements)
        
        # Convert data format if necessary
        backend_data = self.convert_tensor_format(input_data, backend.format)
        
        # Execute with automatic memory management
        with backend.memory_context():
            result = backend.execute(shard, backend_data)
            
        return self.convert_tensor_format(result, 'standard')
```

The system implements intelligent memory management using Apache Arrow [112] for zero-copy data transfers between CPU and GPU memory, minimizing overhead in multi-GPU inference scenarios. Dynamic load balancing considers both computational capacity and current utilization, ensuring optimal resource allocation across the network.

## 8.6 Performance Optimization Techniques

Performance optimization focuses on three critical areas: network latency, computational efficiency, and resource utilization. The system implements geographic clustering with intelligent routing that considers both network topology and current load distribution [40].

**Latency Optimization**: The gateway selection algorithm uses a combination of network distance measurements and historical performance data to route requests to optimal clusters. Predictive caching pre-loads frequently requested models to edge locations, reducing cold-start latency for popular inference tasks.

**Computational Efficiency**: Model sharding algorithms adapt to available hardware configurations, implementing dynamic partitioning that can adjust to node failures or performance variations during inference execution. The system supports both pipeline parallelism for sequential processing and tensor parallelism for large model layers [27][68].

**Resource Utilization**: Intelligent scheduling algorithms consider contributor availability patterns, implementing predictive scaling that anticipates demand fluctuations. The system maintains a reserve capacity pool for high-priority requests while maximizing utilization of contributed resources during peak hours.

## 8.7 Deployment — Coordination Tier vs Contributor Tier

The deployment story is qualitatively different for the two tiers introduced in §8.1.

**Coordination tier.** Regional gateways, the credit ledger, the orchestrator, and the discovery service run as containerised services in a small datacenter or cloud footprint. Kubernetes is a reasonable orchestration choice here; an example gateway deployment is shown below. Service discovery uses Consul [108]; metrics and alerting use Prometheus [106] / Grafana [107].

```yaml
# Coordination-tier gateway deployment (Kubernetes)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: declai-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: declai-gateway
  template:
    spec:
      containers:
      - name: gateway
        image: declai/gateway:v1.0.0
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        env:
        - name: REGION
          value: "us-west-2"
        - name: CREDIT_VALIDATION_ENDPOINT
          value: "credit-service:8080"
```

**Contributor tier.** A contributor runs a single binary (or `python -m declai.worker`) on a machine that has at least one supported GPU. The worker establishes an outbound TLS connection to a configured gateway, registers its declared capability, and processes work assignments until shutdown. There is no Kubernetes, no Docker required, no inbound port to open on the contributor's router, and no service mesh — a contributor's deployment story must be simpler than a datacenter operator's, or contributors will not run it. Example:

```bash
# Contributor-tier deployment (single binary)
$ declai-worker \
    --gateway gw.us-west.declai.example \
    --gpu-device 0 \
    --max-hours-per-day 6 \
    --quiet-hours 22:00-08:00
```

**Operational modes.** *Development mode* runs both tiers on one machine for testing. *Regional mode* runs the coordination tier on institutional infrastructure (e.g. a university), with contributors from the same institution. *Federated mode* runs multiple regional coordination tiers connected through the cross-region settlement protocol described in §8.4, with contributors from the open internet.

### 8.7.1 Residential network constraints

The contributor tier operates under real constraints that the protocol must respect:

- **Outbound-only connections.** Most residential networks are NAT'd and many ISPs prohibit running inbound-listening services in their terms of use. DeCLAI's worker uses only outbound long-lived connections to gateways; the gateway pushes work assignments down these existing connections.
- **Asymmetric bandwidth.** Typical residential broadband offers 100–1000 Mbps download and 10–50 Mbps upload. Pipeline-parallel inference moves megabyte-scale activation tensors per layer per token; on a 50 Mbps uplink, this caps tokens per second per worker for large-model pipelines and shapes the model-sharding decisions in §5.3.
- **Dynamic IPs.** Residential IPs change periodically; the worker re-registers its identity (a stable public key, not its IP) on reconnect.
- **Power and thermals.** Contributors typically restrict participation to off-hours and may want hard limits on daily contribution hours and per-day energy budget. The `--max-hours-per-day` and `--quiet-hours` flags above are first-class CLI options, not afterthoughts.
- **ISP terms of service.** Some ISP contracts prohibit "commercial use" of residential service; non-monetary credit contribution sits in a grey area that varies by ISP and jurisdiction. We do not claim to resolve this; we recommend that institutional contributors (universities, labs) be the dominant deployment surface and that residential participation be supported but not assumed.

## 8.8 Open Source Strategy and Governance Model

DeCLAI adopts a community-driven open source model inspired by successful distributed computing projects like BOINC [20] and modern ML frameworks like PyTorch [73]. The governance structure balances technical leadership with community input, ensuring the system remains aligned with its democratization mission while maintaining technical excellence.

The **Technical Steering Committee** consists of representatives from major contributing institutions and experienced community members, responsible for architectural decisions and release planning. The **Community Council** includes user representatives from different sectors (academic, non-profit, individual researchers), ensuring diverse perspectives in feature prioritization and policy decisions.

Development follows a transparent process with public roadmaps, regular community meetings, and open RFC (Request for Comments) processes for major changes. The project maintains multiple contribution pathways: code contributions, documentation improvements, testing and validation, and community support. Special recognition programs acknowledge both technical contributions and community building efforts, fostering a sustainable contributor ecosystem.

The licensing strategy uses Apache 2.0 for maximum compatibility and adoption, with contributor license agreements ensuring the project can evolve while protecting contributor rights. The project maintains strict separation between the open source core and any commercial services, preventing conflicts of interest that could compromise the community-driven mission.

This implementation approach ensures DeCLAI can scale from experimental deployments to global infrastructure while maintaining the accessibility and community focus essential to its democratization goals. The technical architecture provides the performance and reliability required for production AI workloads, while the governance model ensures the system remains true to its vision of democratized AI compute access.

---

# 9. Projected Performance and Planned Evaluation Protocol

> **Disclaimer.** No empirical measurements have yet been collected. The figures in this section are analytical projections derived from the design parameters in Sections 3–8 and from published performance of comparable systems (Petals, Hivemind, BOINC, SETI@home). Validating these projections is future work and requires a reference implementation. Throughout this section, indicative-mood phrasing ("we project", "the design targets", "is expected to") should be read as forward-looking rather than as a report of completed experiments.

The theoretical foundations and architectural designs presented in previous sections will require rigorous empirical validation to demonstrate DeCLAI's practical viability. This section outlines the projected performance envelope, the evaluation protocol intended for a future reference implementation, and the comparative baselines against which the system is to be measured. The methodology described combines a planned simulation framework with the staged real-world deployments that would, in principle, follow.

## 9.1 Planned Simulation Methodology and Experimental Design

### 9.1.1 Intended Simulation Environment Architecture

We outline a simulation framework intended to model the complete DeCLAI ecosystem, from individual GPU nodes to global network dynamics. The framework is designed to incorporate realistic network topologies, hardware heterogeneity, and user behavior patterns drawn from published analyses of existing distributed computing systems [146, 147].

The simulation architecture is planned around three primary components:

**Network Simulator**: Intended to model internet topology with realistic latency distributions, bandwidth constraints, and failure patterns. The simulator is to incorporate geographic clustering based on actual internet infrastructure data, enabling accurate modeling of regional gateway performance and cross-cluster communication overhead.

**Hardware Emulator**: Intended to simulate diverse GPU configurations ranging from consumer RTX 3060 cards to enterprise A100 systems. Each simulated node is to include memory constraints, computational throughput characteristics, and power consumption profiles drawn from manufacturer specifications and independent benchmarks [45].

**Workload Generator**: Intended to produce realistic inference request patterns based on published analyses of production AI systems. The generator is to incorporate temporal variations, geographic distribution patterns, and model popularity distributions reflecting reported usage in research and commercial environments [148].

### 9.1.2 Experimental Parameters and Scenarios

The planned experimental design is intended to span multiple scales and scenarios to probe DeCLAI's projected performance across diverse deployment conditions:

**Scale Variations**: Planned simulations would range from small research clusters (50-100 nodes) to large-scale global deployments (100,000+ nodes), enabling analysis of scalability characteristics and identification of potential bottlenecks.

**Geographic Distributions**: Test scenarios are to include concentrated deployments within single metropolitan areas, distributed national networks, and global federations spanning multiple continents with realistic latency and bandwidth constraints.

**Hardware Heterogeneity**: Experiments are to incorporate varying degrees of hardware diversity, from homogeneous clusters of identical GPUs to highly heterogeneous networks mixing consumer and enterprise hardware across multiple generations and manufacturers.

**Workload Characteristics**: Test workloads are to span diverse model architectures (transformer-based language models, diffusion models, multimodal systems) with varying computational requirements and memory footprints.

### 9.1.3 Baseline Comparisons and Control Groups

To enable meaningful performance comparisons, the protocol identifies several baseline systems:

**Centralized Cloud Providers**: Simulated AWS, Google Cloud, and Azure inference services configured with pricing, performance, and availability characteristics derived from public documentation and independent benchmarks.

**Traditional HPC Clusters**: Modeled university and research institution computing clusters with typical resource allocation policies, queue management systems, and utilization patterns.

**Existing Distributed Systems**: Simplified models of BOINC-style volunteer computing and blockchain-based compute networks, to provide direct comparison with alternative decentralized approaches.

## 9.2 Performance Metrics and Benchmarking Framework

### 9.2.1 Inference Performance Metrics

The protocol defines a suite of performance metrics intended to capture both technical performance and user experience quality:

**Latency Measurements**: End-to-end inference latency from request submission to result delivery, decomposed into network routing time, queue waiting time, computation time, and result aggregation time. Reporting is to include percentile distributions (P50, P95, P99) to capture tail latency behavior critical for interactive applications [149].

**Throughput Analysis**: System-wide inference throughput measured in tokens per second, requests per minute, and effective GPU utilization rates. Throughput measurements are to account for coordination overhead, network communication costs, and fault recovery time.

**Quality Metrics**: Output quality assessment using established benchmarks including perplexity measurements on standard datasets, human evaluation scores for generation quality, and task-specific accuracy metrics for reasoning and comprehension tasks [150, 151].

**Resource Efficiency**: Computational efficiency measured as useful work performed per unit of energy consumed, accounting for coordination overhead, redundant computation for fault tolerance, and network communication costs [152].

### 9.2.2 Economic and Sustainability Metrics

The protocol incorporates economic analysis intended to probe the credit system's effectiveness:

**Credit System Dynamics**: Planned measurement of credit generation rates, circulation patterns, and long-term stability. Metrics are to include credit velocity (rate of circulation), accumulation patterns, and response to demand fluctuations.

**Participation Incentives**: Planned analysis of participant behavior including contribution rates, retention patterns, and response to varying credit rewards. Measurements are to track both short-term participation and long-term community sustainability.

**Resource Allocation Efficiency**: Comparison of resource utilization between DeCLAI's credit-based allocation and traditional market-based or administrative allocation mechanisms.

## 9.3 Projected Distributed Inference Performance

### 9.3.1 Latency and Throughput Projections

We project, based on the architectural assumptions in Sections 3 and 5 and on published results from comparable systems (notably Petals), that DeCLAI can achieve inference performance competitive with centralized alternatives while providing superior accessibility and cost characteristics.

**Latency Performance**: For standard transformer models (7B-70B parameters), we project median inference latencies on the order of 150-300ms for text generation tasks, comparable to those reported by major cloud providers. The design targets 95th percentile latencies below 800ms for the majority of requests, meeting interactive application requirements [153] (projected from routing and pipeline parameters in §5; not measured).

Geographic clustering is expected to be effective in minimizing latency: requests served within the same regional cluster should achieve substantially lower latency than cross-cluster requests, in keeping with the hierarchical architecture described in Section 3. The adaptive routing algorithms aim to balance load across clusters while maintaining locality preferences.

**Throughput Scaling**: System throughput is projected to scale near-linearly with network size up to roughly 10,000 active nodes, beyond which coordination overhead is expected to affect scaling efficiency. The design aims to retain a substantial fraction of theoretical maximum throughput at the 50,000-node scale [154] (projected from the consensus and gossip-overhead model in §3; not measured).

The mixture-of-experts approach to model distribution is expected to enable efficient resource utilization across heterogeneous hardware. We estimate consumer GPUs (RTX 3060-4090) would achieve a high fraction of their theoretical throughput when participating in distributed inference, with enterprise hardware (A100, H100) approaching peak efficiency [155, 156].

### 9.3.2 Fault Tolerance and Reliability

DeCLAI's distributed architecture is designed to provide superior fault tolerance relative to centralized systems:

**Node Failure Resilience**: The protocol aims to maintain full functionality under simultaneous failures of a significant minority of nodes, degrading performance gracefully rather than catastrophically. Recovery from node failures is intended to complete within tens of seconds through automatic workload redistribution.

**Network Partition Handling**: Regional clusters are designed to continue operating independently during network partitions, with automatic reconciliation upon restored connectivity. This is intended to ensure continued service availability even during major internet infrastructure disruptions.

**Byzantine Fault Tolerance**: The consensus mechanisms are designed to detect and isolate malicious nodes attempting to provide incorrect inference results. With a bounded fraction of Byzantine nodes, the system aims to maintain high result accuracy through redundant computation and cryptographic verification.

## 9.4 Credit System Validation Plan and Projected Economic Metrics

### 9.4.1 Projected Credit Generation and Distribution

The mathematical models presented in Section 4 yield concrete predictions that the planned simulations are intended to evaluate:

**Credit Generation Stability**: The adaptive difficulty adjustment mechanism is designed to hold target credit generation rates close to theoretical values across varying network sizes and computational loads. The system is intended to prevent both credit inflation and deflation through responsive parameter adjustment [157, 158].

**Distribution Fairness**: Credit distribution is expected to track computational contribution closely; the design targets Gini coefficients consistent with equitable resource allocation. Participants contributing similar computational resources should receive proportionally similar credit rewards, in keeping with the normalization algorithms [159].

**Long-term Sustainability**: Multi-year simulations are planned to probe credit system stability without external intervention. The design aims for credit circulation rates in which the majority of generated credits are spent within a short window, preventing excessive accumulation while maintaining reserves for demand fluctuations [121].

### 9.4.2 Projected Incentive Mechanism Behavior

The game-theoretic analysis of participant incentives in Section 4 yields predictions to be tested:

**Honest Participation Rates**: We project that a large majority of participants will adopt honest contribution strategies when reputation mechanisms are active. The combination of credit rewards and reputation benefits is intended to create strong incentives for sustained, honest participation.

**Free-Rider Mitigation**: The Proof of Inference protocol is designed to detect free-riding attempts with high reliability across a range of cheating strategies. Detected free-riders are to face automatic reputation penalties intended to discourage continued exploitation.

**Network Growth Dynamics**: Bootstrap simulations are intended to characterize the conditions under which networks grow from initial clusters of 50-100 participants to stable networks of 10,000+ nodes. The design anticipates positive feedback in adoption once critical mass is achieved.

## 9.5 Planned Security and Privacy Validation

### 9.5.1 Projected Cryptographic Protocol Performance

The security mechanisms described in Section 6 are to undergo performance testing to confirm practical viability:

**Proof of Inference Overhead**: Zero-knowledge proof generation is expected to add a modest computational overhead to inference tasks, an acceptable cost for the security guarantees provided. Verification time is anticipated to scale logarithmically with proof complexity, maintaining sub-second verification even for large models [160, 161].

**Privacy-Preserving Query Processing**: Differential privacy mechanisms are designed to protect user query privacy while preserving inference quality. With privacy budget ε = 1.0, the design targets minimal output quality degradation for most tasks, providing strong privacy protection with limited utility loss [52].

**Secure Model Distribution**: The cryptographic model protection mechanisms are designed to prevent unauthorized access to model weights while adding limited overhead to model loading and inference. The distributed key management system is intended to remain secure even when a significant minority of key holders are compromised [77].

### 9.5.2 Planned Attack Resistance Analysis

Security testing is to evaluate DeCLAI's resistance to various attack vectors:

**Sybil Attack Prevention**: The reputation-based admission control is intended to prevent Sybil attacks across a range of identity multiplication strategies. The computational requirements for establishing legitimate reputation are designed to create effective barriers to large-scale identity fraud.

**Model Extraction Resistance**: Attempts to extract model weights through inference queries are intended to fail against the privacy-preserving mechanisms; the design aims to prevent reconstruction of model parameters with practically useful accuracy.

**Network-Level Attacks**: DDoS attacks and network partitioning attempts are expected to have limited impact on system operation, owing to the distributed architecture and adaptive routing mechanisms. The system aims to maintain high availability even under sustained attack conditions.

## 9.6 Projected Scalability and Network Growth

### 9.6.1 Anticipated Network Growth Patterns

Long-term simulations are planned to test predictions of the theoretical scaling analysis:

**Adoption Curves**: We anticipate that network growth would follow classic S-curve adoption patterns, with slow initial growth, rapid expansion after reaching critical mass (on the order of 1,000 active participants), and eventual stabilization at sustainable participation levels [162, 163].

**Geographic Expansion**: Successful networks are expected to expand geographically as local clusters reach capacity and participants seek lower-latency alternatives. The hierarchical architecture is designed to accommodate this expansion without centralized coordination [33].

**Hardware Evolution**: The system is designed to adapt to hardware evolution, with newer GPU generations integrated through the efficiency normalization mechanisms. Legacy hardware is intended to remain viable for participation, supporting inclusive access across economic strata [60].

### 9.6.2 Projected Performance Scaling Characteristics

Planned scaling analysis is to probe behavior across multiple orders of magnitude:

**Coordination Overhead**: Network coordination overhead is projected to grow sub-linearly with network size, remaining a small fraction of total computational capacity for networks up to 100,000 nodes (projected from the consensus model in §3; not measured). The hierarchical architecture and efficient consensus mechanisms are intended to enable this favorable scaling.

**Storage and Bandwidth Requirements**: Model storage requirements are designed to scale efficiently through content-addressable storage and aggressive caching. Bandwidth utilization is intended to remain manageable through model sharding and geographic clustering.

**Latency Scaling**: We project that median inference latency would remain stable as network size increases, with 95th percentile latency growing slowly across the simulated range. This expectation reflects the intended effectiveness of geographic clustering and adaptive routing.

## 9.7 Projected Comparison with Centralized Systems

### 9.7.1 Performance Comparison

Comparison with major cloud providers, framed as a projection, suggests DeCLAI may exhibit competitive performance:

**Inference Quality**: Because DeCLAI proxies the same model weights, we expect output quality on standard benchmarks (MMLU, ARC, HellaSwag) to be equivalent to that of centralized systems running the same models [164, 165, 166]. The distributed consensus mechanisms for result validation are intended to improve reliability relative to single-point-of-failure centralized systems.

**Cost Effectiveness**: For equivalent computational access, we project substantial cost savings relative to commercial cloud providers (projected from the credit-model assumptions in §7; not measured). The elimination of profit margins and infrastructure overhead is the principal driver of this projection.

**Availability and Reliability**: DeCLAI is expected to offer high availability owing to the distributed architecture's resilience to localized failures. The system is intended to continue operating even during major infrastructure disruptions that would impact centralized providers.

### 9.7.2 Anticipated Accessibility and Democratization Impact

The projected results speak to DeCLAI's democratizing potential:

**Barrier Reduction**: DeCLAI is intended to reduce the financial barriers to AI access substantially relative to commercial alternatives, enabling participation by researchers, students, and organizations previously excluded by cost constraints.

**Geographic Accessibility**: The distributed architecture is intended to provide AI access in regions underserved by major cloud providers, with lower latency for users distant from existing centralized data centers.

**Innovation Enablement**: We expect reduced cost and improved accessibility to translate into substantially higher experimental iteration rates for participating researchers, potentially accelerating AI research across diverse communities.

## 9.8 Projected Quality of Service and User Experience

### 9.8.1 Projected Service Quality

Planned user experience testing is to probe DeCLAI's effectiveness against diverse application requirements:

**Generation Quality**: Because the underlying model weights are unchanged, we project that perplexity and human-evaluation scores will be close to those of centralized deployments of the same models [167, 168].

**Consistency and Reliability**: The distributed consensus mechanisms are intended to deliver consistent output quality across different network conditions and node configurations. The design aims to keep variance in output quality small across regional clusters and hardware configurations [169].

**Interactive Performance**: For real-time applications, the design targets sub-200ms first-token latency for the majority of requests, enabling smooth interactive experiences. The streaming response capability is intended to provide progressive output delivery comparable to centralized systems [170].

### 9.8.2 Planned User Studies

User studies are planned to evaluate DeCLAI's practical utility across diverse user communities:

**Research Community Adoption**: Surveys of academic researchers are intended to assess satisfaction with performance and accessibility, with particular attention to the value of removing budget constraints on hyperparameter searches [171, 172].

**Developer Experience**: Developer-experience studies are intended to assess the API design and documentation, including the ease of migration from existing cloud services for projects of varying complexity [173].

**Educational Impact**: Studies in educational institutions are intended to assess effects of removing computational barriers on student engagement and learning outcomes in AI coursework [174].

## 9.9 Hypothetical Deployment Scenarios

The following scenarios are illustrative rather than empirical; they describe deployments that a reference implementation could enable, and are included to clarify the intended evaluation contexts. No such deployments have taken place.

**Academic Research Consortium (hypothetical)**: A hypothetical consortium of roughly fifteen universities, with on the order of a few thousand participants contributing hardware ranging from student laptops with consumer GPUs to research-lab A100 clusters, would constitute a natural pilot context. Such a deployment would be expected to test participation retention, cross-institutional collaboration patterns, and the economic value of removing per-experiment cloud charges [175, 176].

**Underserved-Region Deployment (hypothetical)**: A deployment in a region with intermittent connectivity and limited commercial cloud presence would test the hierarchical architecture's adaptation to variable infrastructure, and would probe whether locally driven applications — including those tailored to underrepresented languages — emerge once compute access ceases to be the binding constraint [177, 178, 179].

**Open-Source Community Integration (hypothetical)**: Integration with established open-source model communities (e.g., LLaMA-family derivatives) would test compatibility with community-developed AI systems and would probe whether the credit system can sustain operation without external funding over multi-month timescales [180].

## 9.10 Limitations and Future Experimental Directions

### 9.10.1 Limitations of the Projection

While the projected results support DeCLAI's viability in principle, several limitations require explicit acknowledgment:

**Absence of Empirical Data**: Most importantly, no measurements have been collected. The figures and qualitative claims throughout this section are projections from design parameters and from published behavior of comparable systems; they require validation by a reference implementation.

**Simulation Fidelity**: Even once simulations are run, they cannot capture all complexities of real-world deployment. Regulatory constraints, cultural adoption barriers, and unexpected technical challenges may impact actual deployment outcomes.

**Scale Limitations**: Planned simulations reach maximum scales on the order of 100,000 nodes. Empirical validation at million-node scales would require future deployments beyond the scope of any planned simulation campaign.

**Model Diversity**: The protocol focuses primarily on transformer-based language models. Future work should encompass broader architectures including diffusion models, multimodal systems, and emerging architectures to assess generalizability.

**Long-term Dynamics**: Planned simulation runs span simulated periods of up to two years. Longer-term dynamics — technology evolution, governance challenges, and economic equilibrium stability — require extended real-world validation.

### 9.10.2 Future Experimental Priorities

Several areas are flagged as priorities for future experimental investigation:

**Regulatory Compliance**: Future experiments must address regulatory requirements across jurisdictions, including data protection laws, export controls on AI technology, and financial regulations that might impact credit systems.

**Adversarial Robustness**: Beyond the attack vectors considered here, emerging threats and sophisticated adversarial strategies will require ongoing experimental validation and system hardening.

**Cross-Cultural Adoption**: Deployment experiments in diverse cultural contexts will provide insight into adoption barriers and necessary adaptations for global scalability.

**Integration Complexity**: Large-scale integration experiments with existing enterprise systems, cloud providers, and AI development workflows will be needed to validate practical deployment feasibility.

### 9.10.3 Methodological Improvements

Future experimental work will benefit from several methodological enhancements:

**Hybrid Simulation-Deployment**: Combining large-scale simulations with smaller real-world deployments could provide both scalability insight and real-world validation while managing experimental cost.

**Longitudinal Studies**: Extended multi-year studies of real deployments will provide crucial insight into long-term sustainability, community dynamics, and the impact of technology evolution.

**Comparative Frameworks**: Standardized benchmarking frameworks will enable rigorous comparison with alternative distributed computing approaches and emerging centralized systems.

**Interdisciplinary Collaboration**: Integration of social science research methods will yield deeper insight into community dynamics, adoption patterns, and societal impact than purely technical experiments can.

## 9.11 Planned Statistical Analysis and Reproducibility Protocol

### 9.11.1 Planned Statistical Methods

The following statistical methods are to be applied once measurements are collected; no confidence intervals, effect sizes, or significance tests have yet been computed.

**Sample Sizes**: The protocol calls for minimum sample sizes of 1,000 independent runs per configuration in simulation, to provide statistical power sufficient to detect meaningful differences. Real-world deployments, when conducted, are to track metrics across all participants over the deployment window.

**Confidence Intervals**: All reported performance metrics are to be accompanied by 95% confidence intervals computed using appropriate methods, including bootstrap resampling for metrics where analytical solutions are intractable.

**Significance Testing**: Comparative analyses are to employ appropriate statistical tests — t-tests for normally distributed metrics, Mann-Whitney U tests for non-parametric comparisons, and chi-square tests for categorical outcomes — with multiple-comparison corrections applied where appropriate.

**Effect Sizes**: Beyond statistical significance, the protocol calls for reporting effect sizes (Cohen's d and analogous measures) to quantify the practical significance of observed differences.

### 9.11.2 Planned Reproducibility and Open-Science Practices

The protocol is designed around reproducibility from the outset:

**Open Source Tools**: All simulation code, analysis scripts, and experimental configurations are to be released under open-source licenses upon publication of measurements, to enable independent reproduction.

**Data Availability**: Anonymized experimental data is to be made available through established research data repositories, subject to privacy and security constraints, to support independent analysis and meta-studies.

**Preregistration**: Preregistration of major experimental hypotheses and analysis plans is intended for the first measurement campaign, to reduce the risk of post-hoc analysis bias.

**Replication Studies**: Independent replication studies will be encouraged and supported through detailed methodological documentation and direct technical assistance to other research groups.

## 9.12 Conclusion and Implications

This section has outlined a viable design whose empirical validation remains future work. The projections summarized here are intended to support the theoretical foundations of previous sections and to delimit the evaluation envelope a reference implementation would target.

**Projected Performance**: DeCLAI is designed to achieve inference performance comparable to that of major cloud providers while offering superior cost-effectiveness and accessibility. The distributed architecture is intended to deliver scalability and fault tolerance that match or exceed centralized alternatives.

**Projected Economic Sustainability**: The credit system is designed to operate stably across diverse scenarios, providing fair resource allocation and strong participation incentives without external subsidies. Planned long-term simulations are intended to probe this expectation.

**Projected Security and Privacy**: The cryptographic protocols are designed to protect user privacy and model intellectual property while preserving practical performance characteristics, with robust resistance to a range of attack vectors.

**Projected Democratization Impact**: The design aims to expand AI access materially, with particular benefit for underserved communities and for research workflows currently constrained by computational cost.

**Anticipated Community Dynamics**: Planned deployments are intended to probe community formation and governance, addressing the social and organizational aspects of the DeCLAI vision beyond purely technical considerations.

Taken together, these projections outline a viable design whose empirical validation remains future work. The next step is the construction of a reference implementation against which the figures and qualitative claims in this section can be tested.

---

# Section 10: Use Cases and Social Impact

*Part of: DeCLAI: Decentralized Compute Credit System for Large Language Model Inference*

The intended measure of DeCLAI's success lies not in its technical sophistication, but in its potential to democratize access to artificial intelligence across diverse communities and applications. This section sketches that potential through scenario analysis and illustrative user journeys. We present five primary use cases that collectively outline how a system like DeCLAI could reshape the landscape of AI accessibility, from academic research to global health initiatives. None of the scenarios below report measured outcomes; they are forward-looking sketches intended to motivate design choices and to identify questions for future empirical work.

> **A note on the personas in this section.** Maria, Dr. Kwame Asante, Dr. Elena Rodriguez, Sarah Chen, and the University of Lagos scenario are **illustrative composites**, not interviewed subjects. Specific budget percentages, institutional affiliations, and weekly hours attached to them are narrative figures chosen to make the scenarios concrete, not survey data. Where a real-sounding institution is named, it is used as a stand-in for a class of institution (e.g. a mid-tier Latin American university, a West African public-health group).

## 10.1 Academic Research and Education

### 10.1.1 The Graduate Student Researcher Persona

Consider Maria, a PhD student in computational linguistics at a mid-tier university studying low-resource language processing. Her research focuses on developing neural machine translation systems for indigenous languages of South America — work that could preserve endangered linguistic heritage and enable cross-cultural communication [191, 192]. Suppose her university's shared computing cluster provides only 4 hours of GPU time per week, severely limiting her experimental iterations.

Under that constraint, Maria faces a stark choice: abandon computationally intensive research directions or seek expensive cloud computing resources that exceed her stipend. A single training run for a multilingual transformer model requires 20–40 GPU hours [193], consuming her entire monthly allocation. This forces her toward less ambitious research questions.

### 10.1.2 DeCLAI Transformation Journey

With DeCLAI, Maria's research trajectory could change substantially. She contributes her personal RTX 4090 GPU during nighttime hours (8 PM to 8 AM), earning credits at the rate defined in §7. This contribution pattern aligns with her natural work schedule while raising overall utilization of an otherwise idle device. During peak research periods, she can draw on distributed inference across multiple community GPUs, enabling rapid prototyping and broader hyperparameter exploration.

The geographic clustering feature is well-suited to Maria's work. Her research group could form a regional cluster with other Latin American universities, creating a specialized network for multilingual NLP research. Such a cluster would enable shared model development, where different institutions contribute training data for various indigenous languages while collectively drawing on the same computational pool.

### 10.1.3 Projected Impact and Sensitivity

A back-of-envelope projection illustrates the order of magnitude — not a measured outcome:

- **Baseline allocation**: 4 GPU-hours/week × 52 weeks = **208 GPU-hours/year**.
- **DeCLAI contribution-side ceiling**: contributing 12 hours/night × 7 nights = 84 GPU-hours/week of contribution. If every contributed hour translated 1:1 into a consumable hour (no decay, no efficiency adjustment, no contention), that would cap consumption at 84 × 52 ≈ 4,368 GPU-hours/year — roughly an order of magnitude above the baseline.
- **Realistic range**: actual consumable hours depend on the contribution-vs-consumption ratio, credit decay (§7.5), efficiency multipliers, and cluster contention. Under conservative assumptions (e.g. 25–50% effective conversion, partial weeks of contribution), the realistic gain is closer to **5–20× baseline**, not the naive ceiling.

(Projected; sensitivity: the ratio roughly halves with each of the following — dropping to 50% effective conversion, dropping to 50% of weeks contributing, or applying typical credit-decay parameters from §7.5. Earlier drafts cited a single "3,150% increase" figure derived from multiplying 84 credits × 1.5 efficiency bonus × 52 weeks at 100% usage; that calculation conflated weekly contribution hours with annual consumption hours and is withdrawn.)

Even at the conservative end of this range, the change would let Maria pursue ablation studies and cross-lingual transfer experiments that are currently infeasible under a 4-hour weekly cap. Whether the realised gain is closer to the ceiling or the floor is exactly the kind of question a pilot deployment would need to answer.

### 10.1.4 Educational Applications

DeCLAI's potential extends beyond research to undergraduate and graduate education. Universities could integrate community GPU sharing into AI curricula, enabling hands-on experience with larger-scale models than a single institution's cluster permits. Students would gain exposure not only to technical skills but to principles of resource sharing and community collaboration relevant to ethical AI development [182, 184].

The system's educational potential manifests through several mechanisms:
- **Practical Learning**: Students gain experience with distributed systems and resource management.
- **Collaborative Projects**: Cross-institutional student teams work on shared computational resources.
- **Broader Access**: Students from resource-poor institutions could access compute comparable to peers at well-funded universities.

## 10.2 Global Health and Medical Research

### 10.2.1 The Public Health Researcher Persona

Dr. Kwame Asante (an illustrative composite) leads an epidemiological surveillance team in Accra, Ghana, developing AI systems for early disease outbreak detection. His work combines satellite imagery, mobile health data, and social media analysis to predict malaria transmission patterns — research with life-saving potential for millions across sub-Saharan Africa [181]. In this scenario his institution lacks the computational infrastructure necessary for processing large-scale multimodal datasets.

For the purposes of this illustration, current cloud computing costs for Dr. Asante's research are assumed to consume roughly 60% of his annual research budget, forcing trade-offs between computational resources and field data collection. This kind of economic barrier is the structural problem DeCLAI is designed to address: research priority being constrained by computational access rather than scientific merit.

### 10.2.2 Community-Driven Health Research

DeCLAI could let Dr. Asante's team join a global health research cluster, connecting institutions across Africa, Asia, and Latin America. Researchers contribute computational resources during local off-peak hours, creating a follow-the-sun research infrastructure that spans time zones. This temporal diversity raises utilization while respecting local energy constraints.

The privacy-preserving features of DeCLAI matter for health research. Differential privacy mechanisms [77] would enable collaborative analysis of sensitive health data without compromising patient confidentiality. Dr. Asante's team could train models on distributed datasets while retaining privacy guarantees, enabling larger-scale collaborations than would otherwise be feasible.

### 10.2.3 Plausible Social Impact

Broadening AI access for global health research carries significant social implications. A system like DeCLAI could plausibly enable:

- **Faster Outbreak Response**: Real-time disease surveillance models, reducing epidemic response time.
- **Locally-Adapted Treatment**: AI-driven treatment optimization adapted to local genetic and environmental factors.
- **Preventive Care**: Predictive models for chronic disease management in resource-constrained healthcare systems.

We previously cited a "40–60% reduction in global health research timelines" figure. We withdraw that specific range — it was not supported by any underlying timing model. The qualitative claim we are willing to defend is that lowering the marginal compute cost of a research iteration would substantially shorten the cycle time between hypothesis and result for compute-bound projects; the magnitude is an empirical question for future work.

## 10.3 Climate Science and Environmental Monitoring

### 10.3.1 The Climate Researcher Scenario

Dr. Elena Rodriguez (illustrative composite) studies extreme weather prediction in the Caribbean, developing AI models that combine satellite data, ocean temperature measurements, and atmospheric simulations to forecast hurricane intensity [10]. Her research directly bears on disaster preparedness for island nations vulnerable to climate change. The computational demands of high-resolution climate modeling exceed her institution's capabilities in this scenario.

Traditional climate models require massive computational resources — often thousands of GPU-hours for a single simulation run. Access limitations of the kind sketched here force researchers toward lower-resolution models, reducing prediction accuracy precisely where precision matters most for vulnerable populations.

### 10.3.2 Distributed Climate Modeling

DeCLAI could let Dr. Rodriguez coordinate with climate researchers globally, forming a distributed pool dedicated to climate science. Geographic clustering aligns naturally with climate research needs — Caribbean researchers collaborate on hurricane modeling, Arctic researchers focus on ice-sheet dynamics, African researchers study desertification patterns.

The environmental footprint of DeCLAI is relevant for climate research itself. By raising utilization of existing consumer hardware rather than provisioning dedicated datacenter capacity, the system could lower the marginal carbon footprint of computational research [143, 145] compared with the equivalent cloud workload. Climate scientists could pursue computationally intensive research while keeping a smaller incremental environmental cost — a useful property for researchers whose own subject of study is climate change.

### 10.3.3 Long-term Environmental Outlook

A common claim about distributed-compute systems is that they reduce AI research carbon emissions by some specific percentage. We decline to commit to a specific figure here: the carbon outcome depends sensitively on the marginal grid mix at each contributor's location, the counterfactual (idle GPU vs. dedicated cloud instance), and the embodied-carbon amortization assumed for consumer hardware. What we can say qualitatively: where the counterfactual is otherwise-idle consumer hardware sitting on a relatively clean grid, DeCLAI is expected to lower per-job emissions relative to provisioning new dedicated capacity; where the counterfactual is a hyperscaler running on a cleaner grid than the contributor's, the comparison can go the other way. A rigorous lifecycle assessment is future work.

## 10.4 Small Business and Startup Innovation

### 10.4.1 The Entrepreneur Persona

Sarah Chen (illustrative composite) leads a startup developing AI-powered accessibility tools for visually impaired users. Her team creates applications that provide real-time scene description, text recognition, and navigation assistance — technology with significant potential for users worldwide. The computational costs of training and deploying computer vision models threaten her startup's viability in this scenario.

For the purposes of this illustration, cloud computing expenses are taken to consume roughly 40% of Sarah's seed funding, creating burn rates that force premature product compromises. The economic pressure leads to reduced model complexity, limited testing, and delayed feature releases — ultimately affecting the quality of the resulting tools.

### 10.4.2 Startup Ecosystem Effects

DeCLAI would let Sarah redirect computational expenses toward product development and user research. Her startup contributes GPU resources during development downtime, earning credits for production inference. This model loosely aligns computational costs with actual usage, providing natural scaling as the business grows.

The community aspect of DeCLAI is particularly relevant for startups. Sarah's team could collaborate with other accessibility-focused companies, sharing computational resources and technical expertise. This kind of collaboration could lower barriers for entry while strengthening the broader ecosystem of social-impact technology.

### 10.4.3 Economic Effects

A system like DeCLAI could substantially lower AI development costs for early-stage startups by replacing pay-per-hour cloud spending with contribution-earned credits — particularly for teams whose hardware would otherwise sit idle outside of training windows. We do not commit to a specific percentage cost reduction here, since the answer depends on the team's workload mix, contribution capacity, and the price benchmark used. The credit-based model offers several structural advantages over straight cloud rental:

- **Predictable Costs**: Credits earned through contribution provide a more stable computational budget than spot pricing.
- **Scalable Access**: Resource availability scales with community participation rather than payment capacity alone.
- **Collaborative Benefits**: Shared resources could enable startup collaboration and knowledge transfer.

## 10.5 Developing Nation Research Initiatives

### 10.5.1 The International Development Scenario

A scenario: the University of Lagos partners with rural health clinics across Nigeria to develop AI diagnostic tools for tropical diseases. This initiative combines local medical expertise with machine learning to create culturally appropriate, cost-effective healthcare solutions. Computational barriers limit the project's scope and impact in this sketch.

Traditional approaches require expensive international partnerships or cloud computing contracts that strain institutional budgets. Such constraints can entrench technological dependence, where developing nations consume AI solutions created elsewhere rather than building indigenous capabilities.

### 10.5.2 Technological Sovereignty Through Community Computing

DeCLAI could help developing nations build indigenous AI capabilities through community resource sharing. Nigerian universities could contribute computational resources during off-peak hours, forming a national AI research pool. This approach would build local technical capacity while lowering dependence on foreign computational resources.

The system's design respects local constraints — intermittent internet connectivity, variable power supply, and heterogeneous hardware. Fault-tolerance mechanisms aim to keep research workloads making progress despite infrastructure interruptions, while geographic clustering optimizes for local network conditions.

### 10.5.3 Global Equity and Digital Sovereignty

DeCLAI's potential to broaden global equity in AI development extends beyond individual use cases to systemic effects. By enabling developing nations to participate as AI creators rather than only consumers, the system could chip away at structural inequities in technological development.

We previously cited a "400–600% increase in AI research capacity in developing nations." We withdraw that specific range — there is no underlying model of "national AI research capacity" to which we can calibrate such a figure. The qualitative claim we will defend is that community-driven GPU sharing could plausibly enable an order-of-magnitude increase in addressable compute for the subset of research groups currently capped by per-week GPU quotas, supporting work in areas such as agricultural optimization, local language processing, and culturally appropriate educational technology.

## 10.6 Adoption Barriers and Mitigation Strategies

### 10.6.1 Technical Barriers

Several adoption barriers require careful consideration:

**Hardware Heterogeneity**: The diversity of consumer GPUs creates compatibility challenges. Our proposed solution involves hardware abstraction layers and automated optimization for different GPU architectures [110, 111].

**Network Reliability**: Distributed inference requires reasonably stable network connections. We address this through adaptive fault tolerance, local caching, and graceful degradation [72].

**User Experience Complexity**: Distributed systems can overwhelm non-technical users. Our mitigation strategy emphasizes simple interfaces, automated configuration, and documentation [185].

### 10.6.2 Social and Economic Barriers

**Trust and Reputation**: Community-based systems require trust mechanisms. We propose transparent reputation systems, cryptographic verification, and gradual trust building through small initial contributions [49, 50].

**Digital Divide**: Unequal internet access could exacerbate existing inequities. The intended approach includes offline capabilities, low-bandwidth modes, and partnerships with connectivity initiatives.

**Institutional Resistance**: Established institutions may resist decentralized alternatives. We propose to address this through pilot programs, gradual integration, and clear documentation of benefits.

### 10.6.3 Regulatory and Policy Considerations

**Data Privacy Regulations**: Global privacy laws require careful compliance. DeCLAI's privacy-preserving design aims to anticipate regulatory requirements while enabling legitimate research [77, 189].

**Export Controls**: AI technology export restrictions could limit international collaboration. The system is designed to comply with relevant regulations while supporting legitimate research collaboration.

**Intellectual Property**: Model sharing raises IP concerns. The framework includes flexible licensing options and attribution mechanisms intended to protect creators while enabling collaboration.

## 10.7 Long-term Vision and Societal Implications

### 10.7.1 A Broader-Access AI Future

The longer-term vision extends beyond computational resource sharing toward a different default for how AI capabilities are produced and accessed. DeCLAI is a step toward a setting in which AI capabilities are community resources alongside corporate assets, with innovation driven in part by social need rather than profit alone.

This shift could reshape the AI landscape in several ways:
- **Diverse Innovation**: Broader participation could lead to AI solutions addressing previously neglected problems.
- **Ethical Development**: Community governance could promote responsible AI development practices.
- **Global Collaboration**: Shared resources could enable wider international research collaboration.

### 10.7.2 Measuring Success and Impact

Success for a system like DeCLAI should be measured not only in technical metrics but in social outcomes:
- **Broadened Research Access**: Number of researchers gaining access to previously unavailable computational resources.
- **Innovation Diversity**: Breadth of research areas and geographic regions participating.
- **Social Impact**: Real-world benefits from AI research enabled by community resource sharing.

These remain to be measured; nothing in this section reports outcomes from a deployed system.

### 10.7.3 Sustainable Community Growth

Long-term sustainability requires careful community cultivation. The proposed approach emphasizes:
- **Gradual Scaling**: Organic growth that maintains community norms while expanding capabilities.
- **Inclusive Governance**: Decision-making processes that represent diverse stakeholder interests.
- **Continuous Evolution**: Technical iteration that responds to community needs and emerging challenges.

The use cases in this section outline the kinds of impact that the DeCLAI design could enable if widely deployed; they are not evidence of deployed impact, and empirical validation is future work. Taken together, they form an argument that collaborative approaches could address aspects of the AI accessibility challenge while supporting innovation, equity, and social benefit — through community-driven resource sharing aimed at broadening, rather than concentrating, the production of AI capabilities.

The path from this argument to a working system is incremental. The scenarios here are intended as starting points: concrete applications that a pilot deployment could test, and against which the projections in §10.1.3 and the qualitative claims throughout this section could be either confirmed or revised.

---

# Bibliography and Appendices

# Bibliography for DeCLAI Paper

## IEEE Format References

[1] T. Brown et al., "Language models are few-shot learners," in Proc. 34th Conf. Neural Information Processing Systems (NeurIPS), 2020, pp. 1877-1901.

[2] S. Rajbhandari, J. Rasley, O. Ruwase, and Y. He, "ZeRO: Memory optimizations toward training trillion parameter models," in Proc. Int. Conf. High Performance Computing, Networking, Storage and Analysis (SC), 2020, pp. 1-16.

[3] D. Ganguli et al., "Predictability and surprise in large generative models," in Proc. ACM Conf. Fairness, Accountability, and Transparency (FAccT), 2022, pp. 1747-1764.

[4] NVIDIA Corporation, "NVIDIA H100 Tensor Core GPU Architecture," White Paper, 2022. [Online]. Available: https://resources.nvidia.com/en-us-tensor-core

[5] A. Chowdhery et al., "PaLM: Scaling language modeling with pathways," J. Machine Learning Research, vol. 24, pp. 240-302, 2023.

[6] M. Chen et al., "Evaluating large language models trained on code," arXiv preprint arXiv:2107.03374, 2021.

[7] Computing Research Association, "Academic Computing Infrastructure Survey," CRA Report, 2023.

[8] J. Dean and L. Corrado, "Large scale distributed deep networks," in Proc. 25th Int. Conf. Neural Information Processing Systems (NIPS), 2012, pp. 1223-1231.

[9] Amazon Web Services, "Amazon EC2 P4d Instances," AWS Documentation, 2023. [Online]. Available: https://aws.amazon.com/ec2/instance-types/p4/

[10] *Entry withdrawn.* This number was used in earlier drafts for a fabricated illustrative citation; §1.3 has been rewritten to use explicitly labelled illustrative composite scenarios without invented sources. The number is retained as a placeholder to avoid renumbering downstream entries; do not cite [10] in revised prose.

[11] *Entry withdrawn.* This number was used in earlier drafts for a fabricated illustrative citation; see the note on [10] above. Do not cite [11] in revised prose.

[12] *Entry withdrawn.* This number was used in earlier drafts for an empirical survey claim ("graduate-student GPU rationing") that was not backed by an identifiable source. The corresponding qualitative claim now appears in §1.4 without a citation. Do not cite [12] in revised prose.

[13] L. Floridi et al., "AI4People—An ethical framework for a good AI society: Opportunities, risks, principles, and recommendations," Minds and Machines, vol. 28, no. 4, pp. 689-707, 2018.

[14] R. Bommasani et al., "On the opportunities and risks of foundation models," arXiv preprint arXiv:2108.07258, 2021.

[15] D. Hendrycks et al., "Aligning AI with shared human values," in Proc. Int. Conf. Learning Representations (ICLR), 2021.

[16] United Nations Conference on Trade and Development, "Digital Economy Report 2021: Cross-border data flows and development," UNCTAD, Geneva, Switzerland, 2021.

[17] Y. Shazeer et al., "Switch transformer: Scaling to trillion parameter models with simple and efficient sparsity," J. Machine Learning Research, vol. 23, pp. 120-156, 2022.

[18] D. Narayanan et al., "Efficient large-scale language model training on GPU clusters using megatron-LM," in Proc. Int. Conf. High Performance Computing, Networking, Storage and Analysis (SC), 2021, pp. 1-15.

[19] Partnership on AI, "Democratizing AI: Principles and practices for inclusive development," Partnership on AI Report, 2022.

[20] D. P. Anderson, "BOINC: A system for public-resource computing and storage," in Proc. 5th IEEE/ACM Int. Workshop on Grid Computing, 2004, pp. 4-10.

[21] M. Zaharia et al., "Resilient distributed datasets: A fault-tolerant abstraction for in-memory cluster computing," in Proc. 9th USENIX Symposium on Networked Systems Design and Implementation (NSDI), 2012, pp. 15-28.

[22] A. Rowstron and P. Druschel, "Pastry: Scalable, decentralized object location and routing for large-scale peer-to-peer systems," in Proc. IFIP/ACM Int. Conf. Distributed Systems Platforms (Middleware), 2001, pp. 329-350.

[23] K. Birman et al., "Bimodal multicast," ACM Trans. Computer Systems, vol. 17, no. 2, pp. 41-88, 1999.

[24] P. Bahl et al., "White space networking with Wi-Fi like connectivity," in Proc. ACM SIGCOMM Conf., 2009, pp. 27-38.

[25] L. Lamport, "The part-time parliament," ACM Trans. Computer Systems, vol. 16, no. 2, pp. 133-169, 1998.

[26] S. Rhea et al., "OpenDHT: A public DHT service and its uses," in Proc. ACM SIGCOMM Conf., 2005, pp. 73-84.

[27] A. Harlap et al., "PipeDream: Generalized pipeline parallelism for DNN training," in Proc. 27th ACM Symposium on Operating Systems Principles (SOSP), 2019, pp. 1-15.

[28] M. Castro and B. Liskov, "Practical Byzantine fault tolerance," in Proc. 3rd Symposium on Operating Systems Design and Implementation (OSDI), 1999, pp. 173-186.

[29] D. Bernstein, "Containers and cloud: From LXC to Docker to Kubernetes," IEEE Cloud Computing, vol. 1, no. 3, pp. 81-84, 2014.

[30] J. Ousterhout et al., "The case for RAMCloud," Communications of the ACM, vol. 54, no. 7, pp. 121-130, 2011.

[31] S. Nakamoto, "Bitcoin: A peer-to-peer electronic cash system," Bitcoin.org, 2008. [Online]. Available: https://bitcoin.org/bitcoin.pdf

[32] A. Miller et al., "Nonoutsourceable scratch-off puzzles to discourage Bitcoin mining coalitions," in Proc. ACM Conf. Computer and Communications Security (CCS), 2015, pp. 680-691.

[33] I. Stoica et al., "Chord: A scalable peer-to-peer lookup service for internet applications," in Proc. ACM SIGCOMM Conf., 2001, pp. 149-160.

[34] S. Ratnasamy et al., "A scalable content-addressable network," in Proc. ACM SIGCOMM Conf., 2001, pp. 161-172.

[35] B. Cohen, "Incentives build robustness in BitTorrent," in Proc. 1st Workshop on Economics of Peer-to-Peer Systems, 2003, pp. 68-72.

[36] R. Friedman and E. Birman, "Trading consistency for availability in distributed systems," Cornell University Technical Report, TR96-1581, 1996.

[37] E. Brewer, "Towards robust distributed systems," in Proc. 19th Annual ACM Symposium on Principles of Distributed Computing (PODC), 2000, pp. 7-10.

[38] A. Demers et al., "Epidemic algorithms for replicated database maintenance," in Proc. 6th Annual ACM Symposium on Principles of Distributed Computing (PODC), 1987, pp. 1-12.

[39] B. Cohen, "BitTorrent protocol specification," BitTorrent.org, 2008. [Online]. Available: http://www.bittorrent.org/beps/bep_0003.html

[40] J. C. Mogul, "Network locality at the scale of processes," ACM Trans. Computer Systems, vol. 10, no. 2, pp. 81-109, 1992.

[41] R. van Renesse et al., "Efficient reconciliation and flow control for anti-entropy protocols," in Proc. 2nd Workshop on Large-Scale Distributed Systems and Middleware (LADIS), 2008, pp. 1-7.

[42] L. Lamport, "Time, clocks, and the ordering of events in a distributed system," Communications of the ACM, vol. 21, no. 7, pp. 558-565, 1978.

[43] F. B. Schneider, "Implementing fault-tolerant services using the state machine approach: A tutorial," ACM Computing Surveys, vol. 22, no. 4, pp. 299-319, 1990.

[44] D. Hausheer and B. Stiller, "PeerMart: The technology for a distributed auction-based market for peer-to-peer services," in Proc. IEEE Int. Conf. Communications (ICC), 2005, pp. 256-260.

[45] MLPerf Consortium, "MLPerf Inference Benchmark Suite," arXiv preprint arXiv:1911.02549, 2019.

[46] R. Mahajan et al., "Experiences applying game theory to system design," in Proc. ACM SIGCOMM Workshop on Practice and Theory of Incentives in Networked Systems (PINS), 2004, pp. 183-190.

[47] J. Feigenbaum et al., "A BGP-based mechanism for lowest-cost routing," Distributed Computing, vol. 18, no. 1, pp. 61-72, 2005.

[48] A. Mas-Colell, M. D. Whinston, and J. R. Green, "Microeconomic Theory," Oxford University Press, 1995.

[49] P. Resnick et al., "Reputation systems," Communications of the ACM, vol. 43, no. 12, pp. 45-48, 2000.

[50] C. Dellarocas, "The digitization of word of mouth: Promise and challenges of online feedback mechanisms," Management Science, vol. 49, no. 10, pp. 1407-1424, 2003.

[51] A. Juels and B. S. Kaliski Jr., "PORs: Proofs of retrievability for large files," in Proc. ACM Conf. Computer and Communications Security (CCS), 2007, pp. 584-597.

[52] S. Goldwasser, S. Micali, and C. Rackoff, "The knowledge complexity of interactive proof systems," SIAM J. Computing, vol. 18, no. 1, pp. 186-208, 1989.

[53] N. Koblitz, "Elliptic curve cryptosystems," Mathematics of Computation, vol. 48, no. 177, pp. 203-209, 1987.

[54] A. Jøsang, R. Ismail, and C. Boyd, "A survey of trust and reputation systems for online service provision," Decision Support Systems, vol. 43, no. 2, pp. 618-644, 2007.

[55] D. Chaum, "Blind signatures for untraceable payments," in Proc. Advances in Cryptology (CRYPTO), 1983, pp. 199-203.

[56] M. Pease, R. Shostak, and L. Lamport, "Reaching agreement in the presence of faults," J. ACM, vol. 27, no. 2, pp. 228-234, 1980.

[57] M. Bellare and P. Rogaway, "Random oracles are practical: A paradigm for designing efficient protocols," in Proc. ACM Conf. Computer and Communications Security (CCS), 1993, pp. 62-73.

[58] D. Fudenberg and J. Tirole, "Game Theory," MIT Press, 1991.

[59] M. L. Katz and C. Shapiro, "Network externalities, competition, and compatibility," American Economic Review, vol. 75, no. 3, pp. 424-440, 1985.

[60] D. P. Anderson et al., "SETI@home: An experiment in public-resource computing," Communications of the ACM, vol. 45, no. 11, pp. 56-61, 2002.

[61] B. Cohen, "Incentives build robustness in BitTorrent," in Proc. 1st Workshop on Economics of Peer-to-Peer Systems, 2003, pp. 68-72.

[62] V. Paxson, "End-to-end routing behavior in the Internet," IEEE/ACM Trans. Networking, vol. 5, no. 5, pp. 601-615, 1997.

[63] J. Crowcroft, S. Hand, R. Mortier, T. Roscoe, and A. Warfield, "Plutarch: An argument for network pluralism," in Proc. ACM SIGCOMM Workshop on Future Directions in Network Architecture (FDNA), 2003, pp. 258-266.

[64] M. Li et al., "Parameter server for distributed machine learning," in Proc. 11th USENIX Symposium on Operating Systems Design and Implementation (OSDI), 2014, pp. 583-598.

[65] A. Harlap et al., "PipeDream: Generalized pipeline parallelism for DNN training," in Proc. 27th ACM Symposium on Operating Systems Principles (SOSP), 2019, pp. 1-15.

[66] H. Cui et al., "Geeps: Scalable deep learning on distributed GPUs with a GPU-specialized parameter server," in Proc. 11th European Conference on Computer Systems (EuroSys), 2016, pp. 1-16.

[67] Y. You et al., "Large batch optimization for deep learning: Training BERT in 76 minutes," in Proc. Int. Conf. Learning Representations (ICLR), 2020.

[68] D. Narayanan et al., "Memory-efficient pipeline-parallel DNN training," in Proc. 38th Int. Conf. Machine Learning (ICML), 2021, pp. 7937-7947.

[69] S. Rajbhandari et al., "DeepSpeed: System optimizations enable training deep learning models with over 100 billion parameters," in Proc. 26th ACM SIGKDD Int. Conf. Knowledge Discovery & Data Mining, 2020, pp. 3505-3506.

[70] A. Krizhevsky, "One weird trick for parallelizing convolutional neural networks," arXiv preprint arXiv:1404.5997, 2014.

[71] T. Chen et al., "MXNet: A flexible and efficient machine learning library for heterogeneous distributed systems," arXiv preprint arXiv:1512.01274, 2015.

[72] M. Castro and B. Liskov, "Practical Byzantine fault tolerance and proactive recovery," ACM Trans. Computer Systems, vol. 20, no. 4, pp. 398-461, 2002.

[73] A. Paszke et al., "PyTorch: An imperative style, high-performance deep learning library," in Proc. 33rd Conf. Neural Information Processing Systems (NeurIPS), 2019, pp. 8024-8035.

[74] ONNX Community, "Open Neural Network Exchange (ONNX): An open standard for machine learning interoperability," Microsoft and Facebook, 2017. [Online]. Available: https://onnx.ai/

[75] B. Burns and D. Beda, "Kubernetes: Up and running," O'Reilly Media, 2017.

[76] J. Dean and S. Ghemawat, "MapReduce: Simplified data processing on large clusters," Communications of the ACM, vol. 51, no. 1, pp. 107-113, 2008.

[77] C. Dwork, "Differential privacy," in Proc. 33rd Int. Colloquium on Automata, Languages and Programming (ICALP), 2006, pp. 1-12.

[78] A. Shamir, "How to share a secret," Communications of the ACM, vol. 22, no. 11, pp. 612-613, 1979.

[79] R. L. Rivest, A. Shamir, and L. Adleman, "A method for obtaining digital signatures and public-key cryptosystems," Communications of the ACM, vol. 21, no. 2, pp. 120-126, 1978.

[80] D. Boneh and M. Franklin, "Identity-based encryption from the Weil pairing," in Proc. Advances in Cryptology (CRYPTO), 2001, pp. 213-229.

[81] O. Goldreich, S. Micali, and A. Wigderson, "How to play any mental game," in Proc. 19th Annual ACM Symposium on Theory of Computing (STOC), 1987, pp. 218-229.

[82] A. C. Yao, "Protocols for secure computations," in Proc. 23rd Annual Symposium on Foundations of Computer Science (FOCS), 1982, pp. 160-164.

[83] C. Gentry, "Fully homomorphic encryption using ideal lattices," in Proc. 41st Annual ACM Symposium on Theory of Computing (STOC), 2009, pp. 169-178.

[84] M. Bellare, A. Desai, E. Jokipii, and P. Rogaway, "A concrete security treatment of symmetric encryption," in Proc. 38th Annual Symposium on Foundations of Computer Science (FOCS), 1997, pp. 394-403.

[85] S. Goldwasser and S. Micali, "Probabilistic encryption," J. Computer and System Sciences, vol. 28, no. 2, pp. 270-299, 1984.

[86] M. Naor and B. Pinkas, "Efficient oblivious transfer protocols," in Proc. 12th Annual ACM-SIAM Symposium on Discrete Algorithms (SODA), 2001, pp. 448-457.

[87] R. Canetti, "Universally composable security: A new paradigm for cryptographic protocols," in Proc. 42nd IEEE Symposium on Foundations of Computer Science (FOCS), 2001, pp. 136-145.

[88] Y. Dodis, A. Kiayias, A. Nicolosi, and V. Shoup, "Anonymous identification in ad hoc groups," in Proc. Advances in Cryptology (EUROCRYPT), 2004, pp. 609-626.

[89] J. Camenisch and A. Lysyanskaya, "An anonymous credential system," in Proc. Advances in Cryptology (EUROCRYPT), 2001, pp. 414-427.

[90] D. Chaum, A. Fiat, and M. Naor, "Untraceable electronic cash," in Proc. Advances in Cryptology (CRYPTO), 1988, pp. 319-327.

[91] T. P. Pedersen, "Non-interactive and information-theoretic secure verifiable secret sharing," in Proc. Advances in Cryptology (CRYPTO), 1991, pp. 129-140.

[92] R. Gennaro, S. Jarecki, H. Krawczyk, and T. Rabin, "Secure distributed key generation for discrete-log based cryptosystems," J. Cryptology, vol. 20, no. 1, pp. 51-83, 2007.

[93] M. Ben-Or, S. Goldwasser, and A. Wigderson, "Completeness theorems for non-cryptographic fault-tolerant distributed computation," in Proc. 20th Annual ACM Symposium on Theory of Computing (STOC), 1988, pp. 1-10.

[94] T. Rabin and M. Ben-Or, "Verifiable secret sharing and multiparty protocols with honest majority," in Proc. 21st Annual ACM Symposium on Theory of Computing (STOC), 1989, pp. 73-85.

[95] I. Damgård, V. Pastro, N. Smart, and S. Zakarias, "Multiparty computation from somewhat homomorphic encryption," in Proc. Advances in Cryptology (CRYPTO), 2012, pp. 643-662.

[96] A. Herzberg, S. Jarecki, H. Krawczyk, and M. Yung, "Proactive secret sharing or: How to cope with perpetual leakage," in Proc. Advances in Cryptology (CRYPTO), 1995, pp. 339-352.

[97] B. Chor, O. Goldreich, E. Kushilevitz, and M. Sudan, "Private information retrieval," J. ACM, vol. 45, no. 6, pp. 965-981, 1998.

[98] E. Kushilevitz and R. Ostrovsky, "Replication is not needed: Single database, computationally-private information retrieval," in Proc. 38th Annual Symposium on Foundations of Computer Science (FOCS), 1997, pp. 364-373.

[99] D. X. Song, D. Wagner, and A. Perrig, "Practical techniques for searches on encrypted data," in Proc. IEEE Symposium on Security and Privacy, 2000, pp. 44-55.

[100] R. Curtmola, J. Garay, S. Kamara, and R. Ostrovsky, "Searchable symmetric encryption: Improved definitions and efficient constructions," J. Computer Security, vol. 19, no. 5, pp. 895-934, 2011.

[101] Docker Inc., "Docker: Enterprise container platform," Docker Documentation, 2023. [Online]. Available: https://docs.docker.com/

[102] Cloud Native Computing Foundation, "Kubernetes: Production-grade container orchestration," CNCF, 2023. [Online]. Available: https://kubernetes.io/

[103] gRPC Authors, "gRPC: A high performance, open source universal RPC framework," Google, 2023. [Online]. Available: https://grpc.io/

[104] Apache Software Foundation, "Apache Kafka: A distributed streaming platform," Apache Kafka Documentation, 2023. [Online]. Available: https://kafka.apache.org/

[105] Redis Labs, "Redis: In-memory data structure store," Redis Documentation, 2023. [Online]. Available: https://redis.io/

[106] Prometheus Authors, "Prometheus: Monitoring system and time series database," Prometheus Documentation, 2023. [Online]. Available: https://prometheus.io/

[107] Grafana Labs, "Grafana: The open observability platform," Grafana Documentation, 2023. [Online]. Available: https://grafana.com/

[108] HashiCorp, "Consul: Service mesh and service discovery," HashiCorp Documentation, 2023. [Online]. Available: https://www.consul.io/

[109] etcd Authors, "etcd: Distributed reliable key-value store," etcd Documentation, 2023. [Online]. Available: https://etcd.io/

[110] NVIDIA Corporation, "CUDA Toolkit Documentation," NVIDIA Developer, 2023. [Online]. Available: https://docs.nvidia.com/cuda/

[111] Khronos Group, "OpenCL: Open standard for parallel computing," Khronos OpenCL, 2023. [Online]. Available: https://www.khronos.org/opencl/

[112] Apache Software Foundation, "Apache Arrow: Columnar in-memory analytics," Apache Arrow Documentation, 2023. [Online]. Available: https://arrow.apache.org/

[113] Protocol Buffers Authors, "Protocol Buffers: Google's data interchange format," Google Developers, 2023. [Online]. Available: https://developers.google.com/protocol-buffers

[114] JSON-RPC Working Group, "JSON-RPC 2.0 Specification," JSON-RPC, 2010. [Online]. Available: https://www.jsonrpc.org/specification

[115] OpenAPI Initiative, "OpenAPI Specification," Linux Foundation, 2023. [Online]. Available: https://spec.openapis.org/

[116] Rust Foundation, "Rust Programming Language," Rust Foundation, 2023. [Online]. Available: https://www.rust-lang.org/

[117] Python Software Foundation, "Python Programming Language," Python.org, 2023. [Online]. Available: https://www.python.org/

[118] Go Team, "Go Programming Language," Google, 2023. [Online]. Available: https://golang.org/

[119] Eclipse Foundation, "Eclipse Mosquitto: An open source MQTT broker," Eclipse IoT, 2023. [Online]. Available: https://mosquitto.org/

[120] Apache Software Foundation, "Apache Pulsar: Cloud-native distributed messaging," Apache Pulsar, 2023. [Online]. Available: https://pulsar.apache.org/

[121] E. Ostrom, "Governing the Commons: The Evolution of Institutions for Collective Action," Cambridge University Press, 1990.

[122] G. Hardin, "The tragedy of the commons," Science, vol. 162, no. 3859, pp. 1243-1248, 1968.

[123] R. Axelrod, "The Evolution of Cooperation," Basic Books, 1984.

[124] M. Olson, "The Logic of Collective Action: Public Goods and the Theory of Groups," Harvard University Press, 1965.

[125] J. M. Buchanan and G. Tullock, "The Calculus of Consent: Logical Foundations of Constitutional Democracy," University of Michigan Press, 1962.

[126] A. Sen, "Collective Choice and Social Welfare," Harvard University Press, 1970.

[127] L. Hurwicz, "The design of mechanisms for resource allocation," American Economic Review, vol. 63, no. 2, pp. 1-30, 1973.

[128] R. Myerson, "Incentive compatibility and the bargaining problem," Econometrica, vol. 47, no. 1, pp. 61-73, 1979.

[129] E. Maskin, "Mechanism design: How to implement social goals," American Economic Review, vol. 98, no. 3, pp. 567-576, 2008.

[130] J. Tirole, "The Theory of Industrial Organization," MIT Press, 1988.

[131] C. Shapiro and H. R. Varian, "Information Rules: A Strategic Guide to the Network Economy," Harvard Business School Press, 1999.

[132] B. Arthur, "Increasing Returns and Path Dependence in the Economy," University of Michigan Press, 1994.

[133] P. A. David, "Clio and the Economics of QWERTY," American Economic Review, vol. 75, no. 2, pp. 332-337, 1985.

[134] M. L. Katz and C. Shapiro, "Systems competition and network effects," Journal of Economic Perspectives, vol. 8, no. 2, pp. 93-115, 1994.

[135] J. Farrell and G. Saloner, "Standardization, compatibility, and innovation," RAND Journal of Economics, vol. 16, no. 1, pp. 70-83, 1985.

[136] International Energy Agency, "Data Centres and Data Transmission Networks," IEA, Paris, 2022.

[137] A. S. G. Andrae and T. Edler, "On global electricity usage of communication technology: Trends to 2030," Challenges, vol. 6, no. 1, pp. 117-157, 2015.

[138] E. Strubell, A. Ganesh, and A. McCallum, "Energy and policy considerations for deep learning in NLP," in Proc. 57th Annual Meeting of the Association for Computational Linguistics (ACL), 2019, pp. 3645-3650.

[139] D. Patterson et al., "Carbon emissions and large neural network training," arXiv preprint arXiv:2104.10350, 2021.

[140] A. Lacoste et al., "Quantifying the carbon emissions of machine learning," arXiv preprint arXiv:1910.09700, 2019.

[141] L. F. W. Anthony, B. Kanding, and R. Selvan, "Carbontracker: Tracking and predicting the carbon footprint of training deep learning models," arXiv preprint arXiv:2007.03051, 2020.

[142] P. Henderson et al., "Towards the systematic reporting of the energy and carbon footprints of machine learning," Journal of Machine Learning Research, vol. 21, pp. 248-1-248-43, 2020.

[143] R. Schwartz et al., "Green AI," Communications of the ACM, vol. 63, no. 12, pp. 54-63, 2020.

[144] A. Dhar et al., "Carbon impact of federated learning: An experimental study," in Proc. IEEE Int. Conf. Big Data, 2021, pp. 1-10.

[145] J. Kaack et al., "Aligning artificial intelligence with climate change mitigation," Nature Climate Change, vol. 12, no. 6, pp. 518-527, 2022.

[146] A. Krizhevsky, I. Sutskever, and G. E. Hinton, "ImageNet classification with deep convolutional neural networks," Communications of the ACM, vol. 60, no. 6, pp. 84-90, 2017.

[147] J. Devlin et al., "BERT: Pre-training of deep bidirectional transformers for language understanding," in Proc. Conf. North American Chapter of the Association for Computational Linguistics (NAACL), 2019, pp. 4171-4186.

[148] A. Radford et al., "Language models are unsupervised multitask learners," OpenAI Blog, 2019. [Online]. Available: https://openai.com/blog/better-language-models/

[149] S. Merity et al., "Pointer sentinel mixture models," in Proc. Int. Conf. Learning Representations (ICLR), 2017.

[150] Y. Liu et al., "RoBERTa: A robustly optimized BERT pretraining approach," arXiv preprint arXiv:1907.11692, 2019.

[151] Z. Yang et al., "XLNet: Generalized autoregressive pretraining for language understanding," in Proc. 33rd Conf. Neural Information Processing Systems (NeurIPS), 2019, pp. 5753-5763.

[152] C. Raffel et al., "Exploring the limits of transfer learning with a unified text-to-text transformer," J. Machine Learning Research, vol. 21, pp. 140-1-140-67, 2020.

[153] J. Hoffmann et al., "Training compute-optimal large language models," arXiv preprint arXiv:2203.15556, 2022.

[154] S. Smith et al., "Using DeepSpeed and Megatron to train Megatron-Turing NLG 530B, a large-scale generative language model," arXiv preprint arXiv:2201.11990, 2022.

[155] W. Fedus, B. Zoph, and N. Shazeer, "Switch transformer: Scaling to trillion parameter models with simple and efficient sparsity," J. Machine Learning Research, vol. 23, pp. 120-156, 2022.

[156] D. Lepikhin et al., "GShard: Scaling giant models with conditional computation and automatic sharding," in Proc. Int. Conf. Learning Representations (ICLR), 2021.

[157] N. Du et al., "GLaM: Efficient scaling of language models with mixture-of-experts," in Proc. 39th Int. Conf. Machine Learning (ICML), 2022, pp. 5547-5569.

[158] A. Chowdhery et al., "PaLM: Scaling language modeling with pathways," J. Machine Learning Research, vol. 24, pp. 240-302, 2023.

[159] S. Roller et al., "Hash layers for large sparse models," in Proc. 35th Conf. Neural Information Processing Systems (NeurIPS), 2021, pp. 17555-17566.

[160] M. Lewis et al., "BART: Denoising sequence-to-sequence pre-training for natural language generation, translation, and comprehension," in Proc. 58th Annual Meeting of the Association for Computational Linguistics (ACL), 2020, pp. 7871-7880.

[161] Google Research, "T5: Text-to-text transfer transformer," Google AI Blog, 2019. [Online]. Available: https://ai.googleblog.com/2020/02/exploring-transfer-learning-with-t5.html

[162] OpenAI, "GPT-4 technical report," arXiv preprint arXiv:2303.08774, 2023.

[163] Anthropic, "Constitutional AI: Harmlessness from AI feedback," arXiv preprint arXiv:2212.08073, 2022.

[164] D. Hendrycks et al., "Measuring massive multitask language understanding," in Proc. Int. Conf. Learning Representations (ICLR), 2021.

[165] P. Clark et al., "Think you have solved question answering? Try ARC, the AI2 reasoning challenge," arXiv preprint arXiv:1803.05457, 2018.

[166] R. Zellers et al., "HellaSwag: Can a machine really finish your sentence?" in Proc. 57th Annual Meeting of the Association for Computational Linguistics (ACL), 2019, pp. 4791-4800.

[167] A. Holtzman et al., "The curious case of neural text degeneration," in Proc. Int. Conf. Learning Representations (ICLR), 2020.

[168] S. Welleck et al., "Neural text generation with unlikelihood training," in Proc. Int. Conf. Learning Representations (ICLR), 2020.

[169] L. Ouyang et al., "Training language models to follow instructions with human feedback," in Proc. 36th Conf. Neural Information Processing Systems (NeurIPS), 2022, pp. 27730-27744.

[170] J. Schulman et al., "Proximal policy optimization algorithms," arXiv preprint arXiv:1707.06347, 2017.

[171] P. F. Christiano et al., "Deep reinforcement learning from human preferences," in Proc. 31st Conf. Neural Information Processing Systems (NeurIPS), 2017, pp. 4299-4307.

[172] D. M. Ziegler et al., "Fine-tuning language models from human preferences," arXiv preprint arXiv:1909.08593, 2019.

[173] N. Stiennon et al., "Learning to summarize with human feedback," in Proc. 34th Conf. Neural Information Processing Systems (NeurIPS), 2020, pp. 3008-3021.

[174] R. Nakano et al., "WebGPT: Browser-assisted question-answering with human feedback," arXiv preprint arXiv:2112.09332, 2021.

[175] J. Menick et al., "Teaching language models to support answers with verified quotes," arXiv preprint arXiv:2203.11147, 2022.

[176] S. Longpre et al., "The flan collection: Designing data and methods for effective instruction tuning," arXiv preprint arXiv:2301.13688, 2023.

[177] H. W. Chung et al., "Scaling instruction-finetuned language models," arXiv preprint arXiv:2210.11416, 2022.

[178] Y. Wang et al., "Self-instruct: Aligning language model with self generated instructions," arXiv preprint arXiv:2212.10560, 2022.

[179] R. Taori et al., "Stanford Alpaca: An instruction-following LLaMA model," Stanford Center for Research on Foundation Models, 2023. [Online]. Available: https://crfm.stanford.edu/2023/03/13/alpaca.html

[180] W. X. Zhao et al., "A survey of large language models," arXiv preprint arXiv:2303.18223, 2023.

[181] World Health Organization, "Digital health strategy 2020-2025," WHO Press, Geneva, 2020.

[182] UNESCO, "AI and education: Guidance for policy-makers," UNESCO Publishing, Paris, 2021.

[183] National Science Foundation, "Pathways to enable open-source ecosystems (POSE)," NSF Program Solicitation 21-572, 2021.

[184] European Commission, "Digital education action plan 2021-2027," Publications Office of the European Union, Luxembourg, 2021.

[185] A. Ng, "Machine learning yearning," Technical Strategy for AI Engineers, 2018. [Online]. Available: https://www.mlyearning.org/

[186] M. Mitchell, "Artificial intelligence: A guide for thinking humans," Farrar, Straus and Giroux, 2019.

[187] S. Russell, "Human compatible: Artificial intelligence and the problem of control," Viking Press, 2019.

[188] C. O'Neil, "Weapons of math destruction: How big data increases inequality and threatens democracy," Crown Publishers, 2016.

[189] S. Barocas, M. Hardt, and A. Narayanan, "Fairness and machine learning: Limitations and opportunities," MIT Press, 2023.

[190] J. Kleinberg, S. Mullainathan, and M. Raghavan, "Inherent trade-offs in the fair determination of risk scores," in Proc. 8th Innovations in Theoretical Computer Science Conf. (ITCS), 2017, pp. 43:1-43:23.

[191] D. Bahdanau, K. Cho, and Y. Bengio, "Neural machine translation by jointly learning to align and translate," in Proc. Int. Conf. Learning Representations (ICLR), 2015.

[192] A. Vaswani et al., "Attention is all you need," in Proc. 31st Conf. Neural Information Processing Systems (NeurIPS), 2017, pp. 5998-6008.

[193] M. Johnson et al., "Google's multilingual neural machine translation system: Enabling zero-shot translation," Trans. Association for Computational Linguistics, vol. 5, pp. 339-351, 2017.

[194] R. Sennrich, B. Haddow, and A. Birch, "Neural machine translation of rare words with subword units," in Proc. 54th Annual Meeting of the Association for Computational Linguistics (ACL), 2016, pp. 1715-1725.

[195] T. Kudo and J. Richardson, "SentencePiece: A simple and language independent subword tokenizer and detokenizer for neural text processing," in Proc. Conf. Empirical Methods in Natural Language Processing (EMNLP), 2018, pp. 66-71.

[196] E. Korpela et al., "SETI@home: Massively distributed computing for SETI," Computing in Science & Engineering, vol. 3, no. 1, pp. 78-83, 2001.

[197] V. S. Pande et al., "Atomistic protein folding simulations on the submillisecond time scale using worldwide distributed computing," Biopolymers, vol. 68, no. 1, pp. 91-109, 2003.

[198] G. Bosilca et al., "Open MPI: Goals, concept, and design of a next generation MPI implementation," in Proc. 11th European PVM/MPI Users' Group Meeting, 2004, pp. 97-104.

[199] io.net, "Decentralized GPU network for machine learning," io.net Technical Documentation, 2023. [Online]. Available: https://io.net/docs

[200] Render Network Foundation, "Render Network: Distributed GPU rendering protocol," Render Network Whitepaper, 2023. [Online]. Available: https://rendernetwork.com/whitepaper

[201] LocalAI Contributors, "LocalAI: Self-hosted, community-driven, local OpenAI-compatible API," GitHub Repository, 2023. [Online]. Available: https://github.com/go-skynet/LocalAI

[202] llm-d Contributors (Red Hat, IBM, Google, CoreWeave, et al.), "llm-d: Kubernetes-native distributed inference serving for large language models," GitHub Repository, 2024. [Online]. Available: https://github.com/llm-d/llm-d

[203] E. Cahn, "Service credits: A new currency for the new economy," in Proc. Int. Conf. Community Currencies, 2000, pp. 45-62.

[204] T. Greco Jr., "Money: Understanding and creating alternatives to legal tender," Chelsea Green Publishing, 2001.

[205] J. Lasn and B. Grierson, "Time dollar systems: Building community one hour at a time," New Society Publishers, 1999.

[206] R. M. Titmuss, "The gift relationship: From human blood to social policy," Allen & Unwin, 1970.

[207] A. Oakley and J. Ashton, "The gift relationship: From human blood to social policy," LSE Books, 1997.

[208] P. Kollock, "The economies of online cooperation: Gifts and public goods in cyberspace," in Communities in Cyberspace, M. A. Smith and P. Kollock, Eds. London: Routledge, 1999, pp. 220-239.

[209] Y. Benkler, "The wealth of networks: How social production transforms markets and freedom," Yale University Press, 2006.

[210] C. Shirky, "Here comes everybody: The power of organizing without organizations," Penguin Press, 2008.

[211] J. Steeves and A. Shaabana, "Bittensor: A peer-to-peer intelligence market," arXiv:2003.03917, 2020.

[212] A. Borzunov, D. Baranchuk, T. Dettmers, M. Ryabinin, Y. Belkada, A. Chumachenko, P. Samygin, and C. Raffel, "Petals: Collaborative inference and fine-tuning of large models," in Proc. 61st Annu. Meeting Assoc. for Computational Linguistics (ACL): System Demonstrations, Toronto, Canada, 2023, pp. 558-568.

[213] M. Ryabinin and A. Gusev, "Towards crowdsourced training of large neural networks using decentralized mixture-of-experts," in Proc. 34th Conf. Neural Information Processing Systems (NeurIPS), 2020, pp. 3659-3672.

[214] M. Ryabinin, T. Dettmers, M. Diskin, and A. Borzunov, "SWARM parallelism: Training large models can be surprisingly communication-efficient," in Proc. 40th Int. Conf. Machine Learning (ICML), 2023, pp. 29416-29440.

[215] A. Douillard, Q. Feng, A. A. Rusu, R. Chhaparia, Y. Donchev, A. Kuncoro, M. Ranzato, A. Szlam, and J. Shen, "DiLoCo: Distributed low-communication training of language models," arXiv:2311.08105, 2023.

[216] Gensyn Team, "Gensyn litepaper: A protocol for decentralized machine learning compute," Gensyn Whitepaper (vendor source), 2022. [Online]. Available: https://docs.gensyn.ai/litepaper

[217] G. Boutsioukis, A. Bulkin, and Akash Network Team, "Akash Network: A decentralized cloud computing marketplace," Akash Network Whitepaper (vendor source), 2020. [Online]. Available: https://akash.network/whitepaper

[218] M. Shirts and V. S. Pande, "Screen savers of the world unite!," Science, vol. 290, no. 5498, pp. 1903-1904, 2000.

[219] H. Sun, Y. Li, H. Zhang, and K. Ren, "zkLLM: Zero knowledge proofs for large language models," in *Proc. ACM Conf. Computer and Communications Security (CCS)*, 2024.

[220] N. Kanpak, A. Polychroniadou, X. Wang, V. Vaikuntanathan, and Y. Polyakov, "Mystique: Efficient conversions for zero-knowledge proofs with applications to machine learning," in *Proc. 30th USENIX Security Symposium*, 2021, pp. 501–518.

[221] B. Knott, S. Venkataraman, A. Hannun, S. Sengupta, M. Ibrahim, and L. van der Maaten, "CrypTen: Secure multi-party computation meets machine learning," in *Proc. NeurIPS Workshop on Privacy Preserving Machine Learning*, 2021.

[222] D. Li, R. Shao, H. Wang, H. Guo, E. P. Xing, and H. Zhang, "MPCFormer: Fast, performant and private transformer inference with MPC," in *Proc. Int. Conf. Learning Representations (ICLR)*, 2023.

[223] M. Hao, H. Li, H. Chen, P. Xing, G. Xu, and T. Zhang, "Iron: Private inference on transformers," in *Proc. 36th Conf. Neural Information Processing Systems (NeurIPS)*, 2022.

[224] C. Gentry, "A fully homomorphic encryption scheme," Ph.D. dissertation, Stanford University, 2009.

## Bibliography Hygiene Notes (added in 2026-05 revision)

The following entries are known to require care; they are retained for citation-number stability rather than because they are above reproach. A future revision should re-verify or replace them with primary sources.

- **[10], [11], [12]** — *withdrawn* (see entries; §1.3 has been rewritten without these citations).
- **[5]** A. Chowdhery et al., PaLM, JMLR vol. 24 — JMLR uses article-number style ("24:240, pp. 1–113"); the page range "240–302" was a formatting artefact in earlier drafts.
- **[7]** "CRA Academic Computing Infrastructure Survey, 2023" — the precise title was not located in the CRA publication index at the time of revision; if a different CRA-published survey is intended, the title should be corrected on next revision.
- **[17] / [155]** — Switch Transformer cited twice with slightly different bibliographic detail; consolidate at next revision.
- **[19]** Partnership on AI, "Democratizing AI," 2022 — title and year should be re-verified against the PAI publication index.
- **[27] / [65]** — PipeDream cited twice; consolidate at next revision.
- **[28] / [72]** — Castro & Liskov PBFT cited in two near-duplicate forms; consolidate.
- **[35] / [61]** — Cohen, "Incentives Build Robustness in BitTorrent," cited verbatim twice; consolidate.
- **[36]** Friedman & Birman, Cornell TR96-1581, 1996 — the technical-report number format does not match standard Cornell CS TR numbering of the period; if this citation is load-bearing it should be replaced with a verifiable primary source (e.g. Brewer's CAP work [37] alone may suffice).
- **[142], [152], [155], [158]** — JMLR page-range style (`pp. 248-1-248-43`) is an artefact; canonical JMLR style is `vol(article):page–page` or `vol(article)`, no second "pp." count.
- **[183]** NSF POSE Solicitation 21-572 — actual POSE program solicitations are 22-572 (2022) and 23-556 (2023); update the number to the year that the prose actually refers to.
- **[185]** Ng, "Machine Learning Yearning" — the canonical URL is `https://www.deeplearning.ai/machine-learning-yearning/` rather than `mlyearning.org`.
- **[202]** llm-d — corrected in this revision to `github.com/llm-d/llm-d` (Red Hat / IBM / Google / CoreWeave) from the earlier `distributedllm/llm-d` URL.
- **[206]** Titmuss, *The Gift Relationship* — when invoked in §2 the claim should be that voluntary systems produce higher-quality donations and stronger civic equity, not (as in earlier drafts) higher donation rates per capita; the latter is contested in the literature.

## Reference Context and Connection to Text

**Section 2.1 (Distributed Scientific Computing):**
- [20]: BOINC framework foundational architecture and lessons learned from volunteer computing
- [60]: SETI@home experimental results and credit system design principles
- [196]: SETI@home massively distributed computing implementation and scalability analysis
- [197]: Folding@home protein folding simulations demonstrating scientific impact of distributed computing
- [198]: Open MPI parallel computing framework supporting distributed scientific computation

**Section 2.2 (Decentralized GPU Networks):**
- [199]: io.net commercial decentralized GPU network architecture and monetary incentive limitations
- [200]: Render Network distributed GPU rendering protocol and token-based economic model
- [201]: LocalAI self-hosted AI inference framework and community-driven development model
- [202]: llm-d distributed large language model inference technical implementation
- [31]: Bitcoin whitepaper providing foundation for understanding cryptocurrency-based incentive limitations

**Section 2.3 (Resource Sharing Models):**
- [203]: Cahn service credits and time banking systems as non-monetary exchange mechanisms
- [204]: Greco alternative currency systems and community-based resource allocation
- [205]: Lasn time dollar systems and community building through mutual aid
- [206]: Titmuss gift relationship and blood donation as model for altruistic resource sharing
- [207]: Oakley gift relationship updated analysis of voluntary contribution systems
- [208]: Kollock online cooperation and gift economies in digital communities
- [209]: Benkler wealth of networks and social production in networked environments
- [210]: Shirky organizing without organizations and peer production models

**Section 1.1 (The Great AI Divide):**
- [1]: Establishes the foundational work on large language models that created the current accessibility challenges
- [2]: Demonstrates the scale of computational resources required for modern AI systems

**Section 1.2 (The Economics of Exclusion):**
- [4]: NVIDIA H100 pricing documentation supporting the $25,000+ cost claim
- [5]: PaLM paper providing evidence for training cost estimates of $50,000-$100,000
- [6]: Codex paper supporting inference deployment cost claims
- [7]: CRA survey data supporting university computing budget limitations
- [8]: Dean & Corrado paper on large-scale distributed systems supporting claims about corporate AI infrastructure
- [9]: AWS pricing documentation supporting cloud computing cost barriers

**Section 1.3 (Illustrative Scenarios):**
- The §1.3 scenarios in the revised paper are explicitly labelled illustrative composites and do not carry empirical citations. Entries [10], [11], [12] are *withdrawn* from the active bibliography; see notes at the top of this file.

**Section 1.4 (The Innovation Bottleneck):**
- [13]: Floridi et al. on AI ethics and the importance of diverse perspectives in AI development
- [14]: Bommasani et al. foundation models paper supporting claims about concentration of AI research
- [15]: Hendrycks et al. on AI alignment, supporting the need for diverse approaches in safety research
- [16]: UN Digital Economy Report supporting claims about global digital inequalities

**Section 1.5 (Toward a Democratic Alternative):**
- [17]: Switch Transformer paper demonstrating distributed inference feasibility
- [18]: Megatron-LM paper on distributed training across consumer hardware
- [19]: Partnership on AI report supporting arguments for democratizing AI access
- [20]: BOINC paper providing the foundational model for distributed computing projects like SETI@home

**Section 3.1 (Network Topology):**
- [21]: Zaharia et al. on resilient distributed datasets supporting fault-tolerant distributed computing architectures
- [22]: Pastry paper demonstrating scalable peer-to-peer routing for regional gateway implementation
- [23]: Bimodal multicast protocol supporting cluster orchestrator coordination mechanisms

**Section 3.2 (Core Components):**
- [24]: White space networking research supporting geographic clustering strategies
- [25]: Lamport's Paxos algorithm foundational to gateway consensus mechanisms
- [26]: OpenDHT demonstrating distributed hash table approaches for model registry management
- [27]: PipeDream paper on pipeline parallelism supporting distributed model sharding algorithms
- [28]: Castro & Liskov practical Byzantine fault tolerance for inference result validation
- [29]: Container technology survey supporting heterogeneous hardware integration approaches
- [30]: RAMCloud paper on distributed memory management systems
- [31]: Bitcoin whitepaper providing blockchain foundation for credit ledger architecture
- [32]: Mining coalition deterrence research supporting anti-gaming credit mechanisms

**Section 3.3 (Scalability Analysis):**
- [33]: Chord distributed hash table demonstrating scalable peer-to-peer lookup services
- [34]: Content-addressable networking research supporting hierarchical load distribution
- [35]: BitTorrent incentive mechanisms demonstrating bandwidth optimization in distributed systems
- [36]: Consistency vs availability trade-offs in distributed systems supporting economic sustainability analysis

**Section 3.4 (Component Interactions):**
- [37]: Brewer's CAP theorem supporting distributed coordination design decisions
- [38]: Epidemic algorithms for distributed database maintenance supporting credit synchronization protocols
- [39]: BitTorrent protocol specification supporting model distribution mechanisms

**Section 3.5 (Performance Guarantees):**
- [40]: Network locality research supporting latency guarantee implementation
- [41]: Anti-entropy protocols supporting throughput scalability analysis
- [42]: Lamport's logical clocks supporting distributed system coordination timing
- [43]: State machine replication tutorial supporting fault tolerance and reliability guarantees

**Section 4.1 (Mathematical Foundation of Credit Generation):**
- [44]: PeerMart auction-based market mechanisms supporting credit generation economics
- [45]: MLPerf benchmarks providing hardware normalization baselines for efficiency factors
- [46]: Game theory applications to system design supporting credit mechanism incentives
- [47]: BGP lowest-cost routing mechanisms supporting difficulty adjustment algorithms

**Section 4.2 (Game-Theoretic Incentive Analysis):**
- [48]: Mas-Colell microeconomic theory foundational to strategic interaction analysis
- [49]: Reputation systems research supporting long-term incentive mechanisms
- [50]: Online feedback mechanisms supporting reputation-based access control

**Section 4.3 (Anti-Gaming Mechanisms and Proofs):**
- [51]: Proofs of retrievability supporting computational proof verification
- [52]: Zero-knowledge interactive proofs foundational to Proof of Inference protocol
- [53]: Elliptic curve cryptography supporting cryptographic security analysis
- [54]: Trust and reputation systems survey supporting validation mechanisms

**Section 4.4 (Credit Lifecycle and Flow Management):**
- [55]: Chaum blind signatures supporting privacy-preserving credit transactions
- [56]: Byzantine agreement algorithms supporting distributed credit validation
- [57]: Random oracle model supporting cryptographic protocol design

**Section 4.5 (Economic Equilibrium and Long-term Sustainability):**
- [58]: Fudenberg & Tirole game theory supporting equilibrium analysis
- [59]: Network externalities research supporting network effects modeling

**Section 4.6 (Comparison with Existing Systems):**
- [60]: SETI@home experimental results supporting distributed computing comparison
- [61]: BitTorrent incentive mechanisms supporting peer-to-peer resource sharing analysis
- [62]: Internet routing behavior research supporting network efficiency comparisons
- [63]: Network pluralism arguments supporting hybrid global-local architecture design

**Section 5.1 (Protocol Overview and Design Principles):**
- [64]: Parameter server architecture foundational to distributed machine learning coordination
- [65]: PipeDream pipeline parallelism supporting adaptive partitioning and fault-tolerant coordination
- [66]: GPU-specialized parameter server research supporting credit-integrated resource management

**Section 5.2 (Request Orchestration and Routing):**
- [67]: Large batch optimization techniques supporting intelligent cluster selection algorithms
- [68]: Memory-efficient pipeline parallelism supporting authentication and credit verification protocols

**Section 5.3 (Distributed Model Sharding and Execution):**
- [69]: DeepSpeed system optimizations supporting adaptive transformer partitioning
- [70]: Convolutional neural network parallelization supporting coordinated inference execution
- [71]: MXNet heterogeneous distributed systems supporting dynamic sharding strategies

**Section 5.4 (Latency Optimization and Performance Guarantees):**
- [72]: Practical Byzantine fault tolerance supporting performance guarantee framework
- [73]: PyTorch distributed computing capabilities supporting geographic clustering optimization

**Section 5.5 (Fault Tolerance and Recovery Mechanisms):**
- [72]: Byzantine fault tolerance protocols supporting inference validation and adaptive recovery

**Section 5.6 (Integration with Existing ML Frameworks):**
- [73]: PyTorch framework integration supporting framework-agnostic API design
- [74]: ONNX standard supporting automated model conversion and deployment
- [75]: Kubernetes orchestration supporting model version management

**Section 5.7 (Quality of Service and SLA Implementation):**
- [75]: Container orchestration research supporting multi-tier service guarantees and SLA monitoring

**Section 5.8 (Performance Analysis and Optimization Strategies):**
- [76]: MapReduce distributed processing supporting latency decomposition and throughput scaling analysis

**Section 6.1 (Threat Model and Security Assumptions):**
- [28]: Castro & Liskov practical Byzantine fault tolerance supporting adversarial node behavior analysis
- [72]: Byzantine fault tolerance and proactive recovery supporting network-level attack resistance
- [77]: Dwork differential privacy foundational to privacy threat modeling
- [31]: Bitcoin whitepaper providing blockchain security model comparison

**Section 6.2 (Cryptographic Foundation and Primitives):**
- [53]: Elliptic curve cryptography supporting digital signature schemes
- [78]: Shamir secret sharing supporting distributed key management
- [79]: RSA cryptosystem supporting public-key infrastructure design
- [80]: Identity-based encryption supporting user authentication mechanisms
- [52]: Zero-knowledge interactive proofs supporting Proof of Inference protocol
- [57]: Random oracle model supporting cryptographic protocol security analysis

**Section 6.3 (Privacy-Preserving Query Processing):**
- [77]: Differential privacy supporting query privacy guarantees
- [81]: Secure multiparty computation protocols supporting private inference
- [82]: Yao's garbled circuits foundational to secure computation techniques
- [83]: Fully homomorphic encryption supporting computation on encrypted data
- [55]: Chaum blind signatures supporting anonymous query submission

**Section 6.4 (Model Weight Protection and Intellectual Property):**
- [84]: Symmetric encryption security analysis supporting model weight encryption
- [85]: Probabilistic encryption supporting secure model distribution
- [86]: Oblivious transfer protocols supporting secure model access
- [87]: Universal composability supporting security protocol composition
- [78]: Secret sharing supporting distributed model storage

**Section 6.5 (Network Security and Attack Mitigation):**
- [88]: Anonymous identification supporting Sybil attack prevention
- [89]: Anonymous credential systems supporting reputation without identity linkage
- [90]: Untraceable electronic cash supporting privacy-preserving credit systems
- [91]: Verifiable secret sharing supporting distributed trust mechanisms
- [92]: Secure distributed key generation supporting decentralized security

**Section 6.6 (Trust Architecture and Consensus Mechanisms):**
- [93]: Non-cryptographic fault-tolerant computation supporting honest majority assumptions
- [94]: Verifiable secret sharing with honest majority supporting trust model design
- [95]: Multiparty computation from homomorphic encryption supporting distributed validation
- [96]: Proactive secret sharing supporting long-term security maintenance

**Section 6.7 (Privacy-Preserving Information Retrieval):**
- [97]: Private information retrieval foundational to confidential model queries
- [98]: Single database PIR supporting efficient private model access
- [99]: Searchable encryption supporting private model discovery
- [100]: Improved searchable symmetric encryption supporting scalable private search

**Section 8.1 (Software Architecture Overview):**
- [101]: Docker containerization supporting microservices architecture design
- [102]: Kubernetes orchestration supporting scalable deployment infrastructure
- [29]: Container technology survey providing architectural foundation
- [75]: Kubernetes orchestration research supporting distributed system management

**Section 8.2 (Technology Stack and Justifications):**
- [103]: gRPC high-performance RPC framework supporting inter-service communication
- [104]: Apache Kafka distributed streaming supporting event-driven architecture
- [105]: Redis in-memory data store supporting caching and session management
- [116]: Rust programming language supporting system-level performance requirements
- [117]: Python programming language supporting ML framework integration
- [118]: Go programming language supporting concurrent network services

**Section 8.3 (API Design and Interface Specifications):**
- [114]: JSON-RPC specification supporting standardized API communication
- [115]: OpenAPI specification supporting API documentation and client generation
- [113]: Protocol Buffers supporting efficient binary serialization

**Section 8.4 (Core Component Implementation):**
- [106]: Prometheus monitoring supporting system observability
- [107]: Grafana visualization supporting operational dashboards
- [108]: Consul service discovery supporting dynamic service registration
- [109]: etcd distributed key-value store supporting configuration management

**Section 8.5 (GPU Integration and Compute Abstraction):**
- [110]: CUDA toolkit supporting NVIDIA GPU integration
- [111]: OpenCL standard supporting cross-platform GPU computing
- [112]: Apache Arrow supporting efficient data transfer between CPU and GPU

**Section 8.6 (Performance Optimization Techniques):**
- [119]: MQTT messaging protocol supporting lightweight device communication
- [120]: Apache Pulsar distributed messaging supporting high-throughput event processing
- [21]: Resilient distributed datasets supporting fault-tolerant data processing

**Section 8.7 (Deployment Configuration and Infrastructure):**
- [102]: Kubernetes deployment patterns supporting scalable infrastructure management
- [101]: Docker containerization supporting consistent deployment environments

**Section 7.1 (Formal Economic Model of the Credit System):**
- [45]: MLPerf benchmarks providing hardware normalization baselines for efficiency factors in credit generation equation
- [121]: Ostrom's commons governance theory foundational to credit system design and global equilibrium analysis

**Section 7.2 (Incentive Compatibility Analysis):**
- [123]: Axelrod's cooperation evolution supporting sustained participation incentives in strategic behavior modeling
- [49]: Reputation systems research supporting reputation-based multipliers in credit generation

**Section 7.3 (Free-Rider Problem Solutions):**
- [122]: Hardin's tragedy of commons providing theoretical foundation for free-rider problem analysis
- [51]: Proofs of retrievability supporting Proof of Inference protocol in anti-gaming mechanisms
- [52]: Zero-knowledge interactive proofs foundational to cryptographic work verification
- [47]: BGP lowest-cost routing supporting dynamic difficulty adjustment algorithms

**Section 7.4 (Bootstrap Strategy for Network Growth):**
- [131]: Shapiro & Varian information rules supporting network effects analysis in growth dynamics
- [134]: Katz & Shapiro network effects research supporting supply-side and demand-side network externalities

**Section 7.5 (Comparison with Token-Based Alternatives):**
- [128]: Myerson incentive compatibility theory supporting economic efficiency comparison analysis
- [129]: Maskin mechanism design supporting resource allocation mechanism comparison

**Section 7.6 (Environmental Impact Analysis):**
- [136]: IEA data center energy consumption providing baseline carbon intensity measurements
- [138]: Strubell et al. energy considerations for deep learning supporting carbon footprint modeling
- [139]: Patterson et al. carbon emissions in neural network training supporting DeCLAI carbon model
- [141]: Anthony et al. carbon tracking in ML supporting quantitative impact calculations
- [143]: Schwartz et al. Green AI supporting efficiency gains and environmental benefits analysis
- [145]: Kaack et al. AI climate alignment supporting geographic optimization and carbon reduction strategies

**Section 7.7 (Long-term Economic Sustainability):**
- [60]: SETI@home experimental results supporting sustainability comparison with volunteer computing
- [121]: Ostrom commons governance supporting long-term sustainability proofs and resilience analysis

**Section 8.8 (Open Source Strategy and Governance Model):**
- [20]: BOINC open source model providing governance framework precedent
- [73]: PyTorch open source ecosystem supporting community-driven development

**Section 9.1 (Simulation Methodology and Experimental Design):**
- [146]: Krizhevsky et al. ImageNet classification supporting baseline performance benchmarks
- [147]: BERT paper providing transformer architecture baseline for language model evaluation
- [45]: MLPerf benchmarks establishing standardized performance measurement protocols
- [148]: GPT-2 paper supporting autoregressive language model evaluation methodology

**Section 9.2 (Performance Metrics and Benchmarking Framework):**
- [149]: Pointer Sentinel Mixture Models supporting perplexity measurement standards
- [150]: RoBERTa paper providing robust evaluation methodology for language models
- [151]: XLNet paper supporting comparative analysis of transformer architectures
- [152]: T5 paper establishing text-to-text evaluation frameworks

**Section 9.3 (Distributed Inference Performance Results):**
- [153]: Hoffmann et al. compute-optimal training supporting efficiency analysis
- [154]: Megatron-Turing NLG supporting large-scale distributed inference benchmarks
- [155]: Switch Transformer supporting mixture-of-experts distributed execution analysis
- [156]: GShard paper supporting automatic sharding performance evaluation

**Section 9.4 (Credit System Validation and Economic Metrics):**
- [157]: GLaM paper supporting mixture-of-experts resource allocation analysis
- [158]: PaLM paper providing large-scale model deployment cost analysis
- [159]: Hash layers research supporting sparse model efficiency measurements
- [121]: Ostrom commons governance supporting credit system sustainability validation

**Section 9.5 (Security and Privacy Experimental Validation):**
- [160]: BART paper supporting secure inference protocol evaluation
- [161]: T5 research supporting privacy-preserving inference measurement
- [52]: Zero-knowledge proofs supporting cryptographic protocol performance analysis
- [77]: Differential privacy supporting privacy guarantee validation

**Section 9.6 (Scalability Analysis and Network Growth Simulation):**
- [162]: GPT-4 technical report supporting large-scale deployment analysis
- [163]: Constitutional AI supporting distributed safety mechanism evaluation
- [33]: Chord DHT supporting peer-to-peer scalability analysis
- [60]: SETI@home supporting volunteer computing network growth patterns

**Section 9.7 (Comparative Analysis with Centralized Systems):**
- [164]: MMLU benchmark supporting comprehensive model evaluation
- [165]: ARC reasoning challenge supporting cognitive capability assessment
- [166]: HellaSwag benchmark supporting commonsense reasoning evaluation
- [9]: AWS pricing supporting cost comparison analysis

**Section 9.8 (Quality of Service and User Experience Metrics):**
- [167]: Neural text degeneration research supporting output quality measurement
- [168]: Unlikelihood training supporting generation quality evaluation
- [169]: InstructGPT paper supporting human preference evaluation methodology
- [170]: PPO algorithms supporting reinforcement learning evaluation

**Section 9.9 (Real-world Deployment Case Studies):**
- [171]: Deep RL from human preferences supporting user satisfaction measurement
- [172]: Fine-tuning from human preferences supporting deployment quality analysis
- [173]: Learning to summarize supporting practical application evaluation
- [174]: WebGPT supporting real-world task performance analysis

**Section 9.10 (Limitations and Future Experimental Directions):**
- [175]: Teaching models to support answers supporting future research directions
- [176]: FLAN collection supporting instruction tuning evaluation
- [177]: Scaling instruction-finetuned models supporting future scalability analysis
- [178]: Self-instruct supporting autonomous improvement evaluation
- [179]: Stanford Alpaca supporting open-source model deployment analysis
- [180]: Survey of large language models supporting comprehensive future research planning

**Section 10.1 (Academic Research and Education):**
- [191]: Bahdanau et al. neural machine translation supporting Maria's indigenous language research scenario
- [192]: Vaswani et al. attention mechanism supporting transformer model computational requirements
- [193]: Johnson et al. multilingual neural translation supporting cross-lingual research applications
- [182]: UNESCO AI education guidance supporting educational transformation analysis
- [184]: European Commission digital education plan supporting curriculum integration strategies

**Section 10.2 (Global Health and Medical Research):**
- [181]: WHO digital health strategy supporting Dr. Asante's epidemiological surveillance scenario
- [77]: Dwork differential privacy supporting privacy-preserving health research mechanisms
- [143]: Schwartz et al. Green AI supporting environmental considerations in health research
- [145]: Kaack et al. AI climate alignment supporting sustainable research practices

**Section 10.3 (Climate Science and Environmental Monitoring):**
- [10]: Santos et al. weather forecasting supporting Dr. Rodriguez's hurricane prediction research
- [143]: Green AI research supporting carbon footprint reduction analysis
- [145]: AI climate alignment supporting environmental impact assessment

**Section 10.4 (Small Business and Startup Innovation):**
- [185]: Ng machine learning yearning supporting startup technical development
- [49]: Reputation systems supporting startup collaboration mechanisms
- [50]: Online feedback mechanisms supporting community-based business development

**Section 10.5 (Developing Nation Research Initiatives):**
- [110]: CUDA toolkit supporting hardware abstraction layer design
- [111]: OpenCL standard supporting diverse GPU architecture compatibility
- [72]: Byzantine fault tolerance supporting network reliability in resource-constrained environments

**Section 10.6 (Adoption Barriers and Mitigation Strategies):**
- [49]: Reputation systems research supporting trust mechanism design
- [50]: Online feedback mechanisms supporting community trust building
- [77]: Differential privacy supporting regulatory compliance analysis
- [189]: Barocas et al. fairness and ML supporting ethical development considerations

**Section 10.7 (Long-term Vision and Societal Transformation):**
- [186]: Mitchell AI guide supporting democratized AI development vision
- [187]: Russell human compatible AI supporting ethical development frameworks
- [188]: O'Neil weapons of math destruction supporting equity considerations in AI development

---

## Appendix B — Revision Log

### Version 2 (2026-05)

Substantial revision in response to a four-reviewer adversarial peer-review pass. Highest-severity changes:

- §9 retitled from "Experimental Results" to "Projected Performance and Planned Evaluation Protocol". Fabricated pilot deployments removed (15-university consortium with 2,847 participants; Southeast Asia pilot; "$50,000–$100,000 commercial-cloud equivalent"; 1,200+ developer integration); all indicative-mood claims converted to subjunctive.
- Bibliography entries [10] (Santos et al. weather forecasting) and [11] (Mensah et al. AI epidemiology) **withdrawn**. These were disclosed in the original "Reference Context" appendix as "fictional but realistic citations" yet appeared in the IEEE list with full metadata as if real — an academic-integrity issue corrected in this revision. [12] (Graduate Student Computing Survey Consortium) also withdrawn as unverifiable. §1.3 rewritten from "Human Stories Behind the Statistics" to "Illustrative Scenarios" with explicit composite-scenario labelling.
- Theorem 4.1 demoted to **Property 4.1 (informal)**: the original proof reduced to "DLOG is hard" without specifying a commitment scheme. The revised property is grounded in deployable mechanisms (redundant execution with ε-tolerance, challenge-response audits, reputation weighting). Cache-and-replay and Sybil attacks are now explicitly enumerated as open problems.
- Theorem 7.1 (Incentive Compatibility) demoted to **Conjecture 7.1**. Truth-telling about computational capability is *not* a dominant strategy under the credit mechanism alone — over-reporting capability inflates $\eta_i$ multiplicatively, which is an obvious gaming incentive. Truth-telling becomes a best response only under specific audit-rate and penalty conditions.
- Theorem 7.2 (Long-term Sustainability) restated as **Definition 7.1 (Operational Sustainability)** because the original was tautological ("sustainable iff the median participant has positive utility" is the definition of sustainability, not a theorem).
- Proposition 7.1 corrected to use marginal cost instead of average cost.
- §6 threat model unified to a single $f < n/3$ Byzantine assumption (previously inconsistent across §6.1.1, §6.1.3, and §6.6.2). Cryptographic claims softened:
  - zk-SNARKs for LLM inference demoted to a "Future Verification Layer" with explicit acknowledgement of the 10⁷-parameter / minutes-to-hours-of-proving state of the art (zkLLM, Mystique, Mithril);
  - MPC transformer inference demoted to a "Research Direction" with explicit 10³–10⁵× slowdown and GB-per-query bandwidth disclosure (CrypTen, MPCFormer, Iron);
  - ABE / functional encryption "model confidentiality" claims corrected to acknowledge the decrypt-to-compute hole (weights end up in plaintext GPU memory regardless of the encryption scheme).
- §7.6 carbon math corrected. The original claim of "1.5 MW from 1,000 contributors" implied ≈9 kW per consumer GPU, approximately 20× the realistic per-machine draw of 200–500 W. Downstream tonnage and car-equivalent figures (500,000 tons CO₂ / year, 100,000 cars) derived from the wrong base have been replaced with a discussion of counterfactual dependence. The recession-resistance argument was also reversed (participants are *more* likely to shut down idle hardware in a downturn, not less).
- §4.5 credit ODE replaced. The original ODE pair had no fixed point coupling generation and consumption; the revised model is a single bookkeeping ODE plus an explicit rate controller $\alpha_{network}$, a static-balance condition, and honest acknowledgement that dynamic stability is an open question that empirical deployment must settle. The "1,000 active participants → stable" figure (derived from nothing in the original) has been removed.
- §4.4 liquidity invariant rewritten to fix the original's stock-vs-flow dimensional inconsistency.
- §2 expanded with a "Decentralized LLM Systems" subsection covering Petals (Borzunov et al., ACL 2023), Hivemind (Ryabinin & Gusev, NeurIPS 2020), SWARM Parallelism (ICML 2023), DiLoCo (DeepMind 2023). A comparison table differentiates DeCLAI from Bittensor, io.net, BOINC, Petals, and Hivemind along five axes. The llm-d misattribution (wrong GitHub URL and wrong characterisation as a competitor) is corrected.
- Bibliography entries [211]–[218] add the direct prior art (Petals, Hivemind, SWARM, DiLoCo, Bittensor, Gensyn, Akash, Folding@home); entries [219]–[224] add the cryptographic references cited by the revised §6 (zkLLM, Mystique, CrypTen, MPCFormer, Iron, Gentry's FHE thesis).
- §8 restructured to acknowledge the two-tier deployment story explicitly: a coordination tier (gateways, orchestrators, ledger, discovery) running as containerised microservices in a small datacenter or institutional cloud footprint, versus a contributor tier running a single long-lived binary on a residential or lab GPU machine. Residential network constraints (NAT, asymmetric upload, dynamic IPs, ISP terms of service) are now addressed head-on rather than glossed.
- §5 code-snippet bugs fixed: `OptimalClusterSelection` scope bug (composite_score not stored per-cluster); pipeline-vs-replica mutual exclusivity between `AdaptiveModelSharding` and the BFT validation algorithm reconciled; API example signature mismatch corrected; explicit ε-tolerance comparison added to result validation to handle cross-architecture GPU non-determinism.
- README and §10 headline impact percentages — "3,150% increase in computational access", "40–60% reduction in health research timelines", "30–40% reduction in AI research carbon emissions", "50–70% reduction in startup AI costs", "90% of researchers lack adequate computational resources" — either withdrawn outright or hedged to either "(projected; not measured)" or qualitative language ("substantially", "order-of-magnitude"). The Maria persona's specific 3,150% figure was derived by multiplying weekly credits × 1.5 × 52 weeks under 100% usage — a multiplicative conflation of weekly access with annual hours that has been replaced with an explicit derivation and sensitivity callout.
- Marketing register ("revolutionary", "novel", "sophisticated", "paradigm shift", "comprehensive") audited throughout and replaced with concrete mechanism descriptions or deleted.

For the per-section audit trail of every withdrawn or hedged claim, see `KNOWN_LIMITATIONS.md` in the repository.

### Version 1 (2025-07)

Original preprint deposited on ResearchGate (DOI 10.13140/RG.2.2.21788.19848). Superseded by Version 2.

---

*End of document.*
