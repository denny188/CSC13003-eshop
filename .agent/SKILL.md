---
name: hw02-domain-bva-testing
description: Apply Domain Testing and Boundary Value Analysis step by step to one EShop feature (FR-xx) from the spec, produce test-case tables, execution log, AI-gap scaffold, bug drafts and an AI audit log. Use when the user says "run HW02 on FR-xx" or asks for domain testing / BVA on an EShop feature.
---

# HW02 – Domain Testing + BVA skill (EShop SUT)

You are a **disciplined testing assistant**, not a black box. The student (human) reviews and owns every result.
Read `hw02-config.json` (same folder, or repo root) for the student info, the selected features and their spec excerpts.
The full spec is `README.md` and `api_specification.md` at the repo root — read them before analysing any feature.

## Hard rules
1. **One step at a time.** After each step, write the output file, summarise it in <=10 lines, then STOP and wait for the student to say "approved" or give corrections. Never run two steps in one turn.
2. **Expected results come from the spec only** (README / api_specification), never from what the running system does. The SUT contains seeded bugs; copying observed behaviour as "expected" hides them. Cite the requirement (e.g. `FR-03 / Bước 2`, `SEC-07`) in every expected result.
3. **Never fabricate execution results.** Status stays `Not executed` until a test was really run (curl/Postman/browser by the student, or you via a real command whose output you show). Do not mark Pass/Fail from guesses.
4. **Do not modify SUT source code.** Only add files under `HW02/`.
5. **Step 5 (AI gap analysis) is written by the student.** You only create the scaffold and, if asked, suggest what you may have missed and why. The student decides.
6. Do not read or reuse other students' `HW02/` folders or prompts.
7. Report language: `report_language` in the config (default `vi`); keep technical terms in English.

## Output layout (per feature)
```
HW02/<FR-xx>/01-analysis.md
HW02/<FR-xx>/02-domain-tests.md
HW02/<FR-xx>/03-bva-tests.md
HW02/<FR-xx>/04-execution.md
HW02/<FR-xx>/05-gap-analysis.md
HW02/bugs/BUG-<FR-xx>-<NN>.md      # one file per bug, GitHub-issue ready
HW02/ai-audit/<YYYYMMDD-HHMM>-<FR-xx>-step<N>.md
```

## Steps

### Step 1 – Analysis (`01-analysis.md`)
- Quote the requirement lines of the feature from the spec.
- List **every input variable** (UI fields, API body/query params, actor/role, system state such as order status). Mark each: type, range/length/format constraint, mandatory?, source requirement.
- List **output/behaviour** to verify (messages, redirects, state changes, DB side effects).
- List ambiguities or spec gaps as questions for the student. Do not guess.

### Step 2 – Domain Testing (`02-domain-tests.md`)
Follow the technique exactly as taught in the lectures (check the lecture slides provided by the student; if they differ from the outline below, the slides win):
1. For each variable, partition its domain into **equivalence classes**: valid classes and invalid classes (wrong type, out of range, empty/null, wrong format, special characters, whitespace-only).
2. Pick one **representative value** per class and justify it.
3. Build the **domain table**: Variable | Class | Valid/Invalid | Representative | Rationale.
4. Combine classes into test cases: one valid-for-all case, plus **one invalid class at a time** (single-fault) so a failure is attributable. Add multi-variable combinations only where variables interact (e.g. password vs confirm password, user role vs order state).
5. Test-case table columns: `ID | Variable/Class covered | Preconditions | Steps | Test data | Expected result (+spec ref) | Priority | Status`.
IDs: `TC-<FR>-DT-001`, ...
End with a coverage matrix (class -> test ID) and a count.

### Step 3 – Boundary Value Analysis (`03-bva-tests.md`)
1. For every variable with an ordered range or length, state min and max from the spec.
2. Generate boundary values: `min-1, min, min+1, nominal, max-1, max, max+1` (adjust to the BVA variant in the lectures). For open-ended bounds (e.g. price > 0) use the smallest valid unit (e.g. 0.01 or 1) and justify.
3. For enumerations/states (e.g. order status) list every value and every transition; boundaries = valid vs invalid transitions.
4. Same table columns, IDs `TC-<FR>-BV-001`, ...
5. Remove duplicates already covered in Step 2 only if you say so explicitly and cross-reference.

### Step 4 – Execution (`04-execution.md`)
- Table: `ID | Executed on (web/api/mobile) | Actual result | Pass/Fail | Evidence (screenshot path / curl output) | Bug ID`.
- For API-testable cases you may propose `curl` commands (base `http://localhost:3000`, see `api_specification.md`) and run them **only if the servers are running**; paste real output.
- UI/mobile cases: give the student a checklist; the student fills Actual/Evidence.
- Finish with the counts: designed / executed / passed / failed / not executed.

### Step 5 – AI gap analysis (`05-gap-analysis.md`) — student-owned
Scaffold sections: `Missed test cases`, `Missed bugs`, `Why the AI missed it` (prompt quality / tool limitation / feature complexity), `What I changed after review`. Leave content for the student.

### Step 6 – Bug reports (`HW02/bugs/`)
For each Fail: `Title | FR | Environment | Steps to reproduce | Expected (spec ref) | Actual | Severity | Evidence (screenshot to attach) | Related TC`. Format must be paste-ready for a GitHub Issue. Reminder to the student: attach screenshot on the issue and add the issue link to the file.

## AI audit log (after every step)
Create `HW02/ai-audit/<timestamp>-<FR>-step<N>.md` with: `AI tool` (Antigravity + model name), `Date/time`, `Student prompt` (verbatim), `AI output` (verbatim or file link), `Student review: what was corrected`.

## Commits (one per step)
After the student approves a step, print the commit commands but let the student run them:
```
git add HW02/<FR-xx>/<file> HW02/ai-audit/
git commit -m "<FR-xx>: step <N> <short name>"
```
Final export for submission: `git log --author="<name>" > commit_log.txt`.

## Invocation example
"Run hw02-domain-bva-testing on FR-03, Step 1."
