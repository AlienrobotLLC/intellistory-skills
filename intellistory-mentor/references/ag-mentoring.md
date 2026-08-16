<!-- Generated from the IntelliStory agent knowledge base (article `ag-mentoring`, last edited 2026-08-07). Do not edit here — edit the KB and rebuild. -->
# Production Mentor & the Mentor Plan

Mentor Mode is a guided path through the whole production. In the app it is
Orson driving; over MCP it is you, in the **Production Mentor** role. Both
surfaces share ONE plan: `get_mentor_plan` / `update_mentor_plan`. The app's
mentor strip and checklist render whatever you write — keep statuses honest.

## The contract

- Plan = ten phases, fixed ids:
  `foundation, ingest, story, look, assets, breakdown, layout, boards, video_audio, review`.
  Every phase appears in every plan; tailoring is done with status `skipped`,
  never by omitting a phase. Statuses: `done | active | pending | skipped`.
- **The array ORDER you send is saved and is the project's route.** There is no
  canonical sequence — see Routes below. The backend preserves your order.
- Each phase has fixed step ids (the validator rejects invented ones — call
  `update_mentor_plan` with a wrong id and the error names the offender).
  Steps you omit are auto-filled as `pending`.
- `currentPhase` / `currentStep` are the resume cursor. Set them; the next
  mentor session (app or agent) starts there.

## Routes — no fixed order

Story-first and visual-first are both real ways to make a film. Pick the one
that matches how this filmmaker works, put its id in `profile.route`, name it
out loud, and say the other way is equally valid. Never imply they started
wrong.

| `profile.route` | For | Order |
|---|---|---|
| `story_first` | Default. Premise, outline, a loose document, or a blank page. | foundation, ingest, story, breakdown, boards, look, assets, layout, video_audio, review |
| `visual_first` | Has art or a world; the story comes out of it. | foundation, ingest, look, assets, story, breakdown, layout, boards, video_audio, review |
| `script_import` | Finished screenplay / .fdx — story work is repair, not writing. | ingest, foundation, story, breakdown, boards, look, assets, layout, video_audio, review |
| `music_video` | Cut to an existing track; structure follows the audio. | foundation, ingest, look, story, breakdown, assets, layout, boards, video_audio, review |

If the user wants an order no route covers, build one and save it. The plan
serves the film.

## The flow

1. **DISCOVER** — one question at a time: what are you making, what do you
   already have (script / outline / a document from anywhere / artwork / audio /
   nothing), where should it end up, and — only if still open — story-first or
   pictures-first.
2. **PLAN** — propose in route order, adjust with the user, then
   `update_mentor_plan`.
3. **DRIVE** — do each step with the normal tools. Narrate why in one breath.
   Before marking a step done, VERIFY from data (`read_cortex`, `list_assets`,
   `list_shots`, `get_shot_files`) — a step is done when the project shows it,
   not when the conversation remembers it.
4. **RESUME** — `get_mentor_plan` FIRST on any return visit. A null mentor
   means never mentored; `active: false` with phases means paused — continue
   it, don't restart.

## What each phase is really for

- **breakdown** — a script has acts and scenes; those are NOT shots. The
  breakdown produces sequences and shots AND calls out every character and
  location. Review the call-out list with the user and **flag scope**: every
  character needs a sheet, every location a design, so a short with 14 speaking
  parts is a problem worth raising before the artwork exists.
- **boards** — board a sequence or the whole script. Real spend: quote the
  estimate, get a yes. Test one panel first. Boarding before look-dev is normal.
- **look** — New Look → name it → Sheet tab → drag in references (the sheet
  reads them and writes the description) → Save Look. A look is text **and** a
  composite pinboard image. Multiple looks per project is normal — characters,
  environments, a sequence — and they're recalled at generation time.
- **assets** — the breakdown leaves empty cards; that's the head start. Work in
  Generate with reference images and variations before committing, then build
  the sheet. Characters and environments get different sheet types.
- **layout** — the most-skipped phase and the one that most changes the result.
  Runs a sequence or a single shot at a time, works out what must be in frame in
  context, and produces the prompt and camera work.
- **video_audio** — build a shot in layers: board → refined board → final-look
  still → video. Most models animate from a first frame, so lock that still
  deliberately. Shots also take audio and video references (phone footage of the
  action is a legitimate input); everything brought in is stored in the project
  references and reusable.

## Not built yet — never describe as available

- Laying one audio track across a whole sequence to drive generation for its
  shots. (Per-shot audio references DO work.)
- Handles — rendering longer than the cut needs so editorial has trim room.

## Failure modes

- Marking a step done from memory: the app's live status will contradict you
  and the user sees a checklist that lies. Verify, then mark.
- Restarting discovery on a project with a plan: the user already answered.
  Read the profile in the plan instead.
- Implying there's one right order, or that a filmmaker who arrived with
  artwork (or a document written elsewhere) should have started with a premise.
- Driving spend without an estimate: `generate_image`/`generate_video` support
  `estimate_only` — quote first, confirm, then run (`ag-billing.md`).
- Breakdown replaces shots — explicit confirm every time (`ag-pipelines.md`).

Related: `ag-recipes.md`, `ag-billing.md`, `ag-pipelines.md`.
