# Documentation Best Practices

This document is the source of truth for how {{PROJECT_NAME}} documentation is organized, written, verified, and maintained. It applies to `AGENTS.md` and every document under `.subagents/`.

## Goal

Documentation should give an agent the right context at the right time:

1. `AGENTS.md` provides the small set of rules and facts needed at launch.
2. `.subagents/overview.md` provides project orientation and the architecture map.
3. Focused `.subagents/*.md` files provide detail for a subsystem.
4. `.subagents/CHANGELOG.md` contains historical information only.

The top-level guide must be fast to scan. Detailed information should be reachable through a clear pointer instead of being repeated there.

## Information Placement

### Keep in `AGENTS.md`

Include **only** information that can cause an immediate mistake or applies to almost every task:

- What the project is and its main technologies (2–3 lines).
- Build and verification commands, requirements, entry point (5–8 lines).
- Working rules governing agent behavior in this repo (5–10 lines).
- **Critical constraints** — numbered, scannable rules whose violation breaks behavior or loses data (vision gate = rule 0).
- **Compact quick-reference table** (settings, versions, limits).
- **Reference table** pointing to detailed subsystem documents.

**Do not put in `AGENTS.md`:** architecture, runtime workflows, field ownership details, prompt/cache mechanics, service implementations, method signatures, or any detail an agent can read in the owning `.subagents/` doc. If a paragraph explains *how* something works, it belongs in `.subagents/`, not here.

### Keep in `.subagents/overview.md`

Use the overview for information needed to understand the project before selecting a subsystem:

- Project orientation and supported environment.
- The build/run context when it helps explain the project structure.
- Architecture diagrams, layer responsibilities, and major component relationships.
- The location of subsystem documentation.

### Keep in focused subsystem documents

Each detailed document should answer one clear question, such as:

- What state and lifecycle does the public API own?
- How does a core workflow populate and preserve data?
- How are requests, prompts, caches, and responses handled?
- Which configuration keys or permissions exist?

Prefer the existing `.subagents/` document that matches the subject. Add a new document only when no existing document has a natural owner and the new scope can remain focused.

### Keep in `CHANGELOG.md`

Use the changelog for historical fixes, migrations, and prior behavior. Current documentation must describe the implementation that exists now. Do not use historical notes as the only explanation of current behavior.

## Required `AGENTS.md` Structure

Use this order so the most useful context appears first:

```markdown
# AGENTS.md — Project Guide

## Start Here
## Working Rules
## Build and Verify
## Critical Constraints
## Runtime Rules to Preserve
## Quick Reference
## Documentation Rules
## Detailed Reference
```

The exact headings may vary when the project needs another section, but launch-critical rules should remain before detailed references.

## Writing Rules

- Write short, direct sentences. Prefer concrete actions and observable behavior.
- State what exists today. Do not document plans, assumptions, or removed behavior as current facts.
- Include details an agent cannot safely infer from source inspection, especially limits, ownership, ordering, compatibility, and failure behavior.
- Use tables for compact mappings such as settings, fields, triggers, outputs, and document ownership.
- Use diagrams only when relationships or sequencing are difficult to understand in prose.
- Use emphasis sparingly. Reserve `IMPORTANT` and numbered constraints for rules whose violation can break behavior or lose data.
- Avoid repeating the same fact in multiple documents. Keep one authoritative statement and link to it elsewhere.
- When a value changes frequently, document its source of truth and current value only where the value is operationally necessary.
- Keep examples accurate and minimal. An example must reflect the current implementation, not an idealized design.

## Critical Constraint Rules

The `Critical Constraints` section must begin with rule `0` for image handling:

> If the user provides a screenshot or image and the active model cannot see it, stop immediately and notify the user. Never infer, guess, or proceed from assumed image content.

Follow that rule with the project-specific hazards that are most likely to cause hangs, data loss, unsupported behavior, incorrect ownership, or invalid writes. Keep the list short enough to scan. Move explanations and implementation detail to the owning subsystem document.

## Workflow Documentation

For each important workflow, document the trigger, the fields or state it changes, what it must preserve, and the detailed implementation pointer.

```markdown
| Trigger | Populates | Leaves unchanged |
|---|---|---|
| `CoordinateEditorPopover` → `applyGPSCoordinate()` | Location fields | AI fields and Area of Interest |
```

Use the detailed workflow document for flow diagrams, clear operations, error handling, and method-level implementation notes. Keep the top-level table as a quick boundary check.

## Documentation Ownership

Use the most specific existing document:

| Change | Required documentation |
|---|---|
| Bug fix or behavioral change | `.subagents/CHANGELOG.md` and the affected subsystem document |
| New configuration key | `.subagents/config.md` |
| New permission or sandbox behavior | `.subagents/permissions.md` |
| New public API method or state | `.subagents/api.md` |
| New or changed service | Matching `.subagents/services-*.md` document and the reference table |
| Prompt or model version | `.subagents/prompts.md` plus `AGENTS.md` quick reference when operationally relevant |
| Data model, alias, or mapping | `.subagents/models.md` and the affected workflow document |
| Historical migration | `.subagents/CHANGELOG.md` |

Every source-code commit must update `AGENTS.md` and every affected documentation file in the same commit. Documentation is incomplete when it describes behavior that the code no longer implements.

## Safe Reorganization Procedure

When `AGENTS.md` becomes too long:

1. Identify each paragraph that describes a rule, a current fact, a workflow, architecture, or history.
2. Keep launch-critical rules and a concise pointer in `AGENTS.md`.
3. Move architecture to `overview.md`, workflows to the relevant workflow document, and implementation detail to the relevant service or API document.
4. Replace moved material with a specific reference; do not leave an ambiguous pointer such as "see the docs."
5. Remove duplicate or stale wording after confirming the authoritative copy contains the information.
6. Preserve all unique information. Reorganization changes location and presentation, not meaning.

Do not create a standalone document merely to shorten a section. Use an existing document with a matching scope whenever possible.

## Verification Checklist

Before finalizing a documentation change:

- Read the changed documents with fresh eyes and confirm the launch path is scannable.
- Run the "doctor" check: would removing this line make an agent more likely to make a mistake?
- Verify commands, paths, versions, limits, and method names against the implementation.
- Confirm every reference path exists and every moved detail has a reachable owner.
- Search for stale names, removed features, conflicting mappings, and duplicate authoritative values.
- Confirm the mandatory vision rule remains present as constraint `0`.
- Check that no source-code commit has left affected documentation unchanged.
- **AGENTS.md line count ≤ 150** (excluding reference table). If exceeded, move detail to `.subagents/` and replace with pointer.
- Keep unrelated working-tree changes untouched.

## Anti-Patterns

Avoid:

- A long project history in `AGENTS.md`.
- Repeating the same cache, prompt, field, or service description in several files.
- Detailed API signatures where a focused service document is linked.
- Instructions that the agent can infer from ordinary source code.
- Future plans presented as current behavior.
- A reference table that points to missing, stale, or incorrectly named files.
- Deleting unique information simply because it does not belong in `AGENTS.md`.

## Final Standard

Good documentation is concise at launch, detailed on demand, current in every claim, and explicit about the rules that can cause damage. `AGENTS.md` should tell an agent what must be known immediately and where to find everything else.