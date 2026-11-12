# RFC: Support for Hierarchical Skill Suites (Progressive Disclosure) & Runtime Telemetry

## Summary
As AI agent setups mature from 5–10 flat skills to 50+ specialized capabilities, a flat list causes two major problems:
1. **Context Bloat & Instruction Drift:** Loading dozens of skills simultaneously degrades model performance. Industry architectures solve this via **Progressive Disclosure** (Master Suites that expose high-level triggers and lazily load child skills on demand).
2. **Operational Blind Spots:** Users cannot identify which skills are actively used vs. dead weight burning tokens without adding value.

We propose a phased, non-breaking enhancement to SkillsGate:
1. **Phase 1 (Backend/Data Model):** Support hierarchical parent-child relationships in the scanner and cache (`isSuite`, `parentSuite`).
2. **Phase 2 (UI):** Add an optional "Group by Suite / Buckets" view in the SkillsGate dashboard.
3. **Phase 3 (Telemetry):** Read local session logs in read-only mode to surface usage counts and identify "Zombie / Stale" skills.

---

## Motivation & Use Case

Consider a developer using a modern design suite:
```
~/.agents/skills/
├── design-suite/
│   ├── SKILL.md                 <-- Master Suite Orchestrator
│   ├── impeccable/SKILL.md      <-- Child Sub-skill (Anti-slop & polish)
│   ├── shadcn/SKILL.md          <-- Child Sub-skill (Component library)
│   └── wcag-audit/SKILL.md      <-- Child Sub-skill (Accessibility)
└── code-quality-suite/
    ├── SKILL.md
    └── tdd-clean-code/SKILL.md
```

### Current Behavior:
In `ipc-handlers.ts`, if a folder has `SKILL.md`, SkillsGate treats it as a single flat skill and ignores child skills, or flattens nested folders into an unorganized list without parent grouping.

### Proposed Behavior:
1. **Suite Recognition:** SkillsGate recognizes `design-suite` as a Master Suite and groups `impeccable`, `shadcn`, and `wcag-audit` under it.
2. **Zero Breaking Changes:** Flat view remains fully supported and continues to be the default unless the user switches to "Suite View".

---

## Implementation Plan (Atomic PRs)

We are ready to contribute this work via modular, staged PRs:

- **PR #1 (Backend Only):** Detect nested suites in scanner, add `parent_suite` and `is_suite` columns to SQLite `cached_skills`. No UI changes, 100% backward-compatible.
- **PR #2 (UI View):** Add "Suites / Buckets" layout option in `home.tsx` to visualize nested skill groups.
- **PR #3 (Telemetry & Pruning):** Basic invocation counters and "Unused in 30d" filter.

We welcome feedback from the maintainers before submitting PR #1!
