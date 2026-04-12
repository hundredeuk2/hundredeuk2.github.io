---
title: "From Implicit to Explicit Rewards: Why Rubric-Based Evaluators Are the Next Frontier"
date: 2026-04-05
publish: true
tags:
  - RL
  - evaluation
  - rubric
  - reward-model
---

## The Quiet Shift Happening in RL-based LLM Training

Reinforcement learning from human feedback (RLHF) has been the dominant post-training paradigm for large language models since Ouyang et al. (2022) demonstrated that scalar reward signals derived from human preferences could dramatically improve instruction-following behavior [1]. At its core lies a deceptively simple component: the reward model. Given a prompt and a response, it outputs a scalar — a number that tells the policy "this was good" or "this was bad."

But that number has always been opaque.

Nobody — not the engineer, not the model being trained — knows exactly *why* a response scored 0.83 instead of 0.71. The reward is implicit: a compressed judgment with no justification attached. For a long time, this was considered an acceptable trade-off. The numbers worked well enough.

![From implicit to explicit rewards](../asset/rewards_figure_1.avif)

That assumption is now being challenged. As Gunjal et al. (2025) observe, preference-based reward models tend to overfit superficial artifacts — response length, formatting quirks, annotator biases — and require large volumes of pairwise comparisons that are both costly and brittle [2]. Concurrently, Viswanathan et al. (2025) make the case even more directly: in their NeurIPS 2025 paper, they demonstrate that checklist-based feedback — instruction-specific, structured criteria — consistently outperforms traditional reward models across five instruction-following benchmarks, and is the only method to improve performance on every benchmark tested [3].

The field is quietly converging on a new direction: **structured, explicit criteria as the foundation for reward signals.**

## What "Explicit" Actually Means

An explicit reward is one that comes with a reason.

Instead of a scalar produced by a black-box classifier, an explicit reward is derived from a rubric — a structured set of evaluation criteria that the model can read, interpret, and apply. Gunjal et al. (2025) formalize this distinction precisely, defining two reward aggregation strategies [2]:

**Explicit Aggregation** evaluates each criterion independently and combines them into a normalized scalar:

$$
r(x, \hat{y}) = \frac{\sum_{j=1}^{k} w_j \cdot c_j(x, \hat{y})}{\sum_{j=1}^{k} w_j}
$$

where $w_j$ is the weight of criterion $j$ and $c_j$ is a binary correctness function indicating whether the response satisfies that criterion. Crucially, this formulation subsumes **RLVR** as a special case: when $k=1$ and $c_1$ reduces to an exact match function, you recover standard verifiable reward.

**Implicit Aggregation**, by contrast, delegates the entire judgment to an LLM-as-judge in a single forward pass — producing a holistic scalar without exposing intermediate reasoning.

This distinction matters for three reasons. First, **interpretability**: when a reward signal has a rationale attached, both researchers and the model being trained can understand what "quality" means in context. Second, **generalization**: a rubric-based evaluator isn't tied to the distributional patterns of a fixed training set — it can apply consistent principles to novel responses. Third, **controllability**: as Mu et al. (2024) demonstrate in the context of safety alignment, rule-based rewards provide stable, auditable supervision that is far less susceptible to reward hacking than opaque neural reward models [4].

Viswanathan et al. (2025) extend this argument to general instruction following, showing that flexible, instruction-specific checklists — generated synthetically from instructions and graded by a combination of AI judges and specialized verifiers — produce more stable and generalizable training signals than fixed-criteria reward models [3].

## Prometheus: Proof of Concept for Explicit Evaluation

The Prometheus paper (Kim et al., 2023) [5] was among the first to demonstrate that a small language model — trained specifically to evaluate against a rubric — could produce judgments competitive with GPT-4, at a fraction of the cost. The key insight was deceptively simple: don't train a model to output a score; train it to follow an evaluation instruction.

But Prometheus went further than evaluation. In its ablation studies, the paper demonstrated that a rubric-based evaluator sLM could be plugged directly into a reinforcement learning loop as the reward model — replacing the implicit scalar reward with an explicit, criterion-grounded signal. The policy trained against this explicit reward showed stable improvement without the reward hacking behaviors commonly observed in standard RLHF.

More recently, Gunjal et al. (2025) strengthened this case with RaR, achieving up to **31% relative improvement on HealthBench** and **7% on GPQA-Diamond** over popular LLM-as-judge baselines that rely on direct Likert-based rewards — and demonstrating that rubric-based rewards provide more stable supervision across judge sizes [2].

A well-trained rubric-based evaluator can function as a reward model. This is now more than a hypothesis.

## The Implication: Evaluation and Training Are the Same Problem

If Prometheus is right, then the boundary between evaluation and post-training begins to dissolve.

An evaluator that can explain *why* a response is good is, by definition, encoding the same knowledge that a policy needs to *generate* good responses. The two tasks — judging quality and producing quality — are two sides of the same coin.

This is the hypothesis our current research is built on:

> A sufficiently capable explicit evaluator, trained to score responses against interpretable rubrics, can serve as a stable and generalizable reward model for reinforcement learning — making the post-training feedback loop more transparent, more controllable, and more data-efficient.

The practical implication is significant. Rather than training a separate, opaque reward model from human preference data, one could build a single rubric-following evaluator sLM that simultaneously serves both purposes: evaluation for deployment quality assurance, and reward modeling for continued post-training.

## What We're Building Toward

Our own work — currently under review as **E-Star** — attempts to operationalize this hypothesis in the context of Korean language model evaluation. We train a small evaluator model (Gemma-3-12B-IT) to follow structured rubrics across Korean NLP evaluation tasks, and test whether its scoring signal is stable and informative enough to serve as an explicit reward in a post-training loop.

The early results are promising — but they've also surfaced a deeper problem that prior work didn't fully address: the quality of the evaluator is fundamentally constrained by the diversity of the training data it has seen.

That problem, and the architectural solution we're designing around it, is the subject of our next post.

## References

[1] Ouyang, L., Wu, J., et al. (2022). *Training language models to follow instructions with human feedback.* NeurIPS 2022. [arXiv:2203.02155](https://arxiv.org/abs/2203.02155)

[2] Gunjal, A., Wang, A., et al. (2025). *Rubrics as Rewards: Reinforcement Learning Beyond Verifiable Domains.* ICLR 2026. [arXiv:2507.17746](https://arxiv.org/abs/2507.17746)

[3] Viswanathan, V., Sun, Y., et al. (2025). *Checklists Are Better Than Reward Models For Aligning Language Models.* NeurIPS 2025. [arXiv:2507.18624](https://arxiv.org/abs/2507.18624)

[4] Mu, T., et al. (2024). *Rule based rewards for language model safety.* NeurIPS 2024.

[5] Kim, S., et al. (2023). *Prometheus: Inducing Fine-grained Evaluation Capability in Language Models.* [arXiv:2310.08491](https://arxiv.org/abs/2310.08491)
