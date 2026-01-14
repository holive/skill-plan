---
name: plan
description: Bridge codemap understanding and implementation. Cross-references ticket requirements with code structure to surface ambiguities, side effects, and alternatives before coding begins. Produces actionable implementation plans.
---

# Plan

Create implementation plans that bridge understanding (codemap) and execution. Surfaces what tickets don't mention.

## The Difference

| Diving straight in | Using /plan first |
|--------------------|-------------------|
| Discover problems mid-implementation | Surface problems upfront |
| Rework when side effects appear | Side effects documented before starting |
| Guess at scope | Explicit file list with line ranges |
| One approach assumed | Alternatives with trade-offs |
| "Done" is unclear | Verification criteria defined |

## Invocation

```
/plan "add user authentication to the API"
/plan --ticket ./tickets/AUTH-123.md
/plan --codemap ./codemap-auth.md "implement OAuth flow"
/plan --ticket issue.md --codemap codemap-api.md
```

## Input Sources

| Input | Required | Source |
|-------|----------|--------|
| Ticket description | Yes | Inline text OR file path (--ticket) |
| Codemap | Optional | Auto-detect OR explicit (--codemap) OR generate on-demand |

## Workflow

### 1. Parse Input

Extract ticket description:
- If inline text: use directly
- If `--ticket path`: read file contents

Locate codemap:
- If `--codemap path`: use that file
- Otherwise: search for recent codemap files (< 1 hour old) matching topic keywords
- If no codemap found: **ask user**: "No relevant codemap found. Generate one now, or proceed without?"

### 2. Check for Fast Path

Before deep analysis, check if this is a trivial change:

**Triggers for fast path:**
- Single file explicitly mentioned ("fix typo in README.md")
- Clear, narrow scope with no cross-cutting implications
- Pure additions with no dependencies (new isolated utility function)
- Documentation-only changes

If fast path applies, skip to **Quick Plan Output** (see below).

### 3. Cross-Reference Ticket with Codemap

Map requirements to code:
- Which files in codemap relate to each requirement?
- Are there files the ticket implies but codemap doesn't cover?
- Do any gotchas from codemap directly impact this ticket?

If codemap has concurrency sections, flag any requirements touching async code.

### 4. Code Examination

Read the relevant files identified from codemap. For each:
- Trace dependencies and call chains
- Look for patterns that might conflict with proposed changes
- Identify hidden state or side effects
- Note any validation or invariants that must be preserved

### 5. Analysis

#### Risk Classification

Classify as **low**, **medium**, or **high**:

| Level | Criteria |
|-------|----------|
| Low | Single file, no external APIs, no data mutations, styling/docs only |
| Medium | Multiple files, new feature, database reads, API integration |
| High | Security-related, payment handling, data migrations, multi-service changes, breaking API changes |

Risk level determines plan depth:
- **Low**: minimal plan (scope + change + verify)
- **Medium**: standard plan (all sections)
- **High**: detailed plan with rollback strategy and extra verification

#### Scope Definition

List explicitly:
- Files to **modify** (with line ranges if possible)
- Files to **create**
- Files to **delete**
- External dependencies (new packages, APIs, services)

#### Ambiguity Detection

Scan for gaps between ticket and code reality. Document each as:
```
- [ ] [ambiguity] - assumed: [your assumption]
```

Common ambiguities:
- Ticket says "update X" but X has 3 different implementations
- Ticket implies behavior change but doesn't specify edge cases
- New feature needs data but ticket doesn't mention schema changes
- Integration point exists but error handling undefined

**Do not block on ambiguities** - document them and continue. User decides later.

#### Side Effect Discovery

Compare ticket scope with actual code dependencies:
- Will this change affect other features? (shared state, common utilities)
- Are there consumers of code being modified? (API clients, importers)
- Does the change violate any documented invariants?
- Could this cause performance regression?

For each side effect:
```
- [side effect]: [impact and mitigation]
```

#### Alternative Approaches

When multiple valid paths exist, document options:

```
### Option A: [name] (recommended)
- Approach: [description]
- Pros: [list]
- Cons: [list]

### Option B: [name]
- Approach: [description]
- Pros: [list]
- Cons: [list]
```

Only include alternatives when meaningfully different (don't list trivial variations).

### 6. Generate Plan Output

Save to: `plan-[ticket-slug].md` next to codemap file (or current directory if no codemap).

## Output Formats

### Standard Plan (medium/high risk)

```markdown
# Implementation Plan: [ticket title]

## Risk Assessment
- Level: [low | medium | high]
- Reason: [why this risk level]

## Scope Analysis
- Files to modify: [list with line ranges if possible]
- Files to create: [list]
- Files to delete: [list]
- External dependencies: [any new packages, APIs, etc.]

## Ambiguities & Assumptions
> Items that need clarification or where assumptions were made

- [ ] [ambiguity 1] - assumed: [assumption]
- [ ] [ambiguity 2] - assumed: [assumption]

## Potential Side Effects
> Things discovered from code patterns that the ticket didn't mention

- [side effect 1]: [impact and mitigation]
- [side effect 2]: [impact and mitigation]

## Alternative Approaches (if applicable)
> When multiple valid paths exist

### Option A: [name] (recommended)
- Approach: [description]
- Pros: [list]
- Cons: [list]

### Option B: [name]
- Approach: [description]
- Pros: [list]
- Cons: [list]

## Implementation Steps
1. [step 1] - [file(s) affected]
2. [step 2] - [file(s) affected]
...

## Verification Criteria
- [ ] [how to verify step 1]
- [ ] [how to verify step 2]
- [ ] [overall acceptance test]
```

### Quick Plan (low risk / fast path)

```markdown
# Quick Plan: [title]

Scope: [single file or narrow scope]
Change: [what to do]
Verify: [how to check]
```

### High-Risk Addendum

For high-risk changes, add after Verification Criteria:

```markdown
## Rollback Strategy
- Abort points: [where we can safely stop if issues arise]
- Rollback steps: [how to reverse changes]
- Data recovery: [if applicable, how to restore data]

## Pre-Implementation Checklist
- [ ] [required approval / security review]
- [ ] [backup created]
- [ ] [feature flag in place]
```

## Quality Checklist

Before completing the plan:

- [ ] Risk level is justified (not just guessed)
- [ ] All files mentioned exist (paths verified)
- [ ] Ambiguities are documented, not hidden
- [ ] Side effects traced through actual dependencies (not assumed)
- [ ] Implementation steps are in correct order
- [ ] Verification criteria are testable (not vague)
- [ ] For high-risk: rollback strategy defined

## Hard Rules

1. **Never skip codemap context** - if relevant codemap exists, use its gotchas and patterns

2. **Document, don't block** - ambiguities are noted for user decision, not used as excuse to stop

3. **Scope is explicit** - every file that will change must be listed. No "and related files"

4. **Side effects from code, not imagination** - trace actual dependencies, don't invent hypothetical issues

5. **Alternatives only when meaningful** - don't list options that differ by a single line

6. **Risk matches reality** - security changes are high-risk even if small. Large changes can be low-risk if isolated

## Integration with Codemap

The plan should leverage codemap sections:

| Codemap Section | How Plan Uses It |
|-----------------|------------------|
| Quick Start | Starting point for code examination |
| Gotchas | Direct input to ambiguities and side effects |
| Concurrency Map | Flag any async boundaries affected |
| Files That Change Together | Validate scope includes co-change files |
| Error Handling Pattern | Inform implementation steps |
| Key Types | Reference in scope analysis |

## Example Invocations

### Basic usage
```
> /plan "add rate limiting to the API endpoints"

Searching for relevant codemap... found: codemap-api.md (12 minutes ago)

Reading ticket requirements...
Cross-referencing with codemap...
Examining affected files...

Generated: plan-rate-limiting.md

Summary:
- Risk: Medium (new middleware, touches all endpoints)
- Scope: 4 files to modify, 2 to create
- Ambiguities: 2 (rate limit values not specified, per-user vs per-IP unclear)
- Side effects: 1 (existing integration tests may need timeout adjustment)
```

### With explicit inputs
```
> /plan --ticket tickets/AUTH-456.md --codemap codemap-auth.md

Reading ticket from tickets/AUTH-456.md...
Using codemap: codemap-auth.md

...

Generated: plan-auth-456.md
```

### No codemap found
```
> /plan "refactor the payment module"

Searching for relevant codemap... none found.

Options:
1. Generate codemap first (recommended for complex changes)
2. Proceed without codemap (may miss patterns and gotchas)

Which would you like?
```
