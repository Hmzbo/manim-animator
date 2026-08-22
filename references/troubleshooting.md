# Troubleshooting and Pitfalls

Read when a render fails or before writing tricky scenes. Ordered by frequency.

## LaTeX / MathTex failures

| Symptom | Cause | Fix |
|---|---|---|
| `latex ... returned non-zero exit` on render | Missing LaTeX install OR bad TeX string | Check preflight output; if no LaTeX, switch visuals to `Text()`/geometry and inform user |
| Compile error with `{` `}` in string | Braces are TeX grouping | Escape literal braces as `\{ \}`; raw-string it: `MathTex(r"\{x\}")` |
| Unicode char error (`→`, `π`, `≤`) | MathTex is pure LaTeX | Use commands: `\to`, `\pi`, `\leq`; or use `Text()` for that fragment |
| Wrong colors via `t2c` on transformed eq | substring indices shift after transform | Isolate pieces with `{{ }}` syntax instead; re-isolate per equation version |

Raw strings always: `r"\frac{a}{b}"`, never plain strings with backslashes.

## Layout problems

| Symptom | Fix |
|---|---|
| Mobjects cut off at edges | Frame is ~14.22 x 8 units. `.scale_to_fit_width(12)` groups; reduce `font_size`; shrink axes `x_length/y_length` |
| Overlaps | `.arrange(DOWN, buff=0.4)` VGroups; `.next_to(..., buff=0.3)`; verify visually in the draft pass |
| Label far from its object | Chain positioning off the object: `label.next_to(curve, UP)`, not absolute coords |
| New mobject invisible | Forgot `self.add(m)` / never played an entrance animation |

## Animation logic

| Symptom | Fix |
|---|---|
| `.animate` change not visible | Mobject was not added to scene first, or an updater overwrites it every frame - call `clear_updaters()` |
| Updater keeps mutating during later plays | Detach: `m.clear_updaters()`. For `always_redraw`, swap to a frozen copy before transforming it |
| `Transform` leaves stale ghost / weird morph | Structurally different objects: use `ReplacementTransform`, or `TransformMatchingShapes` for text |
| Equation transform scrambles terms | Hint matches: `TransformMatchingTex(old, new, key_map={"x": "y"})`; isolate with `{{ }}` |
| Scene ends mid-motion | End every act settled: finish plays, add closing `self.wait(0.5)` before the final FadeOut |

## Rendering and environment (uv-based)

| Symptom | Fix |
|---|---|
| First render hangs minutes | Normal: uv is downloading manim + managed Python into cache. Use generous timeouts; subsequent runs are fast |
| `uv: command not found` | Install uv, do NOT fall back to pip. Windows: `powershell -c "irm https://astral.sh/uv/install.ps1 \| iex"`; POSIX: `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| Render slow at `-qh` on 3D acts | Expected. Draft everything at `-ql`; keep 3D surfaces coarse (`resolution_factor`), shorten ambient rotations |
| Output path confusion | Always run manim from the project dir; output lands in `./media/videos/<script>/<quality>/<Class>.mp4` |
| PowerShell quoting | Wrap paths with spaces in double quotes; prefer single-line commands over backtick continuations |
| Wrong scene rendered | Class name typo, or stale `media` from an older edit - delete `media/videos/<quality>/` for that class and re-render |

## Stitching failures

| Symptom | Fix |
|---|---|
| ffmpeg missing | stitch.py auto-falls back to pyav via `uv run --with av`; ensure invocation includes `--with av` |
| Dimension mismatch error while stitching | Clips were rendered at different qualities - re-render ALL clips with the same flag before stitching |
| Concatenated video glitches at cuts | Re-run stitch.py letting pyav re-encode instead of stream-copy |

## Debugging workflow

1. Reproduce at `-ql` with only the failing class name.
2. Read the LAST traceback line first; map to tables above.
3. If a visual looks wrong but renders fine, extract a frame to inspect:
   `uv run --with av python -c "import av;c=av.open('clip.mp4');f=next(c.decode(video=0)) if False else None"` -
   or simply re-render the single suspicious beat as its own mini scene.
4. After 2 failed fix attempts, stop and report the blocker to the user honestly.

