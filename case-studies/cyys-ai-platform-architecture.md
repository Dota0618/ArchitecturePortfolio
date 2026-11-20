# Structuring an AI-powered analytics application

CYYS AI Platform lets someone ask a question about a published dataset, review the proposed interpretation and run it to produce a table, chart or report. I would organise this kind of application around the parts that change at different rates: dataset definitions, model interpretation, permitted execution and presentation.

## Give each capability a home

The implementation has a backend module for AI functionality. The outline below shows how I would organise responsibilities within that module and its boundaries; it is not a source directory listing.

```text
datasets/          Published fields, metrics and discovery rules
planning/          Question interpretation and reviewable plans
execution/         Checks, authorised query execution and results
reports/           Saved views, report versions and export
integrations/      Provider and storage adapters
```

Datasets own the meaning of a field or metric. Planning can read allowed definitions and propose a plan. Execution uses those definitions and the user's current permissions to decide what may run. Reports consume results and apply their own presentation rules. They should not take a shortcut back to raw data or to the model. Put the model connection behind an interface used by planning. Keep that adapter responsible for provider requests and responses, not for reading reports or granting query access.

## Pick a query interface before picking a model

A structured query plan is the contract between planning and execution. It describes the dataset, measure, grouping, filters and date basis in terms the application understands. For “requests received in October by category,” the plan needs to say whether it counts creation dates or closure dates. The caller can review that choice; execution can reject fields and operations outside the published dataset.

I would use a model here for interpreting variable language, not for choosing what the user may query. A direct SQL path can shorten the first prototype, but ties interpretation to schema details and makes it harder to inspect what the user agreed to. At the other end, a fixed query builder gives more control and suits recurring reports with agreed definitions. The structured plan lets both routes use the same execution rules. It costs more to define and maintain than passing a string through, and new query capabilities must be added deliberately.

## Reuse the contract, not the privileges

The table, chart and report can reuse the shape of a query result. They can also share field labels and formatting conventions. They should not share a permission decision made at an earlier stage. For example, a report that inherits source permissions needs to apply them when someone opens or exports it, not only when a query first runs. Likewise, catalogue discovery and permission to see individual rows are separate. I kept the existing catalogue permissions rather than giving AI analytics a separate way to discover source and field names. That keeps one set of discovery rules, at the cost of making some datasets harder to find.

For a concrete extension, add a “requests by department” report without writing another model integration or query executor. Define the supported dimension in the catalogue, use the same query plan and result contract, and keep the chart and layout choices in reports. If the metric stops changing and several users ask for the same view, publish a saved query rather than making a model interpret it each time.

[Back to the case studies](../README.md#architecture-guides)
