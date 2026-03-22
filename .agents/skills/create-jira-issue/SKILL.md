---
name: create-jira-issue
description: Create a Jira bug or task in the AP project backlog using the Atlassian MCP. Optionally assigns to the current or a named sprint. Use when the user wants to file a bug, create a task, report an issue, log a defect, or add a ticket.
---

# Create Jira Issue

## Prerequisites

Read `project-context/SKILL.md` first if you haven't already — it has the branch naming convention you'll need for the output.

## Constants

- **Cloud ID**: `deb5ee5f-2c5f-46bf-a2a4-98575f4404b1`
- **Project key**: `AP`
- **Site URL**: `https://capacit.atlassian.net`

## Issue type resolution

| User says | `issueTypeName` |
|-----------|-----------------|
| bug, defect, issue, problem, broken | `Bug` |
| task, ticket, story, work item, todo | `Task` |
| ambiguous or unspecified | **Ask the user** |

Default to `Bug` only when the conversation clearly describes broken behavior.

## Workflow

1. **Gather details from the user or conversation context**
   - **Issue type**: determine from the table above.
   - **Summary** (required): concise one-line title.
   - **Description** (optional): markdown body. For bugs include steps to reproduce, expected vs actual behavior. For tasks include acceptance criteria or scope if available.
   - If the user described the issue during the conversation, infer summary and description from that context. Confirm with the user before creating.

2. **Create the issue**

   Use the Atlassian MCP `createJiraIssue` tool:

   ```json
   {
     "cloudId": "deb5ee5f-2c5f-46bf-a2a4-98575f4404b1",
     "projectKey": "AP",
     "issueTypeName": "<Bug | Task>",
     "summary": "<title>",
     "description": "<markdown description>"
   }
   ```

   The response contains the issue `key` (e.g. `AP-421`).

3. **Sprint assignment (only when requested)**

   By default the issue lands in the **backlog** — do nothing extra.

   If the user asks for a specific sprint or "current sprint":

   a. Find the sprint by fetching a full issue that's already in it:

   ```
   Tool: searchJiraIssuesUsingJql
   jql: "sprint in openSprints() AND project = AP"   (or sprint = "<name>" for a named sprint)
   maxResults: 1
   ```

   Then fetch the returned issue with `getJiraIssue` (no field filter) and read `customfield_10020[0].id` for the sprint ID.

   b. Assign the issue to the sprint. The sprint field is `customfield_10020` and requires a **plain integer**, not an object:

   ```
   Tool: editJiraIssue
   issueIdOrKey: "<AP-XXX>"
   fields: { "customfield_10020": <sprint_id_number> }
   ```

4. **Return the link and branch suggestion**

   Always end by providing:

   a. The clickable Jira URL:
   ```
   https://capacit.atlassian.net/browse/AP-XXX
   ```

   b. A ready-to-paste git command to start working on the issue. Generate the branch name from the issue key and summary using the project convention (`ap-<number>-<kebab-case-short-description>`):
   ```
   To start working on this:
   git checkout -b ap-421-retry-logic-embedding-pipeline
   ```

   This bridges the gap between "issue created" and "work started" — the user can copy-paste the command directly.

## Example interactions

```
User: "Create a bug for the overflow issue we just fixed"
→ Issue type: Bug. Infer summary/description from conversation, confirm, create, return link + branch command.

User: "File a bug: Login page crashes on Safari when password field is empty"
→ Issue type: Bug. Use the provided text as summary, create, return link + branch command.

User: "Create a task to add retry logic to the embedding pipeline"
→ Issue type: Task. Use the provided text as summary, create, return link + branch command.
→ Branch suggestion: `git checkout -b ap-421-retry-logic-embedding-pipeline`

User: "Create a ticket and put it in the current sprint"
→ Ask whether it's a bug or task, then create, find active sprint, assign, return link + branch command.

User: "Log a bug in sprint 'Sprint 24'"
→ Issue type: Bug. Create, find sprint by name, assign, return link + branch command.
```
