# Issue Tracker & Specs

This repository tracks specifications, requirements, and development tasks locally in Markdown files according to the **Spec-Driven Agentic Development (SDAD) 4-Gate Protocol**.

## Conventions & File Paths

- **Gate 1 (Requirement Specs):** `specs/requirements/XXXX-slug.md` (e.g., `specs/requirements/0001-historical-pattern-search.md`)
  - Sequenced monotonically from `0001` to `9999`.
  - Must include **User Value Rationale** and **Scope vs. Negative Scope**.
  - Must obtain explicit user approval before proceeding.
- **Gate 2 (Test Specs / Golden Evals):** `specs/tests/XXXX-slug.md`
  - 1-to-1 match with the requirement spec ID and slug.
  - Frozen once approved.
- **Gate 3 (Implementation Tickets & Tasks):**
  - Minor tasks/sub-tasks: `.scratch/<feature-slug>/issues/NN-<slug>.md`
  - Or GitHub Issues when the user works via `gh` CLI (`gh issue create`).
- **Gate 4 (Change & Decision Records):** `specs/changes/XXXX-slug.md`

## When a Skill Says "Publish to the Issue Tracker" or "To-Spec"

- For technical specifications: Write to `specs/requirements/XXXX-slug.md` following the Gate 1 requirements template.
- For break-down task tickets: Write to `.scratch/<feature-slug>/issues/NN-<slug>.md` (numbered from `01`).

## When a Skill Says "Fetch the Relevant Ticket"

- Read the file from `specs/requirements/XXXX-slug.md` or `.scratch/<feature-slug>/issues/`.
- If a GitHub issue number `#NN` is passed and `gh` is available, view via `gh issue view NN`.
