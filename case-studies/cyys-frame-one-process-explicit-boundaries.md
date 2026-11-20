# One backend release, two different rules for access

CYYS Framework is a Java backend for administering organisations, accounts and departments within one management hierarchy. One job is to let an operator manage departments across several authorised organisations while making each edit in a specific organisation. The department screen made the difference between those two scopes visible.

## The department tree needed a wider read, not a wider write

The client initially sent the selected organisation with the tree request as well as with edit requests. The first browser check used simulated responses. It confirmed that switching organisations sent the right selection for an edit; it did not establish what the real API would return for the tree.

The client stopped sending the current selection on the tree request. The backend still worked out which organisations the operator was allowed to see. Edit requests continued to use the selected organisation.

| Scenario | What the server test showed |
| --- | --- |
| Open the management tree with access to two organisations. | Departments from both authorised organisations appeared. |
| Try to save a department in one organisation with a manager or parent from another. | The cross-organisation assignment was rejected. |

These are results from the 17 November test, not a claim about every read and write path. Separately, the integration test checks both directions of the read rule: an organisation outside the operator’s scope is absent from the tree, then appears when that test operator is granted broader access. A [separate creation test](https://github.com/Dota0618/cyys_frame/blob/238aaa575f024acca72ae8354e8eb4faa45eb06d/backend/cyys-application/src/test/java/com/cyys/application/DepartmentIntegrationTest.java#L58-L68) checks the rejected assignments. The [client change](https://github.com/Dota0618/cyys_frame/blob/238aaa575f024acca72ae8354e8eb4faa45eb06d/admin-ui/src/services/system/departments.ts#L13-L14) and [tree integration test](https://github.com/Dota0618/cyys_frame/blob/238aaa575f024acca72ae8354e8eb4faa45eb06d/backend/cyys-application/src/test/java/com/cyys/application/DepartmentIntegrationTest.java#L18-L32) show how the rule is expressed in the project.

## Keep policy with the code that owns it

I separate three questions: which organisation is the operator acting in, may they perform this action, and which records are in scope for this operation? The tree and edit behave differently because an answer to one question cannot stand in for the other two. On a paged list, for example, I'd check that the total count is filtered as well as the returned rows.

CYYS ships a single executable JAR. Its application module assembles administration and common infrastructure. Common defines the interface it needs for resolving identity; administration supplies the implementation because it owns the rules for accounts and organisations. Keeping both in one process avoids a remote call on this path. It also makes careless shortcuts easy: an import rule catches one module using another module's internal code, but it won't catch a hand-written query against a table it doesn't own.

## What would justify a different deployment?

| If this becomes the problem | What I'd try first | What I'd check before committing |
| --- | --- | --- |
| Exports slow ordinary requests. | Limit concurrent exports and find the contested resource; move jobs to a worker if the API process needs isolation. | With an export running, compare ordinary request latency and database load. A second process can still compete for the same database. |
| An API restart is no longer acceptable during active use. | Replicate the same application and revisit session storage. | Sign in on one instance, send the next request to another, then stop the first. Repeat with a revoked session. |
| One capability keeps holding up unrelated releases. | Give that capability a stable interface and an explicit owner before considering a separate service. | Can the two parts ship independently? If a previously atomic write now crosses a network, what happens when only one side commits? |

The single-instance setup keeps sessions in the API process. A restart interrupts service and users have to sign in again. That's a deployment constraint; it doesn't follow from shipping a single JAR. Before a schema change, I would test the old JAR against the new schema. If compatibility cannot be preserved, I would choose between a forward fix and a restore before deploying, because a restore can discard writes made since the backup. For a restore that includes uploads, I'd check that recovered records still open their files; starting the API is not enough. Those are proposed checks, separate from the department test above.

For the department change, the boundary that mattered was the scope of the operation. I would keep that rule even if the API later ran on several instances or the administration module moved to its own service.

[Back to the case studies](../README.md#architecture-guides)
