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
