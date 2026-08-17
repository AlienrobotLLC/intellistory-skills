<!-- Generated from the IntelliStory agent knowledge base (article `ag-generation`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Generating Images, Video and Audio

## The shape of every generation

1. Pick a model — `list_image_models` for what's available and what it costs
2. Call `generate_image` / `generate_video` / `generate_speech`
3. Poll with `wait_for_job` (or the job id it returns)
4. The finished file lands as a version; check it with `get_shot_files` or
   `get_asset_files`

Generation is asynchronous. A returned job id is not a finished render, and a
job that reports `created` has been queued but not yet picked up — that's a
normal transient state, not a failure. Don't re-submit on the assumption the
first call was lost; poll.

## Regenerating a whole sequence

`regenerate_sequence` re-renders EVERY shot in a sequence with overrides — a
newer/better model, a higher resolution, or `media_type: "video"` for animatic
clips instead of stills. It reuses each shot's stored prompt (video mode
prefers the compiled layout video prompt, and uses the shot's approved still
as the i2v source when one exists) and queues one job per shot as a NEW
version — nothing existing is touched, and approvals stay where they are
until the user moves them.

The workflow is Always Ask: call it first with `estimate_only: true` to get
`{shot_count, per_shot_usd, total_usd}`, show the user the total, and only
run after their go-ahead. Poll with `get_populate_job`; each run's files are
grouped by `regen_batch_id` in version metadata.

## Cost is real money

Every generation bills the account. Two consequences:

- **Don't fan out speculatively.** Generating eight variations to see which is
  best is a decision for the user to make, not a default.
- **Record the cost on the version** (see `ag-versions.md`). A render with no
  recorded cost can't be reconciled against the bill later.
- **Estimate first.** `generate_image` / `generate_video` /
  `regenerate_sequence` all take `estimate_only: true` — get the number, show
  the user, then run. See `ag-billing.md`.

`list_image_models` is the source of truth for pricing — it reflects what's
actually configured. Don't quote prices from memory; they change, sometimes on
promotion. `ag-billing.md` covers credits and who is exempt.

Some models are offered on more than one host at different prices — Seedream
4.5 / 5 Lite / 5 Pro, Nano Banana 2 / Pro, Qwen Image Edit Plus among them.
`list_image_models` reports each entry's `host` and a shared `family` id for
those. Treat a family as ONE model: don't offer the user "Seedream 5 Pro (host
A)" and "Seedream 5 Pro (host B)" as if they were different renders. Pick the
member with the lower `pricing` for the size you're rendering (some hosts are
flat across resolutions; others double above 2K), pass that member's key. If the
user has their own provider key configured, that host is free to their wallet —
prefer it. The app's own model pickers do exactly this.

## Reference media

Most models accept reference images. Some accept video and audio input. Pass them
through the generation call, and then — separately and just as importantly —
record them in `refs` on the resulting version, or the user will never see what
the render was built from.

Two known constraints worth knowing before you promise a user something:

- **Not every video model accepts every input kind.** Video-to-video and
  keyframe input are supported by some models and not others. `list_video_models`
  answers this per model, along with *how many* references of each kind it takes
  — the range is wide (one start frame on some, fifty mixed attachments on
  others). Check it before attaching a stack of references: anything over a
  model's limit is dropped or rejected, and the clip comes back looking almost
  right, which is very hard to diagnose after the fact. The same listing reports
  whether the numbers were measured or merely inferred, so treat "assumed" as a
  hint rather than a promise.
- **Some video models reject reference footage containing recognisable people.**
  The platform has a supported workaround for it. If a video-ref job comes back rejected, the error explains which
  case you hit — read it rather than retrying identically.

## Character consistency

The reliable path is a character sheet on the asset, then reference-driven
generation from it — not prompt description alone. Prompt text drifts across
renders; a reference doesn't. `read_asset_profile` tells you what's established
for a character before you generate anything new.

## Storyboards in bulk

`generate_storyboard` boards a run of shots rather than one frame. It's the right
tool when a sequence needs covering; looping `generate_image` per shot is slower
and costs the same.

## What to tell the user

Describe what happened, not the plumbing: "I've boarded the first six shots of
the sequence" — not the model routing, the queue, or the pass structure behind
it. See the confidentiality note in the README.

Related: `ag-versions.md`, `ag-billing.md`, `ag-pipelines.md`.
