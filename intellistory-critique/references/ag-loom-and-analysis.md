<!-- Generated from the IntelliStory agent knowledge base (article `ag-loom-and-analysis`, last edited 2026-08-06). Do not edit here — edit the KB and rebuild. -->
# The Loom, Big Picture and Story Core

Three analysis surfaces, three different jobs. All three are readable *and*
writable — a common mistake is treating them as read-only outputs and duplicating
their content into a document instead.

## Story Core

The project's creative north star: what it's about, why it exists, the spine.
`read_story_core` / `write_story_core`.

Read it before doing anything creative. If it's empty, filling it in with the
user is a better first move than generating around a void.

## The Loom

Eleven narrative threads, each tracking one dimension across the whole script.
`read_loom_threads` reads them; `spin_loom_threads` builds them from the script;
`write_loom_thread` lets you author one by hand.

The threads, and the role each is written in:

| Thread key | Voice | Tracks |
|---|---|---|
| `script_map` | Story Editor | per scene: anchor line, beat label, emotional waypoints, tension 1–10, setups |
| `setup_registry` | Story Editor | every promise the script makes, and whether it pays off |
| `character_state` | Script Supervisor | per character entry/exit state: emotion, physical, knows / doesn't know |
| `info_asymmetry` | Story Editor | who knows what and when — including the audience |
| `motif_registry` | Film Scholar | recurring motifs and how their meaning evolves |
| `subtext_map` | Script Editor | what each scene is really about beneath the dialogue |
| `world_rules` | World Builder | rules the world establishes, and whether they've been tested |
| `decision_ledger` | Arc Analyst | choices under pressure: chose, gave up, cost |
| `audience_position` | Suspense Engineer | what the audience knows, believes, fears, wants |
| `voice_samples` | Dialogue Coach | per character: vocabulary, tics, rhythm, representative lines |
| `production_map` | Production Designer | per scene: action beats, props, environments, fx, lighting, sound |

**Write a thread when you have analysis the script can't yield** — world rules
from a series bible, voice samples from an interview, production notes from a
location recce — or to correct a thread that a spin got wrong.

Call `write_loom_thread` with just `project_id` + `thread_key` to get the full
brief for that thread: the role to write as, what to track, what to skip, and the
exact content shape. Use it. A thread written without the brief reads visibly
different from a spun one.

Two mechanics that matter:

- `prose` is the Markdown a user reads. `content` is the structured payload. Fill
  both — prose alone leaves the thread with no data behind it.
- Your keys **merge** over the last build by default, so writing prose won't
  destroy the structured data.
- **A later `spin_loom_threads` on the same thread will overwrite your hand-written
  version.** Tell the user that if they've asked you to author one.

## Big Picture

Project-level story analysis. `read_big_picture` / `write_big_picture`, and
`write_big_picture` also carries an authoring brief.

`narrative_health_check` is the quick read when a user asks "how's the story
doing" and you don't need the full surface.

## Test screening (reception, not craft)

`run_test_screening` is the other half of critique: instead of reviewing the
writing, it simulates an AUDIENCE watching one sequence — shots, camera,
durations, dialogue — through a chosen archetype lens (general, festival,
short-form, family-animation, horror, comedy, short-drama, or any free-text
description) and returns bucketed notes (structure / pacing / clarity /
emotional engagement / audio-visual), per-shot moment notes with severity, and
a prioritized top-fixes list.

Use it when the question is "will this land?" rather than "is this well
made?" — after layout has run (dialogue + durations make the screening much
sharper), and again after edits to see whether the fixes moved the engagement
score. Each run persists as a `test_screening` project document, so you can
compare runs and cite earlier screenings.

## Reading order for a critique

`read_cortex` first for the whole project state, then `read_story_core` for
intent, then the threads that bear on the question. `setup_registry` and
`decision_ledger` are the two that most often surface something the user hasn't
noticed — unpaid setups and choices that cost the character nothing.

Related: `ag-pipelines.md`, `ag-ids-and-refs.md` for the @-IDs threads cite.
