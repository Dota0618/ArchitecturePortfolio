# Designing a desktop pipeline for 2D game animation

gameFrameAnimation is a desktop prototype for making 2D animation frames from character references. Its workflow is to submit a reference and a prompt, review the candidate frames, change their timing and export them for use in a game. A generator works in images; the game needs named actions, directions, frame order and timing. I would put that translation in the application, rather than relying on a particular model or game engine to own it.

## Organise around the production workflow

This is the layout I would use for the next version; it is not a directory listing of the prototype:

```text
desktop/              Window, file selection and editing UI
animation/            Actions, directions, frames, timing and anchors
generation/
  hosted/             Remote requests and job handling
  local/              Local runtime and device setup
export/               Sprite-sheet layout and engine-specific output
```

The desktop interface assembles those pieces. The animation module owns the project's editable data and doesn't know how requests or exports are made. A generator returns candidate frames and their provenance; it does not rewrite the approved animation. Export reads the edited animation and writes a package. I would store the selected frame files and their timing with the project, so reopening an asset does not depend on a provider still serving it.

## Choose the shell, generation and output independently

The prototype uses Electron. It fits a desktop workflow with local references, previews and file export, at the cost of a heavier runtime than a small native utility. A browser-first version could make distribution simpler, but I would choose it only if local file handling and offline generation were no longer important enough to shape the experience. The choice of shell does not decide which model runs.

I would keep hosted and local generation as separate routes. A hosted service avoids installing the model on each workstation, but introduces network dependence and requires a decision about whether artwork can be sent off the machine. A local route can keep art on the workstation when the entire workflow runs locally, but makes hardware and runtime setup part of the product. Their job lifecycles need not look identical. The common output should stay small: frames, the direction they belong to and enough information to review their origin.

Sprite-sheet layout belongs in export. The same edited animation can be packed differently for different consumers without teaching the editing screen about each engine. Don't bake file naming, trimming or atlas packing into the model response.

## Keep the editor reusable

Suppose a developer adds an eight-direction walk and later asks for an attack animation. The editor can reuse selection, ordering, playback, timing and anchors. The action-specific requirements, such as directions or when a strike should occur, belong in the action data or its own settings. Similarly, adding another generator should not require an export change, and adding an export format should not require generating the frames again.

I would avoid making a general pipeline framework up front. Keep editing and export stable, and give generator-specific choices a place of their own. That is enough structure for a second route without asking the first route to define every future one.

[Back to the case studies](../README.md#architecture-guides)
