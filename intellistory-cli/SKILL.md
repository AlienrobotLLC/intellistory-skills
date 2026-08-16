---
name: intellistory-cli
version: 0.1.0
description: |
  How to reach IntelliStory from any agent: the `intellistory` CLI (install, sign in,
  pick a project, call ANY tool with schema-checked options, upload/download media,
  connect Claude/Codex/Cursor/Gemini/Hermes) and the same tools over MCP. Load this
  first when a task mentions IntelliStory, a project code (HGRD, sq010_sh0020), shots,
  storyboards, looks, generation, or "put this render on the shot". Other intellistory-*
  skills assume this one: they name tools; this one says how to call them.
argument-hint: "[login | projects | use <project> | <tool> --option value | upload <file> --shot <code>]"
allowed-tools: Bash
---

# IntelliStory CLI

IntelliStory is a pre-production platform for film, TV and animation: story development,
look development, storyboards, layout, AI image/video/audio generation, review. Everything
lives under a workspace → project → (documents · acts/scenes · sequences/shots · assets ·
references · versions).

## Two ways to call the same tools

Every IntelliStory operation is a **tool** with a name like `list_shots`, `write_beats`,
`generate_image`. There are ~200. You reach them one of two ways:

1. **MCP** — if an `intellistory` MCP server is connected in this session, call the tools
   directly. Same names, same arguments as below.
2. **CLI** — otherwise, run them from the shell. `intellistory <tool> --option value` is
   the tool call; the CLI reads the live tool registry, so anything that exists as a tool
   exists as a command.

Both hit the same platform with the same permissions. Prefer MCP when it's there (no
process spawn); use the CLI whenever it isn't, or for anything that touches local files.

## Step 0 — bootstrap (CLI path)

```bash
intellistory whoami            # signed in? which API? default project?
```

- `command not found` → `npm i -g intellistory` (Node 18+; no other dependencies).
- `Not signed in` → ask the user to run **`intellistory login`** (opens the browser, they
  pick a workspace, done) or, on a headless box, `intellistory login --with-key <key>` with
  a workspace key from Settings → API keys. Wait for them; don't guess keys.
- No default project → `intellistory projects`, then `intellistory use <code|name|id>`.
  From then on every command that takes `project_id` fills it in; `--project <uuid|code|name>`
  overrides for one call.

## Calling a tool

```bash
intellistory tools --search loom                 # find tools by name/description
intellistory help write_shot_prompt              # options, required ones marked, defaults, enums
intellistory list_shots --json                   # call — project_id injected from `use`
intellistory write_shot_prompt --shot-id <id> --prompt "…"          # dashes or underscores, either
intellistory generate_image --prompt "…" --estimate-only --json    # ask the price first
intellistory write_beats --args '{"beats":[…]}'                    # raw JSON for nested input
intellistory write_beats --args - < beats.json                     # …or from stdin
```

Rules the CLI enforces for you (read the error, it names the option):
- options are coerced from the tool's JSON schema — `--limit 5` → number, `--full` → true,
  `--tags a,b` → array, `--settings '{…}'` → object;
- unknown options and missing required ones exit 2 with the option name;
- `--json` gives compact JSON on stdout for parsing; without it you get readable JSON.
- Exit codes: 0 ok · 1 the tool returned an error · 2 usage · 3 not signed in · 4 network · 5 timeout.

Wrap long-running work: `intellistory wait <job_id>` polls a generation job and exits 0 on
success, 1 on failure (`--timeout 900` for long renders).

## Local files ↔ the project (CLI only — MCP can't see the disk)

```bash
intellistory upload ./takes/*.mp4 --shot sq010_sh0020 --pass-type previz     # versions on a shot
intellistory upload ./board.png --shot sq010_sh0020 --pass-type storyboard
intellistory upload ./moodboard/*.jpg --ref --category mood --tags night,rain # project references
intellistory upload ./cleo_front.png --character "Cleo"                       # onto a character
intellistory download sq010_sh0020 -o ./renders/                              # newest version on the shot
intellistory download <file_id> -o ./x.mp4        # a specific version;  --all for every version
```

Uploads are ≤ 36 MB per file for now — for a longer render, upload a shorter or lower-bitrate
encode rather than an external link. Video must have its audio **muxed in** before upload; the
platform plays exactly the bytes you send. `pass_type` is the production STAGE: images →
`storyboard` / `concept`; video → `previz` / `animatic` / `final`; audio → `vo`.

## IDs — the one thing that goes wrong most

Four different things are called an "id" (`references/ag-ids-and-refs.md`). In short:
- **UUID** = the row. Most write tools also accept a **reference code** (`sq010_sh0020`,
  `CHAR_CLEO`) and resolve it for you — pass codes, don't guess UUIDs.
- **@-IDs** (`@S4`, `@C1`) inside scripts are a registry: `register_element` with the canonical
  name and use exactly what comes back; never invent one; never strip them when editing.
- Shots, assets and versions return an `app_url` — give the user that (or `create_share_link`),
  not a raw media URL.

## Connecting an agent (one command each)

```bash
intellistory setup claude-code      # claude mcp add … (user scope)
intellistory setup claude-desktop   # stdio proxy — no key lands in the config
intellistory setup cursor | codex | gemini | hermes | openclaw
intellistory setup <agent> --print  # show what would be written, write nothing
intellistory skills install         # npx skills add AlienrobotLLC/intellistory-skills
```

`intellistory mcp` is the stdio MCP proxy those setups use; it adds one local-only tool,
`stash_pasted_images`, that files images the user pasted into chat as project references.

## Behaviour

- Describe what happened, not the plumbing: "boarded six shots of sq020" — not the model
  routing or the queue. Don't narrate "calling the CLI".
- Money is real: `generate_*` and `regenerate_sequence` take `estimate_only: true` — quote,
  get a yes, then run (`references/ag-billing.md`).
- Read before you write on an existing project: `read_cortex` gives the whole state.
- After a batch of meaningful story/structure changes, call `invalidate_cortex` once.
- When a read comes back empty, check project → entity code → upstream exists → stale
  view, in that order (`references/ag-failure-modes.md`). Don't invent an ID, a price or a
  capability; `list_knowledge_base` / `read_knowledge_article` are authoritative for
  platform questions.

## References

- `references/cli-reference.md` — full command list (generated from `intellistory --help`)
- `references/ag-ids-and-refs.md` — UUIDs vs codes vs @-IDs vs deep links
- `references/ag-failure-modes.md` — the silent no-ops and how to read an empty result
- `references/ag-billing.md` — estimate-first, credits, what to tell the user
