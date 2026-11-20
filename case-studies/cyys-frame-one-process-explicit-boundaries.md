# One deployable, explicit boundaries: the CYYS backend

**Case type:** Architecture decision analysis · **Evidence:** repository snapshot, 20 November 2026

## Decision in brief

CYYS is a Java administration framework for one organisation hierarchy. The backend is a Maven multi-module application assembled into one executable JAR and documented to run as one API instance. Code boundaries separate shared infrastructure from administration rules; the deployment boundary keeps the backend on one release and restart cycle. Identity, allowed operations and row scope are still evaluated separately inside that process.

The central trade-off is straightforward: keeping one deployable avoids coordinating release and session state across several API instances, but an API restart interrupts service and requires users to sign in again. This is a rational choice only where a maintenance window and re-authentication are acceptable. The repository documents the model, not a measured availability target or proof that any particular deployment accepts that cost.

## 1. Why modularise the code without distributing the runtime?

The repository describes one management hierarchy with a real root organisation. An organisational unit is a scope within that hierarchy; the case does not describe a multi-tenant SaaS isolation model. The challenge is to keep reusable technical policies independent of application-specific identity and administration rules without requiring a separate network service for every code boundary.

- `cyys-common` provides shared technical mechanisms and owns the identity-resolution interface. It must not query administration business tables or depend on downstream business modules.
- `cyys-admin` implements administration behaviour and depends on `cyys-common`. Its identity implementation supplies the organisation-aware rules behind that common interface.
- `cyys-application` is the composition root. It brings the modules together and builds the sole executable backend JAR. The frontend is a separate build artifact.

The likely rationale, inferred from the dependency graph, is to make the code easier to extend and review while shipping one coherent backend version. An undifferentiated application could also ship a single JAR, but it would not express the same build-time dependency direction. Independently deployed services would make separate rollouts and fault isolation possible, but would introduce network failure modes and contracts across process boundaries. The repository does not show that either alternative was formally evaluated.

The cost of the chosen design is shared deployment, compute and failure boundaries. A module cannot be scaled or restarted in isolation. Maven dependencies express direction, and the repository includes a test for module package and import rules. Neither catches all semantic coupling, such as shared-table assumptions or a business rule leaking into a supposedly generic helper.

## 2. The security boundary inside one process

Three questions drive access decisions. A positive answer to one must not silently answer the others.

| Question | Evidence in the design |
| --- | --- |
| **Who is acting, and in which organisation?** | The framework resolves authenticated identity through a common interface. Selecting an organisation is a separate authorisation decision. Business code reads the resolved context rather than inventing its own. Request context is cleared at the request boundary. |
| **May that actor perform this operation?** | Entry-point permissions are checked at controllers. Switching the selected organisation does not change the actor's action permissions. |
| **Which records are in scope?** | Protected mappers use a shared data-scope interceptor. The selected organisation narrows the actor's authorised row scope rather than broadening it; new records belong to the selected organisation. The rules call for rejection if context is missing or scope cannot be resolved. Mappers declared as manual-scope must apply their own rules; they do not inherit automatic filtering. |

**Example, not an incident:** An operator with permission to list records still needs authorisation to select a given organisation. Once selected, a protected listing should return only records within that selection **and** the actor's permitted scope. Granting write permission would change the allowed operation, not widen the set of records. The same distinction matters in asynchronous work: the documented rule is to capture the actor and organisation, re-authorise at execution and clear the context afterwards.

Central enforcement makes it easier to apply one policy across modules, but it also concentrates risk. A newly introduced access path, manual-scope query or incorrectly filtered join can undermine the invariant. These are application-level controls, not database-enforced tenant isolation. A scoped query and its count or pagination counterpart need consistent predicates. The source includes tests for separately filtered join tables, count and union queries, rejection when context is absent, and write rejection through read-only views. Those tests were inspected, not run.

## 3. Deployment and the recovery unit

The deployment guide describes one API instance, process-local sessions, MySQL with application-run Flyway migrations, Redis as a separate dependency, and private files backed by a local directory or S3-compatible storage. Redis is documented for display caching, with live data used for authorisation checks; its presence does not make sessions shared. A restart invalidates the process-local sessions; it does not by itself imply loss of database rows or files.

The recovery unit is larger than the executable. The release procedure calls for stopping writes during maintenance and retaining a matching database, private files and external configuration alongside the release. The guide allows switching back to a matching image when the database schema remains compatible with the previous image. When it does not, the documented path is a coordinated restore of database, private files, external configuration and the matching image, with writes still stopped. This is a recovery procedure, not proof of an atomic snapshot or of a successfully measured restore. Redis recovery and any required storage key material should be explicitly checked in a restore exercise; no blanket assumption is made that Redis can be discarded or that a storage backup alone is sufficient.

## 4. Failure modes worth rehearsing

The scenarios below are design analysis. None is presented as a past production incident.

| Scenario | Possible effect | Proposed verification or control |
| --- | --- | --- |
| API process restarts during a request | Users must sign in again. A write with no returned response may have an uncertain outcome for the caller. | Rehearse restart and session invalidation. For important writes, test retry behaviour with a lost response and use idempotency where duplicate effects would matter. |
| A second API instance is enabled without redesigning session handling | Requests can reach a process that does not recognise the session. Routing affinity would not preserve it after the owning process fails. | Test sign-in against two instances with alternating routing and a forced node loss before supporting this topology. |
| A new query path mishandles actor or row scope | An authorised operation could see or affect unauthorised records, or be denied incorrectly. | Exercise cross-organisation reads and writes, missing context, manual-scope paths, joins and counts. Test delayed work after permissions change. |
| An old image starts against an incompatible migrated schema | Merely rolling back the artifact may fail or leave the application behaving incorrectly. | Check schema compatibility before image-only rollback. Rehearse restoring the matching release and data together under stopped writes. |
| Database and private files are restored to different points | Metadata and stored content may disagree; a database-only check can miss inaccessible files. | In a restore drill, check both representative database records and corresponding file reads, including the configured object-storage access path where applicable. |

## 5. When would I redesign it?

| Trigger to verify | First change I would consider |
| --- | --- |
| The business requires rollout without a login interruption, or recovery from an API node failure within a target that one process cannot meet. | Keep the code modular. Evaluate a shared session store, including revocation and store failure, then test multi-instance routing, connection draining and node loss. Before adding replicas, also review file staging, scheduled work and migration coordination. |
| Restore drills cannot meet agreed recovery-time or recovery-point objectives, or backups cannot be restored coherently. | Improve coordinated backup and restore, test compatible migrations and measure recovery before adding new services. The release unit alone is not the recovery strategy. |
| A specific workload demonstrably needs independent scaling, a separate release cadence or fault containment. | Profile it first and isolate the workload at its existing module boundary. Consider a worker or separate service only if the measured pressure or ownership need justifies the extra contract and failure handling. |
| The product must isolate independent customers as tenants. | Revisit the isolation model explicitly, including storage, database enforcement, background work and operational access. Do not reinterpret organisational filtering as a tenant boundary. |

A second API instance and a separate business service solve different problems. The former can address API availability after state and coordination are accounted for; the latter only makes sense when a particular domain needs its own runtime boundary.

## Evidence and limits

This case is based on static review of `cyys_frame` at commit `29f329805e9d1bfb74c1c7932d9f4041bd601889`. It separates documented behaviour and inspected implementation from inferred rationale and proposed checks. It does not claim verified live topology, traffic, latency, recovery timings, customer outcomes, incident history or who made the original decisions. Tests and recovery drills were not executed for this case. No customer details or environment-specific values are reproduced.

1. [Project structure and dependency rules](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/docs/development/project-structure.md#L3-L25) · [Application assembly POM](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-application/pom.xml#L13-L28) · [Admin dependency direction](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-admin/pom.xml#L16-L23)
2. [Identity and organisation context](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/docs/development/backend/identity-context.md#L3-L21) · [Operation permissions and data scope](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/docs/development/backend/data-permission.md#L3-L28)
3. [Release and runtime model](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/deploy/README.md#L5-L24) · [Recovery procedure](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/deploy/README.md#L47-L49)
4. [Identity resolver implementation](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-admin/src/main/java/com/cyys/admin/auth/service/IdentityService.java#L23-L75) · [Scope declaration checks](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-common/src/main/java/com/cyys/common/mybatis/DataPolicyRegistry.java#L38-L57)
5. [Module and mapper boundary tests](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-application/src/test/java/com/cyys/application/ArchitectureRulesTest.java#L17-L95) · [Row-scope tests](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/backend/cyys-application/src/test/java/com/cyys/application/DataScopePolicyTest.java#L40-L79) · [Cache rules](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/docs/development/backend/cache-usage.md#L3-L9)
