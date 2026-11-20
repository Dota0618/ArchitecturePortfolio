# From a question to a reviewable query plan

CYYS AI Platform lets users ask questions about published datasets and turn the answer into a table, chart or report. The model proposes a plan. The user can read it before choosing Run. The backend is responsible for checking what may actually be queried.

## Put the interpretation where the user can see it

Consider this example: “How many requests did we get in October, by category?” In a made-up dataset, two requests were created in October and closed in November; a third was created in September and closed in October. Counting by creation date gives two. Counting by closure date gives one. Both queries can run successfully, but the system needs to know which event the user meant to count.

For that case, I'd want the user to see this before execution:

```text
Dataset: Requests
Measure: Count requests
Date basis: Created date
Period: 1 October inclusive to 1 November exclusive
Group by: Category
```

Letting the model execute directly removes the review step, but may hide a wrong interpretation behind a convincing chart. A saved query is better when the same metric comes up often and its definition is already agreed. A reviewable plan is useful for the questions in between. It adds a step, so make the choices that affect the result visible rather than asking the user to approve technical detail.

The person confirms the meaning; the backend checks the current dataset definition and the person's authority to run the query. The platform stayed with its existing catalogue visibility rules rather than creating another path that exposed source and field names. The cost is that permitted data can be harder to discover. If discovery becomes a recurring problem, I'd test a limited catalogue view with its own permission before making source metadata more widely visible.

## What changed in testing

| Check | Observed result | Limit |
| --- | --- | --- |
| October acceptance test with a three-table test dataset | A task had finished, but the application still treated its plan as unfinished. After the state check was corrected, the original failing case passed on rerun. | This checked when a completed plan became available for review. |
| Separate desktop workflow using a model substitute | After Run, the table and chart appeared. Refresh preserved the plan and result. Export worked and both versions of a regenerated report could still be opened. | This checked the workflow, not the model's interpretation. |
| Separate check with a real model provider | An undefined metric prompted a clarification question. | One observed response; it does not establish accuracy on other questions. |

The first failure changed where I'd put the regression test: start before the task has finished and observe the transition. Opening a plan that was already saved would miss the bug.

## Two failures I'd test next

If the model response is lost, the provider may have already accepted the request. Look up the original request if the provider supports it. Only retry automatically when the provider guarantees that the same operation will not be repeated. Otherwise, show that the outcome is unknown and let the user decide whether to try again.

If source access is revoked after a report is saved, try opening and exporting the saved version. If the report inherits source permissions, both actions should be denied. Revoking access at the application does not retract a file that a user already downloaded. These are proposed checks, not part of the acceptance results above.

[Back to the case studies](../README.md#architecture-guides)
