# How I make architecture decisions

I start with a change someone needs to make or a failure they need to handle. “Should we use microservices?” is too broad. “An export can slow down normal requests. Where should that job run?” gives me something I can investigate. I'd first check where the time goes. A worker may help with API resource contention, but it won't fix an overloaded database on its own.

## Decide what must hold

Write down the rule you cannot relax and the cost you can accept. In an administration system, widening a user's read view must not widen their write authority. In a game asset tool, it may be fine to wait longer for an export if the result needs less manual repair. Those are different kinds of trade-off. I don't put them in the same scorecard.

## Compare options using the same event

Pick a specific request or failure and follow it through each option. Start with today's design. Identify where data is owned, where permission is checked, and what the operator can recover if the request stops halfway through. For example, if a model provider may have accepted a request but the application lost the response, look at what the application can establish before it sends anything again.

A useful scenario has a visible result. “Improve isolation” is too vague. “An operator can manage departments in A and B. While editing in A, assigning a parent in B must fail” tells me what to test.

## Keep the result next to the decision

Record what changed and rerun the case that previously failed. Also check the behaviour that should remain intact. A department tree that shows more records provides little reassurance until you try the write that should still be denied. And a successful export with existing frames says something about the editor, not about the generator. I label future tests as future tests so the next person knows what they can rely on.

## Leave a way to change course

Name the first condition that would make the design worth revisiting. If maintenance can no longer interrupt active use, test whether users can move between API instances and keep their sessions. If people keep correcting the same model interpretation, offer them a saved query with an agreed definition. Set a numerical target when there's a requirement or baseline behind it.

### Further reading

[Michael Nygard on architecture decision records](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) explains how the context and consequences help someone revisit a decision. The [SEI's Architecture Tradeoff Analysis Method](https://www.sei.cmu.edu/library/atam-method-for-architecture-evaluation/) is useful for turning concerns into scenarios. [Simon Brown's dynamic diagrams](https://c4model.com/diagrams/dynamic) show one way to make the order of actions visible when a list of components doesn't explain the behaviour.

[Back to the case studies](README.md#architecture-guides)
