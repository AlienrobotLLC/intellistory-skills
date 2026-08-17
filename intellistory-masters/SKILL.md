---
name: intellistory-masters
version: 0.1.0
description: |
  Attach full-quality masters to IntelliStory shots — HDR movies (ProRes 4444/422, HEVC
  PQ/HLG, DNxHR) and image sequences (EXR, DPX, TIFF, PNG) — and get them back byte-exact.
  Files go straight to object storage through presigned URLs (never through the API or MCP),
  the platform renders a tone-mapped SDR preview for the web player, and every spec (codec,
  bit depth, primaries, transfer, HDR metadata, EXR channels/compression, frame range) is read
  from the file. Use when the user says "attach the master", "put the EXRs on the shot",
  "upload the HDR render", "download the original", "pull the sequence", "what is this
  master", or asks why the browser preview looks different from their grade.
argument-hint: "[push <shot> <file|glob|dir> | pull <master_id> | list <shot>] [--colorspace acescg] [--fps 24] [--label L]"
allowed-tools: Bash
---

# IntelliStory — Masters (HDR movies, EXR sequences)

A **master** is the real deliverable on a shot. Three facts govern everything here:

1. **Bytes are stored exactly as uploaded** and come back byte-exact. Nothing re-encodes them.
2. **The browser cannot show HDR.** What the IntelliStory player shows for an HDR or linear
   master is a **tone-mapped SDR preview** the platform rendered — not the HDR image, not
   the grade. Say so every time you describe or link a preview. `dynamic_range` tells you:
   `hdr_pq | hdr_hlg | dolby_vision | linear` → the preview is not the real thing;
   `sdr` → the preview is a plain transcode.
3. **Specs come from the file** (`get_shot_master`), not from memory or the filename.

Masters are **not takes**: they never appear in Versions, review, or animatics.

## Push (attach) — CLI, the normal way

```
intellistory masters push <shot> ./renders/sq010_sh0010.####.exr --colorspace acescg --fps 24 --label "v003 final comp"
intellistory masters push <shot> ./delivery/sq010_sh0010_hdr.mov --label "HDR10 master"
intellistory masters push <shot> ./exr_folder/            # every exr/dpx/tif/png in the folder
```

`<shot>` is a shot code (with a project selected via `intellistory use <project>` or
`--project`) or a shot UUID. What happens: the CLI hashes each file (sha256), asks the
server for **one presigned PUT per file** (`init_master_upload`), streams the files straight
to the bucket in parallel, then `finalize_master_upload` verifies every object by size and
starts processing. It prints the `master_id`; `intellistory masters list <shot>` shows the
status flip `processing → ready`.

Rules the server enforces (say them rather than fight them):
- One master = **one movie** or **one image sequence** (all files one extension). Frame
  numbers are parsed from names like `shot.1001.exr`, `plate_0042.dpx`; gaps are reported as
  warnings, not refused. Un-numbered images are ordered alphabetically.
- **5 GB per file** in this release (single presigned PUT). Larger single files: tell the
  user the multipart path is coming; suggest splitting/downscaling for now.
- `--colorspace` matters for EXR: `acescg`, `aces2065`, `linear_rec709`, `linear_rec2020`
  choose the matrix used for the preview. If the header has chromaticities the platform
  guesses; a declaration wins. Movies are read from their own tags (PQ/HLG/primaries).
- `--take <file_id>` links the master to the take it finals (optional, useful).
- Anyone with project access can push; deleting needs the uploader, a project owner, or a
  workspace admin.
- `storage_not_configured` → the server has no masters bucket; `masters_cap` → the workspace
  hit its allowance on IntelliStory's store — point them at Workspace → Integrations →
  Buckets to connect their own bucket (AWS S3 / B2 / Wasabi) with the **Masters** role on
  (`connect_org_bucket` roles.masters for agents; one key per bucket, roles Source/Masters/Export).

## Push over MCP only (no shell) — the contract

```
init_master_upload(shot_id, files:[{filename,size,sha256?}], label?, source_file_id?, declared:{colorspace?,fps?})
  → master_id + files[{file_id, filename, put_url, headers}]      (URLs valid 2 h)
PUT each file's bytes to its put_url with the given headers        ← whoever has the bytes does this
finalize_master_upload(master_id, files?:[{file_id, sha256}])
  → { ok, verified, total }  |  { ok:false, missing[], mismatch[] }  → re-PUT those, call again
```

Bytes never travel through MCP or the API. If you have a shell, `curl -X PUT -H
"Content-Type: …" --data-binary @file "<put_url>"` works; without one, hand the URLs to the
user (or use the CLI). Cancelled or abandoned uploads are cleaned up after 24 h; a half-open
master can also be `delete_shot_master`ed.

## Read

```
list_shot_masters(shot_id)   → status, kind, format, dynamic_range, preview_url / thumb_url (1 h), usage
get_shot_master(master_id)   → files, full spec + color block, processing.recipe, processing.warnings
```

CLI: `intellistory masters list <shot>` · `intellistory get_shot_master --master_id <id>`.

The **spec line** to give a user (from `get_shot_master`):
- Movie: `ProRes 4444 XQ · 3840×2160 · 24 fps · 12-bit · BT.2020 PQ · HDR10 · 1.2 GB` —
  from `format`, `width/height`, `fps`, `color.bit_depth`, `color.primaries/transfer`,
  `dynamic_range`, `total_bytes`. `color.mastering_display`, `max_cll`, `max_fall` exist for
  HDR10.
- Sequence: `OpenEXR · 4096×1716 · 240 frames [1001–1240] · half RGBA · PIZ · ACEScg linear`
  — from `format`, `frame_count/frame_start/frame_end`, `color.pixel_format`,
  `color.channels`, `color.compression`, `color.declared_colorspace`.
- `processing.recipe` is the exact tone-map used for the preview (e.g.
  `pq→linear(npl100)→bt709→hable→bt709`, `linear AP1→709→hable→bt709`); `warnings` carry
  gaps, deep/multipart EXR (stored, no preview), untagged transfer.

## Pull (download) — byte-exact

```
intellistory masters pull <master_id> -o ./out --verify      # every file, parallel, sha256-checked
```

Over MCP: `get_master_download(master_id[, file_id])` → presigned attachment URLs (1 h) for
all files or one. **Use GET, not HEAD** (signatures are method-bound). For sequences prefer
the CLI over a hundred links.

## Fix / remove

- `reprocess_master(master_id)` re-runs probe + preview (after a failed run, or when the
  declared colour space was wrong — re-`init` with the right one is cleaner). Originals are
  untouched.
- `delete_shot_master(master_id)` removes the files and the preview. Confirm first; not
  reversible. CLI: `intellistory masters delete <id>`.

## What to say

- "The player is showing an SDR preview; the master itself is HDR (PQ, 10-bit, BT.2020) —
  download it to see it properly." Never imply the browser image represents the HDR grade.
- For EXR: "linear ACEScg, half float, PIZ, frames 1001–1240 — the preview is a Rec.709
  tone-map, not your look."
- Deep/multipart EXR: stored, no browser preview, download works.
- Report outcomes, not plumbing: "attached the 240-frame EXR to sq010_sh0010 (5.8 GB), preview
  ready" — not the presigned URLs or the ffmpeg chain.

## References

- `references/ag-masters.md` — the full contract, warnings, what to say
- `references/ag-versions.md` — takes vs masters (masters are not versions)
- `references/ag-workspaces.md` — bring-your-own bucket lives under workspace integrations
