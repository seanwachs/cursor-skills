---
name: update-all-docs
description: Runs update-docs and update-release-notes together after a PR is approved, presenting a consolidated review before applying changes to Confluence. Use when updating docs, updating documentation, syncing docs, documenting a PR, or updating confluence.
---

# Update All Documentation

Invoke after a PR is approved and **before merging into main** to review and update **both** regular documentation **and** release notes in one go.

---

## What this skill does
1. Reads the git diff / recently changed files to understand what was changed
2. Identifies which Confluence documentation pages are affected
3. Drafts proposed edits for those pages
4. Drafts a release note entry for the change
5. **Presents everything for human review and approval before making any changes**
6. Only applies approved changes to Confluence

---

## Steps

### Step 1 — Understand what changed
This skill is designed to run on an approved PR branch **before it is merged into main**. Run the following to capture everything this branch introduces compared to main:
```bash
git diff main...HEAD --stat
git diff main...HEAD
git log main..HEAD --oneline
```
The `main...HEAD` triple-dot syntax diffs the branch tip against the point where it diverged from main — giving a clean, complete picture of the PR's changes regardless of how many commits are on the branch.

If the user has described the change in chat, use that description as the primary source of truth and use the diff as supporting detail.

### Step 2 — Run the two sub-skills in sequence
Invoke both sub-skills and collect their outputs:
- `@update-docs` — identifies affected documentation pages and proposes edits
- `@update-release-notes` — drafts the release note entry

### Step 3 — Present a consolidated review
Output a structured summary in this format:

```
## 📋 Documentation Update Proposal

### 📄 Documentation Changes
[Output from @update-docs — list of pages with proposed edits]

### 📢 Release Note Draft
[Output from @update-release-notes — full draft]

---
Do you approve these changes? You can:
- Type `approve all` to apply everything
- Type `approve docs` to apply only documentation changes
- Type `approve release notes` to apply only the release note draft
- Edit any section above before approving
- Type `skip [section]` to skip a specific section
```

### Step 4 — Apply approved changes
Once the user approves, apply changes using the Atlassian MCP integration. See sub-skills for exact page IDs and update instructions.

### Step 5 — Output diff links for verification
After applying changes, output a diff link for every updated page so the user can manually verify the changes. Both `@update-docs` and `@update-release-notes` will produce these links — collect and present them all here.

See the sub-skills for the diff link format.

---

## Notes
- **Never modify Confluence pages without explicit approval.**
- If the user says "this is a minor fix" or "just a patch", the release note entry should reflect that — keep it brief and categorise it under Bug Fixes.
- If unsure which pages are affected, err on the side of suggesting more rather than fewer, and let the user decide.
