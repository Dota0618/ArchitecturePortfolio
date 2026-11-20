# Designing a reusable Java backend

CYYS Framework is a starting point for Java administration applications. It provides accounts, organisations, permissions and shared technical capabilities so that new business modules do not have to build those from scratch. The design needs to fit two goals: keep business code independent enough to extend, and keep the first deployment simple enough to operate as one application.

## Organise by responsibility

The main directories are:

```text
backend/
  cyys-common/        Shared technical mechanisms
  cyys-admin/         Accounts, organisations and administration rules
  cyys-application/   Startup, configuration and module assembly
admin-ui/             Administration frontend
migrations/           Database schema and migration scripts
deploy/               Packaging and deployment
scripts/              Build and operating scripts
docs/                 Development conventions and version notes
```

Inside administration, group by feature first. A department change should be traceable inside the department package. Add a controller, service or mapper where it serves a purpose; do not create an interface and implementation for every class. For a new business area, add a peer module rather than a new collection of tables inside administration.

The application module depends on the modules it assembles. Administration can use common infrastructure. A business module can use common and, when needed, administration. Neither of those modules imports the new business module. If common needs identity, it defines the resolver interface it needs; administration supplies the implementation. The dependency follows the interface even though the call stays in the same process.

## Choose the backend and deployment separately

**Java, Spring Boot and Maven.** Maven modules give the project separate build-time dependencies; Spring Boot gives it one executable JAR. A flat application would be easier to set up but would leave more of the dependency discipline to convention. Separate services would allow independent releases but would add network calls and deployment coordination. A module is enough if the requirement is to keep code and its dependencies separate.

**MySQL, MyBatis Plus and Flyway.** Accounts, roles and departments have relationships that fit a relational database. MyBatis Plus fits a design where explicit SQL still matters, particularly when controlling which rows a query can see. The cost is that a query written outside the standard path has to apply the same scope rules. Flyway gives schema changes a single migration path; the ability to restore an earlier JAR still depends on whether it can work with the changed schema.

**React and TypeScript for the administration UI.** The UI is built separately from the JAR. Ant Design Pro supplies the standard forms, tables and navigation. That saves writing a framework around ordinary administration screens, but the API contract still has to stay compatible across frontend and backend releases.

**Single API process for the standalone installation.** Redis is a separate dependency. Files are stored outside the API process, in a mounted directory or object storage. Redis being present does not make sessions shared: the standalone setup holds them in memory, so restarting the API requires users to sign in again. An additional API instance would make sense when availability requires it, provided session handling is redesigned. That does not require splitting the business code into services.

## Set a rule for what goes in common

A request's identity, permission to perform an action and access to individual records answer different questions. Common code can expose identity context and provide mechanisms for applying a scope. Administration owns the meaning of that scope for its accounts and organisations. A department operation decides which authorities it needs; it does not turn its rule into a generic utility.

Apply the same rule to a hypothetical inventory module. Put stock movements, quantity rules and their API in `cyys-inventory`. Reuse the shared identity interface and file export helpers. Keep “who can adjust stock” and “which stock records they can see” in the inventory feature. Register the module in the Maven build and add it to the application assembly. The existing common and admin modules do not need to learn about inventory.

[Project layout](https://github.com/Dota0618/cyys_frame/blob/29f329805e9d1bfb74c1c7932d9f4041bd601889/docs/development/project-structure.md) · [Back to the case studies](../README.md#architecture-guides)
