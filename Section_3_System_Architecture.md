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