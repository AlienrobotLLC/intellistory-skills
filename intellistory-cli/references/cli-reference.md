<!-- Generated from `intellistory --help` (intellistory 0.1.0 (dev)). Do not edit here. -->
# intellistory CLI — command reference

```
intellistory 0.1.0 — IntelliStory from the terminal

Usage:
  intellistory <command> [options]
  intellistory <tool> [--option value …]     any IntelliStory tool, e.g. `intellistory list_shots --limit 20`
  istory …                                    (alias)

Commands:
  login [--with-key <key>]               Sign in — opens the browser; `--with-key <key>` for headless machines
  logout                                 Forget the stored credential
  whoami                                 Show the current credential, API and default project
  projects [--archived]                  List projects you can reach
  use <project> | --clear                Set the default project (by code, name or id)
  tools [--search q]                     Browse every IntelliStory tool you can call (`intellistory <tool> …`)
  help <tool>                            Show options for a tool: `intellistory help <tool>`
  call <tool> [options]                  Call any tool explicitly: `intellistory call <tool> [options]`
  wait <job_id> [--timeout s]            Poll a generation job until it finishes (exit 0 on success, 1 on failure)
  upload <file…> --shot S4-02            Put local renders, stills or audio into a project (as a shot version, on an asset, or as a reference)
  download <file_id|shot> [-o path]      Fetch a generated file (by file id, or the newest version on a shot)
  masters push|pull|list|delete          HDR movies / EXR sequences on shots — direct-to-storage push, verified pull
  artpacks coverage|list|sync|show|pull|map Production art packs — coverage per sequence, sync from the source bucket, browse, pull originals
  open [project] [--shot <code>]         Open a project (or a shot) in the web app — default project if none given
  mcp                                    Run the stdio MCP proxy (for Claude Desktop, Codex, Hermes, …); adds the local stash_pasted_images tool
  setup <agent>                          Connect an agent: claude-code, claude-desktop, cursor, codex, gemini, hermes, openclaw
  skills install [-- <skills args>]      Install the IntelliStory agent skills into your agents (npx skills add AlienrobotLLC/intellistory-skills)
  version                                Print version and build info

Coming:
  import <script.fdx|.fountain>          Import a screenplay
  export animatic                        Export the current cut

Global options:
  --json               machine-readable output (compact JSON)
  --project <id>       project for this call (otherwise the `use` default)
  --api <url>          API base (default https://api.intellistory.net; or INTELLISTORY_API)
  --key <key>          credential for this call (or INTELLISTORY_KEY; else the stored login)
  --refresh            re-fetch the tool registry instead of using the 24h cache
  --no-color           plain output
  -h, --help           this help    ·    -v, --version

Start:  intellistory login --with-key <key>   →   intellistory projects   →   intellistory use <code>
Docs:   https://intellistory.net/cli
```

## Calling any tool

```sh
intellistory tools --search <word>          # find it
intellistory help <tool>                    # its options, required ones marked
intellistory <tool> --option value …        # call it (project_id from `use`, or --project <uuid|code|name>)
intellistory <tool> --args '{"key":"value"}' # raw JSON, or --args - to read stdin
intellistory <tool> … --json                # compact JSON for parsing
```

Options are checked against the tool's schema: numbers, booleans, comma-separated arrays and JSON
objects are coerced; unknown options and missing required ones are reported by name (exit 2).
Exit codes: 0 ok · 1 tool/runtime error · 2 usage · 3 not signed in / unauthorized · 4 network · 5 timeout.
