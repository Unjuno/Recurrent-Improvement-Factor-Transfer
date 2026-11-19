# RIFT — Recurrent Improvement Factor Transfer

**Post-hoc extraction and transfer of recurrent improvement operators from pretrained Transformers.**

**研究プロジェクト / Research project.** RIFT investigates whether the task-relevant computations of a pretrained looped Transformer can be isolated and transferred to an independent lightweight recurrent operator. The teacher Transformer runs **once at inference**, and the extracted operator performs subsequent state updates.

> **Status (2026-11-19): Exploratory proof of concept, not peer reviewed.** Current evidence comes from small synthetic permutation-following tasks. The general method, its novelty relative to prior work, and any advantage of energy-based updates remain unverified.

## Research question / 中心課題

Can we identify an existing recurrent model's **causally important state-update computation**, transfer it to a smaller independent recurrent module, and preserve accuracy and long-horizon behavior at lower inference cost?

```text
Training / extraction:
  Pretrained looped Transformer
     -> state trajectories + causal interventions
     -> identify candidate update factors
     -> learn a compact operator (direct / energy-based / hybrid)

Inference:
  Input -> Transformer F (once) -> initial query and memory states
                                     |
                                     v
                            lightweight operator G
                            (repeated; no further F)
                                     |
                                     v
                                   output
```

**This is not just structural pruning.** Finding removable FFNs/attention heads can support the analysis, but the core goal is to **reconstruct and transfer the recurrent update rule**, even when the student's architecture differs from the teacher.

## What has been observed

| Stage | Question | Summary | Status |
| --- | --- | --- | --- |
| 1–3 | Can components be ablated or compressed? | Small attention/FFN/head circuits can be compressed on synthetic tasks; scratch-trained small models sometimes reach the same accuracy. | Supporting controls, not the central RIFT claim |
| **4** | Can a separately learned factor-update module replace later Transformer passes? | Yes, in a six-node synthetic task. Controlled trials achieved 100% three-hop accuracy for 4/5 teachers (the fifth: 98.61%); ~65% estimated MAC reduction; 2.12× CPU inference speedup at batch 128. | **Existence example** |
| 5 | Are input-dependent factor bases consistently better? | Mixed results; student initialization strongly affects success. | No general superiority established |
| **6** | Can off-trajectory teacher queries improve long rollouts? | Six-hop mean accuracy: ordinary distillation 60.60%; uniform off-trajectory update distillation 80.42%; causally weighted sampling 80.97% (5 teacher seeds). | Exploratory trend; uncertainty remains |

The teacher must still be loaded for the **first** pass; smaller inference computation does **not** imply smaller total resident model weights. The reported CPU speedups are hardware- and batch-dependent.

**Important:** Stage 4–6 used the same small synthetic task family and repeatedly examined a particular evaluation split. A newly held-out task/split is necessary for confirmatory claims. **Energy-based RIFT has not yet been implemented or validated.**

## Next experiments

1. Freeze a **new evaluation protocol** spanning additional sequential-computation tasks and unseen graph sizes, with independent teacher and student seeds.
2. Compare direct low-rank updates, a **learned low-dimensional energy gradient**, and an energy-gradient-plus-residual hybrid under matched model size and teacher-query budgets.
3. Test whether the teacher's projected update field is **approximately conservative** before claiming an energy-only representation is sufficient.
4. Benchmark against ordinary trajectory distillation, off-trajectory uniform distillation, on-policy distillation, random-factor controls and small recurrent models trained from scratch.
5. Measure short/long-horizon accuracy, factor ablation effects, teacher-query budget, estimated MACs, latency and loaded-parameter memory separately.

See [research plan (日本語)](docs/RESEARCH_PLAN_JA.md) and [experiment history (日本語)](docs/EXPERIMENT_HISTORY_JA.md).

## Relationship to prior work

Existing work already addresses recurrent inference, circuit distillation, on-policy imitation, and energy-based updates. RIFT's **candidate** differentiator is **post-hoc identification and transfer of recurrent update operators from an already-trained teacher**, with interventions and rollout-faithfulness tests. Novelty is a research question, **not an established finding**.

Selected references for comparison:

- [Energy-Based Reasoners (2026)](https://arxiv.org/abs/2611.08977)
- [Inside the Loop (2026)](https://arxiv.org/abs/2610.24135)
- [What Mechanisms Does Knowledge Distillation Distill? (2024)](https://proceedings.mlr.press/v243/wu24a.html)
- [Circuit Distillation (OpenReview)](https://openreview.net/pdf?id=HVShyTzRgy)
- [A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning (DAgger)](https://proceedings.mlr.press/v15/ross11a.html)

## Repository status and reproducibility

This initial commit documents the research and its limitations. Source-code archives, checkpoints, training logs and the original experimental reports are **not yet published in this repository**; they require a dedicated reproducibility and release review. Do not treat the numerical summaries as independently replicated benchmarks.

Research notes are not a license grant. Code and dataset licensing will be selected before their public release.
