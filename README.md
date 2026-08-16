# IntelliStory Skills

Agent skills for [IntelliStory](https://intellistory.net) — the visual-storytelling platform for
film, TV and animation pre-production: story development, look dev, storyboards, layout, AI
image/video generation and review.

Skills are plain Markdown (`SKILL.md` + `references/`) that teach an AI coding agent when and how
to use IntelliStory. They work in **Claude Code, Codex, Cursor, OpenClaw, Hermes, Gemini CLI**
and any other agent that loads Markdown skills — no MCP required, though they use it when it's there.

> **Status: being populated.** The install commands below already work; the first skills land
> here as they're ready. Watch this repo or <https://intellistory.net/cli>.

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

## Planned skills

| Skill | What it's for |
|---|---|
| `intellistory-story` | Develop a story from a spark: premise, characters, structure, beats |
| `intellistory-critique` | Honest, structured feedback on a script or outline |
| `intellistory-look` | Visual language: palette, composition, references, Director's Brief |
| `intellistory-boards` | Breakdown → layout → storyboards |
| `intellistory-generate` | Image / video / audio generation, with cost estimates first |
| `intellistory-mentor` | Guided end-to-end production on a saved plan |
| `intellistory-ingest` | Bring local media, scripts and references into a project |
| `intellistory-cli` | Command reference for the `intellistory` CLI |

## Layout

```
<skill-name>/
  SKILL.md          # frontmatter (name, description, allowed-tools) + instructions
  references/       # supporting docs the skill points the agent at
evals/              # golden prompts per skill
```

## Contributing

Issues and PRs welcome. This repo is public: **never commit keys, tokens or anything from a
private IntelliStory deployment.** CI greps every push for common secret shapes.

## License

MIT © Alienrobot LLC
