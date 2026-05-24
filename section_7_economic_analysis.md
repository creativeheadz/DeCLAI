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
