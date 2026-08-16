<!-- Generated from the IntelliStory agent knowledge base (article `ag-failure-modes`, last edited 2026-07-28). Do not edit here — edit the KB and rebuild. -->
# Failure Modes — What Silently Goes Wrong

Drawn from things that actually broke. The pattern in almost every case: the call
returned success and the result didn't appear where the user was looking.

## Silent no-ops

These succeed and do nothing useful. There is no error to catch.

| What you did | What happened |
|---|---|
| Described references in `metadata` (`character_ref`, `references`, `motion_driver`, `source_url`) | Sources strip stays empty. Only `refs` renders. |
| Put version observations in `metadata` keys | Not shown anywhere. `notes` is the column that displays. |
| Tagged a video `pass_type: storyboard` | Auto-corrected to `final`. Your label was discarded. |
| Wrote a long test writeup into a shot's `description` | It's a logline that seeds prompts — now the prompt is polluted. |
| Stripped `@S` / `@C` tags while editing a script | Scenes and dialogue silently unlinked from their entity records. |
| Guessed an `@-ID` instead of registering | Points at a different element, or nothing. |
| Uploaded a silent video and set audio metadata | Plays silent. The record plays the bytes you uploaded. |

## Things that look like failures and aren't

- **Job status `created`** — queued, not yet picked up. Poll. Re-submitting
  duplicates the work and the charge.
- **A video-ref job rejected** — some video models refuse reference footage with
  recognisable people. The error says which case you hit; the platform has a supported workaround for it. Retrying the identical call fails identically.
- **A shot with no versions** — normal for an un-boarded shot. Not an error state.

## Things that look fine and aren't

- **A generation you didn't record cost on.** Reconciliation later is impossible.
- **A breakdown re-run on a project with existing shots.** It replaces them. Ask.
- **A layout run scoped to the whole project when you needed six shots.** Pass
  `shot_ids`.
- **A hand-written Loom thread.** A later spin overwrites it. Say so.

## Getting an empty read

If a read tool returns nothing where you expected content, check in this order:

1. Right project? `list_projects` — a user's default org may not hold the project
   you're thinking of.
2. Right entity? A `ref` code that doesn't exist resolves to nothing, quietly.
3. Does the upstream thing exist? No script means no scenes means no shots. The
   chain in `ag-pipelines.md` has to be walked in order.
4. Stale view? `invalidate_cortex` after significant changes, then re-read.

## When you're not sure

Say so, and read rather than guess. Inventing an ID, a price, or a capability
costs the user more than a clarifying question does. For platform behaviour,
`list_knowledge_base` and `read_knowledge_article` are authoritative — answer
from them instead of from memory.

Related: `ag-versions.md`, `ag-ids-and-refs.md`, `ag-pipelines.md`.
