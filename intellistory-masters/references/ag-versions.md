<!-- Generated from the IntelliStory agent knowledge base (article `ag-versions`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Versions, Files and Sources

Every render you produce becomes a version on a shot or an asset. Four fields
decide whether it shows up correctly, and three of them fail silently when wrong.

## pass_type is the production stage, not the media kind

The media kind (image / video / audio / 3d) is derived from `content_type`.
`pass_type` says what **stage** this version represents:

| pass_type | Media | Means |
|---|---|---|
| `storyboard` | image | boarded key frame |
| `concept` | image | concept / style frame |
| `previz` | video | rough blocking or motion |
| `animatic` | video | timed animatic |
| `final` | video | finished shot render |
| `vo` | audio | voice-over / dialogue |
| `model3d` | 3d | `.glb` asset |

Tagging a video as `storyboard` is the classic mistake — that's an image stage. A
type that conflicts with the actual media gets auto-corrected to the default for
that kind (image → `storyboard`, video → `final`, audio → `vo`, 3d → `model3d`),
so a wrong value doesn't error, it just quietly becomes something else. Pick the
right one.

Do not invent types outside this list.

## refs[] is the only thing that renders Sources

If a version was made *from* reference media, you **must** pass `refs` with the
actual media URLs the generator consumed:

```json
"refs": [{ "url": "https://…/cleo_front.png", "type": "image", "usage": "character", "strength": 0.8 }]
```

Describing references in prose metadata — `character_ref`, `references`,
`motion_driver`, `rich_ref`, `source_url` — **does not display them**. The
Sources strip reads `refs` and nothing else. This is not a formatting preference;
prose ref keys are invisible to the user and always have been.

URLs in `refs` are auto-ingested and deduped into the project's ref library, so
you don't need to upload them separately first.

`update_generated_file` accepts `refs` too, so Sources can be retrofitted onto an
existing version in place — no need to create a duplicate version to fix it.

## notes is a real column

Extensive observations — what you tried, what worked, settings, comparisons —
go in `notes`. It's a column, and it renders in the version viewer as Markdown.

Do not scatter that material across `metadata` keys. It won't be shown, and the
next agent won't find it.

Keep the shot's own `description` short — it's a logline that seeds prompts, not
a place for a test writeup.

## Always record how it was made

Include `prompt`, `model`, and a cost (`cost_usd` or `cost_credits`) in
`metadata` on every version. Without them the version is an image with no
provenance, and no one can reproduce or price it later.

## Video with sound

The record plays exactly the bytes you upload. A silent render stays silent no
matter what you put in metadata — mux the audio in **before** uploading:

```bash
ffmpeg -i video.mp4 -i audio.wav -map 0:v -map 1:a -c:v copy out.mp4
```

## A file on your disk does not need a public host

`upload_media` takes base64 bytes — video, image or audio — and writes a real
version straight onto the shot. No S3, no external URL, no borrowed bucket. A
poster frame is extracted from video automatically, and `create_media_link` will
mint a shareable public URL for the result afterwards if a human needs one.

```
upload_media({ project_id, media_data: <base64>, content_type: "video/mp4",
               target: "shot", target_id: <shot uuid>, pass_type: "previz" })
```

Ceiling is ~36MB of actual bytes — base64 has to fit a 50MB request body. Above
that, upload a shorter or lower-bitrate encode rather than reaching for an
external host. Encode the file **client-side** and stream it into the argument;
never read a media file into your own context first.

`insert_generated_file` is the other door, and it is for media that already has
a URL — a provider result, or an archive row you deliberately leave at the
source. It cannot accept bytes.

Refs are not attached at upload time: follow with
`update_generated_file({ file_id, refs })` to cite what the version was made
from.

## Posters: slates and black heads are skipped automatically

The poster (thumbnail + preview — what the timeline and shot tiles show) of an
**uploaded / ingested video** is not blindly frame 0. At ingest the head of the
clip is analysed:

- a **static card that hard-cuts to picture** (a vendor slate, a leader) → the
  poster is taken from the first clean picture frame after it;
- a **blank head** (black or white fade-in) → same, first real picture frame;
- otherwise → frame 0, exactly as before.

Generated takes (renders the platform made itself) are not analysed — frame 0
is what the artist expects for a 5-second AI clip.

What gets recorded on the version, readable through `get_shot_files` /
`get_asset_files` (`metadata`):

```
metadata.poster = { time_s, frame, reason: "default"|"slate_skipped"|"blank_head"|"manual",
                    source: "auto"|"manual", at }
metadata.slate  = { detected, kind: "none"|"slate"|"blank", head_frames, head_seconds,
                    fps, frame_count, checked_at,
                    ocr?: { is_slate, show, title, link, version_name, version,
                            frame_range:{first,last}, date, artist, vendor, shot_type,
                            submitted_for, submission_note, other_fields, raw_text },
                    ocr_model?, ocr_provider?,
                    mismatch?: [ { field: "link"|"frame_range"|"version", slate, expected, source } ] }
```

When the head is a slate the card is **OCR'd** (one small vision call) and its
fields are **cross-checked** against what the file already claims: `link` vs the
plate token in the filename and the shot name (`079_AFS_1400`), `frame_range`
length vs the picture frames actually in the file, `version` vs any `_vNNNN` in
the filename. Anything that disagrees lands in `mismatch[]` — that is the
signal a delivery is mislabelled (wrong version, wrong shot folder). An empty
`mismatch` with `ocr` present means the slate and the file agree. The OCR is
tolerant of l/I/1 and O/0 confusions, so those never count as a mismatch.

The submission note on a slate is captured verbatim in `ocr.submission_note` —
it is usually the vendor's own change list for that version and is worth
quoting when summarising a delivery.

If the auto pick is wrong, `set_poster_frame({ file_id, time_s })` (or
`{ file_id, frame }`, or `{ file_id, mode: "auto" }` to re-run detection)
rewrites the poster and records `metadata.poster.source = "manual"`. It works on
any video version, ingested or generated. Browsers may hold the old thumbnail for
up to a day; the media proxy serves the new one immediately.

## Required fields

`insert_generated_file` needs `project_id`, `entity_type`, `entity_ref`,
`pass_type`, and one of `storage_path` or `public_url`. Everything else is
optional but see above — `refs` and `notes` are optional in the schema and
load-bearing in practice.

## The review loop — how your work reaches a human

Versions carry a `review_status`: `ip` (in progress),
`rev` (waiting for approval), `apr` (approved), `rtk` (retake), `rej`
(rejected), `fin` (final). Everything you insert defaults to `ip` — WIP that
never demands the owner's attention.

What the owner sees while you iterate: the Shots tab has a **Latest / Approved**
toggle. In Latest view (the default) your newest take shows on the shot tile the
moment it lands, flagged with its version number until it's approved; in
Approved view only the approved pick shows. So don't assume unapproved work is
invisible — but the formal review still happens through the queue below.

The contract:

1. **Iterate freely.** Make as many takes as the shot needs; they all stay `ip`.
2. **Submit your best candidate(s)** — `submit_for_review` with a one-line note
   on why this one (or `for_review: true` + `submit_note` directly on the
   insert). Submitted versions appear in the owner's Review queue. Submit
   candidates, not every take.
3. **Poll for verdicts** — `get_review_queue` with `review_status: 'rtk'`
   returns your sent-back versions; `review_note` says exactly what to change.
   Fix it, submit again (resubmission clears the old verdict). `'apr'` confirms
   approvals.

Versions are permanent history: no state transition deletes or archives
anything, and submitting one candidate never touches its siblings.

Related: `ag-ids-and-refs.md` for `entity_ref` vs `entity_id`,
`ag-generation.md` for producing the media in the first place.
