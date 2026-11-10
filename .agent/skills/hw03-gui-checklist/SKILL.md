---
name: hw03-gui-checklist
description: Systematically design, critically review (human-in-the-loop AI gap analysis), and execute a >40-item GUI Checklist across IA-01 (General UI), IA-02 (Forms), IA-03 (Navigation), and IA-04 (Feedback/State) for EShop screens, producing Markdown/Excel checklists, failure screenshots, bug reports, and AI audit logs.
---

# HW03 Skill 1 — GUI Checklist Design, Review & Execution

You are a **disciplined GUI testing assistant** operating at Bloom-AI levels **G9.3 (Analyse)** and **G9.4 (Collaborate)**. Never treat GUI testing as a single black-box prompt. Follow these steps strictly one by one.

## Hard Rules
1. **One phase at a time with Human Review:** Present the generated files for the current step and wait for the student's review.
2. **Git Commit ONLY After Explicit Approval:** NEVER run `git add` or `git commit` automatically right after generating files. Always wait until the student explicitly approves the work (e.g., says "approved", "oke", "làm tiếp đi"), and only then commit the approved changes.

## Inputs
1. Read `.agent/hw03-config.json` for student metadata, selected screens, and Interface Aspect (`IA-01`..`IA-04`) mappings.
2. Read `README.md` (specifically `FR-01`..`FR-24`) for ground-truth business and GUI requirements.
3. Inspect the React source components (`frontend-web/src/App.jsx` and target page `.jsx` files) to understand the DOM structure.

## Step-by-Step Workflow

### Step 1: Initial AI-Generated Checklist (`01-ai-initial-checklist.md`)
- Generate an initial baseline checklist of **40 items** covering all 4 Interface Aspects across the selected screens:
  - `IA-01`: General UI Standards (language, button colors, currency format, single `<h1>`, Tab order).
  - `IA-02`: Forms (`*` on required labels, `type="email"`, `type="password"`, error placement above submit, Step Indicator on multi-step forms).
  - `IA-03`: Navigation (active navbar link highlight, cart badge count, `'Đăng xuất'` label, breadcrumbs on sub-pages).
  - `IA-04`: Feedback & State (empty state illustration/message, confirmation before destructive action, validation/success feedback, image/icon accessibility).
- Save raw output to `HW03/task1-gui/01-ai-initial-checklist.md` and log the prompt/output in `HW03/ai-audit/`.

### Step 2: Critical Human Review & AI Gap Analysis (`02-human-review.md`)
- Review the initial 40 items for redundancy, vagueness, or missing edge cases.
- Add **15+ human-contributed items** covering blind spots that LLMs typically miss when prompted from text specifications alone:
  1. **Accessibility (WCAG 2.1 AA):** `<label htmlFor>` to `<input id>` programmatic binding, color contrast ratios (e.g., `text-gray-400` on white), visible keyboard focus ring (`:focus-visible`), screen-reader ARIA live regions for errors.
  2. **Layout & Viewport:** Mobile responsive overflow at `375px` width, browser zoom at `200%`, long Vietnamese diacritic strings / text wrapping in tables, RTL / flex alignment resilience.
  3. **Browser & Form Ergonomics:** Password visibility toggle (`Show/Hide`), `autocomplete` attributes (`email`, `current-password`, `new-password`, `tel`), preventing double submission while request is in flight, Dark Mode (`prefers-color-scheme: dark`) readability.
- For **every added item**, explicitly document **why the AI missed it** (prompt scope limitation, lack of visual/computed-style perception, or spec-centrism bias).

### Step 3: Checklist Execution & Screenshot Capture (`03-execution.md` + `.xlsx`)
- Execute all checklist items against the running SUT (`http://localhost:5173`).
- Mark each item as **Passed** or **Failed**.
- Populate the **Notes** column with the exact root cause and DOM/code evidence for every **Failed** item.
- Capture screenshots **for Failed items only** with the student watermark (`<student_email> | <student_id> - <name>`) and save under `HW03/task1-gui/screenshots/`.
- Export the complete checklist and summary statistics to `HW03/task1-gui/HW03_GUI_Checklist.xlsx`.

### Step 4: Bug Reporting (`HW03/bugs/BUG-GUI-*.md` & GitHub Issues)
- Consolidate related failed checklist items into structured bug reports with Steps to Reproduce, Expected vs. Actual Result, Code Location, Severity, and attached screenshots.
- Publish each bug to GitHub Issues and record the Issue URL.
