# Scenarios

Golden prompts across the IntelliStory skills: one user request, what a good run does, how
to score it. Run by a human (or an agent acting as the user) in a fresh session with the
skills installed and `intellistory whoami` succeeding.

---

## 1 — First contact, existing project (cli · story)

**User:** "Let's work on Ashfall."

**Expected:** `intellistory whoami` (or MCP equivalent) → `projects` → `use` the match →
`read_cortex` → `read_story_core` → greets with something specific from the project ("last
time you were working the second-act turn"), asks ONE question. No "how can I help?", no
tool narration.

**Fail:** asks for a project id, dumps JSON, asks three questions at once.

## 2 — Board a sequence with the price first (boards · generate)

**User:** "Board sequence 3."

**Expected:** confirms layout has run for that sequence (`read_layout` on a shot, or runs
`run_layout_pipeline` scoped to it with the user's direction) → `regenerate_sequence(…,
estimate_only: true)` → states `{shot_count, total_usd}` as an estimate → waits for a yes →
runs → `get_populate_job` → reports "boarded N shots, ~$X" with an `app_url`. Suggests one
test panel first if the sequence is large.

**Fail:** generates without a quote; loops `generate_image` per shot; quotes a price from memory.

## 3 — Put a local render on a shot (cli · ingest)

**User:** "I've got the new take of sq010_sh0020 at ~/renders/take7.mp4, put it on the shot."

**Expected:** `intellistory upload ~/renders/take7.mp4 --shot sq010_sh0020 --pass-type previz`
(or `final` if the user says it's final) → reports the version and its link. If > 36 MB,
explains the limit and offers a re-encode. Does NOT try to read the file into context, does
NOT ask for a public URL.

**Fail:** `pass_type: storyboard` on a video; asks the user to host the file somewhere.

## 4 — Notes on a script (critique)

**User:** "Give me notes on the script — be honest."

**Expected:** `read_cortex` → `read_story_core` → `read_document(script)` → `read_loom_threads`
(spins them if empty) → Good / Bad / Ugly with named scenes and lines, judged against the
Story Core; writes the notes into the project; ends with "which do you want to tackle first?"
Does not rewrite scenes.

**Fail:** generic praise; a rewritten scene; ten notes with no ranking.

## 5 — Add a character properly (story · look)

**User:** "Add a new character, Cleo, the harbourmaster's daughter."

**Expected:** `register_element(character, "CLEO")` → uses the returned @-ID verbatim →
`create_asset` → `write_asset_profile` from what the user says (asks one question if
thin) → offers a character sheet next. Never invents `@C7`.

**Fail:** writes "@C7" into a script without registering; edits the script and strips tags.

## 6 — Resume the mentor (mentor)

**User:** "Where were we?"

**Expected:** `get_mentor_plan` FIRST → resumes at `currentPhase`/`currentStep`, restates the
route ("we're story-first; next is layout for sq020") → verifies the previous step from data
before moving on. Does not re-run discovery.

**Fail:** asks "what are you making?" on a project with a plan.

## 7 — Dialogue take that ships (generate)

**User:** "Render Sarge's line 'Rain, Tim. Rain.' on sq040_sh0030 so it lip-syncs."

**Expected:** reads the character's `voice_profile` → `generate_speech(text, voice)` →
`generate_video(shot_id, image_url: approved still, reference_audio_url, estimate_only)` →
quote → yes → run → `transcribe_audio` the result and compares to the line before surfacing.
Records `refs`, prompt, model, cost on the version.

**Fail:** uses native audio for a shipping line; both native and reference audio; no estimate.

## 8 — Wrong flag, right recovery (cli)

**User:** "List the shots in sequence 2, just the codes."

**Expected:** `intellistory help list_shots` (or reads the error) → `intellistory list_shots
--sequence-id <id> --json | jq -r '.[].code'` — never guesses an option; if `--limit` errors
as unknown, adapts without asking the user.

**Fail:** repeated failed calls with invented options; asks the user for the option name.
