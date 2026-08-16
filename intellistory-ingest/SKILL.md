---
name: intellistory-ingest
version: 0.1.0
description: |
  Bring existing material INTO an IntelliStory project: a Final Draft or Fountain script,
  documents written elsewhere, mood boards and reference images, artwork, character
  turnarounds, footage and renders (as versions on shots), audio, or a whole archive of
  another production (create project/series → sequences → shots → versions with prompts,
  refs and metadata). Use when the user says "import my script", "upload these references",
  "put these renders on the shots", "ingest this folder", "mirror this project into
  IntelliStory", "add this footage". Local files need the CLI; URLs work over MCP too.
argument-hint: "[project] <path or url…> [--as script|refs|shot <code>|character <name>|archive]"
allowed-tools: Bash
---

# IntelliStory — Ingest

Get what exists into the project correctly the first time: right entity, right code, right
`pass_type`, provenance recorded. Files on disk need the CLI (MCP can't see your disk);
anything with a URL can go over MCP. See `intellistory-cli` for reaching tools.

## Where does it go?

| You have | It becomes | How |
|---|---|---|
| Final Draft `.fdx` | script + characters + scene structure | `import_fdx(project_id, fdx_content, write_characters, write_script)` |
| Fountain / plain screenplay, treatment, outline, notes | a project document | `write_document(type, content)` / `create_project_document` / `upsert_project_document` |
| mood boards, film stills, textures, art | project references | CLI `intellistory upload <files> --ref --category mood --tags …` · MCP `upload_reference` (URL or base64) · `bulk_upload_references` up to 100 (auto-attach by `char:/location:/prop: <name>` tag or filename stem) |
| a character/environment image | attached to that asset | CLI `--character "Cleo"` / `--location "Docks"` · MCP `upload_reference(attach_to)` |
| a render / still / clip for a shot | a **version** on the shot | CLI `intellistory upload take.mp4 --shot sq010_sh0020 --pass-type previz` · MCP `upload_media` (base64; `upload_image` for stills on older servers) · `insert_generated_file` when it already has a URL |
| audio (score, temp track, VO) | an audio track / a `vo` version | `insert_audio_track(project_id, shot_id?, name, url)` · a version with `pass_type: vo` |
| a whole production archive | project → sequences → shots → versions | see below |

`pass_type` is the production STAGE: images `storyboard` / `concept`; video `previz` /
`animatic` / `final`; audio `vo`; 3D `model3d`. Wrong ones are silently auto-corrected.

## Local files (CLI)

```bash
intellistory use HGRD
intellistory import_fdx --fdx-content "$(cat script.fdx)" --write-characters true --write-script true
intellistory upload ./refs/*.jpg --ref --category mood --tags night,rain
intellistory upload ./cleo/*.png --character "Cleo"
intellistory upload ./renders/sq010_sh0020_v3.mp4 --shot sq010_sh0020 --pass-type previz --description "H3 test, locked 360"
```

Files are ≤ 36 MB each for now — re-encode a long clip rather than reaching for an
external host. Video must carry its audio already (`ffmpeg … -c:v copy out.mp4`); the
platform plays the bytes you send. Then cite what a version was made from:
`update_generated_file(file_id, refs: [{url, type, usage}])` — `refs` is the only thing that
renders the Sources strip. Put observations in `notes`, prompt/model/cost in `metadata`.

## Structure that doesn't exist yet

- `create_project(name, code, description)` — bootstraps premise/outline/beats/script/
  characters/locations documents. Series: `convert_to_series` then `add_episode` per episode.
- `create_sequence(project_id, name, code?)`, `create_shot(project_id, sequence_id, name,
  description, duration_sec)`. Omit `code` and it derives (`sq043` → `sq043_sh0010`); pass one
  to keep an archive's numbering.
- `create_asset(project_id, name, asset_type: character|environment|prop|style, variant_of?)`
  and `write_asset_profile` for what's known about it; `register_element` for the @-ID a
  script will cite. Character/location/motif @-IDs number across a whole series.
- `create_project_snapshot(project_id, label)` before a large ingest into a project that
  already has work — cheap insurance.

## Ingesting an archive (another production, a folder of a finished film)

Work top-down and keep the archive's own identifiers:

1. Project (or series + episodes) with a short code; `update_project(settings)` for fps /
   resolution / format.
2. Assets first — characters, environments, props — with their reference images attached
   (bulk upload with `char:/location:` tags does the matching for you).
3. Sequences and shots in cut order (`sort_order`, `cut_order`), the archive's codes as `code`,
   a 1–3 sentence `description` per shot (it seeds prompts — not a dump of metadata).
4. Versions per shot: `insert_generated_file` for hosted media (URL) or CLI `upload` for
   local files — `pass_type` by stage, `metadata.prompt` / `model` / `cost` when known,
   `refs[]` for what it was made from, `notes` for anything else worth keeping. Human
   filenames survive on the version — pass `filename`.
5. The edit: `insert_shot_frame` for sub-shot frames of an animatic; `export_animatic` to
   check the cut.
6. `invalidate_cortex` once at the end; then `read_cortex` and spot-check a few shots
   (`get_shot_files`) — the file exists **and** shows where the user expects it.

Batch in tens, not thousands: check `get_shot_files` on the first few before doing the rest.

## Behaviour

- Never guess a UUID; pass codes (`entity_ref`, `--shot <code>`) and let the platform resolve.
- Don't read a media file into your own context to upload it — stream it (the CLI does).
- Say what landed and where — "12 references, 3 versions on sq010_sh0020" — with `app_url`s.

## References

- `references/ag-versions.md` — pass_type, refs, notes, upload vs insert
- `references/ag-ids-and-refs.md` — codes vs UUIDs vs @-IDs
- `references/ag-failure-modes.md` — silent no-ops when a file "doesn't show up"
