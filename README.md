# ArchitecturePortfolio

How I make architecture decisions: what I look at first, how I test a choice, and when I change course.

## Architecture guides

- [Split the code before you split the deployment](case-studies/cyys-frame-one-process-explicit-boundaries.md) — Decide whether a growing Java backend needs clearer modules, another worker, or another service.
- [Put the quality test before the generation pipeline](case-studies/game-frame-animation-first-pass-quality.md) — Design a first-pass acceptance test before choosing between hosted and local generation.
- [Let the model propose the query. Decide who can run it.](case-studies/cyys-ai-platform-architecture.md) — Review generated query plans, check access on the server, and handle uncertain retries.
