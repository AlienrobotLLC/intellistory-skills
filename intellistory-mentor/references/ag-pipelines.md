<!-- Generated from the IntelliStory agent knowledge base (article `ag-pipelines`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Pipelines — What Each Pass Writes

The passes are ordered, and each one consumes what the last produced. Running
them out of order mostly "works" and produces thin results, which is worse than
failing.

## The order

```
script  →  structure   run_structure_analysis         →  Acts / Scenes, structure read (poll read_big_picture)
        →  breakdown   run_pipeline  — or by hand:     →  Sequences > Shots
                       create_sequence / create_shot
        →  layout      run_layout_pipeline             →  per-shot camera / lighting / sound / blocking
        →  boards      generate_image (shot_id) or     →  boarded frames, one version per shot
                       regenerate_sequence (whole sequence)
```

(The app runs the same passes from its Breakdown / Layout / Storyboard buttons;
`analyze_script` and `suggest_breaks` are in-app chat tools, not MCP tools.)

Nothing downstream of the script improves if the script isn't there. If a project
has a premise and no script, the useful next move is story work, not a breakdown
that will invent shots out of an outline.

## What each pass owns — and overwrites

| Pass | Writes | Overwrites? |
|---|---|---|
| `run_structure_analysis` | Acts, Scenes, Big Picture structure read | re-runs replace the structure read |
| breakdown (`run_pipeline`, or the app) | Sequences and Shots | **destructive to existing shots** — `run_pipeline` works on its own branch |
| `run_layout_pipeline` | per-shot layout | replaces layout on the shots it touches |
| `generate_image` / `regenerate_sequence` | new versions | additive — old versions stay |

The two to be careful with: **a breakdown re-run can replace shots a user has
already worked on**, and a layout run replaces layout wholesale for the shots in
scope. Both accept a narrower scope — pass specific `shot_ids` to
`run_layout_pipeline` rather than letting it take the whole project when you only
need a few.

Ask before re-running either on a project that already has work in it.

## Layout specifics

`run_layout_pipeline` needs both `project_id` and `org_id`. It returns a `jobId`;
poll `get_layout_job`. `direction` takes director notes and genuinely changes the
result — pass the user's intent through rather than running it bare.

Two cheap partial modes are NON-destructive (they merge one field group and
leave everything else alone):

- `dialogue_only: true` — backfills `layout.dialogue` (verbatim, speaker-
  attributed) from the scene script, plus its beat bridge.
- `camera_only: true` — a scene-level **camera coherence pass**: one DP review
  per scene sees all its shots' camera specs together and re-plans them as a
  set (coverage grammar, the 180° line, size/angle variety, escalation).
  Updates only `layout.camera`, `shot_role`, and the scene's camera_language
  anchor. Run it after a full layout when a scene's camera feels templated, or
  after shots are reordered. Shots that were already expanded keep their old
  prompts — re-expand them (force) so the new camera reaches the dense prompts.

`clear_layout` exists for when layout is wrong rather than merely old.

## The autonomous pipeline

`run_pipeline` runs the whole chain and **creates a dedicated branch** to do it.
That's the safety property that makes it usable — it doesn't touch the working
state. `approval_mode` defaults to `human`; leaving it there means the user sees
each stage. Set it to `auto` only when the user has asked for that.

Merge with `merge_branch` when they've approved.

## After significant changes

Call `invalidate_cortex` once you've made meaningful changes to a project's story
or structure. Without it, later reads can serve a stale view of the project and
you'll act on out-of-date context. Once, at the end of a batch of changes — not
after every write.

## Realtime

Shots, sequences and assets publish changes live. Something you insert appears in
the user's open window without a reload. Useful to know when narrating: you don't
need to tell them to refresh.

Related: `ag-ids-and-refs.md` for the tags the script pass relies on,
`ag-loom-and-analysis.md` for the analysis layer.
