# CLAUDE.md — IntelliStory Skills

## What this is

Eight skills that drive IntelliStory — pre-production for film, TV and animation — through
the [`intellistory` CLI](https://www.npmjs.com/package/intellistory) or the IntelliStory MCP.

```
intellistory-cli       →  how to reach the platform: install, login, projects, ANY tool, upload/download, setup <agent>
intellistory-story     →  Story Mentor: premise → characters → structure → tagged script
intellistory-critique  →  Critique Partner: Good/Bad/Ugly from the Loom, Big Picture, test screening
intellistory-look      →  Look Dev: brief, category prompts, references, variants, sheets
intellistory-boards    →  structure → breakdown → layout → beats/dialogue → boards → animatic
intellistory-generate  →  image/video/audio with estimate-first, refs, versions, review loop
intellistory-mentor    →  Production Mentor on the shared ten-phase plan
intellistory-ingest    →  scripts, references, renders, audio, whole archives → the project
```

## Layout

```
<skill>/SKILL.md          frontmatter (name, description, version, allowed-tools) + instructions
<skill>/references/       ag-*.md — GENERATED from the IntelliStory agent knowledge base; do not edit here
evals/scenarios.md        golden prompts + expected behaviour per skill
.claude-plugin/ .codex-plugin/ .cursor-plugin/   plugin manifests
```

## Rules for this repo

- It is public. No keys, no internal system names, no provider routing, no pipeline
  mechanics. CI runs a secret scan on every push; the reference build runs a denylist.
- `references/ag-*.md` are build outputs — edits go to the knowledge base upstream and are
  regenerated. Edit `SKILL.md` freely.
- Every tool named in a SKILL.md must exist on the platform (`intellistory tools`).
- Product names (Cortex, the Loom, Story Core, Big Picture, IntelliBeat) are fine to say;
  how they work inside is not something a skill describes.
