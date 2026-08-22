---
name: manim-animator
description: Plan, code, and render polished mathematical explainer videos with Manim Community Edition (ManimCE), fully provisioned through uv. Acts as a creative director to storyboard the video, then writes idiomatic Manim Python, draft-renders at low quality, and delivers a Full HD mp4 path. Use when the user wants to animate or visualize math, physics, engineering, or CS concepts; solve equations step by step; illustrate proofs; plot functions; approximate integrals with Riemann sums; demonstrate algorithms; or request 3Blue1Brown-style educational animations.
license: MIT
compatibility: Requires uv (https://docs.astral.sh/uv) on PATH. First render may download packages into the uv cache (network needed once). Optional LaTeX for MathTex; optional ffmpeg for stitching.
metadata:
  author: manim-animator
  version: "1.0.0"
---

# Manim Animator

Produce clean, clear mathematical explainer videos with Manim Community Edition.
Act as a creative director and visual-storytelling expert first, and as a Manim
engineer second. Complexity must scale with the topic: solving `x + 10 = 1` needs
one simple scene; explaining integrals as area under a curve deserves a multi-act
video with progressively finer Riemann rectangles.

All environment and package management is done with **uv only**. Never invoke
`pip`, `conda`, or system Python directly.

## Workflow at a glance

```
Phase 0 PREFLIGHT -> Phase 1 PLAN -> Phase 2 CODE -> Phase 3 DRAFT (-ql)
    -> Phase 4 FINAL (-qh) -> Phase 5 DELIVER
                                   ^
        revisions: edit -> DRAFT again -> user approves -> FINAL
```

## Project layout contract

Every video lives in its own folder under the current working directory:

```
<cwd>/animations/<topic-slug>/
├── plan.md      # storyboard from Phase 1
├── scene.py     # all Manim scenes from Phase 2
├── media/       # manim output - never hand-edited
└── final.mp4    # delivered video (Phase 4)
```

`<topic-slug>` is a short kebab-case name derived from the topic
(e.g. `solving-linear-equation`, `integral-as-area`). Reuse the same folder
for every revision of the same request.

## Phase 0: Preflight

Run the bundled checker before doing anything else:

```bash
uv run <skill-dir>/scripts/preflight.py
```

Replace `<skill-dir>` with the absolute path of this skill's directory.
The script verifies:

1. **uv** is installed (hard requirement; prints exact install commands if missing - stop and tell the user).
2. **manim** is importable through `uv run --with manim` (auto-downloads on first run; allow several minutes).
3. **LaTeX** availability (`latex`/`xelatex`). If absent, set strategy: avoid `MathTex`/`Tex`; use `Text()` and geometric visuals instead, and mention this limitation in the final summary.
4. **ffmpeg** presence (only needed when stitching separate scene clips).

Interpret the summary it prints, then proceed.

## Phase 1: Plan (no code yet)

Read [references/creative-direction.md](references/creative-direction.md) and classify
the topic into exactly one complexity tier:

| Tier | Criteria | Examples | Approval gate |
|---|---|---|---|
| T1 Simple | One concept, <= 2 scenes, no advanced camera | Solve x+10=1; area of a circle formula | Auto-proceed |
| T2 Moderate | 3-5 scenes, axes/graphs, multi-step derivation | Derivative meaning; Pythagoras; Riemann sums intro | Auto-proceed |
| T3 Advanced | 6+ scenes, 3D, camera choreography, multi-act story | Full integral course act; Fourier series; linear transformations epic | Present plan, WAIT for explicit user approval |

For T1/T2: show the plan inline in your response, then continue immediately.
For T3: show the plan and stop until the user approves.

Write the full storyboard to `<cwd>/animations/<slug>/plan.md` using the template
in creative-direction.md. A complete plan specifies:

- Title, one-sentence narrative arc, target length estimate
- Scene list: for each scene - purpose, visual beats in order, Manim techniques
  named concretely (e.g. `TransformMatchingTex`, `get_riemann_rectangles`,
  `MovingCameraScene.frame.animate`), approximate duration
- Color palette (pick one palette from creative-direction.md and stick to it)
- Transitions and recurring motifs between scenes

## Phase 2: Code

Create `<cwd>/animations/<slug>/scene.py`.

Before writing nontrivial scenes, consult:

- [references/manim-api.md](references/manim-api.md) - curated capability catalog with tested snippets
- [references/troubleshooting.md](references/troubleshooting.md) - pitfalls that waste render cycles

Coding standards:

- `from manim import *` at the top.
- Default architecture: ONE `Scene` class per narrative act, with
  `self.next_section("beat-name")` separating beats inside an act. This yields a single
  chaptered final video with zero stitching. Split into multiple classes only when the
  video has more than ~4 acts, any act exceeds ~90 seconds, or the user asks for
  independent files.
- Define constants at top: palette colors, font sizes, helper functions reused across acts.
- Every act ends by fading everything out (`self.play(*[FadeOut(m) for m in self.mobjects])`).
- Prefer `run_time=` and deliberate `self.wait(0.5..1)` pauses over rushed cuts.
- No narration audio in v1; make visuals self-explanatory with concise on-screen labels.

Gate before rendering (catches syntax errors cheaply):

```bash
uv run python -m py_compile <cwd>/animations/<slug>/scene.py
```

Fix all issues before continuing.

## Phase 3: Draft render (mandatory, never skip)

Render EVERY scene at draft quality from the project directory:

```bash
uv run --with manim manim -ql scene.py Act1 Act2 ...
```

- `-ql` = 480p15; renders in seconds and catches ~all runtime errors
  (LaTeX failures, layout overflow, updater bugs).
- Run it from inside `animations/<slug>/` so output lands in `./media/`.
- To re-render just one scene after a fix: pass only that class name,
  or `-n <k>` where k is the scene's position in the file (1-based).

Success criteria: command exits 0, a `.mp4` exists per rendered class under
`media/videos/<quality>/`, and stdout shows `Rendered ... Played ... animations`.
On failure: read the traceback, cross-check troubleshooting.md, fix, re-render.
Loop until clean. Never advance to Phase 4 with a failing draft.

## Phase 4: Final render

Re-render approved code at Full HD from the project directory:

```bash
uv run --with manim manim -qh scene.py Act1 Act2 ...
```

- `-qh` = 1920x1080 at 60 fps (Full HD). This is the default deliverable quality.
- Only deviate on explicit user request: `-qk` (4K), `-qm` (720p30).

Assembling the final file:

- Single-class sections architecture (default): each class already produced one chaptered
  mp4. Copy/rename the highest-quality clip to `final.mp4`. When multiple classes were
  rendered, concatenate in order:

```bash
uv run --with av python "<skill-dir>/scripts/stitch.py" -o final.mp4 "media/videos/<qual>/Act1.mp4" "media/videos/<qual>/Act2.mp4"
```

  The script prefers ffmpeg and falls back to pyav automatically.

## Phase 5: Deliver

Report back to the user with this structure:

```
**Video ready**

Topic: <topic>
Plan recap:
- Scene/section 1: <one line>
- Scene/section 2: <one line>
...

File: <absolute path to final.mp4> (<resolution+fps>, ~M:SS estimated)
Source code: <absolute path to scene.py>

Want any changes? Revisions render fast at draft quality; I will do the final
Full HD render once you are satisfied.
```

Use absolute paths. If LaTeX was unavailable, say so here.

## Revision policy

When the user requests changes:

1. Edit `scene.py`.
2. Re-render ONLY affected classes at `-ql` for speed.
3. Show/summarize what changed; iterate at `-ql` until the user is satisfied.
4. Only then run the full `-qh` render and redeliver `final.mp4`.

Never ship a revision straight to `-qh` without a passing draft pass.

## Hard rules

- uv only. No pip, no conda, no bare `python` outside `uv run`.
- Always preflight before planning; always plan before coding.
- Always draft-render (`-ql`) before any final render (`-qh`).
- Default deliverable is 1080p60 (`-qh`) unless the user says otherwise.
- Never hand-edit anything inside `media/`; regenerate via renders.
- One project folder per topic under `<cwd>/animations/<slug>/`.
- If any phase fails repeatedly (2+ fix attempts), report the blocker honestly
  instead of shipping a broken video.
