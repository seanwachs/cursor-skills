---
name: create-pr
description: Creates a pull request for the current branch using gh CLI. Extracts ticket numbers from branch names, builds title and description from diff and commit history, pushes and opens the PR. Use when creating a PR, opening a pull request, submitting changes for review, or pushing a branch for review.
---

# Create Pull Request

## Prerequisites

Read `project-context/SKILL.md` first if you haven't already — it has branch naming conventions, PR title format, and Jira metadata you'll need.

## Workflow

1. **Check branch state**
   - Run `git status` to check for uncommitted changes.
   - Run `git log` and `git diff <base>...HEAD` to understand all commits on the branch.
   - Determine the base branch (usually `main` or `master`).

2. **Handle uncommitted changes**
   - If there are uncommitted changes, ask the user whether to commit them first or proceed without.

3. **Extract ticket number**
   - Parse the branch name using the convention from `project-context`: `ap-365-some-description` → `AP-365`.
   - If no ticket is found, skip the prefix.

4. **Fetch Jira context (when ticket found)**
   - Use the Atlassian MCP to fetch the Jira issue summary and description:
     ```
     getJiraIssue:
       cloudId: "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1"
       issueIdOrKey: "AP-XXX"
     ```
   - The Jira issue often captures the *why* behind a change better than commit messages do. Use it to enrich the PR body — the diff tells you *what* changed, but the Jira issue tells you *why* it was needed.
   - If the Atlassian MCP is not available, fall back to deriving context from the branch name and commits only.

5. **Build PR title**
   - Format: `AP-XXX: Short descriptive title` (if ticket found) or just `Short descriptive title`.
   - Always include the colon after the ticket number — this matches the GitHub Actions workflow that enforces the format.
   - Derive the title from the Jira issue summary (preferred) or branch name + commit messages.

6. **Build PR body**
   - Write a clear, concise description of what the PR does and why.
   - Base it on: the Jira issue context (the *why*), the full diff (the *what*), and the commit history.
   - Do NOT use a structured template with sections. Just write a plain description paragraph.
   - If the Jira issue has acceptance criteria or a description, weave the relevant parts into the PR body naturally.

7. **Push and create**
   - Push the branch: `git push -u origin HEAD`
   - Create the PR: `gh pr create --title "..." --body "..."`
   - Use a HEREDOC for the body to preserve formatting.

8. **Return the PR URL** so the user can open it.

## Example usage

```
User: "Create a PR"
User: "Push and open a PR for this branch"
User: "Submit this for review"
```

## Notes

- Never force-push or amend commits without explicit user approval.
- If the branch has no commits ahead of the base, inform the user instead of creating an empty PR.
- If `gh` CLI is not authenticated, let the user know.
