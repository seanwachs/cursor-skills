---
name: deslop
description: Remove AI-generated code slop and clean up code style. Catches unnecessary comments, defensive bloat, any-casts, verbose naming, and deeply nested code. Use when deslopping, cleaning up AI code, removing code slop, or tidying up a branch before review.
---

# Remove AI code slop

Check the diff against main and remove AI-generated slop introduced in the branch.

## Prerequisites

Read `project-context/SKILL.md` first — especially the "Intentional patterns" section. The AP codebase has architectural patterns (two-layer model split, explicit mappers, dual-identity tool nodes, `SharedContextData` with `MessageCaptureHandler`) that look like unnecessary complexity at first glance but are deliberate design decisions. Don't flag these as slop.

If an `architecture.md` exists in the repo, read it too for a deeper understanding of what's intentional.

## Focus Areas

- **Extra comments** that are unnecessary or inconsistent with local style — AI tends to over-comment obvious code
- **Defensive checks or try/catch blocks** that are abnormal for trusted code paths — not every function needs a try/catch
- **Casts to `any`** used only to bypass type issues — fix the types instead
- **Deeply nested code** that should be simplified with early returns
- **Verbose naming slop** — AI loves overly descriptive names that add noise rather than clarity. Watch for:
  - Unnecessary `Handler`/`Manager`/`Service`/`Helper`/`Processor` suffixes that don't distinguish the class from alternatives
  - Variables like `isCurrentlyLoading` when `loading` would do
  - Methods like `performDataValidation` when `validate` is clear from context
  - Redundant type information in names like `userDataObject` or `itemsList`
- **Other patterns** inconsistent with the file and surrounding codebase

## Guardrails

- Keep behavior unchanged unless fixing a clear bug.
- Prefer minimal, focused edits over broad rewrites.
- Don't touch patterns that are architectural decisions (check project-context first).
- When in doubt about whether something is intentional, leave it and mention it in the summary.
- Keep the final summary concise (1-3 sentences).
