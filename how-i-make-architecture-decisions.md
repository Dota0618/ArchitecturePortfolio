# How I make architecture decisions

I start with a decision someone actually has to make. “How should we architect the backend?” is too broad. “Should exports run outside the API process?” gives me a workload to investigate, a failure to reproduce and a reason to compare alternatives.

## 1. Name the problem and the limit

Describe what a user or operator is trying to do and what goes wrong today. Then name what the design must preserve. For example, a report taking longer is a performance problem; one organisation seeing another's records is an access-control failure. I don't trade those against each other as if both were a few points on a scorecard.

## 2. Put the alternatives at the right level

Include the option of keeping the current shape. A module can clarify ownership in the code. A worker can give a slow job its own resources. Another instance can help the API survive a process failure. A service can have its own contract and release cycle. The right comparison depends on what I need to change; these aren't four sizes of the same solution.

## 3. Follow a request through success and failure

I use one concrete request to compare the options. Who can start it? Who owns the data it changes? Who can release the code independently? What if the operation finishes but the response never comes back? I also check what the team would have to run, observe and recover. This tends to surface the expensive parts sooner than comparing tools by feature lists.

## 4. Test the assumption that could change the choice

In a backend, that might be running a rollback against a changed schema. For generated sprites, it's playing the first output at game scale, where an attractive preview can hide a broken direction. For AI queries, it's checking that a saved report is still protected when someone loses access. I want the smallest test that could make me choose differently.

## 5. Define when I'd revisit the decision

I record the reason for the choice, the downside I'm willing to accept, and what would make it stop being a good choice. “Move sessions out of the API when a restart can no longer sign users out” is useful. “Move to microservices when we grow” doesn't say what would change or why.

The articles below apply that process to three different problems. The project background matters because it explains which constraint carries the most weight in each case.

- [CYYS Framework: where to split a backend](case-studies/cyys-frame-one-process-explicit-boundaries.md)
- [gameFrameAnimation: how to test output before choosing a pipeline](case-studies/game-frame-animation-first-pass-quality.md)
- [CYYS AI Platform: how to separate model interpretation from execution authority](case-studies/cyys-ai-platform-architecture.md)
