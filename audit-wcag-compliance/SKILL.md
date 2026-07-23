---
name: audit-wcag-compliance
description: >
  Guides an agent through a full WCAG conformance audit. Use when a user asks
  for a compliance check, conformance audit, or accessibility report against
  WCAG 2.0, 2.1, or 2.2 at Level A, AA, or AAA.
compatibility: Requires a11y MCP server (a11y_manifest, a11y_read, a11y_search)
metadata:
  tags: [wcag, a11y, audit, compliance, accessibility]
  mcp-prerequisites:
    - name: a11y
      tools: [a11y_manifest, a11y_read, a11y_search]
      docs: "https://wiki.tools.csod.svc/display/ER/Accessibility+MCP+Guide"
---

# WCAG Conformance Audit

Run a structured WCAG conformance audit that generates a checklist, reviews each
success criterion, and produces an audit report with pass/fail status.

## Prerequisites

Before proceeding, verify the required MCP server is connected:

1. **Required:** Accessibility (a11y) MCP server
2. **Verify:** Run `a11y_manifest()` — if this returns available operations, proceed.
3. **If not connected:** Ask the user to set up the a11y MCP server:
   - **HTTP (recommended):** Add to your MCP config:
     ```json
     { "a11y": { "type": "http", "url": "https://flare.docs.int.csod.svc/mcp-a11y" } }
     ```
   - **Setup guide:** [Accessibility MCP Guide](https://wiki.tools.csod.svc/display/ER/Accessibility+MCP+Guide)

Do not proceed until the MCP server is confirmed accessible.

## When to Use This Skill

Use this skill when the user:
- Asks for a "WCAG audit", "conformance check", or "compliance report"
- Wants to verify a page or application meets a specific WCAG level
- Needs a structured accessibility report with pass/fail per success criterion
- Mentions "audit", "conformance", or "compliance"

Do NOT use this skill when:
- The user asks about a single component → use `analyze-component-accessibility` instead
- The user has a quick question about a specific SC → call `a11y_read` directly
- The user wants to search WCAG content → call `a11y_search` directly

## Scoping the Audit

Before launching the workflow, determine these parameters:

### Conformance Level

| Level | When to choose |
|-------|---------------|
| **AA** (default) | Standard target for most projects. Covers A + AA criteria. |
| **A** | Minimum baseline. Use for legacy systems or initial assessments. |
| **AAA** | Enhanced accessibility. Use when required by policy or regulation. |

If the user doesn't specify, default to **AA**.

### WCAG Version

| Version | When to choose |
|---------|---------------|
| **2.2** (default) | Latest standard. Includes new SCs for cognitive and mobile. |
| **2.1** | Use when the project explicitly targets 2.1 compliance. |
| **2.0** | Legacy. Use only when required by existing contracts or regulations. |

If the user doesn't specify, default to **2.2**.

### Audit Scope

Ask the user what pages or flows to audit. Examples:
- "Login page and checkout flow"
- "The entire application"
- "Just the navigation component"

A narrower scope produces faster, more actionable results.

## Running the Audit

Launch the conformance audit workflow:

```
start_workflow('conformance_audit', {
  level: '<A|AA|AAA>',
  version: '<2.0|2.1|2.2>',
  scope: '<optional description of pages/flows>'
})
```

The workflow handles:
1. **Checklist generation** — fetches all required SCs for the chosen level/version
2. **Systematic SC review** — evaluates each SC with techniques (parallelized)
3. **Custom guidelines check** — includes org-specific requirements
4. **Report generation** — produces a pass/fail matrix with remediation steps

Follow the workflow instructions returned by `start_workflow`. Do not duplicate
the workflow logic here — the workflow content is authoritative.

## Interpreting Results

After the workflow completes, help the user understand the report:

### Priority Order for Remediation

1. **Level A failures** — these are critical. Fix first.
2. **Level AA failures** — required for standard compliance. Fix next.
3. **Level AAA failures** — enhanced. Fix if targeting AAA.
4. **Custom guideline failures** — org-specific. Prioritize per policy.

### Common Failure Patterns

| Pattern | Typical SCs | Quick Fix |
|---------|------------|-----------|
| Missing alt text | 1.1.1 | Add descriptive `alt` attributes |
| Low contrast | 1.4.3, 1.4.6 | Adjust color values to meet ratio |
| No keyboard access | 2.1.1 | Add keyboard event handlers |
| Missing form labels | 1.3.1, 4.1.2 | Associate `<label>` with inputs |
| No skip navigation | 2.4.1 | Add skip link to main content |

### Using Tools for Follow-Up

After reviewing the report, use these tools for deeper investigation:

- `a11y_read(operation: "get_success_criterion", params: {sc: "<id>", version: "<ver>"})` — get full SC details
- `a11y_read(operation: "get_techniques_for_sc", params: {sc: "<id>"})` — get implementation techniques
- `a11y_search(query: "<topic>")` — search for guidance on specific topics

## Error Handling

- If the workflow fails to generate a checklist, report the error to the user
- If individual SC fetches fail, note them as "Error" in the report and continue
- If more than 20% of SCs fail to load, ask the user whether to continue with a partial audit or retry
