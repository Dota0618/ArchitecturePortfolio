# Split the code before you split the deployment

CYYS Framework is a Java administration backend for organisations that belong to the same management hierarchy. An operator can select an organisation, but the selection doesn't grant new actions or make every record in that organisation visible. The framework needs to keep those rules out of shared infrastructure that other modules use.

CYYS separates common infrastructure, administration and application assembly in code, while shipping them in one backend release. That setup raises a useful question: which parts need a clear boundary in the code, and which actually need to run, scale or ship on their own?

## Draw the dependencies first

In CYYS, the common module defines what it needs from identity resolution. Administration supplies the implementation because it owns the account and organisation rules. A third module assembles the application. That keeps the dependency pointed towards common code without making identity resolution a network call.

I make the direction of dependencies explicit. A build check should fail if common code starts importing administration classes. This catches one kind of coupling. It won't catch common code quietly querying an administration table, so review data ownership as well as imports.

I keep the access checks separate for the same reason: each one answers a different question. An operator may be allowed to list records and may be allowed to select a branch, but some records in that branch can still be outside the operator's scope. Test those conditions separately. For a paged list, verify that the count follows the same rules as the rows you return.

## Compare what you'd gain by separating the runtime

**Data and transactions.** Suppose creating an account and granting its initial role must succeed or fail together. If both writes run inside one local transaction, the database can enforce that rule. If separate services own the account and the grant, decide what users see when the first write succeeds and the second fails. You might need a pending state, a retry or a way to reverse the first change. On the other hand, a single database can turn into an unspoken interface between modules. Give each module ownership of its writes even if the tables live in the same database.

**Releases and ownership.** One release is convenient when features change together: the code and schema can be tested as a unit. When one team needs to ship frequently without waiting for another, look at what actually holds up releases. A separate service can help if the team owns it end to end and can keep its API compatible. If every release still needs a simultaneous change to both services, you've added coordination without gaining independence.

**Load and failures.** A slow export job can exhaust resources used by normal requests. Give that workload a separate worker and limits before splitting an entire business domain. Likewise, two copies of the same backend can help survive an API failure without creating two different services. Neither gives strong isolation if both still overload the same database. Each new runtime also needs its own deployment, health checks and way to trace failures across calls. Take on that work where it buys a specific separation.

## Rehearse one restart and one rollback

The standalone CYYS setup runs one process with sessions in memory. Restarting it interrupts requests and signs users out. Ask whether that's acceptable during a maintenance window. If you need two instances, first test signing in on one and sending the next request to the other. Then stop the first instance. Keeping a user on the same instance can hide the problem until that instance fails. A shared session store is one option, but you also need to check what happens when a session is revoked or the store is unavailable.

Run a separate rollback exercise. After a database change, can the previous release still read and write correctly? If it can't, restoring the old application alone won't get you back. If the system stores attachments separately, restore a matching set of records and files and open a few attachments. A database health check doesn't tell you if the files are there.

I choose the smallest split that changes the outcome: modules to separate responsibilities, replicas to remove dependence on one API process, and services when a specific capability needs independent releases, scaling or failure handling. A system can need all three, but it rarely needs them for the same reason.
