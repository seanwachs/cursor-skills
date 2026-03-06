---
name: create-pr
description: Creates a pull request for the current branch using gh CLI. Extracts ticket numbers from branch names, builds title and description from diff and commit history, pushes and opens the PR. Use when creating a PR, opening a pull request, submitting changes for review, or pushing a branch for review.
---

# Create Pull Request

## Workflow

1. **Check branch state**
   - Run `git status` to check for uncommitted changes.
   - Run `git log` and `git diff <base>...HEAD` to understand all commits on the branch.
   - Determine the base branch (usually `main` or `master`).

2. **Handle uncommitted changes**
   - If there are uncommitted changes, ask the user whether to commit them first or proceed without.

3. **Extract ticket number**
   - Parse the branch name for a ticket prefix (e.g., `ap-365-some-description` → `AP-365`).
   - If no ticket is found, skip the prefix.

4. **Build PR title**
   - Format: `AP-XXX: Short descriptive title` (if ticket found) or just `Short descriptive title`.
   - Always include the colon after the ticket number — this matches the GitHub Actions workflow that enforces the format.
   - Derive the title from the branch name and commit messages.

5. **Build PR body**
   - Write a clear, concise description of what the PR does and why.
   - Base it on the full diff and commit history — not just the latest commit.
   - Do NOT use a structured template with sections. Just write a plain description paragraph.

6. **Push and create**
   - Push the branch: `git push -u origin HEAD`
   - Create the PR: `gh pr create --title "..." --body "..."`
   - Use a HEREDOC for the body to preserve formatting.

7. **Return the PR URL** so the user can open it.

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
