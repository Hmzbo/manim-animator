# Creative Direction Guide

How to think about planning a Manim explainer video. Read this before writing any plan.

## Mission

Produce a video that is clean, clear, and compelling: one idea at a time, visuals that
carry the argument, motion that has meaning. Every animation must answer "what does this
help the viewer understand?" If an animation is decoration only, cut it or replace it with
a deliberate pause so the viewer can absorb.

## Complexity calibration

Match ambition to the topic. Under-animating a rich topic wastes it; over-animating a
simple topic buries it.

### T1 - Simple (1-2 short scenes)

Single concept, single visual thread, default camera.

- Solving `x + 10 = 1`: equation centered, each algebraic step via `TransformMatchingTex`,
  the moving term recolored, solution circled at the end.
- Definition slides: term + formula + one illustrative figure.
- Target length: 20-60 seconds.

### T2 - Moderate (3-5 scenes)

A small arc: setup -> core mechanism -> payoff.

- Derivative as slope: secant line sliding into tangent (`ValueTracker` + updater).
- Pythagoras: triangle plus squares on sides, area comparison, formula reveal.
- Riemann sums intro: curve draws, coarse rectangles appear, refine twice, converge.
- Target length: 1-3 minutes.

### T3 - Advanced (6+ scenes and/or 3D / camera choreography)

Multi-act story with escalating insight. Requires user approval of the plan before coding.

- Integral as area, full act structure: naive sum -> finer partitions -> limit ->
  antiderivative connection -> real-world framing.
- Fourier series: rotating circles building a wave term by term.
- Linear transformations epic: grid transforming under matrices, determinant as area.
- Target length: 3+ minutes.

## Narrative arc

Even a 40-second clip benefits from a micro-arc:

1. **Hook** (5-10%): pose the question visually. A paradox, a shape, an unfinished equation.
2. **Setup** (15-25%): establish objects and notation the viewer needs.
3. **Build** (40-55%): the mechanism, step by step. One transformation per beat.
4. **Payoff** (15-20%): the result lands; highlight what changed in understanding.
5. **Recap** (5-10%, optional for T2/T3): compress the journey into one summary frame.

## Scene beats

Plan every scene as ordered beats; a beat is one `self.play(...)` plus its pause.

Rules of thumb:

- 4-10 beats per scene for T2/T3; 3-6 for T1.
- One visual idea per beat; never move more than ~5 mobjects simultaneously.
- Hold each new state with `self.wait(0.5)` to `self.wait(1)`.
- Vary entry animations across scenes: rotate through Write / Create / FadeIn /
  GrowFromCenter / LaggedStart instead of reusing one everywhere.

## Aesthetic guardrails

These separate professional output from generic output:

1. **Palette discipline.** Pick ONE palette per video from the table below, declare it in
   plan.md and in scene.py constants, then never use an off-palette color except
   `WHITE`/`BLACK` neutrals.
2. **Density limit.** Max ~5 visible mobjects on screen at once. Fade out finished ones.
3. **Typography hierarchy.** Titles `font_size=44-52`, body/labels `24-36`, never below 18.
   Use `Text` for words, `MathTex` for math only.
4. **Motion economy.** Default durations: entrances 1-2s, transforms 1-2s, emphasis 0.75s.
   Long slides or spins read as padding.
5. **Camera restraint.** Static frame is fine for most content. Reach for
   `MovingCameraScene` when zooming into detail genuinely helps; reach for
   `ThreeDScene` only when dimensionality is the point.
6. **Highlight language.** Choose consistent semantics: YELLOW = focus, RED = wrong/
   subtracted, GREEN = result/positive. Keep these meanings stable all video.

### Palettes (dark background default)

| Name | Background | Primary | Secondary | Accent |
|---|---|---|---|---|
| Classic | `#1C1C1C` | BLUE `#58C4DD` | GREEN `#83C167` | YELLOW `#FFFF00` |
| Warm academic | `#2D2B55` | `#FF6B6B` | `#FFD93D` | `#6BCB77` |
| Neon tech | `#0A0A0A` | `#00F5FF` | `#FF00FF` | `#39FF14` |
| Monochrome | `#16161E` | WHITE | GREY_B | GOLD |

Set background via `config.background_color = "#..."` at top of scene.py.

## Technique matrix

Map topic type to the Manim toolkit (snippets in manim-api.md):

| Topic type | Key classes/animations |
|---|---|
| Equation solving / algebra steps | `MathTex`, `TransformMatchingTex`, isolate parts with `{{ }}`, recolor terms |
| Functions & graphs | `Axes`, `.plot()`, `Create`, `get_area`, tangent via `TangentLine` + tracker |
| Integrals / area | `axes.get_riemann_rectangles(dx=...)` refinement loop, `get_area`, limit notation |
| Geometry / proofs | `Polygon`, `Angle`, `RightAngle`, `Brace`, `DrawBorderThenFill`, labels |
| Linear algebra | `NumberPlane`, `ApplyMatrix`, `LinearTransformationScene`, determinant shading |
| Probability / data | `BarChart`, `SampleSpace`, `DecimalTable` |
| Algorithms / graphs | `Graph`, vertex recoloring, arrows, step captions |
| Vector fields / ODEs | `ArrowVectorField`, `StreamLines`, `MoveAlongPath` particles |
| 3D geometry / surfaces | `ThreeDScene`, `ThreeDAxes`, `Surface`, `begin_ambient_camera_rotation` |
| Numbers changing live | `ValueTracker` + `always_redraw` + `DecimalNumber` updaters |

## plan.md template

```markdown
# <Video Title>

## Overview
- Topic: <core concept>
- Tier: T1 | T2 | T3
- Hook: <opening question/mystery>
- Narrative arc: <2-3 sentences>
- Estimated length: <N seconds/minutes>
- Palette: <name> - bg <hex>, primary <hex>, secondary <hex>, accent <hex>

## Act 1: <Act name>            [class Act1]
Purpose: <what this act accomplishes>
Beats:
1. <beat description>          [technique: Write/Create/..., ~Ns]
2. ...
Transition out: <how>

## Act 2: ...                   [class Act2]
...

## Shared elements
<recurring motifs, e.g. axes persist between acts; yellow = focus everywhere>
```

