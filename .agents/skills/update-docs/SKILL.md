---
name: update-docs
description: Identifies and updates Confluence documentation pages affected by a PR. Analyses the diff, proposes targeted edits, and applies approved changes via Atlassian MCP. Use when updating documentation, updating confluence pages, or syncing docs after a PR.
---

# Update Documentation

Use after a PR is approved and **before merging into main** to identify which regular documentation (not release notes) needs updating in Confluence.

Covers the "Agents" Confluence space at: https://capacit.atlassian.net/wiki/spaces/Agents

---

## Documentation page catalog

Below is the full list of documentation pages that may need updating. Match changed code/infra to relevant pages.

### 🏗️ Infrastructure & Setup

| Page | Confluence ID | URL | Covers |
|------|--------------|-----|--------|
| Whitelabel infrastructure guide | `584843265` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/584843265) | Docker setup, container names, environment variables, deployment steps, infra config for white-label customers |
| IDA: Infrastructure on-prem guide | `588447749` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/588447749) | On-prem setup for IDA customer, specific infra differences from whitelabel |
| Developer setup guide | `625147955` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/625147955) | Local dev environment setup, prerequisites, first-time setup steps |
| Widget app (for self-hosting) | `626065409` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/626065409) | Self-hosted widget configuration |
| Deployment guide | `769916930` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/769916930) | Deployment procedures and checklists |
| Developer handbook | `791478275` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/791478275) | Development standards, conventions, patterns |
| Incident, Bug & Security Agreement | `611713026` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/611713026) | SLA and incident handling procedures |

### 🔌 API Documentation

**Scope:** API documentation only covers endpoints intended for **external callers** (i.e. systems other than the frontend). Currently the only externally-facing API endpoints are: **workflow export, workflow import, workflow validate, and workflow deployment**. Do NOT update API docs for endpoints that are only called by the frontend.

| Page | Confluence ID | URL | Covers |
|------|--------------|-----|--------|
| API documentation (root) | `821133317` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/821133317) | Overview of available external API endpoints |
| Workflow Export & Import | `821592085` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/821592085) | API endpoints for workflow export, import, and validate |
| Workflow Deployment | `820805643` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/820805643) | API endpoints for workflow deployment |

### 🖥️ Platform UI Documentation

| Page | Confluence ID | URL | Covers |
|------|--------------|-----|--------|
| Dashboard | `719749148` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/719749148) | Dashboard UI features and metrics |
| Workflow overblik | `722599957` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/722599957) | Workflow overview UI |
| Schemas | `806584329` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/806584329) | Schema management UI |
| API Settings | `722599950` | [link](https://capacit.atlassian.net/wiki/spaces/Agents/pages/722599950) | API settings configuration UI |

---

## Steps

### Step 1 — Analyse the changes
This skill is designed to run on an approved PR branch **before it is merged into main**. Get a full picture of what the PR changes compared to main:
```bash
git diff main...HEAD --stat
git diff main...HEAD
git log main..HEAD --oneline
```
The `main...HEAD` triple-dot syntax diffs the branch tip against the point where it diverged from main — giving a clean view of the entire PR regardless of how many commits it contains.

Then categorise what was affected:
- **External API endpoints changed?** (workflow export/import/validate/deployment only) → Check API documentation pages. Ignore endpoints that are only called by the frontend.
- **Environment variables / config changed?** → Check whitelabel guide, IDA guide, developer setup
- **Docker / container names / versions changed?** → Check whitelabel guide, IDA guide, deployment guide
- **UI features changed?** → Check Dashboard, Workflow, Schemas, API Settings pages
- **Dev tooling / local setup changed?** → Check developer setup guide, developer handbook

### Step 2 — Fetch current content of affected pages
For each affected page, fetch its current content so you can make precise, targeted edits:
```
Use the Atlassian MCP to call getConfluencePage with:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  pageId: <page ID from table above>
  contentFormat: "markdown"
```

### Step 3 — Propose specific edits
For each affected page, output a proposal in this format:

```
### 📄 [Page Title]
🔗 [URL]

**Reason this page is affected:** [Brief explanation]

**Proposed change:**
- Section: [Section name or heading]
- Current text (abbreviated): "[...]"
- Proposed new text: "[...]"

[If it's a new section, show the full new section in markdown]
```

### Step 4 — Wait for approval
Do NOT apply any changes. Present all proposals and ask:
```
Ready to apply changes to the pages above. Please review and confirm:
- `approve [page name]` to apply a specific page
- `approve all` to apply all proposed changes
- `skip [page name]` to skip
- Or edit any proposal above before approving
```

### Step 5 — Apply approved changes
For each approved page, update via Atlassian MCP:
```
updateConfluencePage:
  cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
  pageId: <page ID>
  contentFormat: "markdown"
  body: <updated full page content>
  versionMessage: "Updated by Cursor agent: [brief reason]"
```

**Important:** Always send the **full page content**, not just the changed section.

### Step 6 — Output diff links for verification
After each page is updated, output a diff link so the user can visually verify the changes. The link compares the new version (N) against the previous version (N-1):

```
https://capacit.atlassian.net/wiki/pages/diffpagesbyversion.action?pageId=<PAGE_ID>&selectedPageVersions=<N-1>&selectedPageVersions=<N>
```

To get the current version number, check the response from `updateConfluencePage` — it returns the new version. Use `newVersion - 1` as the old version.

**Example output:**
```
✅ Updated: Workflow overblik
🔗 Diff: https://capacit.atlassian.net/wiki/pages/diffpagesbyversion.action?pageId=722599957&selectedPageVersions=2&selectedPageVersions=3
```

Always output diff links for **every** page that was updated.

---

## Rules
- Never edit pages without explicit user approval
- If a page requires screenshots or visuals (e.g., dashboard UI changes), note this in the proposal and ask the user to add the screenshots manually
- Prefer minimal, targeted edits — don't rewrite sections that haven't changed
- When container image tags or version numbers appear in docs, update them to match the new version
- The Atlassian cloud ID is always: `deb5ee5f-2c5f-46bf-a2a4-98575f4404b1`
