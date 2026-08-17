<!-- Generated from the IntelliStory agent knowledge base (article `ag-art-packs`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Art Packs — Production Reference Bundles

An **art pack** is production's per-scene folder of cleared reference material —
concept art, cleared stills, graphics, design files — that lives in the client's
own bucket. IntelliStory **indexes it in place** (never copies the originals) and
models each pack as an **asset of type `art_pack`** attached to the shots of its
sequence. That is the whole trick: an art pack behaves like any other asset, so
the tools you already know do the work.

| You want to… | Use |
|---|---|
| see which sequences have a pack and which don't | `art_pack_coverage(project_id)` |
| list packs (project or one sequence) | `list_art_packs` |
| read one pack: files, subfolders, categories, sequence, shots attached | `get_art_pack` |
| get the pack's stills as references for a shot | `get_shot_assets(shot_id, include_latest_approved: true)` — the pack is in the cast; or `get_asset_files(asset_id)` |
| download originals (presigned, 1 h, from the client bucket; GET not HEAD) | `get_art_pack_download` (all / `file_id` / `category: "image"`) |
| pull new packs / new files from the client bucket | `sync_art_packs(project_id)` |
| fix a pack that landed in *Unassigned* | `map_art_pack(art_pack_id, sequence_id)` |
| connect the client bucket (workspace admin — once, one key) | Workspace → Integrations → **Buckets** with the **Source** role on (prefix e.g. `Scene_Bin/`); agents: `connect_org_bucket({ roles: { source: { enabled: true, prefix } } })` → `test_org_bucket`. Never re-paste a stored key — `sync_art_packs` finds the bucket by its Source role. (`set_org_source` still works as a wrapper.) |

## Status is ours, not production's

Production names folders `Temp_Art_Pack`, `Cleared_Art_Pack`, `Updated_Art_Pack`.
We keep that word only as `source_status_hint`. The pack's `status` uses the
pipeline codes everything else uses:

| status | meaning |
|---|---|
| `wtg` | the sequence has no pack yet (coverage rows only) |
| `ip` | only a temp pack on hand |
| `app` | a cleared / updated / plain pack is on hand |
| `fin` | locked (manual) |
| `archived` | superseded by a newer pack on the same sequence — still browsable |

Say "no art pack yet" / "temp only" / "cleared pack on hand" — not production's words.

## How discovery works (so you can explain oddities)

`sync_art_packs` walks the workspace's **production source** prefix
(`Scene_Bin/` for KS): `<NNN>_(<ABC>)_<NAME>/<*_locations>/From_Prod/<pack folder>`.
Folders whose name says *art pack* (any spelling) become packs; `Lidar`,
`Photogrammetry`, `Scans`, `Design_WIP`, `Additional graphics`, `_archive` are
skipped on purpose (useful, but not what production means by art pack). Each pack
is mapped to a sequence by **letters first (`ABC` = sequence code), then number**;
anything unmatched sits in **Unassigned** until `map_art_pack`. Re-running sync is
safe: new files are added, vanished files flagged `missing`, and a cleared pack
arriving on a sequence archives that sequence's temp pack.

Inside a pack every object is catalogued (`get_art_pack` → `files[]` with
`category`). Only images and video get thumbnails and become asset files (that's
what the shot rail and ref slots read); PSD / AI / PDF / OBJ / LAS are listed with
size and a download link only.

## Using packs for generation

The point of indexing them: **use the cleared pack as references** when boarding
or rendering a shot in that sequence. `get_shot_assets` returns the pack with the
rest of the cast; pick 1–3 stills that carry the look/props/location and pass them
as `image_urls` / ref slots. Prefer `app` (cleared) packs over `ip` (temp) ones and
say which you used.

## Costs and caveats

- Reading the client bucket is **their** egress; thumbnailing a big pack the first
  time reads its images once. Say so if a user asks why the first sync took a while.
- Presigned links expire after an hour; thumbnails are ours and permanent.
- Deleting a pack in IntelliStory removes our catalogue/asset/thumbnails only —
  the client's files are never touched.

Related: `ag-generation` (refs), `ag-workspaces` (connected buckets — Source role),
`ag-masters` (masters are outputs; art packs are inputs).
