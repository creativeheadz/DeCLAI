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
