# Let the model propose the query. Decide who can run it.

CYYS AI Platform helps users ask questions about published datasets and turn the results into tables, charts and reports. Its workflow gives users a chance to read the model's plan before they run the query. That matters because a question can be clear in conversation and still be ambiguous when translated into a query.

“How many requests did we get last month, by category?” sounds clear until you have to choose a date column. Should you count requests created last month or requests closed last month? A valid query can answer the wrong question. I give the person asking a chance to check the interpretation before the query runs. I leave the decision about what can run with the backend.

## Make the plan readable and keep enforcement on the server

Show the dataset, measure, date range, grouping and filters in terms the reader understands. Let the user fix the date or ask for clarification. On execution, have the backend validate the plan against the dataset and check the user's current permissions. A user clicking Run confirms the intended question. The server still decides which fields, operations and records are allowed.

For this platform, we kept the existing catalogue visibility rules: it avoids creating a second way to expose source and field names. The downside is that someone who could query a permitted subset of rows may never see the published dataset. If that happens often, I'd consider a small catalogue view with its own permission and a short list of safe fields. Test discovery, query execution and access to saved reports separately. Finding the name of a dataset shouldn't let you read its rows.

## What an acceptance test caught

An October acceptance run used a dataset built from three tables. It exposed a timing bug: a task had just finished, but the application still treated its plan as unfinished. The state check was corrected, and the original failing case passed when rerun. The lesson was to test the transition from a running task to a reviewable plan, not just to start a test with a plan already in place.

In a separate desktop test, the team reviewed the plan, explicitly ran the query, saw a table and chart, and refreshed without losing the plan or result. Export worked, and both versions of a regenerated report could still be opened. That run used a model substitute. A separate check with a real provider tested an undefined metric and returned a request for clarification. Those two tests answered different questions: whether the application kept its state through the workflow, and whether the real model would ask instead of guessing.

## Try three interruptions before adding more automation

1. **Refresh after reviewing a plan.** Restore the plan so the user can continue. Don't ask the model to interpret the same question again unless the user asks for a new plan. If the dataset definition changed in the meantime, check it again before execution.
2. **Lose the model response after sending the request.** The provider may already have done the work. If you can look up the original request or retry with an identifier the provider honours, use that to decide what to do. Otherwise show that the result is unknown and let the user start a new attempt. An automatic retry can repeat billable work and produce a different answer.
3. **Remove someone's access after they save a report.** Try opening and exporting the saved version. If report access is supposed to follow source access, enforce it again there. Keeping the old report version is useful for understanding what was shared at the time. It won't let you take back a file someone already downloaded.

If most requests turn into a few recurring questions, I'd offer those as saved queries with agreed definitions and the same server-side checks. Save model interpretation for questions people actually need help expressing.
