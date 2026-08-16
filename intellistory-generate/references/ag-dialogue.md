<!-- Generated from the IntelliStory agent knowledge base (article `ag-dialogue`, last edited 2026-08-10). Do not edit here — edit the KB and rebuild. -->
# Dialogue Shots — Voice, Lip-Sync and Prompting

How to make characters speak on screen with a stable voice and correct lip-sync.

## The data flow

1. **Voice identity is character-level.** Each character asset carries a
   `voice_profile` section (`read_asset_profile` / `write_asset_profile`):
   archetype, timbre, pitch, accent, cadence, emotion_baseline, tts_voice_id.
   Write it once — never re-describe a voice per shot.
2. **The layout pipeline extracts dialogue** into `layout.dialogue`
   (verbatim `{character, line, delivery}` per shot) and auto-bridges it into the
   shot's beats with each speaker's voice parenthetical attached. It also stamps
   `speaking_time_estimate_s` and, when the speech can't fit any engine's max
   clip, `dialogue_split_warning`.
3. **Beats carry structured dialogue.** In `write_shot_beats` /
   `write_sequence_beats`, a spoken line is
   `dialogue: { speaker, line, delivery?, voice?, language? }` (or an array for
   an exchange inside one beat) — the compile emits
   `SPEAKER (voice) says delivery: "line"` plus a dry-audio clause. The freeform
   `dialog` string is legacy; prefer the structured form so the RIGHT character
   lip-syncs when two are in frame.

## Rendering the voice — two modes

- **Native**: `generate_video` with `generate_audio: true` — the engine voices
  the quoted line in the prompt. Cheap and fast; voice identity drifts between
  takes. Good for tests and one-off lines.
- **Anchored**: `generate_speech` (use the character's `tts_voice_id`) →
  pass the returned `reference_audio_url` to `generate_video` — the platform
  builds the performance anchor, lip-sync is frame-locked, and the voice is
  identical across every shot. Use for anything that ships.
- Never both: an attached reference VO always wins and native audio is disabled.

## Prompt rules (any surface that writes prompts)

- **No emotion words on bodies.** "Miserable, defeated guard" physically shrinks
  the character between shots. Emotion lives in facial expression, gesture, and
  the delivery word only.
- **Speaker-first, verbatim line, clause last:**
  `Dialogue language: English. SARGE (booming, low, slow and drawn-out) says wearily: "Rain, Tim. Rain."`
- **One sync instruction max** — "Precise mouth synchronization." Over-directing
  mouth mechanics degrades the sync.
- **Render dry:** end with "Studio isolation, dry acoustics, the background is
  silent." Ambience/Foley/weather SOUND are post-production; visual weather is fine.
- **Name the target of every interaction** — "shows the photograph to RAY", not
  "slides the photograph across the table". Ambiguous targets play to camera.
- **Duration covers the words**: slow cadence ≈ 1.8 words/sec, neutral ≈ 2.5,
  rapid ≈ 3.5, plus ~1s of handles. Max clip length is ENGINE-SPECIFIC — heed
  the `warnings` returned by `write_shot_beats` and `dialogue_split_warning`
  on the layout; split long exchanges across shots at layout time.
- **Cadence is a control, not flavor**: slow/drawn-out → wide mouth shapes and
  heavy gestures; rapid/deadpan → minimal twitches. Contrast your characters.

## Working with IntelliBeat (the beats tools)

`write_shot_beats` / `write_sequence_beats` / `read_shot_beats` share storage
(`layout.beats`) and the compiler with the UI's IntelliBeat editor — beats you
write appear as editable rows in the user's beat editor, and beats the user
authors come back to you through `read_shot_beats`. Practical consequences:

- Structured `dialogue` you write shows up as speaker/delivery/line rows in the
  UI; a bare `dialog` string shows up as an unattributed legacy line the user is
  prompted to convert. Always prefer the structured form.
- The UI shows the user the same cadence-based speaking-time math you should be
  doing: if `write_shot_beats` returns dialogue-fit `warnings`, fix the split
  BEFORE generating — the user will see the same overrun flagged in red.
- The layout pipeline may re-bridge `layout.dialogue` into beats (adding voice
  parentheticals from asset profiles) on its next run — don't fight it: keep
  `layout.dialogue` as the source of truth for WHAT is said, and treat the beat
  `dialogue` entries as the compiled-toward form.
- `continuity` on `write_shot_beats` / `write_sequence_beats` controls how beats
  join: `cuts` (default — "Shot transition." markers) or `single-take` (one
  unbroken camera move: "SEQUENCE SHOT. NO CUT." + timed `N-Ms:` segments — the
  Seedance sequence-shot form for long continuous takes). Stored in
  `layout.continuity`; the UI exposes it as the Continuity switch in IntelliBeat
  Studio, so respect an existing stored value unless the user asks to change it.
- Per-beat override: `beat.transition` (`cut` | `continuous`) is the join INTO
  that beat and wins over the shot mode. Mixed boundaries compile as one flowing
  take with "Hard cut." only where marked — use this when some beats are camera
  moves within a continuous action and others are true cuts. The Studio timeline
  shows each boundary as a clickable ✂/→ chip.
- The reverse direction exists: `parse_prompt_to_beats` decomposes an EXISTING
  video prompt into beats — timestamped segments ("shot 2 (3-6s)", "0-3秒"),
  cut-separated ("切镜"/"Cut."), or single-flow, in any language. By default it
  reads the shot's ingested generated-video prompt (imported shots),
  preserves the original verbatim as `generation_prompts.video.prompt`, and
  writes the parsed beats as the editable decomposition. Use `write:false` to
  preview. Quoted lines come back as structured `dialogue`; reference
  placeholders (`<<<image_1>>>`, `@name`) are preserved verbatim. The same
  action is the "Import" button in the UI's IntelliBeat tab.

## Batch and verify

Locked lip-sync is brute force — generate several takes per dialogue shot and
expect rejections. `transcribe_audio` a take and compare against the intended
line before surfacing it to the user.
