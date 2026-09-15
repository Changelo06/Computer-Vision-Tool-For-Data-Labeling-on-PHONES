# Executive Code Reviewer

## Role and authority

Act as the project's final technical reviewer before a push. Inspect the proposed changes, the relevant surrounding codebase, and verification evidence against this policy and the [project plan](README.md). Produce a reasoned verdict; do not approve based only on code appearance or the author's claims.

This document is an instruction for human and coding-agent reviewers. It does not execute itself, install a Git hook, or enforce GitHub branch protection. Repository agent instructions require this review; automated enforcement can be added when the build and test system exists. A passing review is a quality decision, not permission to publish beyond the user's authorized scope.

## Vision

Make dependable computer vision dataset preparation accessible on a phone, anywhere, without an internet connection. Users should be able to produce well-defined, reviewable training datasets through precise manual annotation and transparent exports.

## Mission

Deliver a focused offline application that lets users inspect imported images and video frames, manually annotate them, review their work, prepare reproducible dataset splits and optional transformations, and export files that have been verified against explicitly supported training pipelines.

Protect the user's source data and labeling effort. Keep progress understandable, interactions practical on a touchscreen, and documentation accurate as the codebase evolves. Dataset quality and model performance must not be guaranteed merely because an export succeeds.

## Non-negotiable product policy

1. Core dataset operations work locally without required accounts, servers, or network access. Do not introduce undisclosed uploads or telemetry carrying user data.
2. Annotation remains manual. AI labeling, automatic detection, and automatic segmentation are outside the approved scope.
3. Preserve the workflow: Import Review → Labeling Queue → Final Review → Export. Opening an image must not mark it completed.
4. Keep viewed state, annotation progress, intentional empty-image disposition, and exclusion distinct. Editing approved annotations returns them to review.
5. Preserve original source files and recover saved work after interruptions. Do not silently discard edits or annotations.
6. Geometry, class identity, masks, and image orientation remain consistent through editing, persistence, and transformations.
7. Split original data before augmentation. Keep derived images in their source split and related capture groups together by default. Apply augmentation to training data by default.
8. Export conventions must be explicit and tested. Never silently lose annotation meaning in conversion or advertise generic compatibility with every YOLO, CNN, or computer vision model.
9. Keep the interface focused on mobile manual labeling. In-app training, cloud services, and collaboration are outside the initial scope unless the project owner explicitly revises it.
10. Document implemented behavior, known limitations, and significant technical decisions in the same change that introduces them.

## Review scope and development stage

Review all proposed commits, not only the last commit. Read related modules and callers to understand impact. For an initial review, inspect the repository baseline; for later reviews, inspect the complete change against a recorded base revision and trace affected behavior through the existing codebase.

Distinguish **ready to push this change** from **ready to release the application**. Incremental and documentation-only changes may pass without every planned feature existing. They must not break existing behavior or present unimplemented features as complete. Apply checks to the implemented scope and record a reason for every not-applicable category.

Do not choose unresolved platforms, frameworks, zoom limits, or export versions on behalf of the owner merely to finish a review. Evaluate against recorded decisions. Identify only decisions that actually block the proposed change.

## Review procedure

1. **Establish the candidate:** record the base revision, candidate commit, working-tree status, changed files, intended behavior, and development stage. Include uncommitted content in preliminary reviews; final approval must identify the committed content that will be pushed.
2. **Read the requirements:** inspect this policy, README, applicable agent instructions, and relevant requirements, architecture decisions, data-model, and export documentation that actually exist.
3. **Trace implementation:** inspect the changed behavior and relevant surrounding code. Look for regressions, incomplete paths, hidden network dependencies, data loss, invalid geometry, and scope violations.
4. **Verify proportionately:** run the available build, static checks, and meaningful tests appropriate to the change. Use documented commands; do not invent results or substitute a successful build for behavioral checks. Inspect relevant device/manual evidence when interaction or persistence is affected.
5. **Check documentation:** confirm feature claims, examples, schemas, setup steps, and compatibility statements match the implementation. Documentation-only changes need content/link/consistency checks, not nonexistent application tests.
6. **Report actionable findings:** cite file and line, concrete trigger, expected versus actual behavior, impact, and required correction. Separate demonstrated defects from unverified concerns.
7. **Resolve and reassess:** fix issues within the authorized scope or report them. Re-run affected checks after corrections. Do not waive failures merely to produce a passing verdict.
8. **Issue the verdict:** use the report format below. Any subsequent code or configuration change invalidates approval for the earlier candidate and requires review of the new candidate.

## Evidence checklist

For each applicable category, record **PASS**, **FAIL**, or **NOT VERIFIED**, with evidence. Use **NOT APPLICABLE** only with a scope-based explanation.

| Category | Evidence to seek when affected |
| --- | --- |
| Scope and offline operation | Implemented paths match the mission; core workflow exercised without connectivity; no required remote service introduced. |
| Import and source handling | Keep/empty/exclude behavior is distinct; unreadable inputs fail clearly; original files remain intact; video frames retain source and timestamp. |
| Annotation correctness | Boxes/polygons/masks retain correct coordinates and classes after edit, zoom, orientation changes, undo/redo, and save/reload; display toggles do not mutate labels. |
| Workflow and review | Viewed status is independent; completion is explicit; edits invalidate approval; empty images cannot retain contradictory annotations. |
| Persistence and recovery | Interrupted saves do not corrupt projects; recovery and backup/restore preserve images, classes, annotations, and status; schema changes have a migration strategy. |
| Splits and transformations | Recorded settings reproduce assignments; source groups and derived images do not leak across splits; transformations preserve label alignment and discrete mask class IDs. |
| Export integrity | Valid class mappings, paths, dimensions, coordinates, and empty-image representation; invalid annotations are rejected; representative output loads in the named supported training loader/version. |
| Mobile usability | Relevant gestures, selection, zoom, brush/eraser, and navigation are exercised on a stated device or emulator; any remaining physical-device checks are disclosed. |
| Resource handling | Affected large-image/video/export operations have bounded memory behavior, clear progress/failure handling, and safe handling of storage exhaustion or interruption. |
| Code quality | Responsibilities are understandable; errors are handled; dependencies serve the agreed scope; changed behavior has appropriate tests rather than tests that only mirror implementation. |
| Privacy and repository hygiene | No credentials or private datasets committed; imported paths/archive entries cannot escape intended storage; permissions match the feature's needs. |
| Documentation | Claims reflect actual support; relevant format, architecture, testing, and user guidance are updated. |

Do not treat every row as mandatory work for every change. For example, changing prose does not require a device test. Conversely, changing an exporter cannot pass on prose review alone. Record environment limitations honestly; required checks that cannot run remain unverified.

## Finding severity

- **Critical:** data loss/corruption, credential or private-data exposure, or another immediate severe failure.
- **Major:** broken core behavior, incorrect labels/exports, dataset leakage, mission violation, or a material regression.
- **Minor:** a limited defect with no material effect on correctness, recoverability, or the agreed workflow.
- **Suggestion:** optional improvement without a demonstrated defect; not a blocker by itself.

Critical and major findings block a push. A minor finding may remain only when its limited impact and follow-up are recorded and it violates no mandatory acceptance criterion. Avoid using stylistic preferences or speculative issues as blocking findings.

## Final verdict rules

Choose exactly one:

- **PASS — READY TO PUSH:** all applicable required checks passed, no critical/major findings remain, mandatory acceptance criteria are met, and evidence matches the candidate commit. List any non-blocking follow-ups.
- **FAIL — CHANGES REQUIRED:** a demonstrated blocking defect, policy violation, or failed required check remains. State what must change and how to verify the correction.
- **BLOCKED — VERIFICATION INCOMPLETE:** required evidence is unavailable or a necessary decision prevents a defensible conclusion. State the exact missing check, constraint, or decision and the next action. Do not equate unavailable evidence with success.

Only PASS satisfies this review policy for pushing the candidate. FAIL and BLOCKED require correction or completion of the missing verification before a new verdict. A review verdict does not authorize deployment or assert that the whole application is release-ready.

## Required review report

Include the report in the task response or pull request; a permanent report file is optional. Do not put private datasets, secrets, or sensitive logs in it.

```text
Executive Code Review
Base revision:
Candidate commit:
Scope and development stage:
Working-tree status:

Vision and mission alignment:
Files/modules inspected:

Checks and evidence:
- [PASS / FAIL / NOT VERIFIED / NOT APPLICABLE] Check — evidence or reason

Findings:
- Severity — file:line — trigger, impact, required correction

Non-blocking follow-ups:
Verification limitations:
Release readiness: Not assessed / Not ready / Ready within stated release scope

FINAL VERDICT: PASS — READY TO PUSH
              OR FAIL — CHANGES REQUIRED
              OR BLOCKED — VERIFICATION INCOMPLETE
Reason:
Required next action:
```

## Final reviewer instruction

Before issuing your verdict, ask: **Does this candidate preserve the project's offline, manual computer vision labeling mission, work as claimed within its stated scope, protect user data, and have sufficient verification evidence?**

Approve only when the evidence supports the answer. Never fabricate checks, conceal failures, weaken this policy merely to pass a change, or claim release readiness from a partial review. If the owner changes the mission or scope, record that decision explicitly and update the relevant documentation before assessing against the revised policy.
