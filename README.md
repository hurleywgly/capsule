# /capsule

Portable, actionable knowledge transfer between Claude Code systems.

## What It Does

Capsules are self-contained markdown artifacts that package knowledge, patterns, skills, and integration plans into a format any receiving agent can act on. They're not context dumps — they're dispatch packages with built-in instructions for the receiver.

A capsule answers: **"What does the receiving system need to know, and what should it do with it?"**

## Quick Start

```
/capsule "Transfer the authentication patterns to the new project"
```

```
/capsule load path/to/capsule.md
```

```
/capsule
```
Bare invocation walks you through it interactively.

## Modes

### Create

Builds a new capsule from session context, referenced files, or a description you provide.

**Workflow:**

1. **Intake** — What to transfer and where it's going
2. **Complexity Detection** — Auto-triage into Quick, Standard, or Deep tier based on content signals
3. **Extraction & Assembly** — Compress, classify (narrative vs. structured), and assemble through optional lens sequence
4. **Sanitization** — Strip credentials, internal URLs, proprietary data
5. **Verification** — Self-check for actionability, completeness, proportionality
6. **Output** — Write the capsule to disk

### Load

Reads an existing capsule and walks through its integration plan.

```
/capsule load path/to/capsule.md
```

Parse -> present the plan -> check prerequisites -> execute step-by-step with confirmation -> verify results.

Quick-tier capsules with no integration plan are presented as context briefings.

## Tiers

Auto-detected from content complexity. You can override.

| Tier | Target Size | Use When | Includes |
|------|-------------|----------|----------|
| **Quick** | 50-150 lines | 1-2 focused topics, pure knowledge transfer | Dispatch Summary, Core Content |
| **Standard** | 150-400 lines | 3-5 topics, session handoffs, meeting prep | + Patterns, Integration Plan, Signals |
| **Deep** | 400+ lines | Skill replication, schemas, infrastructure setup | + Skill Artifacts, Verbatim Payloads |

The key difference: Quick capsules inform. Standard capsules instruct. Deep capsules replicate.

## Categories

Orthogonal to tiers — describes **receiver intent**, not output volume.

| Category | Receiver Needs To... | Preserve |
|----------|---------------------|----------|
| **Full System** | Operate within the system | Constraints, boundaries, operating rules |
| **Knowledge Pillar** | Understand a domain | Conceptual relationships, the "why" |
| **Conversation Thread** | Continue in-flight work | Decisions, open questions, temporal context |
| **Living Reference** | Stay current on evolving state | Structure for incremental updates |

## Archetypes

Orthogonal to both tiers and categories — describes the **temporal transfer pattern**, how the knowledge behaves after delivery.

Ordered by shelf-life (shortest to longest):

| Archetype | Transfer Pattern | Optimize For |
|-----------|-----------------|--------------|
| **Perishable** | Use once, discard | Aggressive compression, task-specific context only |
| **Handoff** | Baton pass between sessions | Decisions, open threads, where-we-left-off state |
| **Seed** | Intentionally incomplete, receiver expands | The question/direction, not the answer |
| **Steroid** | Rapid large-scale absorption | Action items and critical context for immediate execution |
| **Delta** | Incremental sync over time | Changes only — assumes base context exists |
| **Trainer** | Permanent learning | Conceptual relationships, the "why," principles |
| **Dormant** | Pre-loaded, inactive until triggered | Trigger conditions and full payload |

**Audience-targeting** is a modifier, not an archetype. Any capsule can carry attributes defining who sees it, what's redacted, and what level of detail is appropriate. This connects to the sanitization layer (same-owner vs. cross-team vs. public).

## Capsule Schema

Every capsule follows a canonical structure. Sections are included or omitted based on tier — never left blank.

```
# [NAME].CAPSULE.md

> One-line tagline

Metadata (version, date, from, to, purpose, tier, archetype)

1. Dispatch Summary              [all tiers]
2. Core Content                  [all tiers]
3. Patterns and Learnings        [Standard+]
4. Skill Artifacts               [Deep only]
5. Integration Plan              [Standard+]
6. Signals                       [Standard+]
7. Sanitization Notes            [when applied]
8. Verbatim Payloads             [Standard+, when applicable]
```

## Key Concepts

**Artifact classification** — Source material is split into NARRATIVE (compress normally) and STRUCTURED (copy verbatim). The rule: if changing a single character could change system behavior, it goes to Verbatim Payloads untouched.

**Sanitization** — Credentials removed, company data redacted, names generalized to roles. Same-owner transfers (both systems belong to the same person) can skip personal sanitization.

**Verification** — Before output, capsules are checked against six criteria: actionability, completeness, assumption clarity, proportionality, receivability, and verbatim fidelity.

**Integration Plans** — Standard and Deep capsules include numbered steps with prerequisites, actions, verification checks, and rollback instructions. Each step is marked `[auto]` (agent executes) or `[manual]` (human required).

## Customization

**Lenses**: The skill can optionally use lens files (perspective-based review passes) during extraction. If lens files aren't available, the skill proceeds with default perspectives. Lenses enhance quality but aren't required.

**Output path**: Defaults to a `packages/` directory. Configure in the SKILL.md to match your workspace structure.

**Sanitization rules**: The same-owner exception and sanitization categories can be adjusted for your team's needs.
