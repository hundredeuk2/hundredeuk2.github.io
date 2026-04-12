---
title: "Trains Itself: A Co-Evolution Approach to Rubric-Based RL"
date: 2026-04-05
publish: true
tags:
  - RL
  - evaluation
  - rubric
  - distillation
---

## We Tried the Obvious Thing First

After establishing that rubric-based evaluators can serve as explicit reward models, the natural next step was simple: take Prometheus — the best-known rubric-following evaluator — fine-tune it on Korean financial data, and plug it into our evaluation pipeline.

So we did exactly that.

Prometheus was trained on a carefully curated dataset of 99.7K feedback instances. It works remarkably well in English. The model follows rubric instructions, produces coherent critiques, and generates scores that align with GPT-4 judgments at an impressive rate.

Then we switched to Korean. And things fell apart.

![DrZero](../asset/drzero_figure_1.avif)

## The Numbers That Surprised Us

The degradation wasn't subtle. Measuring evaluator reliability via Pearson correlation with human judgments, the numbers told a stark story:

| Setting | Pearson r |
| --- | --- |
| English (original Prometheus) | 0.832 |
| Korean (direct translation) | 0.377 |
| Faithfulness (Korean) | 0.333 |
| Context Relevancy (Korean) | 0.392 |

A direct translation of the prompts — no other changes — cut correlation by more than half. The model wasn't just slightly worse. It was operating in a fundamentally different regime, one where its scores were barely more reliable than chance on certain criteria.

The immediate diagnosis was straightforward: Prometheus's internal representations are deeply English-centric. Its understanding of what "accurate" or "faithful" means has been baked in through 99.7K English preference pairs. Korean isn't just a different language — for this model, it's effectively out-of-distribution (OOD) data.

## The Deeper Problem: 99.7K Is Both Too Much and Too Little

This is where the problem gets structurally interesting.

Prometheus's 99.7K dataset isn't something you can simply replicate in Korean. It represents an enormous curation effort — diverse prompts, carefully written rubrics, detailed feedback, and human-validated scores across many domains. Building an equivalent Korean dataset from scratch would require comparable resources, and even then, you'd be solving for distribution coverage rather than generalization.

We ran additional experiments to confirm this wasn't just a translation artifact. Fine-tuning on the available Korean data improved in-distribution performance, but OOD performance — responses the model hadn't seen response patterns for — remained fragile. The evaluator had learned *which* responses to score well, not *why* a response is good.

This is the core failure mode: a dataset-dependent evaluator memorizes scoring patterns rather than internalizing evaluation principles.

The fix isn't more data. The fix is a different kind of learning.

## Before We Get to Co-Evolution: Why RL Alone Won't Save Us

At this point, one might ask: can we simply apply reinforcement learning to the evaluator and let it self-improve? Run RL, get a better evaluator, done.

Recent work from Yue et al. (2025) — "Does Reinforcement Learning Really Incentivize Reasoning Capacity in LLMs Beyond the Base Model?" (NeurIPS 2025 Best Paper Runner-Up) [1] — gives us a clear and sobering answer: no, not on its own.

Their key finding, demonstrated across math, coding, and vision benchmarks: RLVR (Reinforcement Learning with Verifiable Rewards) does not expand a model's reasoning capacity. It only improves sampling efficiency within the solution space the base model already has. When they measure pass@k, RL-trained models outperform base models at small k, but base models consistently catch up and surpass RL-trained models as k grows. Every correct solution the RL model produces was already present in the base model's distribution.

The mechanism is intuitive once you see it. Under a binary 0/1 reward scheme, if the base model never samples a correct solution for a given problem, RL receives no gradient signal — it literally has nothing to learn from. The RL optimizer can only amplify what the base model already knows, not discover what it doesn't.

The implication for a small model like Kanana — or our evaluator sLM — is direct and uncomfortable: if the base model's solution space is narrow, RL will optimize within that narrow space. It cannot escape it. For a Korean evaluator trained on limited data, RL would simply reinforce the same limited scoring patterns more efficiently. The OOD fragility wouldn't improve — it would harden.

Yue et al. draw the same conclusion explicitly:

> Distillation can genuinely introduce new knowledge into the model. Distilled models often exhibit an expanded scope of reasoning capability beyond that of the base model, in contrast to RLVR-trained models whose capacity remains bounded by the base.

This is the missing ingredient. Before RL can do useful work, the model needs a distribution that's worth optimizing. That distribution expansion only comes through distillation — from a frontier model that has seen and solved the diversity of inputs we care about.

## The Hypothesis: To Evaluate Well, You Must Know What Perfect Looks Like

The Limit of RLVR finding reframes our problem precisely. The evaluator's OOD fragility isn't a data quantity problem — it's a distribution problem. Our Korean evaluator hasn't been exposed to the full space of possible responses. It doesn't know what a high-quality Korean response looks like across diverse rubric criteria, because its training data never showed it.

This leads to a deeper hypothesis:

> An evaluator that can *generate* a high-quality response — one that satisfies all rubric criteria — will produce more reliable and generalizable evaluations than one trained purely on comparison data.

A skilled human evaluator doesn't just recognize bad responses through pattern matching. They have an internalized model of what an ideal response looks like. They can generate the gold standard mentally, then compare against it. Their evaluation is grounded in *generative understanding* — and that understanding was built through exposure to diverse, high-quality examples.

To build that kind of evaluator, we need two things in sequence: first, a distillation phase to expand the base distribution — exposing the model to diverse, frontier-quality responses; then a co-evolution phase to refine the evaluation signal through iterative self-improvement.

## Designing the Co-Evolution Loop

Inspired by **Absolute Zero [2]** and **Dr. Zero (Yue et al., 2026) [3]**, we are designing a 2-way co-evolution framework where the evaluator doesn't just passively score responses — it actively participates in generating the training signal it needs to improve.

The architecture involves two roles:

- **Solver** — generates candidate responses to prompts, attempting to satisfy rubric criteria and maximize evaluator score.
- **Evaluator** — scores responses against rubric criteria, producing explicit per-criterion feedback.

Training proceeds in two alternating phases, structured as an EM-like optimization:

### Phase A — Evaluator alignment (Solver frozen)

The solver generates responses. A frontier model (GPT-4) provides ground-truth rubric scores. The evaluator updates its weights $\phi$ to minimize the gap between its scores and the frontier model's judgment:

$$
\mathcal{L}_{\text{eval}} = \mathcal{L}\bigl(E_\phi(x, \hat{y}),\ y_{\text{oracle}}\bigr)
$$

This phase sharpens the evaluator's understanding of what rubric-satisfying responses actually look like — crucially, across the diverse responses the solver produces, not a fixed dataset.

### Phase B — Solver training (Evaluator frozen)

The evaluator's weights are now fixed — a stable, reliable judge. The solver updates via **GRPO**, maximizing the reward signal from the frozen evaluator:

$$
\nabla J_{\text{solver}} \approx \mathbb{E}\bigl[(E_\phi(x, \hat{y}) - b)\ \nabla_\pi \log S_\pi(\hat{y} \mid x)\bigr]
$$

A better solver generates more diverse, more challenging responses — which in the next Phase A round, forces the evaluator to handle inputs it has never seen before. This is the mechanism through which distributional diversity is maintained.

### Why alternating, not joint?

Updating both models simultaneously creates two failure modes we explicitly want to avoid. **The moving target problem:** if the evaluator's scoring criteria shift while the solver is trying to learn them, neither model converges. **The collusion problem:** both models can find a degenerate solution — the solver learns to produce responses that game the evaluator's weaknesses, and the evaluator learns to reward them. Freezing one while updating the other, as demonstrated in Dr. Zero [3] and R-Zero, prevents both.

## The Full Picture: Distillation First, Then Co-Evolution

Putting the Limit of RLVR finding together with our framework, the design philosophy becomes clear:

- **Distillation phase**: expose the Solver and Evaluator base models to diverse, frontier-quality response distributions. This is the only mechanism that can genuinely expand what the models are capable of — RL cannot do this alone.
- **Co-evolution phase**: with a sufficiently rich base distribution established, alternate between Evaluator alignment (Phase A) and Solver training (Phase B). Each phase feeds the other: a better evaluator produces a more reliable reward signal, a more capable solver generates harder training examples.

The Limit of RLVR doesn't undermine the co-evolution approach — it clarifies where it sits in the training pipeline. RL is still essential for extracting and sharpening what the model knows. But it cannot create knowledge that was never there. Distillation lays the foundation; co-evolution builds on top of it.

## What We're Testing Next

The central question of our upcoming experiments:

> Does an evaluator trained through this co-evolution loop — on top of a distillation-expanded base — produce more reliable and generalizable reward signals than a conventionally trained reward model?

If yes, it opens a concrete training recipe for small, domain-specialized models: distill first to expand distribution, then co-evolve to refine precision. Results to follow.

## References

[1] Yue, Y., Chen, Z., et al. (2025). *Does Reinforcement Learning Really Incentivize Reasoning Capacity in LLMs Beyond the Base Model?* NeurIPS 2025. [arXiv:2504.13837](https://arxiv.org/abs/2504.13837)

[2] Zhao, A., Wu, Y., et al. (2025). *Absolute Zero: Reinforced Self-play Reasoning with Zero Data.* [arXiv:2505.03335](https://arxiv.org/abs/2505.03335)

[3] Yue, Z., Upasani, K., et al. (2026). *Dr. Zero: Self-Evolving Search Agents without Training Data.* [arXiv:2601.07055](https://arxiv.org/abs/2601.07055)

[4] Kim, S., et al. (2023). *Prometheus: Inducing Fine-grained Evaluation Capability in Language Models.* [arXiv:2310.08491](https://arxiv.org/abs/2310.08491)
