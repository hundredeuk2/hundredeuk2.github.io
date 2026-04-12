---
title: Heondeuk Lee
---

<div class="landing-hero">
  <img src="asset/profile.JPG" alt="Heondeuk Lee" class="landing-hero-photo" />
  <div class="landing-hero-text">
    <h1 class="landing-hero-name">Heondeuk Lee</h1>
    <p class="landing-hero-role">AI Research Engineer</p>
    <p class="landing-hero-bio">
      I build models that Train better and Evaluate more honestly.
      I design post-training data pipelines to shape how models think,
      and develop evaluation frameworks to verify they actually do.
      Training and evaluation elevating each other — that's the cycle I'm building toward,
      one paper and one deployment at a time.
    </p>
    <p class="landing-hero-tags">EMNLP 2025 · Financial sector deployment (3 companies) · Open-source contributor</p>
  </div>
</div>

<div class="landing-grid">
  <section class="landing-col">
    <h2>Research Interests</h2>
    <div class="landing-item">
      <h3>Efficient RL &amp; Data Synthesis</h3>
      <p>Making reinforcement learning more accessible by reducing its dependency on massive compute. CAC-CoT is one example: a data synthesis pipeline that shapes reasoning behavior without expensive RL, achieving the same goal more efficiently.</p>
    </div>
    <div class="landing-item">
      <h3>Explicit &amp; Explainable Evaluation</h3>
      <p>Building evaluation frameworks where scoring is not a black box. A good evaluator should tell you not just how much, but why — making the feedback loop between training and evaluation transparent and controllable.</p>
    </div>
    <div class="landing-item">
      <h3>Evaluator as Reward Model</h3>
      <p>Exploring whether a well-designed explicit evaluator can replace implicit reward signals in RLHF, turning evaluation into a direct driver of better training.</p>
    </div>
    <a class="landing-cta" href="research-statement">Read full research statement →</a>
  </section>

  <section class="landing-col">
    <h2>Publications</h2>
    <div class="landing-item">
      <h3>CAC-CoT: Connector-Aware Compact Chain-of-Thought for Efficient Reasoning Data Synthesis Across Dual-System Cognitive Tasks</h3>
      <p class="landing-venue">EMNLP 2025 Findings · Accepted</p>
      <p>Dynamically injects Reflection or Confidence connectors based on certainty at each reasoning step, terminating unnecessary reasoning at the data level — without RL.</p>
      <p class="landing-metric">&gt;20% accuracy on S1-Bench over SOTA (o3, LIMO) · ~4–5× token reduction to ~500 tokens</p>
    </div>
    <div class="landing-item">
      <h3>Enhancing Multi-step Reasoning with Improved Representation from Large Language Models</h3>
      <p class="landing-venue">2023</p>
      <p>Supervised Alignment Tuning (SAT): a KD-based approach that self-evaluates LLM-generated CoT rationales and learns from preferred outputs via a combined Cross-Entropy and Ranking Loss.</p>
      <p class="landing-metric">150% performance gain per parameter — GPT-3 (6.7B) 6.7 pts → Ours (0.7B) 9.62 pts</p>
    </div>
  </section>
</div>

