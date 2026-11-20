# ArchitecturePortfolio

I write about how I make architecture decisions: what I look at first, what I test, and when I change course.

Start with [How I make architecture decisions](how-i-make-architecture-decisions.md). Each case below starts with a real project and uses it to work through one decision.

## Architecture guides

1. **CYYS Framework** · [Split the code before you split the deployment](case-studies/cyys-frame-one-process-explicit-boundaries.md) — A Java administration backend. How to compare modules, workers, replicas and services by what they let you change independently.
2. **gameFrameAnimation** · [Put the quality test before the generation pipeline](case-studies/game-frame-animation-first-pass-quality.md) — A game animation tool in design. How to compare hosted and offline generation by the first result you can actually use.
3. **CYYS AI Platform** · [Let the model propose the query. Decide who can run it.](case-studies/cyys-ai-platform-architecture.md) — An analytics workflow. How to separate interpreting a question, approving the plan and enforcing access.
