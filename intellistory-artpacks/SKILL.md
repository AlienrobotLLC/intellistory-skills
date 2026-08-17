---
name: intellistory-artpacks
version: 0.1.0
description: |
  Work with production Art Packs in IntelliStory — the per-scene folders of cleared reference
  material a studio delivers into its own bucket. Coverage per sequence (which scenes still lack a
  pack), sync/index from the studio's bucket, browse a pack's files, pull originals, and — the
  point — use a sequence's cleared pack as references when boarding or rendering its shots. Use
  when the user says "art pack", "art packs", "cleared pack", "temp pack", "which sequences are
  missing a pack", "sync from the client bucket", "use the art pack as refs".
argument-hint: "[coverage | sync | show <id> | pull <id> --category image] [--project P]"
allowed-tools: Bash
---

# IntelliStory — Art Packs

An **art pack** is production's per-scene bundle of cleared reference material (concept art,
stills, graphics, design files) that lives in the studio's own bucket. IntelliStory **indexes it
in place** — originals never move — and models each pack as an **asset of type `art_pack`
attached to the shots of its sequence**. So it behaves like any asset: `get_shot_assets`,
`get_asset_files`, ref slots, `search_images` all already see it.

## Verbs (MCP names; CLI is `intellistory artpacks …`)

| Task | Tool |
|---|---|
| which sequences have a pack, which don't | `art_pack_coverage(project_id)` · CLI `artpacks coverage` |
| list packs (project / one sequence) | `list_art_packs` · `artpacks list [--sequence S]` |
| one pack: files by folder/category, sequence, shots attached | `get_art_pack` · `artpacks show <id>` |
| index new packs / new files from the studio bucket | `sync_art_packs(project_id)` · `artpacks sync` |
| a pack landed in *Unassigned* | `map_art_pack(art_pack_id, sequence_id)` · `artpacks map` |
| download originals (presigned 1 h, from the studio bucket) | `get_art_pack_download` (all / `file_id` / `category:"image"`) · `artpacks pull <id> --category image` |
| connect the studio bucket (workspace admin, read-only key) | `set_org_source` → `test_org_source` |

## Status is IntelliStory's, not production's

Production folders are named `Temp_Art_Pack` / `Cleared_Art_Pack` / `Updated_Art_Pack`; that word
is kept only as `source_status_hint`. The pack's `status`:
`wtg` none yet · `ip` temp only · `app` cleared/updated on hand · `fin` locked · `archived`
superseded. Say "no art pack yet", "temp only", "cleared pack on hand".

## Using a pack for generation

1. `get_shot_assets(shot_id, include_latest_approved: true)` — the pack is in the cast.
2. Pick 1–3 stills that carry the look/props/location; pass them as refs to `generate_image`
   / `generate_video` (see `intellistory-generate`). Prefer `app` packs over `ip`.
3. Say which pack and stills you used.

## How discovery works (for explaining oddities)

`sync_art_packs` walks the workspace's production-source prefix
(`Scene_Bin/` for KS): `<NNN>_(<ABC>)_<NAME>/<*_locations>/From_Prod/<pack folder>`. Folders
whose name says *art pack* become packs; `Lidar`, `Photogrammetry`, `Scans`, `Design_WIP` are
skipped on purpose. Mapping is by **letters first (`ABC` = sequence code), then number**;
unmatched → *Unassigned*. Re-running is safe (new files added, vanished files flagged, a cleared
pack archives that sequence's temp pack). Only images/video get thumbnails and become asset
files; PSD/AI/PDF/OBJ are catalogued with a download link.

## Behaviour

- Reading the studio bucket is **their** egress; the first sync reads each image once. Say so
  if asked why it took a while.
- Presigned links expire in an hour; thumbnails are ours.
- Deleting a pack in IntelliStory removes catalogue/asset/thumbnails only — never their files.

## References

- `references/ag-art-packs.md` — the contract, statuses, discovery
- `references/ag-generation.md` — using references in generation
- `references/ag-workspaces.md` — production source credential lives under workspace integrations
