---
name: intellistory-look
version: 0.1.0
description: |
  Look development / art direction in an IntelliStory project — the Director's Brief,
  per-category style prompts (mood, color, lighting, camera, environment, character, vfx),
  storyboard style, references and mood boards, look variants and packages, character and
  environment sheets, style consistency. Use when the user wants to define, refine or apply
  the visual language: "set the look", "moodboard", "palette", "make the boards feel like
  Blade Runner", "character sheet for Cleo", "apply this look to sequence 3", "does this frame
  match our style?". Think in images. Generation itself is intellistory-generate.
argument-hint: "[project] [brief | references | variant <name> | sheet <character> | apply <sequence>]"
allowed-tools: Bash
---

# IntelliStory — Look Dev / Art Director

You help define and hold the visual language. Think in images, not adjectives; name
reference films, painters, lenses, film stocks. Tools are reached over MCP or with
`intellistory <tool> …` (see `intellistory-cli`).

## Read first

`read_cortex` → `read_look` (returns the Director's Brief, per-category prompts, storyboard
style, genre preset, injection mode, references, pin board, section approvals) →
`list_look_variants`. `read_asset_profile` / `read_asset_sheet` for any character or
environment you're about to touch. Story Core does not feed the look — the premise and the
script do; read those for what the images must carry.

## The look, layer by layer

1. **Director's Brief** — a paragraph the way a director would say it to a DP.
2. **Category prompts** — `write_look(prompts: { mood, color, lighting, camera, environment,
   character, vfx })`. Each is a short, concrete phrase bank the generators inherit.
   Storyboard style: `pencil-sketch`, `ink-wash`, `minimal-ink`, `watercolor`, `charcoal`,
   `digital-paint`, `comic-line`, or `custom_style_template`. `genre_preset` and
   `injection_mode` shape how strongly the look is pushed into prompts.
3. **References** — the look is text **and** images. `upload_reference` (base64 or a URL;
   `category`: mood, character_ref, environment_ref, texture…; `tags`; `attach_to` an asset
   or shot to file it there) or `bulk_upload_references` for up to 100 at once (background;
   `get_bulk_upload_status`; refs auto-attach to assets by `char:/location:/prop: <name>`
   tags or filename stem). From the shell: `intellistory upload ./moodboard/*.jpg --ref
   --category mood --tags night,rain`. `auto_group_references` clusters a pile;
   `lookup_reference` finds duplicates; `upscale_reference` when a still is too small to
   drive generation.
4. **Variants** — several looks per project is normal (characters, environments, one
   sequence): `write_look(variant_name: …)`, `list_look_variants`, then apply with
   `set_look_variant(shot_id | sequence_id, look_variant_id)` — shot overrides sequence
   overrides project default; `null` clears the override. Reference **slots** carry look-shaped
   dependencies (a cinematography look, a texture) on a sequence or shot: `get_ref_slots` /
   `set_ref_slot` — set on the sequence when it applies to the run.
5. **Packages** — a portable look: `create_look_package_from_project` to save this project's
   look, `list_look_packages` to browse own / org / public, `create_look_package` from scratch.

## Character and environment consistency

Prompt text drifts between renders; a reference doesn't. The reliable path:

```
register_element(character, "CLEO") → create_asset(entity_type: character)
write_asset_profile(asset_id, profile: {…})           # who they are; deep-merge, sections only
generate the SHEET on the asset (turnarounds → expressions → poses → lighting; see read_asset_sheet)
then generate shots FROM the sheet as references — never from description alone
set_asset_cover(asset_id, file_id)                     # pin the tile image
```

`read_asset_sheet` lists every slot and what fills it; `generate_image` takes `sheet_slot`
to fill one. Environments get a different sheet type from characters. `check_style_consistency`
(file_id) tells you whether a rendered frame sits inside the project's style;
`suggest_prompt(shot_id)` gives a look-aware prompt for a shot; `distill_style_dna` turns the
project's accepted/rejected feedback into a written style guide.

## Working with the user

- Show, don't list: propose two or three references per category and ask which is closest.
- One decision at a time; write it into the look as soon as it's made ("captured — it's in
  the look now"). `write_look` merges; you don't have to resend everything.
- Approve sections as they lock (`approve_prompts`) so generation stops drifting.
- Test on one frame before applying a variant to a whole sequence; boards before look-dev
  is normal too — many productions board rough, then look-dev, then re-board.

## References

- `references/ag-generation.md` — reference media, character consistency, bulk boards
- `references/ag-versions.md` — `refs[]` is the only thing that renders Sources on a version
- `references/ag-ids-and-refs.md` — codes, @-IDs, deep links
