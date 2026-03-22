---
name: update-release-notes
description: Drafts and publishes release note entries to Confluence after a PR is approved. Categorises changes into breaking changes, features, bug fixes, and pipeline items using the canonical release note format. Use when updating release notes, drafting release notes, or adding a release note entry.
---

# Update Release Notes

Use after a PR is approved and **before merging into main** to draft a release note entry capturing everything the PR introduces.

Covers the Release Notes folder at: https://capacit.atlassian.net/wiki/spaces/Agents/folder/711819265

---

## Discovering the current release notes

Rather than relying on a hardcoded list, **always discover** the current state of the Release Notes folder dynamically at the start of every run.

Call `getConfluencePageDescendants` on the release notes folder:
```
getConfluencePageDescendants:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  pageId: "711819265"
  depth: 1
  limit: 50
```

From the results, build a picture of the existing release notes:
- Each child page has an `id` and `title`.
- A page whose title contains `(draft)` is the **current draft** — note its `id` and version number.
- The page with the **highest semver** that does NOT contain `(draft)` is the **most recent published** release note. Use it as the gold-standard reference for format and tone.
- If no draft page exists, a new page will be created in Step 5.

Store these values for use throughout the remaining steps:
- `DRAFT_PAGE_ID` — the `id` of the draft page (or `null` if none exists)
- `DRAFT_VERSION` — the semver extracted from the draft title (e.g. `0.2.9`)
- `LATEST_PUBLISHED_ID` — the `id` of the most recent published page
- `LATEST_PUBLISHED_VERSION` — its semver (e.g. `0.2.7`)

---

## Canonical release note format

Use this exact structure. Do not deviate from section names or order. Use the **most recent published** release note (discovered above) as the gold standard example for format and tone.

```markdown
# Release Notes

**{{version}} = X.Y.Z**

_All containers and builds are named according to the release version. Follow the IDA: Infrastructure on-prem guide for detailed instructions._

* Node build name: {{version}}.js
* API: agentsappcrprod.azurecr.io/agent-platform-api:{{version}}
* Python executor: agentsappcrprod.azurecr.io/agent-platform-python-executor:{{version}}

## Breaking changes

[List breaking changes with detailed explanation. If none, write: "No breaking changes in this release."]

## New Features & Improvements

* **[Feature name] NEW**: [Description of the feature. Include screenshots if relevant.]
* **[Improvement name]**: [Description]

## Bug Fixes

* [Description of fix — what was broken, what was fixed]

## Known Issues

* **[Issue title]**: [Description and workaround if available]

[If none: "No known issues at this time."]

## Features in Pipeline

* [Short description of what's coming in future releases]
```

---

## Steps

### Step 0 — Discover current release notes
Before anything else, run the `getConfluencePageDescendants` call described in "Discovering the current release notes" above. This gives you `DRAFT_PAGE_ID`, `DRAFT_VERSION`, `LATEST_PUBLISHED_ID`, and `LATEST_PUBLISHED_VERSION`.

If a draft page exists, tell the user:
> "Found existing draft release note: **X.Y.Z (draft)** — I'll update this page."

If no draft exists, tell the user:
> "No draft release note found. I'll create a new page after you approve."

### Step 1 — Gather change information
This skill is designed to run on an approved PR branch **before it is merged into main**. Get a full diff of everything the branch introduces compared to main:
```bash
git diff main...HEAD --stat
git diff main...HEAD
git log main..HEAD --oneline
```
The `main...HEAD` triple-dot syntax diffs the branch tip against the point where it diverged from main — giving a clean, complete picture of the PR's changes regardless of how many commits are on the branch.

Also ask the user: **"What is the target version number for this release?"** if not already known.

### Step 1b — Fetch Jira context for customer-facing language

Extract the ticket number from the branch name (see `project-context/SKILL.md` for the naming convention). Then fetch the Jira issue:

```
getJiraIssue:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  issueIdOrKey: "AP-XXX"
```

The Jira issue summary and description are often written in customer-facing language, which is exactly what release notes need. Use the Jira context to inform your descriptions — the diff tells you *what* changed technically, but the Jira issue tells you *what it means for users*.

If the Atlassian MCP is not available or the branch has no ticket number, fall back to the diff and commit messages.

### Step 2 — Categorise changes
Sort all changes into the four buckets:
- **Breaking changes** — anything that requires customer action, migration, or will break existing setups
- **New Features & Improvements** — new functionality or UX improvements
- **Bug Fixes** — fixes to existing functionality
- **Features in Pipeline** — work in progress / next release items (ask the user if they want to add these)

**Criteria for "Breaking change":**
- API endpoint removed or renamed
- Environment variable renamed or required
- Database migration that is irreversible
- Container name or image tag format change
- Config file format change
- Any behaviour that external customers rely on that has changed

### Step 3 — Draft the release note
Write the full release note in the canonical format above. Use clear, customer-facing language — not internal jargon. Include:
- `NEW` badge on brand new features
- Short, factual descriptions (not marketing language)
- Mention if a fix is related to a customer-reported issue
- Prefer language from the Jira issue when it's clearer than what the diff suggests

### Step 4 — Present for review
Show the full draft and ask:

```
## Release Note Draft — v[X.Y.Z]

[Full draft here]

---
Please review the draft above. You can:
- `approve` to create this as a new Confluence page in the Release Notes folder
- `approve as draft` to create it with "(draft)" in the title
- Edit any section above before approving
- `skip` if you don't want to create a release note
```

### Step 5 — Create or update the Confluence page

**If `DRAFT_PAGE_ID` is `null` (no existing draft) — create a new page:**
```
createConfluencePage:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  spaceId: "488505348"
  parentId: "711819265"   ← Release notes folder
  title: "X.Y.Z"          ← or "X.Y.Z (draft)" if draft
  contentFormat: "markdown"
  body: <approved release note content>
```

**If `DRAFT_PAGE_ID` exists — update the existing draft page:**
```
updateConfluencePage:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  pageId: <DRAFT_PAGE_ID>          ← discovered in Step 0
  title: "X.Y.Z (draft)"
  contentFormat: "markdown"
  body: <approved release note content>
  versionMessage: "Updated by Cursor agent"
```

### Step 6 — Output diff link for verification
After creating or updating the page, output a diff link so the user can visually verify the changes:

```
https://capacit.atlassian.net/wiki/pages/diffpagesbyversion.action?pageId=<PAGE_ID>&selectedPageVersions=<N-1>&selectedPageVersions=<N>
```

For a **new page**, the first version is 1 — skip the diff link (there's no previous version to compare against).

For an **updated page**, use `DRAFT_PAGE_ID` and the version number from the `updateConfluencePage` response:
```
Updated: X.Y.Z (draft)
Diff: https://capacit.atlassian.net/wiki/pages/diffpagesbyversion.action?pageId=<DRAFT_PAGE_ID>&selectedPageVersions=<N-1>&selectedPageVersions=<N>
```

---

## Writing guidelines
- Write for **customers and operators**, not internal developers
- Avoid referencing internal ticket numbers or PR numbers unless asked
- For infrastructure changes, always mention whether the **IDA on-prem guide** needs to be re-followed
- Mark anything that requires **manual migration steps** very clearly with a warning
- "Known Issues" should be honest — don't hide real problems
- "Features in Pipeline" gives customers a preview of what's coming; keep it high-level

---

## Rules
- Never create or modify pages without user approval
- Always confirm the version number with the user before creating the page
- The Atlassian cloud ID is: `deb5ee5f-2c5f-46bf-a2a4-98575f4404b1`
- The space ID is: `488505348`
- The release notes folder ID is: `711819265`
