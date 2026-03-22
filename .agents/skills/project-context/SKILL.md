---
name: project-context
description: Shared project conventions, branch naming, architecture overview, and Jira/GitHub metadata for the AP (Agent Platform) project. Other skills reference this file for cross-cutting context. Read this skill whenever you need to understand project conventions, branch format, ticket prefixes, or the codebase architecture — especially before creating PRs, triaging comments, deslopping code, or proposing AI rule updates.
---

# Project Context — Agent Platform (AP)

This is the shared context that other skills reference. It captures conventions, naming patterns, and architectural knowledge that cut across multiple workflows. If you're working on a task that touches the AP codebase, read this first.

---

## Project identity

- **Project key:** AP
- **Jira site:** https://capacit.atlassian.net
- **Jira Cloud ID:** `deb5ee5f-2c5f-46bf-a2a4-98575f4404b1`
- **Confluence space:** Agents — https://capacit.atlassian.net/wiki/spaces/Agents

---

## Branch naming convention

All branches follow this format:

```
ap-<issue-number>-<short-description>
```

- `ap` — project prefix (Agent Platform), always lowercase
- `<issue-number>` — the Jira issue number (digits only)
- `<short-description>` — kebab-case summary of the task or bug

**Examples:**
- `ap-123-login-feature` → Jira issue `AP-123`
- `ap-456-fix-schema-validation` → Jira issue `AP-456`

**Parsing rule:** Given a branch name, extract the ticket by uppercasing the prefix and number: `ap-365-some-description` → `AP-365`. The Jira URL is then `https://capacit.atlassian.net/browse/AP-365`.

---

## Architecture overview

The codebase uses a **two-layer model design** for workflow nodes. A full description lives in `architecture.md` in the repo root (or the docs folder). Here's the summary so other skills can make informed decisions:

### Layer 1 — Storage Models
Pydantic models persisted in `workflow_nodes.config`. All extend `BaseNodeConfigModel`, discriminated by `entity_type`. Located in `src/core/nodes/<node_type>/`.

### Layer 2 — Engine Models
Flat, fully-resolved configs consumed at runtime. Located in `src/workflow_platform/core/entities/`. No additional resolution per-run.

### The Mapper
`WorkflowConfigBuilder.build` (`src/core/mappers/workflow_config_builder.py`) bridges the layers. Per-type mapper functions live in `src/core/mappers/node_mappers/`.

**Key rule:** Never pass a storage model through to the engine layer. Each layer has its own model shape.

### Five node categories

| Category | Storage model | Engine model | Runtime agent |
|----------|--------------|-------------|---------------|
| Trigger | `TriggerConfigModel` | `TriggerConfig` | (not an agent — publishes initial message) |
| Return | `ReturnConfigModel` | `ReturnConfig` | (not an agent — applies formatting) |
| LLM/Agent | `AgentConfigModel` | `AutonomousAgentConfig` | `AutonomousLLMAgent` |
| Tool | `*ToolConfigModel` | `DirectToolNodeConfig` or `ToolConfig` | `ToolAgent` |
| Logic | `ConditionalRouterConfigModel` | `ConditionalRouterConfig` | `ConditionalRouterAgent` |

### Intentional patterns (not slop)

These patterns are architectural decisions, not code smell:

- The two-layer split (storage vs. engine) with explicit mappers between them
- Tool nodes having dual identity (standalone via execution flow edges, attached via tool attachment edges)
- `BaseWorkflowAgent` hierarchy extending autogen's `RoutedAgent`
- `SharedContextData` dict keyed by node ID with `MessageCaptureHandler` intercepting publishes
- Per-type mapper files in `node_mappers/` rather than a single monolith mapper
- Subscription building from `execution_flow_edges` during `WorkflowConfigBuilder.build`

Understanding these patterns helps avoid flagging them as unnecessary complexity during code review or deslopping.

---

## Commit message convention

```
<prefix>: <concise what>

<why this change is needed>
```

Prefixes: `fix:`, `feat:`, `refactor:`, `chore:`, `docs:`, `test:`

---

## PR title convention

```
AP-XXX: Short descriptive title
```

Always include the colon after the ticket number — this matches the GitHub Actions workflow that enforces the format.
