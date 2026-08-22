# manim-animator

An [Agent Skill](https://agentskills.io) that turns any AI coding agent into a mathematical
animation studio. Give it a topic — *"explain how x+10=1 is solved"* or *"show me why an
integral is the area under a curve"* — and it acts as a creative director: it plans a clean,
compelling video storyboard, writes idiomatic [Manim Community Edition](https://www.manim.community)
Python, drafts at low quality to verify everything works, then renders the final video in
Full HD and hands you the file path.

Everything runs through [uv](https://docs.astral.sh/uv) — no conda, no pip, no global
installs, no manual environment setup.

## What it produces

```
<your-cwd>/animations/<topic-slug>/
├── plan.md        # The storyboard written during the planning phase
├── scene.py       # Idiomatic ManimCE code, one Scene class per act
├── media/         # Manim render output (draft + final passes)
└── final.mp4      # Delivered video - 1080p60 Full HD by default
```

Multi-act videos use Manim's Sections API so the deliverable is a single chaptered `final.mp4`
(no stitching required in most cases; a stitch script with a pyav fallback covers the rest).

## Workflow

```
preflight -> PLAN -> CODE -> DRAFT RENDER (-ql) -> fix loop -> FINAL RENDER (-qh) -> DELIVER
                                                ^__________|
                                           revision loop (-ql until satisfied)
```

| Phase | What happens |
|---|---|
| Preflight | Verifies uv, provisions manim on demand, detects LaTeX/ffmpeg availability |
| PLAN | Creative-director storyboard: narrative arc, per-scene beats, techniques, palette. No code |
| CODE | Writes `scene.py` following curated Manim patterns and pitfall avoidance |
| DRAFT | Renders every scene at 480p15 (seconds, not minutes) and fixes errors before paying for quality |
| FINAL | Re-renders approved code at 1080p60 Full HD |
| REVISIONS | Changes re-render at draft quality for speed; Full HD only once approved |

## Install

With the skills CLI:

```bash
npx skills add Hmzbo/manim-animator
```

Or manually clone into your agent's skill directory (the whole repo *is* the skill):

```bash
# opencode (project)          .opencode/skills/
# opencode (global)           ~/.config/opencode/skills/
# Claude Code (global)        ~/.claude/skills/
# any agentskills client      ~/.agents/skills/

git clone https://github.com/Hmzbo/manim-animator ~/.config/opencode/skills/manim-animator
```

## Requirements

| Component | Required | Notes |
|---|---|---|
| [uv](https://docs.astral.sh/uv) | Yes | The skill never uses pip/conda. Install: `powershell -c "irm https://astral.sh/uv/install.ps1 \| iex"` (Windows) or `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| Network access | First run only | uv fetches manim and a managed Python into its cache |
| LaTeX (MiKTeX / TeX Live / MacTeX) | Optional | Needed for `MathTex` equation rendering. Without it the skill degrades gracefully to plain-text visuals |
| ffmpeg | Optional | Only for stitching independently-rendered scenes; pyav fallback included |

Run the bundled checker anytime:

```bash
uv run <skill-dir>/scripts/preflight.py
```

## Example prompts

- "Animate solving the equation x + 10 = 1 step by step."
- "Make a video explaining integrals as area under a curve, with Riemann rectangles getting finer."
- "Visually explain the Pythagorean theorem."
- "Show gradient descent converging on a function."

## License

[MIT](LICENSE)
