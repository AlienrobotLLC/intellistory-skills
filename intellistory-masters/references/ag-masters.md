<!-- Generated from the IntelliStory agent knowledge base (article `ag-masters`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Shot Masters — HDR Movies and EXR Sequences

A **master** is a full-quality deliverable attached to a shot: an HDR movie
(ProRes 4444 XQ, HEVC PQ/HLG, DNxHR…) or an image sequence (EXR first; DPX,
TIFF, PNG work too). It is **not** a generation and never becomes a take.

Three things are always true about a master, and you must keep them straight
when you talk about it:

1. **The bytes are stored exactly as uploaded** in an S3-compatible bucket
   (IntelliStory's own, or the workspace's bring-your-own bucket). Downloads
   are byte-exact originals.
2. **What the web player shows is a tone-mapped SDR preview** rendered by us.
   HDR (PQ/HLG) is tone-mapped; linear EXR is put through a display transform.
   **It is not the HDR image and it is not the grade.** Say so whenever you
   describe or link a preview. `dynamic_range` tells you which case you're in:
   `hdr_pq | hdr_hlg | dolby_vision | linear` all mean "the browser is not
   showing the real thing"; `sdr` means the preview is a plain transcode.
3. **Specs come from the file, not from anyone's memory** — codec, profile,
   bit depth, primaries, transfer, mastering-display metadata for movies;
   resolution, channels, pixel type, compression, frame range for EXR. Read
   them from `get_shot_master` before answering "what is this?".

## The upload contract (bytes never cross MCP)

```
init_master_upload   → master_id + one presigned PUT URL per file (2 h)
   PUT each file's bytes to its URL (Content-Type from the response headers)
finalize_master_upload → verifies every object; starts processing
   status: processing → ready (preview_url appears) | failed (see processing.error)
```

- **Movies**: exactly one file. **Sequences**: many files, one extension,
  frame numbers parsed from names like `shot_010.1001.exr` / `plate_0042.dpx`.
  Gaps are reported as warnings, not errors.
- Per-file cap in this release: **5 GB** (single presigned PUT). Larger single
  files → tell the user the multipart path is coming; suggest splitting.
- `declared.colorspace` matters for EXR: `acescg`, `aces2065`, `linear_rec709`,
  `linear_rec2020` pick the matrix used for the preview. If the header has
  chromaticities we guess; the declaration wins. Wrong guess → `reprocess_master`
  after `init` with the right declaration next time (or accept the preview is
  only a preview).
- `source_file_id` links the master to the take it finals; optional but useful.
- Anyone with project access can attach masters; deleting one needs the
  uploader, a project owner, or a workspace admin.
- If `init` answers `storage_not_configured` the server has no masters bucket;
  `masters_cap` means the workspace hit its allowance on the platform store —
  point them at Workspace → Integrations to connect their own bucket.

## Reading a master back

- `list_shot_masters` — status per master + `preview_url` / `thumb_url` (1 h).
- `get_shot_master` — files, spec, `processing.recipe` (the exact tone-map
  used), `processing.warnings` (gaps, deep/multipart EXR = no preview, untagged
  transfer).
- `get_master_download` — presigned attachment URLs for all files or one
  `file_id`; **use GET, not HEAD** (signatures are method-bound). For sequences
  point users at `intellistory masters pull <master_id>` rather than a
  hundred links.

## What to say

- "The player shows an SDR preview; the master itself is HDR (PQ, 10-bit,
  BT.2020) — download it to see it properly." Never imply the browser image is
  representative of the HDR grade.
- For EXR: "linear ACEScg, half float, PIZ, frames 1001–1240 — the preview is a
  Rec.709 tone-map, not your look."
- Deep/multipart EXR: stored, no browser preview, download works.

Related: `ag-versions` (takes are a different thing), `ag-workspaces` (BYO
bucket lives under workspace integrations).
