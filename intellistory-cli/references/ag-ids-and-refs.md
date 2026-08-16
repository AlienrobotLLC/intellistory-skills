<!-- Generated from the IntelliStory agent knowledge base (article `ag-ids-and-refs`, last edited 2026-08-14). Do not edit here — edit the KB and rebuild. -->
# IDs, @-IDs and Entity References

Four different things in this system are called an "id". Using the wrong one is
the single most common way an agent writes a row that points at nothing.

## The four kinds

| Kind | Looks like | What it addresses |
|---|---|---|
| **UUID** | `9dd55476-…` | the database row. What foreign keys hold. |
| **Reference code** | `sq010_sh0010`, `CHAR_CLEO` | a shot or asset, human-readable and stable |
| **@-ID** | `@S47`, `@C1`, `@L2` | a story element as cited *inside* documents |
| **Short code** | the tail of an `app_url` | a shareable deep link |

Most write tools accept a reference code and resolve the UUID for you.
`insert_generated_file` takes `entity_ref` and fills `entity_id` itself for shots
and assets — pass the ref and let it resolve. Passing a UUID you guessed, or a
ref for an entity that doesn't exist yet, is how you get an orphan row.

**Shot codes inherit their sequence's code.** `create_shot` under a sequence coded
`sq043` generates `sq043_sh0010`, and files the shot under `sq043/`. Sequences the
breakdown creates are numbered in tens (`sq010`, `sq020`); ones you code yourself
or that arrive from an ingest need not be. You don't have to pre-compute either
value — omit `code` and let it derive, or pass one explicitly and it is used as
given.

## @-IDs are a registry, not a naming convention

Scripts in Tagged Fountain and the compressed context formats cite @-IDs
directly — `@S47 [@L2]: @C1 finds his name absent`. Those tags are what link a
line of script to an entity record.

Prefixes: `@S` scene · `@C` character · `@L` location · `@M` motif · `@T` thread ·
`@B` beat · `@H` shot · `@F` frame.

**Never invent an @-ID.** A guessed `@C7` points at whatever character actually
holds 7, or at nothing at all. Call `register_element` with the canonical name
and use exactly what comes back. It's idempotent — the same name always returns
the same ID — so call it freely instead of tracking IDs yourself.

`list_elements` reads the registry, and is how you find out what an ID you found
in a script or a thread actually refers to.

Scope differs by type, and this catches people out on series:

- `character` / `location` / `motif` / `thread` number across a **whole series** —
  `@C1` is the same character in every episode
- `scene` / `beat` / `shot` / `frame` restart **per project**

Names match case- and whitespace-insensitively. IDs never change once assigned.

**When editing a script, never strip `@S` / `@C` tags.** They are not decoration.
Removing them silently unlinks scenes and dialogue from their entity records, and
nothing will error to tell you.

Call `register_element` with just `project_id` for the full brief on types,
scoping and naming.

## Two keys on generated files

`generated_files` rows carry both `entity_id` (the UUID) and `entity_ref` (the
code). Both. They are not alternatives — a row with only one of them is half
attached, and the consequence is usually "the file exists but doesn't show up
where the user expects it."

Let the tool populate both from `entity_ref`. Don't hand-write one.

## Deep links

Shots, assets and versions come back with an `app_url`. Give that to a user
rather than a raw media URL: it opens the exact entity, and for assets it pins
the specific version rather than landing on whatever is newest.

`create_share_link` returns a members-only short link. It is login- and
permission-gated, unlike a raw media URL — prefer it any time you're handing
someone a pointer to something.

Related: `ag-versions.md` for what goes on the file record itself.
