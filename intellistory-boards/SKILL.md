---
name: intellistory-boards
version: 0.1.0
description: |
  From script to storyboards in an IntelliStory project: structure read → sequences and
  shots (breakdown) → layout (camera / lighting / sound / blocking, per shot, with the
  director's intent) → timed beats and dialogue (IntelliBeat) → boarded frames → animatic.
  Use when the user says "break this down", "make the shot list", "board sequence 3",
  "run layout", "write the beats for this shot", "export the animatic", or wants a
  sequence covered. Knows which passes overwrite what, and asks before those. Rendering
  details live in intellistory-generate.
argument-hint: "[project] [structure | breakdown | layout <sequence> | beats <shot> | board <sequence> | animatic <sequence>]"
allowed-tools: Bash
---

# IntelliStory — Breakdown, Layout, Boards

The passes are ordered and each consumes what the last produced. Out of order mostly
"works" and gives thin results — worse than failing. Tools are reached over MCP or with
`intellistory <tool> …` (see `intellistory-cli`).

## The order

```
script            read_document(script)            nothing downstream improves without one
structure         run_structure_analysis           acts / scenes; minutes; results in read_big_picture
breakdown         run_pipeline  — or by hand:      sequences > shots  (+ every character/location called out)
                  create_sequence / create_shot
layout            run_layout_pipeline              per-shot camera/lighting/sound/blocking; pass `direction`
beats + dialogue  write_shot_beats / write_sequence_beats  (IntelliBeat) — timed, compiled to a video prompt
boards            generate_image(shot_id) per shot · regenerate_sequence for a whole sequence
animatic          export_animatic(sequence_id)
```

Then `invalidate_cortex` once.

## Breakdown — two doors, both need a yes

- **`run_pipeline`** runs the chain autonomously on a **dedicated branch** — that's the safety
  property. Leave `approval_mode: "human"` (the default) so the user sees each stage; `auto`
  only if they asked. `merge_branch` when they approve; `cancel_pipeline` to stop.
- **By hand**: `create_sequence(project_id, name, code?)` then `create_shot(project_id,
  sequence_id, name, description, duration_sec, dialogue?)`. Codes derive if you omit them —
  a shot under `sq043` becomes `sq043_sh0010`; the app's breakdown numbers sequences in tens.
  Keep `description` a 1–3 sentence logline: it seeds the prompt.

A breakdown that replaces existing shots is destructive. On a project that already has
shots, **show the proposed sequence/shot list and ask** before creating anything, and flag
scope: every character needs a sheet and every location a design — a short with fourteen
speaking parts is worth raising before artwork exists. Cast assets into shots with
`link_assets_to_shot` (many at once; check `failed`); audit coverage with
`get_sequence_dependencies` ("which shots is this character missing from?").

## Layout — the most-skipped pass and the one that changes the most

```
run_layout_pipeline(project_id, org_id, shot_ids?, direction)   → jobId → get_layout_job
```

- `direction` takes the director's notes and genuinely changes the result — pass the
  user's intent through, never run it bare.
- Scope it: pass `shot_ids` for the shots you need. A whole-project run **replaces layout
  wholesale** on every shot it touches; ask before that on a project with work in it.
- Two cheap **non-destructive** partial modes: `dialogue_only: true` backfills verbatim,
  speaker-attributed dialogue from the scene script; `camera_only: true` re-plans a
  scene's shots as a coherent set (coverage grammar, the line, size/angle variety,
  escalation) — run it after a full layout when a scene feels templated, or after reordering.
- `read_layout(shot_id)` / `write_layout(shot_id, camera|lighting|sound|blocking|vfx,
  director_notes)` for one shot by hand; `clear_layout` (one scope) when it's wrong, not old.

## Beats and dialogue — IntelliBeat

`write_shot_beats(shot_id, beats, duration, continuity, …)` authors timed beats and compiles
the video prompt; `write_sequence_beats` does many shots in one call; `read_shot_beats`
reads them back (they're the same rows the user edits in the app). Rules that matter:

- Spoken lines are structured: `dialogue: { speaker, line, delivery?, voice?, language? }`
  (array for an exchange) — so the RIGHT character lip-syncs. Not a bare string.
- Duration must cover the words (slow ≈ 1.8 words/s, neutral ≈ 2.5, rapid ≈ 3.5, plus ~1s
  handles). Heed the `warnings` the tool returns; split long exchanges across shots.
- `continuity`: `cuts` (default) or `single-take`; per-beat `transition: cut | continuous`.
  Respect a stored value unless asked to change it.
- No emotion words on bodies; name the target of every interaction; one sync instruction
  max; render dry (`references/ag-dialogue.md`).
- Voice identity is character-level: `write_asset_profile(voice_profile: {…})` once, not
  per shot.

## Boards

- One shot: `generate_image(shot_id, prompt?)` — with no prompt it uses the shot's stored
  prompt (`write_shot_prompt` / `suggest_prompt` to set one). Reference images: the shot's
  cast (`get_shot_assets(include_latest_approved: true)`) is the right source.
- A whole sequence: `regenerate_sequence(project_id, sequence_id, model?, media_type:
  "image" | "video", estimate_only: true)` → show `{shot_count, per_shot_usd, total_usd}` →
  run → `get_populate_job`. New versions only; approvals untouched.
- Test **one panel first**. Quote the estimate. Get a yes. Boards before look-dev is normal.
- Every board is a version on the shot: `pass_type: storyboard`; record `prompt`, `model`,
  cost, and `refs` (`references/ag-versions.md`).

`export_animatic(sequence_id)` queues the cut the Shots view offers; poll and hand back the
result's `app_url` (or `create_share_link`).

## Behaviour

- Ask before anything destructive (breakdown on existing shots, whole-project layout).
- Narrate the *why* in one breath; describe what happened, not the passes behind it.
- Verify from data before saying a step is done: `list_shots`, `read_layout`,
  `get_shot_files`.

## References

- `references/ag-pipelines.md` — what each pass writes and overwrites
- `references/ag-dialogue.md` — dialogue shots, voice, lip-sync, IntelliBeat rules
- `references/ag-recipes.md` — script → boarded sequence, end to end
- `references/ag-ids-and-refs.md` — codes, @-IDs, `app_url`
