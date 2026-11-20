# A usable sprite sheet starts with a quality test

gameFrameAnimation is a desktop tool for turning a character reference into 2D animation frames for games. A game developer needs frames that can be played at the right scale and speed, without the character changing shape when it turns. Both hosted and offline routes are possibilities. Before choosing one, I want to know if the output can go straight into a game.

## What the prototype got through to export

The prototype started with a reference image and a prompt. In the November desktop check, an existing sequence of frames stood in for generated output. The check went from job submission through frame selection and timing changes to export. The prototype took those known frames through export. It did not establish that a model could create an acceptable sequence.

That distinction affects the design. I can make the review and export path work with existing frames before investing in a generator. I wouldn't choose a generation route based on what that editor test showed.

## A walk cycle that could change the choice

I'd use one character with a visible shoulder guard on one side and a walk in eight directions. Give both candidates the same reference, game scale, frame timing and export rules. Keep the first output and count missing views as failures. For this test, all eight directions must be usable after the same automatic packaging, with no retries, substitutions or manual fixes. I'd then repeat the test with a different character and action before standardising on a route.

Here's how I'd separate two problems that can look similar in a preview:

| What you see in the game | Check before blaming the model | What I'd do if confirmed |
| --- | --- | --- |
| The character jumps sideways during the northeast walk. | Compare the point used to place the character in each frame. The export may have moved that point when it trimmed transparent pixels. | Fix the origin used by the exporter and replay the exact same frames. |
| The shoulder guard changes sides when the character turns. | Check the original frames for the two directions and whether the exporter mirrored an image. | Fix the mirroring rule if it caused the change. If the raw frames already disagree, constrain generation or try rendering from a rig. |

These are examples of what to look for, not defects observed in the prototype. Play the sheet in the target engine at game scale, switch directions while moving, then return to idle. [Godot's sprite animation guide](https://docs.godotengine.org/en/stable/tutorials/2d/2d_sprite_animation.html) shows how to preview frames or a sprite sheet. Record how many directions passed before manual cleanup and how many needed new frames. Keep the cleanup time separate from generation time.

## When each route earns its place

Hosted generation avoids maintaining the model on the workstation, but I'd check whether the reference art can leave the machine and whether retries make the job expensive. Offline keeps the input local if the workflow runs entirely on the workstation; it needs hardware and a local runtime that someone can maintain. A rig takes work up front, but limits how much of the character can change between directions. I'd compare it if appearance keeps drifting in the raw output.

I haven't verified a complete eight-direction export from a real model for this case. The next decision depends on the first-pass results and the effort to repair them. The prototype only settled the earlier question: existing frames can go through selection, timing and export.

[Back to the case studies](../README.md#architecture-guides)
