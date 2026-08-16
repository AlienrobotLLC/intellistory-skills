# IntelliStory Skills

Agent skills for [IntelliStory](https://intellistory.net) — the visual-storytelling platform for
film, TV and animation pre-production: story development, look dev, storyboards, layout, AI
image/video generation and review.

Skills are plain Markdown (`SKILL.md` + `references/`) that teach an AI coding agent when and how
to use IntelliStory. They work in **Claude Code, Codex, Cursor, OpenClaw, Hermes, Gemini CLI**
and any other agent that loads Markdown skills — no MCP required, though they use it when it's there.

## Install

```sh
npm i -g intellistory                                # the CLI the skills drive
intellistory login                                   # browser sign-in (or --with-key for headless)
npx skills add AlienrobotLLC/intellistory-skills     # this repo → your agent
```

Also: `gh skill install AlienrobotLLC/intellistory-skills`, or in Claude Code
`/plugin marketplace add AlienrobotLLC/intellistory-skills`.

## Connect an agent over MCP (works today)

Any MCP-capable agent can use IntelliStory directly with a workspace key from the app:

```
https://api.intellistory.net/api/mcp
Authorization: Bearer <your workspace key>
```

Claude Code: `claude mcp add intellistory --url https://api.intellistory.net/api/mcp --header "Authorization: Bearer <key>"`

## The skills

| Skill | Use it when the user wants to… |
|---|---|
| [`intellistory-cli`](intellistory-cli/SKILL.md) | reach IntelliStory at all — install, sign in, pick a project, call **any** tool, move files in and out, connect an agent. The others assume it. |
| [`intellistory-story`](intellistory-story/SKILL.md) | develop a story: premise, characters, world, structure, beats, a tagged script |
| [`intellistory-critique`](intellistory-critique/SKILL.md) | honest notes: Good / Bad / Ugly, unpaid setups, subtext, a simulated test screening |
| [`intellistory-look`](intellistory-look/SKILL.md) | define the visual language: brief, palette, references, look variants, character sheets |
| [`intellistory-boards`](intellistory-boards/SKILL.md) | go from script to boards: structure → breakdown → layout → beats/dialogue → boards → animatic |
| [`intellistory-generate`](intellistory-generate/SKILL.md) | render images, video, voice — estimate first, references, versions, the review loop |
| [`intellistory-mentor`](intellistory-mentor/SKILL.md) | be guided through a whole production on a saved ten-phase plan |
| [`intellistory-ingest`](intellistory-ingest/SKILL.md) | bring in scripts, references, renders, audio, or a whole archive |

Each skill names the tools it uses. Over MCP you call them directly; from a shell,
`intellistory <tool> --option value` is the same call — the CLI reads the live tool registry,
so nothing here goes stale when a tool is added.

Agents: read [`INSTALL_FOR_AGENTS.md`](INSTALL_FOR_AGENTS.md). Golden prompts:
[`evals/scenarios.md`](evals/scenarios.md).

## Layout

```
<skill-name>/
  SKILL.md          # frontmatter (name, description, version, allowed-tools) + instructions
  references/       # ag-*.md — generated from IntelliStory's agent knowledge base (don't edit here)
evals/              # golden prompts per skill
.claude-plugin/ .codex-plugin/ .cursor-plugin/    # plugin manifests
```

## Contributing

Issues and PRs welcome. This repo is public: **never commit keys, tokens or anything from a
private IntelliStory deployment.** CI greps every push for common secret shapes.

## License

MIT © Alienrobot LLC
