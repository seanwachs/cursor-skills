---
name: get-pr-comments
description: Fetch and summarize unresolved review comments from the active pull request. Groups feedback by severity and actionability, returning a prioritized action list. Use when checking PR feedback, reviewing comments, seeing what reviewers said, or getting a summary of PR review status.
---

# Get PR comments

## Trigger

Need a concise, actionable summary of unresolved feedback on the active pull request. Typical invocations:

- `/get-pr-comments` — fetch all unresolved comments
- `/get-pr-comments blockers` — only show blocking/critical items
- `/get-pr-comments nits` — only show minor suggestions

## Severity filter

The user can optionally specify a severity filter to focus on what matters:

| Filter | Shows | Hides |
|--------|-------|-------|
| (none / all) | Everything unresolved | Nothing |
| `blockers` | Blocking issues, bugs, required changes | Nits, suggestions, questions |
| `nits` | Minor suggestions, style feedback, optional improvements | Blockers, bugs |
| `questions` | Open questions from reviewers that need answers | Fixes, suggestions |

If no filter is specified, show everything grouped by severity.

## Workflow

1. Resolve the active PR for the current branch:
   ```
   gh pr view --json number,url,headRefName,baseRefName
   ```
   Extract owner, repo, and PR number.

2. Fetch review threads with resolution status using the GraphQL API:
   ```
   gh api graphql -f query='{
     repository(owner: "<owner>", name: "<repo>") {
       pullRequest(number: <number>) {
         reviewThreads(first: 50) {
           nodes {
             isResolved
             comments(first: 10) {
               nodes {
                 id body author { login }
                 path line originalLine diffHunk
                 url
               }
             }
           }
         }
       }
     }
   }'
   ```
   Filter to `isResolved: false` threads only — resolved threads are already handled and just add noise.

3. Classify each unresolved comment by severity: **blocker** (must fix), **suggestion** (should consider), **nit** (minor/optional), **question** (needs answer).
4. Apply the severity filter if one was specified.
5. Group feedback by severity and actionability.
6. Return a concise action list.

## Output

- Grouped feedback summary (blockers first, then suggestions, nits, questions)
- Action list ordered by priority
- Open questions that still need clarification
- Count per severity level (e.g. "2 blockers, 3 suggestions, 1 nit, 1 question")
- Note how many resolved threads were skipped (e.g. "5 resolved threads hidden")

If a severity filter was applied, note what was filtered out: "Showing blockers only. 4 other comments (2 suggestions, 1 nit, 1 question) hidden — run without filter to see all."
