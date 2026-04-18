# Generative AI Analysis Report
## LLM-Based Operations Advisor for Multimodal Inference Clusters

---

## 1. Overview

This project implements an **LLM-based operations advisor** that generates operational guidance from inference cluster telemetry. Then uses the augmentation model to generate scenarios and transforms structured metrics into diagnostic narratives with recommendations.

**Pipeline:**
1. **Real dataset** — Azure Multimodal Inference Dataset 2025 (1M production requests)
2. **Scenario augmentation** — Transformer model extrapolates stress-test conditions
3. **LLM advisor** — Generates operational guidance 

## 2. Dataset: Azure Multimodal Inference Dataset 2025

### Source and Citation
> Zhang, J., et al. (2025). ModServe: Modality- and stage-aware resource disaggregation for scalable multimodal model serving. *ACM Symposium on Cloud Computing (SoCC '25)*.

### Schema
| Column | Description |
|--------|-------------|
| TIMESTAMP | Request invocation time |
| NumImages | Images per request (0 to 9,409) |
| ContextTokens | Input text tokens (0 to 148,569) |
| GeneratedTokens | Output tokens (0 to 16,384) |

### Exploratory Data Analysis

**Scale:** 1,000,000 inference requests over 7 consecutive days.

**Distribution characteristics:**
| Feature | Mean | Median | Key observation |
|---------|------|--------|-----------------|
| NumImages | 14.3 | 0.5 | 50% text-only, 33% single-image |
| ContextTokens | 2,911 | 1,124 | Right-skewed, log transform applied |
| GeneratedTokens | 187 | 98 | Right-skewed, log transform applied |

**Correlations:**
- NumImages ↔ ContextTokens: **0.76** (strong positive)
- ContextTokens ↔ GeneratedTokens: 0.01 (negligible)

**Temporal patterns:** Clear diurnal cycles, burst structure at short timescales.

## 3. Pipeline Design

### Stage 1: Scenario Augmentation

**Architecture:** 2-layer Transformer, 64-dim, dropout 0.3, weight decay 1e-3

**Training results:**
| Metric | Value |
|--------|-------|
| Best validation NLL | 0.71 |
| Final train - val gap | -0.48 |
| Early stopping | Epoch 20 |

**Observation:** Overfitting persists despite regularization. Validation loss increases after epoch 5.

### Stage 2: LLM Operations Advisor

**Input:** System state with request rate, latencies, modality mix, queue trends, historical context

**Output:** Markdown advisory with situation assessment, root cause analysis, recommendations, impact, risks

## 4. Evaluation Results

### Scenario Augmentation Fidelity

| Feature | KS p-value | Mean diff | Assessment |
|---------|------------|-----------|------------|
| inter_arrival_time | <0.001 | 12% | Acceptable |
| NumImages | <0.001 | **342%** | Failure |
| ContextTokens | <0.001 | 35% | Moderate drift |
| GeneratedTokens | <0.001 | 14% | Acceptable |

**Correlation preservation:** MAE = 0.15

**Critical finding:** The model fails on NumImages. In real data, 83% of requests have 0 or 1 images. The model spreads values across the range, producing a mean of 58 versus the real mean of 14.

### LLM Advisor Output Review

| Scenario | Key input | Advisory response |
|----------|-----------|-------------------|
| normal_operations | 179 req/min, queue decreasing, P99 above SLA | Recommends scaling resources / intervention |
| peak_load | Elevated rate, increasing queue | Recommends scale-up |
| off_peak_idle | Low utilization but severe P99 latency | Recommends scaling resources / investigation, not scale-down |
| image_heavy_shift | High image fraction | Flags modality-specific concerns |
| high_latency_alert | P99 above SLA | Recommends immediate intervention |

**Assessment:** The advisor produces scenario-appropriate responses that reference input metrics. However, without ground truth or operator feedback, accuracy cannot be quantified. Production deployment would require validation against operator decisions.

## 5. Ethical Considerations

### Bias in Augmented Data

**Evidence from our analysis:**

| NumImages | Real % | Synthetic % | Issue |
|-----------|--------|-------------|-------|
| 0 | 50.0% | ~41% | Under-represented |
| 1 | 33.0% | ~0.4% to 0.6% | **Severely under-represented** |
| 2+ | 17.0% | ~58% | Over-represented |

**Impact:** Stress tests using augmented data over-weight multi-image scenarios. Capacity planning tuned on this data may under-serve text-only users (the majority).

### Over-reliance on AI Advisors

LLM-generated guidance is fluent and confident even when potentially incorrect. During high-pressure incidents, operators may defer to the advisor without verification.

**Mitigation:** Display raw metrics alongside advisory; require human approval for scaling actions.

### Fairness

83% of requests have 0-1 images. If augmentation over-represents multi-image workloads, capacity planning may over-provision for minority workloads at the majority's expense.

## 6. Limitations and Future Work

**Current limitations:**
- Augmentation model fails on discrete NumImages distribution (342% error)
- Overfitting persists (train/val gap: 0.48) despite regularization
- Advisor evaluation is qualitative — no validated ground truth
- Single-cluster, 7-day window limits generalization

**Future improvements:**
- Categorical output for NumImages (discrete distribution)
- Reduce model capacity or use simpler baseline
- Validate advisor against operator decisions
- Multi-cluster training data

## 7. Interpretation for a Non-Technical Audience

This project asks a practical question: can AI help operations teams understand what is happening inside a busy multimodal inference system and suggest what to do next? The answer is partly yes. The advisor can turn raw system metrics into readable guidance, and it generally responds in ways that match the type of scenario it is shown.

The main limitation is that the scenario generator does not reproduce the real workload mix well enough, especially for image counts. In the real data, most requests contain zero or one image, but the synthetic data produces far too many multi-image requests. That means the generated stress tests can exaggerate image-heavy traffic and give operators a misleading view of what the system usually faces.

The practical takeaway is that the language-model advisor looks promising as a decision-support tool, but the synthetic data used to test it is not yet reliable enough for high-confidence planning. In plain terms: the reporting layer is useful, but the artificial scenarios feeding it still need improvement before they should influence real operational decisions.

## 8. References

Zhang, J., et al. (2025). ModServe: Modality- and stage-aware resource disaggregation for scalable multimodal model serving. *ACM Symposium on Cloud Computing (SoCC '25)*.

Vaswani, A., et al. (2017). Attention is all you need. *NeurIPS*.

Floridi, L., et al. (2018). AI4People—An ethical framework for a good AI society. *Minds and Machines*, 28(4), 689-707.
