<!-- Generated from the IntelliStory agent knowledge base (article `ag-recipes`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Recipes — Common Jobs End to End

## Start of any session on an existing project

```
list_projects                → find it
read_cortex                  → whole project state
read_story_core              → what it's for
```

Then say something that shows you read it. "Good to see you back on Ashfall —
last time you were working the second-act turn." Not "how can I help?"

## Script → storyboarded sequence

```
1. read_document(script)          confirm there IS a script
2. run_structure_analysis         acts / scenes / structure — poll read_big_picture
3. run_pipeline                   breakdown → layout → boards, on a DEDICATED BRANCH,
                                  approval_mode "human" so the user sees each stage
   — or by hand: create_sequence / create_shot, then run_layout_pipeline (pass the
     user's direction), then generate_image per shot (shot_id) — or
     regenerate_sequence to board a whole sequence at once (estimate_only first)
4. merge_branch                   when the user approves what the pipeline made
5. invalidate_cortex              once, at the end
```

The app's own Breakdown / Storyboard buttons are not MCP tools — over MCP the
breakdown happens inside `run_pipeline` (safely, on its branch) or by creating
sequences and shots yourself. Either way, show the user the sequence/shot list
before boarding it: a breakdown they didn't agree to is a breakdown they'll want
undone, and it replaces existing shots.

## Add a character and keep them consistent

```
register_element(type: character, canonical_name: "CLEO")   → @C1, use it verbatim
create_asset(entity_type: character)
write_asset_profile                                          → who they are
generate a character sheet on the asset
then generate shots FROM the sheet as refs — not from prompt description
```

The sheet is the consistency mechanism. Prompt text drifts between renders; a
reference doesn't.

## Retrofit Sources onto old versions

A version made from references but missing `refs` shows no Sources strip. Fix it
in place:

```
get_shot_files                            → find the version
update_generated_file(refs: [...])        → same version, Sources appear
```

Don't create a new version to fix metadata on an old one — it clutters the rail
and the user loses the render they approved.

## Author a Loom thread by hand

```
write_loom_thread(project_id, thread_key)   ← bare call, returns the brief
write_loom_thread(… prose + content …)      ← now write it properly
```

Tell the user a later spin will overwrite it.

## Deliver a link to something

```
create_share_link          → members-only, permission-gated
```

Not a raw media URL. And for a specific version, the `app_url` on the version
pins that version rather than landing on whatever is newest.

## Before an expensive batch

```
get_billing_status
→ tell the user the rough cost
→ wait for them to say yes
```

Related: `ag-pipelines.md`, `ag-versions.md`, `ag-billing.md`, `ag-failure-modes.md`.
