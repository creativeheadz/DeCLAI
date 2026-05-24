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