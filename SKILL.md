---
name: capsule
description: Create and load Claude Capsules — self-contained markdown artifacts for agent-to-agent knowledge transfer with automatic complexity detection and mandatory actionability
disable-model-invocation: true
---

# /capsule

Create or load Claude Capsules for system-to-system knowledge transfer. Capsules are self-contained markdown artifacts that package knowledge, patterns, skills, and integration plans into a portable format any receiving agent can act on.

## Purpose

Formalize the capsule pattern with:
- **Automatic complexity detection** — Quick, Standard, or Deep tier based on content signals
- **Mandatory actionability** — Every Standard/Deep capsule includes a concrete integration plan
- **Skill portability** — Deep capsules include complete skill artifacts inline so receiving systems can replicate without follow-up questions
- **Sanitization** — Strip sensitive data before cross-system transfer

## Modes

| Mode | Invocation | What It Does |
|------|-----------|--------------|
| **Create** | `/capsule` or `/capsule "description"` | Build a new capsule from session context or referenced files |
| **Load** | `/capsule load [path]` | Read an existing capsule and execute its integration plan |

---

## Create Mode: 6-Phase Workflow

### Phase 1: Intake

Determine what to transfer and where it's going.

- If invoked with a description or file references, use those as source material
- If bare invocation, scan the current session context and ask:
  - "What should we transfer?"
  - "Where is it going?" (which system/context will receive this)
- Identify all source material: session context, files, URLs, patterns from conversation

### Phase 2: Complexity Detection (Auto-Triage)

Assess source material and auto-select tier:

| Signal | Quick | Standard | Deep |
|--------|-------|----------|------|
| Topic count | 1-2 focused | 3-5 related | 6+ or cross-domain |
| Artifact types | No skills/code | Patterns, learnings | Skills, schemas, infrastructure |
| Action needed | "Read and understand" | "Integrate these ideas" | "Replicate and build" |

**Decision tree:**
1. Includes skill definitions, code, schemas, or infrastructure changes? -> **Deep**
2. Includes 3+ distinct learnings, patterns, or strategic shifts? -> **Standard**
3. Otherwise -> **Quick**

Announce the detected tier and rationale. Let the user override if they disagree.

### Phase 3: Extraction & Assembly

Behavior varies by tier. Load lenses from `/lenses/` as specified.

**Quick (50-150 lines)**
- Direct extraction, structured sections
- Lens: Synthesizer only
- Focus on clarity and conciseness
- No Integration Plan required

**Standard (150-400 lines)**
- Multi-topic extraction with pattern identification
- Generate numbered action items and integration steps
- Lens: Synthesizer -> Guardian
- Include Patterns & Learnings, Integration Plan, and Signals sections

**Deep (400+ lines)**
- Full extraction with skill artifacts inline
- JSON schemas if relevant
- Concrete integration plan with install/setup commands
- For each skill artifact, include ONE of:
  - Complete SKILL.md inline (if <200 lines)
  - Public URL + install instructions
  - Full specification sufficient to recreate from scratch
- Lens: Synthesizer -> Architect -> Guardian -> Sculptor
- Apply `/projector` principles: discovery, fidelity assessment, knowledge file curation

**Lens activation**: For each lens in the sequence, read the lens file from `/lenses/[name].md` and apply its perspective to the assembled content. Each lens reviews and enriches the output of the previous lens.

**Source compression**: When source material is verbose relative to the target tier, compress before assembling:
- Strip ceremony (headers, boilerplate, repeated context the receiver won't need)
- Preserve specifics (exact commands, file paths, config values, code snippets)
- Convert narrative into structured form (paragraphs become bullet points or tables)
- Keep the ratio: if a 500-line source doc feeds a Quick capsule, extract the 10 lines that matter, not a 150-line summary of 500 lines

### Phase 4: Sanitization

Scan all assembled content before writing:

| Category | Action |
|----------|--------|
| Company data | Replace with generic descriptions or `[REDACTED]` |
| Credentials/tokens | Remove entirely, note in Sanitization Notes that they're needed |
| Personal names (non-sender/receiver) | Generalize to roles (e.g., "the engineering lead") |
| Internal URLs | Remove or describe what was linked |
| Proprietary metrics | Strip specific values, keep directional insight |

**Same-owner exception**: If both sending and receiving systems belong to Ryan, offer to skip personal sanitization. Company/proprietary data is still stripped.

### Phase 5: Verification

Self-check before writing to disk. Evaluate each criterion and report results:

1. **Actionability**: Does every section inform or instruct? Flag any purely descriptive sections at Standard/Deep tiers.
2. **Completeness**: For skills, is there enough to replicate without follow-up questions? Flag gaps.
3. **Assumptions**: Are implicit assumptions made explicit? (e.g., "receiving system has Claude Code", "requires curl + jq")
4. **Proportionality**: Does depth match tier? Flag bloated Quick capsules or thin Deep capsules.

Present verification results to the user. User approves, requests adjustments, or overrides.

### Phase 6: Output

Write the capsule to `~/ryos/packages/[slug]-capsule.md`.

Report:
```
Capsule written to ~/ryos/packages/[slug]-capsule.md

Tier: [Quick | Standard | Deep]
Lines: [N]
Sections: [list]
Sanitization: [Applied / Skipped (same-owner)]
Verification: [All passed / N issues flagged]

Ready for dispatch.
```

---

## Load Mode: `/capsule load [path]`

Read an existing capsule and execute its integration plan.

### Load Workflow

**Step 1: Read & Parse**
- Read the capsule at the given path
- Identify tier and sections from the canonical schema
- Validate structure (warn if non-standard format)

**Step 2: Briefing**
- Present the Dispatch Summary and Integration Plan to the user
- "This capsule wants to do N things. Here's the plan:"
- List each integration step with a brief description

**Step 3: Prerequisites Check**
- Verify the receiving system has what's needed (tools, directories, permissions)
- Flag gaps before starting execution
- If prerequisites are unmet, present options: install them, skip affected steps, or abort

**Step 4: Step-by-Step Execution**
- Walk through the Integration Plan steps sequentially
- For each step:
  - Explain what it will do
  - Execute (create files, install skills, update configs)
  - Confirm success before moving to the next step
- If a step fails, pause and ask the user how to proceed

**Step 5: Skill Installation** (Deep capsules only)
- If a Skill Artifacts section exists, install each skill:
  - **Inline SKILL.md**: Write to `.claude/skills/[name]/SKILL.md`
  - **URL reference**: Fetch and install
  - **Specification**: Generate SKILL.md from spec, confirm with user before writing
- Check for conflicts with existing local skills. If found, flag and ask user to resolve before overwriting.

**Step 6: Verification**
- Run the capsule's verification criteria (from Integration Plan)
- Report what landed and what didn't

**Step 7: Eject**
- Report final results: steps completed, skills installed, issues encountered
- "Capsule consumed. [N/M] steps completed successfully."

### Load Edge Cases

| Scenario | Behavior |
|----------|----------|
| No Integration Plan (Quick tier) | Present content as context briefing, no execution steps |
| Skill conflicts with existing local skill | Flag conflict, show diff, ask user to resolve |
| Unknown/untrusted source | Full sanitization review before executing any steps |
| Malformed capsule (missing sections) | Parse what exists, warn about missing sections, proceed with available content |

---

## Canonical Capsule Schema

All capsules follow this structure. Sections are included or omitted based on tier.

```markdown
# [NAME].CAPSULE.md

> [One-line tagline in system-to-system voice]

**Version:** [1.0]
**Created:** [YYYY-MM-DD]
**From:** [Sending system/context]
**To:** [Receiving system/context]
**Purpose:** [1-2 sentences]
**Tier:** [Quick | Standard | Deep]

---

## 1. Dispatch Summary                    [ALL TIERS]
What's being transferred and why. 2-5 sentences.

## 2. Core Content                        [ALL TIERS]
Primary knowledge payload. Structure varies by content type.

## 3. Patterns and Learnings              [STANDARD+]
Transferable principles. Each pattern includes:
- **Name**: Short identifier
- **What**: The pattern itself
- **When**: Trigger conditions
- **Why**: The insight behind it

## 4. Skill Artifacts                     [DEEP only]
Per skill: purpose, invocation, then one of:
  - Inline SKILL.md (if <200 lines)
  - Public URL + install commands
  - Full specification to recreate from scratch

## 5. Integration Plan                    [STANDARD+]
Numbered steps. Prerequisites listed upfront.
Each step: what to do, where, and how to verify.
THIS IS THE SECTION THAT SOLVES THE ACTIONABILITY GAP.

## 6. Signals                             [STANDARD+]
Suggestions for the receiving system to evaluate.
Not mandates — prompts for the receiver's own judgment.

## 7. Sanitization Notes                  [ALL, if applied]
What was stripped and why. Helps the receiver know
what context is missing and whether to request it.

---

*End of dispatch. Eject capsule when ready.*
```

---

## Lens Sequences by Tier

| Tier | Sequence | Rationale |
|------|----------|-----------|
| Quick | Synthesizer | Clarity and focus only |
| Standard | Synthesizer -> Guardian | Strategic extraction + completeness/risk check |
| Deep | Synthesizer -> Architect -> Guardian -> Sculptor | Full analysis: strategy, systems, protection, then narrative polish |

---

## Usage Examples

**Simple transfer:**
```
/capsule "Transfer the notification hook learnings to RyOS"
```

**Multi-pattern transfer:**
```
/capsule "Package up the lens sequence patterns, compounding loop, and zero-dependency architecture for a new workspace"
```

**Skill replication:**
```
/capsule "Replicate /jira, /today, and /pdf skills for an external system"
```

**Load an existing capsule:**
```
/capsule load ~/ryos/packages/ryos-work-dispatch-capsule.md
```

**Bare invocation (guided):**
```
/capsule
```

## Integration

Works with other ryOS skills:
- **Input from**: Any session context, `/update-from-session` learnings, skill definitions
- **Output to**: `~/ryos/packages/` directory, receiving systems via load mode
- **Related**: `/projector` (sub-skill principles for Deep tier packaging)

## Output Location

Capsules are saved to `~/ryos/packages/[slug]-capsule.md` by default.
