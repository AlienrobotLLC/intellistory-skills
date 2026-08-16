<!-- Generated from the IntelliStory agent knowledge base (article `ag-workspaces`, last edited 2026-08-16). Do not edit here — edit the KB and rebuild. -->
# Workspaces, Seats and Project Access

A **workspace** (organization) is where projects live and where the bill goes.
Two different things decide what a person can do, and agents mix them up:

| Relationship | Row | Grants |
|---|---|---|
| **Seat** in a workspace | `org_members` | tenancy + billing; a *role* in the workspace |
| **Grant** on a project | `project_members` | access to that one project (and its episodes) |

A seat does **not** open any project by itself, and a project grant does **not**
create a seat. Someone with grants but no seat is a **guest**: they see exactly
the projects they were put on, in a workspace they don't otherwise belong to.

## Workspace roles

| role | Can |
|---|---|
| `owner` | everything below, plus billing, delete the workspace, transfer ownership, grant/revoke `owner` |
| `admin` | manage members, settings, integrations and API keys; **sees every project** in the workspace |
| `member` | seated; sees only projects they were granted; may create projects |
| `viewer` | read-only seat; may not create (the demo workspace auto-joins people as viewer) |
| *guest* | not a role you can set — it's the absence of a seat |

Rules the platform enforces, so don't try to work around them:

- Only an owner can grant or revoke `owner`, transfer ownership, or delete the workspace.
- The **last owner can never be removed or demoted** — transfer ownership first (`transfer_org_ownership`).
- Admins cannot change their own role or remove owners.
- API keys act *as their workspace*, need the `admin:org` scope for any workspace write, and can never transfer or delete.
- Personal workspaces (one per user) have no member management and cannot change owner.

## Which tool for which job

| You want to… | Use |
|---|---|
| know which workspaces the caller is in, and as what | `list_organizations` |
| see who is in a workspace (and, for admins, the guests) | `list_org_members` |
| **put someone on a project** | `create_project_member` — this is the normal "add crew" |
| give someone workspace-wide oversight | `add_org_member` with `role: admin` |
| give someone a seat that bills to the workspace but no projects yet | `add_org_member` with `role: member` |
| bring in someone who has no account yet | `invite_to_org` (email link; optional `project_ids`) |
| take away a project | `delete_project_member` |
| take away a seat (and, by default, every project grant in that workspace) | `remove_org_member` — pass `keep_project_grants: true` to leave them as a guest |
| change name / slug / logo | `update_organization` |
| find a person the workspace already knows, for a picker | `search_org_users` |
| see what admins did recently | `read_org_audit_log` |
| point the workspace's masters at its own bucket (BYO storage) | `set_org_storage` → `test_org_storage` (`get_org_integrations` to read, `remove_org_storage` to drop) |
| set a workspace-level model-provider key (e.g. fal) | `set_org_provider_key` |

`add_org_member` only works for people who already have an IntelliStory account
(by `email` or `user_id`). If it answers `user_not_found`, use **`invite_to_org`**
instead: it emails a single-use link bound to that address (7 days) and can carry
`project_ids` so they land on projects the moment they accept. `list_org_invites`,
`resend_org_invite`, `revoke_org_invite` manage the pending ones. If the response
says `mail.sent: false`, mail isn't configured on that server — hand the returned
`url` to the user to send themselves.

## Bring-your-own storage (masters)

A workspace can keep its full-quality masters (HDR movies, EXR sequences) in a
bucket it owns — AWS S3, Backblaze B2, Wasabi or any S3-compatible endpoint.
`set_org_storage` stores the config and sealed keys (never returned);
`test_org_storage` runs a live probe (reach bucket, write/read/delete a probe
object, presigned download, list, CORS) and only a passing test (`status: ok`)
switches the workspace over. Everyday generations and the database are not
affected — only masters. If a user asks how to prepare their bucket, point them
at the runbook (Workspace → Integrations → Storage explains the IAM policy and
CORS; the doc is `byo-storage-runbook`).

## Crew management is privilege, not access

`create_project_member`, `update_project_member` and `delete_project_member`
require **crew authority**: workspace owner/admin of the project's workspace, a
project `owner` member, or an API key issued for that workspace. Being able to
*see* a project is not enough — an artist cannot add or promote anyone, including
themselves. Removing yourself from a project (leaving) is always allowed.

Project roles (`owner | director | producer | artist | viewer`) are a different
vocabulary from workspace roles; the default for a new grant is `artist`.

## Practical sequences

**"Add Sam to the Kid Show project."**
1. `search_org_users` in the project's workspace with `query: "sam"` → user_id.
2. `create_project_member` with that user_id (role `artist` unless told otherwise).
3. Sam gets a notification with a link; if Sam's session predates the grant they reload, and if the project lives in another workspace they switch to it from the workspace menu — say so.

**"Make Priya an admin here."** `add_org_member` (`role: admin`), or `update_org_member_role` if Priya already holds a seat.

**"Hand the studio over to Priya."** She must hold a seat first; then `transfer_org_ownership`. The caller becomes admin (or stays a co-owner with `keep_owner: true`).

**"Remove a contractor when the job is done."** `remove_org_member` — the default also revokes their project grants in that workspace, which is almost always what's meant. Confirm before calling; it's not reversible short of re-adding them.

Related: `ag-versions` (what a grant lets someone see), `ag-billing` (which wallet a project spends from — the project's workspace, never the caller's).
