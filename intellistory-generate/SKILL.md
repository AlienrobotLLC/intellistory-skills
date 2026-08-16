---
name: intellistory-generate
version: 0.1.0
description: |
  Generate images, video and audio inside an IntelliStory project — boards, concept
  frames, character sheets, previz/animatic/final shots, voice-over and lip-sync — with
  the platform's rules: pick a model from the live list, estimate before you spend, use
  references for consistency, record prompt/model/cost/refs on the version, submit your
  best takes for review. Also: put a render made ELSEWHERE onto a shot (upload), and the
  review loop. Use when the user says "generate", "render", "board this shot", "make a take",
  "animate the approved still", "voice this line", "how much would it cost", "upload these
  renders to the shot", "submit for review".
argument-hint: "[project] [image|video|speech] [--shot <code>] [--estimate] [--upload <files>]"
allowed-tools: Bash
---

# IntelliStory — Generation

Every generation bills a real account. Every finished file becomes a **version** on a shot
or asset. Those two facts shape everything below. Tools are reached over MCP or with
`intellistory <tool> …` (see `intellistory-cli`).

## The shape of every generation

```
1. list_image_models / list_video_models      what's available, what it costs, what refs each takes
2. generate_image / generate_video / generate_speech  … with estimate_only: true first
3. show the user the number → yes → same call without estimate_only → job_id
4. wait_for_job(job_id)   (CLI: intellistory wait <job_id>)     — status "created" = queued, not lost; don't resubmit
5. get_shot_files / get_asset_files              the version landed; hand back its app_url
```

**Always Ask.** `generate_image`, `generate_video`, `regenerate_sequence` all take
`estimate_only: true` and return the price without queueing. Quote it as an estimate ("about
$0.61 for a 5 s 1080p take"), get a yes, then run. If the wallet can't cover it the call
returns `insufficient_funds` with the estimate — relay plainly and stop. Don't fan out
speculatively; eight variations "to see" is the user's decision. `get_billing_status`
before a large batch; don't quote prices from memory — `list_image_models` is the source.

## Targets

- **A shot**: pass `shot_id` (or the code) — the result is a version on it. Without a prompt,
  `generate_image` uses the shot's stored prompt (`write_shot_prompt`; `suggest_prompt` for a
  look-aware suggestion). Reference images: the shot's cast —
  `get_shot_assets(include_latest_approved: true)`.
- **An asset** (character/environment sheet slot): `entity_type`, `entity_id`, `sheet_slot`
  (`read_asset_sheet` for the slots). Then generate shots FROM the sheet as references —
  prompt text drifts between renders; a reference doesn't.
- **Scratch**: `project_id: "playground"` — the workspace's scratchpad, nothing attached.
- **A whole sequence**: `regenerate_sequence(project_id, sequence_id, model, media_type:
  "image"|"video", estimate_only: true)` → `{shot_count, per_shot_usd, total_usd}` → run →
  `get_populate_job`. New versions only; approvals stay where they are. Video mode animates
  from each shot's approved still when there is one — lock that still deliberately first.

## Video, references, audio

- `list_video_models` **before** attaching references: models differ in what they take
  (start frame only vs dozens of mixed attachments) — over-limit refs are dropped or
  rejected and the clip comes back subtly wrong. Some models refuse reference footage with
  recognisable people; the error says which case you hit and what to do — read it, don't
  retry identically.
- Inputs: `image_url` (i2v / character still), `reference_video_url` (motion), `image_urls`
  / `video_urls` / `audio_urls` (multi-ref), `last_frame_url`, `aspect_ratio` (match the
  master), `duration`, `resolution`, `seed`, `generate_audio`.
- **Dialogue** — two modes. *Native*: `generate_audio: true`, the engine voices the quoted
  line; cheap, voice drifts between takes. *Anchored*: `generate_speech(text, voice)` →
  `reference_audio_url` → `generate_video(image_url, reference_audio_url)`; lip-sync locked,
  identical voice across shots — for anything that ships. Never both. Voice identity is on
  the character (`write_asset_profile(voice_profile)`); prompt rules and cadence math in
  `references/ag-dialogue.md`. `transcribe_audio` a take and compare to the intended line
  before surfacing it — expect rejections; lip-sync is brute force.

## The version record — four fields, three fail silently

- `pass_type` is the production **stage**, not the media kind: images `storyboard` /
  `concept`; video `previz` / `animatic` / `final`; audio `vo`; 3D `model3d`. A conflicting
  value is auto-corrected without error — pick the right one.
- `refs[]` — `[{url, type, usage, strength}]` of the media the generator consumed — is the
  **only** thing that renders the version's Sources strip. Describing references in prose
  metadata shows nothing. Retrofit with `update_generated_file(file_id, refs)`.
- `notes` — observations, settings, comparisons; renders as Markdown in the viewer. Do not
  bury this in `metadata`, and do not put it in the shot `description` (a logline that
  seeds prompts).
- `metadata.prompt`, `metadata.model`, `metadata.cost_usd` (or `cost_credits`) — every time.

## A render made elsewhere

- **From disk** (CLI): `intellistory upload ./take.mp4 --shot sq010_sh0020 --pass-type previz`
  — the platform stores it as a real version; audio must be muxed in first
  (`ffmpeg -i v.mp4 -i a.wav -map 0:v -map 1:a -c:v copy out.mp4`); ≤ 36 MB per file for
  now. Over MCP the equivalent is `upload_media` (base64) where the server has it, else
  `upload_image` for stills.
- **From a URL** you already have: `insert_generated_file(project_id, entity_type,
  entity_ref, pass_type, public_url, metadata{prompt, model, cost}, refs, notes)`. Pass the
  code as `entity_ref`; the UUID is resolved for you.
- `create_media_link(file_id)` mints a public URL (default 7 days) when a human needs one;
  otherwise hand back `app_url` or `create_share_link`.

## The review loop

Everything you insert is `ip` (in progress) — WIP that never demands the owner's attention.
Iterate freely, then `submit_for_review(file_id, note)` your **best** candidate(s) — not every
take. Poll `get_review_queue(review_status: "rtk")` for send-backs (`review_note` says what to
change; fix, resubmit) and `"apr"` for approvals. Approve/send back yourself only if you're
acting as the owner (`approve_file`, `send_back_version` — note required).

## Behaviour

- Say what happened, not the plumbing: "boarded six shots of sq020, ~$1.20" — not the model
  routing or the queue.
- One test frame before a batch. Quote → yes → run. Record cost on every version.
- `search_images(query)` finds existing frames before you make new ones.
- After a batch that changed the project's shape: `invalidate_cortex` once.

## References

- `references/ag-generation.md` — the shape, references, character consistency, bulk
- `references/ag-billing.md` — Always Ask, credits, what to say
- `references/ag-versions.md` — pass_type, refs, notes, upload_media vs insert_generated_file, review states
- `references/ag-dialogue.md` — voice, lip-sync, prompt rules, IntelliBeat
- `references/ag-failure-modes.md` — silent no-ops and false alarms
