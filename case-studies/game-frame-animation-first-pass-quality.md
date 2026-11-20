# Put the quality test before the generation pipeline

gameFrameAnimation is a tool I'm designing to turn a character reference into 2D animation frames for games. I'm considering both hosted and offline generation. The first thing I need to know is whether either route produces something I can use in the game without redrawing it. A good preview doesn't answer that question: I need to see the same character move and turn at game scale.

I'd start with a walk cycle in eight directions. Choose a character with an asymmetrical feature, like a shoulder guard. Agree on the eight directions, frame size, timing and playback scale. Run both routes on the same reference. Keep the first output, including missing views; don't replace a bad result and call the replacement a first pass.

### Agree on what counts as a fix

An export step can arrange the frames, normalise their size and put them on a sprite sheet. Give both routes the same export rules. If someone has to repaint a frame, realign one direction by hand or generate the missing views, record that work separately. Otherwise, “one click” can mean five attempts and ten minutes of cleanup, and the results won't tell you what to automate.

Play the animation at game scale. Switch direction while walking, then return to idle. [Godot's animation guide](https://docs.godotengine.org/en/stable/tutorials/2d/2d_sprite_animation.html) shows how to preview a sheet or individual frames. A foot that jumps because the frame is offset is an export problem. A shoulder guard that jumps to the other side may be an artwork problem. Record which direction failed and why, and count the usable ones before and after rework.

### Use the failure to choose what to build next

- If the art holds together but the origin drifts, fix packaging and replay the same frames.
- If the generator keeps changing the character, try tighter motion constraints or compare it with rendering a rig. A better packer won't give the character its shoulder guard back.
- If both routes produce usable frames, I compare the time and cost per usable animation, including retries and cleanup. Also check whether the artwork is allowed to leave the workstation and whether the local machine can run the workflow.

One good walk cycle gets you to the next test, not to a general quality claim. Try another character and action that you didn't tune for. I'd add queues and retry logic when job duration or volume starts getting in the way. They won't make a bad animation usable.

The eight-direction run is the next test for this design; I'm not treating a preview as a result.
