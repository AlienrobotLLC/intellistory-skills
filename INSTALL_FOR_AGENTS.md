# Install for Agents

You are an AI coding agent. The user asked you to set up IntelliStory. Follow this exactly.

## Step 1 — the CLI

```bash
npm i -g intellistory
```

Needs Node 18+. Verify: `intellistory version` → `intellistory 0.x.y (…) built …`.

## Step 2 — sign in

Ask the user to run, in their own terminal:

```bash
intellistory login
```

It opens the browser; they pick a workspace and click Authorize. On a headless machine they
can paste a workspace key instead: `intellistory login --with-key <key>` (Settings → API keys
in the app). Wait for them to confirm. Never ask a user to paste a key into the chat.

Verify: `intellistory whoami` → API, key prefix, projects visible.

## Step 3 — pick a project

```bash
intellistory projects
intellistory use <code|name|id>
```

## Step 4 — the skills

Preferred, cross-agent:

```bash
npx skills add AlienrobotLLC/intellistory-skills
```

Alternatives: `gh skill install AlienrobotLLC/intellistory-skills` (GitHub CLI 2.90+);
Claude Code: `/plugin marketplace add AlienrobotLLC/intellistory-skills` then
`/plugin install intellistory@intellistory`; or clone this repo and point your agent at a
skill directory.

## Step 5 (optional) — MCP instead of / as well as the CLI

`intellistory setup <claude-code|claude-desktop|cursor|codex|gemini|hermes|openclaw>` writes
the agent's MCP config in one go (`--print` to preview). Over MCP the tools have the same
names as the CLI commands; the skills work either way.

## Verify

`intellistory list_shots --json | head -c 200` returns JSON. `intellistory help generate_image`
lists options. If a command says "not signed in", go back to Step 2.
