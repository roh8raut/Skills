---
name: analyze-component-accessibility
description: >
  Guides an agent through analyzing a UI component for accessibility issues.
  Use when a user asks to check a component, validate an a11y defect, or get
  WCAG guidance for a specific element.
compatibility: Requires a11y MCP server (a11y_manifest, a11y_read, a11y_search)
metadata:
  tags: [a11y, component, wcag, accessibility]
  mcp-prerequisites:
    - name: a11y
      tools: [a11y_manifest, a11y_read, a11y_search]
      docs: "https://wiki.tools.csod.svc/display/ER/Accessibility+MCP+Guide"
---

# Accessibility Component Analysis

Analyze a UI component or reported defect for accessibility issues, with
configurable focus area, depth, and conformance level.

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
- Asks "is this component accessible?" or "check this for a11y"
- Reports an accessibility defect and wants WCAG guidance
- Wants to analyze a specific UI element (button, form, modal, etc.)
- Needs remediation advice for a component

Do NOT use this skill when:
- The user wants a full-page or full-app audit → use `audit-wcag-compliance` instead
- The user has a quick factual question about WCAG → call `a11y_search` directly
- The user wants to browse WCAG content → call `a11y_read` directly

## Choosing Parameters

### Focus Area

Determines which WCAG principle to emphasize:

| Focus | When to choose | Example triggers |
|-------|---------------|-----------------|
| **general** (default) | Broad analysis across all principles | "check this component" |
| **perceivable** | Text alternatives, contrast, captions, adaptable content | "alt text", "contrast", "image", "video" |
| **operable** | Keyboard access, timing, navigation | "keyboard", "focus", "tab order", "skip link" |
| **understandable** | Labels, error messages, predictable behavior | "form label", "error message", "language" |
| **robust** | ARIA, name/role/value, status messages | "aria", "role", "live region", "screen reader" |

Infer the focus from the user's description. If unclear, use **general**.

### Analysis Depth

| Depth | When to choose | What you get |
|-------|---------------|-------------|
| **standard** (default) | Most requests | Relevant SCs + techniques + remediation |
| **quick** | "Just a quick check", time-sensitive | Top violations only, no technique details |
| **deep** | "Thorough analysis", compliance-critical | All related SCs, custom guidelines, cross-references |

### Conformance Level

| Level | When to choose |
|-------|---------------|
| **AA** (default) | Standard target for most projects |
| **A** | Minimum baseline check |
| **AAA** | Enhanced accessibility requirements |

## Running the Analysis

Launch the analysis workflow:

```
start_workflow('analysis', {
  component: '<description of the component or defect>',
  focus: '<general|perceivable|operable|understandable|robust>',
  depth: '<standard|quick|deep>',
  level: '<A|AA|AAA>'
})
```

The workflow handles:
1. **Component identification** — understands what is being analyzed
2. **SC matching** — finds relevant success criteria for the component type
3. **Technique lookup** — retrieves sufficient and advisory techniques
4. **Remediation guidance** — provides specific fix recommendations

Follow the workflow instructions returned by `start_workflow`.

## Interpreting Findings

### Severity Classification

- **Critical** — blocks users entirely (e.g., no keyboard access, missing alt text on functional images)
- **Major** — significant barrier (e.g., low contrast, missing form labels)
- **Minor** — usability issue (e.g., inconsistent focus indicators, verbose alt text)

### Follow-Up Tools

After the analysis, use these for deeper investigation:

- `a11y_read(operation: "get_success_criterion", params: {sc: "<id>"})` — full SC details
- `a11y_read(operation: "get_techniques_for_sc", params: {sc: "<id>"})` — implementation techniques
- `a11y_search(query: "<topic>")` — search for related guidance
- `a11y_read(operation: "get_conformance_requirements", params: {level: "<level>", includeCustom: true})` — check custom org guidelines

## Common Component Patterns

| Component | Key SCs | Common issues |
|-----------|---------|--------------|
| Button | 4.1.2, 2.1.1 | Missing accessible name, no keyboard handler |
| Image | 1.1.1 | Missing or decorative alt text misuse |
| Form input | 1.3.1, 3.3.2, 4.1.2 | No associated label, missing error identification |
| Modal/dialog | 2.1.2, 1.3.1 | Focus trap missing, no role="dialog" |
| Navigation | 2.4.1, 2.4.5, 1.3.1 | No skip link, missing nav landmark |
| Table | 1.3.1 | Missing headers, no caption |
| Video/audio | 1.2.1–1.2.5 | Missing captions, no audio description |
