# How I structure a new project

I start by describing what the software needs to do and what is likely to change. A Java backend that will host several business areas needs somewhere to put the next one. A game asset tool needs somewhere to put the next generator without rebuilding the editor. Those are different design problems, even if both projects end up with a folder called `common`.

## Put code that changes together in one place

Start with the main capabilities. Give each one a home and keep the code involved in a typical change close together. In a backend, I prefer a package for departments with its own API, service and persistence code to three global folders that mix code from every feature. Within a feature, I add a layer when there is work for it to do. An empty layer tells the next developer very little.

Then write down who may depend on whom. If reusable code needs information about the logged-in user, it can ask through a small interface. It doesn't need to know how the administration module stores users. The code that starts the application can wire the two together.

## Select technology by the work it needs to do

Choose the deployment model separately from the code structure. A modular backend can ship as one JAR. That is a good starting point when one release can carry the changes and the team can operate it that way. If one workload needs its own resources, a worker may be enough; a new service also brings a network contract and a separate release to maintain.

Make the same distinction with a replaceable technology. If a local generator and a hosted generator both produce candidate frames, keep their credentials, device setup and job protocols outside the editor. The editor needs a reliable description of frames and timing, regardless of how generation works.

## Make shared code earn its place

I look for a stable contract, not just matching code. Spreadsheet export can reuse a writer, but a feature still decides which columns and rows a person may export. A query plan can feed both a chart and a report, but their presentation rules can stay separate. When two callers share a helper only by passing flags for their differences, I would rather keep those rules with the callers.

A useful architecture document makes the next change easy to place. It should show where a new feature goes, which dependency is allowed and which decisions must stay in that feature.

[Back to the case studies](README.md#architecture-guides)
