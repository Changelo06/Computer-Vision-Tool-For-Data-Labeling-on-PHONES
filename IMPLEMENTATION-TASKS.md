# Implementation tasks and usage-aware work sessions

Status: proposed execution plan; application implementation has not started.

Progress: **T01 — Done (requirements documentation)**. See [first-release requirements](docs/product-requirements.md) for acceptance examples and open decision gates. T02 is next; all implementation tasks remain Planned. Completion of T01 does not mean platform/export choices or application behavior are verified.

Owner-confirmed direction from T01: Android first; semantic masks with brush painting/erasing prioritized. Platform tooling, device versions, and exact export consumers still need technical validation.

Follow the [project plan](README.md), [agent instructions](AGENTS.md), and [executive review policy](Executive-Code-Reviewer.md). This backlog sequences the agreed scope without choosing an unresolved technology stack.

## Planning around account availability

Plan work in bounded sessions, not promised daily token counts. Account tooling exposes remaining usage percentages and reset times rather than a guaranteed daily token allocation. Usage depends on model, task complexity, context, and tools; a fixed number of features or messages cannot be inferred from a remaining percentage. See [official Codex usage guidance](https://learn.chatgpt.com/docs/pricing).

Check current five-hour and weekly availability before a work session. They are separate constraints; a five-hour reset does not replenish the weekly allowance. Do not commit personal account identifiers, balances, or usage snapshots to the repository.

### Initial daily rhythm

- Start with **one small or medium task per active workday**, including its verification and documentation. This is a conservative starting pace, not a capacity guarantee or delivery deadline.
- After completing it, check remaining usage before adding another task. Do not consume all available capacity on implementation and leave review unfinished.
- As a planning heuristic, allocate roughly 60% of planned session effort to implementation, 25% to verification/review, and 15% to documentation and handoff. These are effort allocations, not measurable token reservations.
- For steady daily use, divide the weekly capacity the user intends to spend on this project by the remaining planned workdays before its reset. Account for other tasks using the same account. Recalculate when availability changes.
- When capacity is low, select a small task or finish an existing checkpoint rather than starting a broad feature. If verification cannot finish, preserve work locally with a precise handoff and do not push it as approved.
- Never purchase credits, consume a reset, switch the user's model, or schedule recurring runs merely to meet this plan.

### Calibrate after the first three implementation sessions

Keep a private session note with task ID, model, start/end usage readings, whether a reset or concurrent account activity occurred, and actual completion status. Percentage differences are only approximate consumption indicators and cannot isolate this task when other work is running or a reset occurs.

Use those observations to adjust task size. Do not convert them into invented exact token costs. Keep the repository progress record focused on deliverables, evidence, and next steps.

## Task sizing and completion

- **S — Small:** one narrow document, behavior, or validation change.
- **M — Medium:** one feature slice with a clear input, output, and focused verification.
- **Split:** too uncertain or broad to start as one session; first divide it into small/medium tasks with separate acceptance criteria.

Sizes are preliminary, not session-duration estimates. If a medium task grows, split it at a coherent boundary. Every task includes relevant documentation and the executive review; those are not postponed to the end of the project.

Task statuses: **Planned → Active → In review → Done**, with **Blocked** for a specific unresolved dependency. A task is Done only when acceptance criteria and applicable checks pass. A push additionally requires a PASS verdict tied to the candidate commit.

## Milestone A — Decide and establish the foundation

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T01 | S | Define first-release requirements | None | Document target user workflow, explicit exclusions, acceptance examples, and owner decisions needed for platform, segmentation priority, and first export target. |
| T02 | M | Select platform and technical approach | T01 | Record owner-aligned platform/framework/storage decisions, available development and device tooling, alternatives, and a feasible local build/verification path. |
| T03 | M | Define internal project data model | T02 | Document image identity, source groups, coordinates/orientation, stable class IDs, independent workflow states, empty disposition, annotation types, and schema versioning with examples. |
| T04 | M | Create runnable app foundation | T02, T03 | App starts on the chosen target; navigation shell works offline; setup/build/check commands are documented and reproducible. |
| T05 | M | Persist projects and classes | T04 | Create/reopen a project and rename a class without losing class identity; save/reload and invalid-input checks pass. |

## Milestone B — Import and manually label images

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T06 | M | Import local images into staging | T05 | Valid images receive project-owned records; unreadable images produce clear errors; originals remain intact; orientation behavior is verified. |
| T07 | M | Implement Import Review | T06 | Keep, empty, exclude, undo, bulk selection, and batch confirmation work; unconfirmed imports do not silently enter the labeling queue. |
| T08 | M | Build labeling queue and status filters | T07 | Viewed status is independent from completion; thumbnail/status counts and filters agree with stored records. |
| T09 | M | Implement image inspection controls | T08 | Fit, bounded zoom, pan, and image/overlay viewing work without editing annotations; document tested zoom behavior. |
| T10 | M | Draw and edit bounding boxes | T09 | Create/select/move/resize/delete classed boxes; coordinates stay aligned across pan/zoom and save/reload; boundary checks pass. |
| T11 | M | Add edit undo/redo and interruption recovery | T10 | Edit sequences undo/redo correctly; interrupted-save recovery preserves valid state and reports recoverable problems. Basic persistence must already exist in earlier tasks. |
| T12 | M | Implement final review and validation | T11 | Approve/flag/return-to-label works; edits invalidate approval; zero-area geometry, missing classes, and contradictory empty images are detected. |

## Milestone C — First complete detection dataset

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T13 | M | Create repeatable source-grouped splits | T12 | Ratios and seed are recorded; no source group spans splits; small/imbalanced datasets produce transparent outcomes instead of fabricated exact ratios. |
| T14 | M | Implement the selected detection exporter | T13 | Named format/version, class mapping, coordinates, empty-image representation, split folders and metadata are documented; representative output loads in the chosen training loader. |
| T15 | M | Add export preflight and local file output | T14 | Counts and exclusions are accurate; invalid labels block export; generated archive can be inspected and imported; failure never masquerades as success. |
| T16 | M | Validate the offline detection journey | T15 | On a stated target device, import → label → save/reopen → review → split → export works without connectivity; document failures and limitations. |

Milestone C is a **detection prototype**, not the complete planned app. It does not remove segmentation from the product scope.

## Milestone D — Manual segmentation

T01 confirms semantic segmentation first, prioritizing brush painting/erasing. Within this milestone, implement T17 → T19a → T19b before T18, then T20. Polygons represent semantic class regions. T02 can move segmentation earlier in the overall prototype sequence while retaining its dependencies and validated exports.

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T17 | M | Implement segmentation storage rules | T03, T12 | Chosen mode has explicit instance or pixel-class identity, overlap/background/ignore rules, serialization, and representative fixtures. |
| T18 | M | Implement polygon editing | T17, T09 | Create/close/edit/delete polygons with validity checks, undo/redo, and reliable reload; retain instances when the selected mode requires them. |
| T19a | M | Implement brush and eraser operations | T17 | Brush strokes/erasures modify the intended mask pixels; class values, edges, and overlap rules are verified on fixtures. |
| T19b | M | Integrate painting into the mobile editor | T19a, T09, T11 | Brush size, opacity, labels-only view, zoomed strokes, undo/redo, and recovery work together on the target device. |
| T20 | M | Export and validate the first segmentation mode | T18, T19b, T15 | The chosen consumer loads representative masks/polygons; unsupported holes or geometry cause explicit handling rather than silent loss. |

Support for a second segmentation mode is a follow-on task group requiring its own storage, editing, and export acceptance criteria; T20 does not imply both modes are complete.

## Milestone E — Video and dataset transformations

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T21a | M | Extract video frames locally | T06 | Interval/range extraction preserves video identity and timestamps; rotation, invalid inputs, and bounded resource behavior are checked. |
| T21b | M | Add frame selection and import preview | T21a, T07, T13 | Frame count/storage preview, manual selection and cancellation work; extracted frames pass through Import Review and retain grouping for splits. |
| T22 | M | Add resize/padding preprocessing | T15, T20 | Previewed output dimensions match exports; boxes/polygons/masks stay aligned; class-ID masks keep discrete values. |
| T23 | M | Add rotation and flip augmentation | T22 | Transformed labels match images; source originals remain intact; derived images stay in their source training split; settings are recorded. |
| T24 | M | Add color augmentation controls | T23 | Independent saturation, brightness/exposure and temperature controls have toggles/ranges/previews; masks are unaffected; generated copies are limited. |

## Milestone F — Additional outputs and reliability

| ID | Size | Task | Depends on | Acceptance / evidence |
| --- | --- | --- | --- | --- |
| T25 | M | Implement full project backup and restore | T20, T21b | Editable annotations, classes, statuses and provenance survive round-trip; corrupt/unsafe archives fail safely; existing data is not overwritten silently. |
| T26 | Split | Add additional task types/exporters | T16, relevant data model | Separate single-label classification, multi-label classification, second segmentation mode, and each extra exporter into individual tasks; each has a named consumer and verified fixtures. |
| T27 | Split | Harden resource and device behavior | T24, T25 | Split into storage-exhaustion, large-image/video memory, cancellation, and device-interaction tasks; record actual tested limits. |
| T28 | M | Prepare the scoped release | T26 and T27 as scoped for release | Reconcile implemented features with release claims, finish user/setup guidance, run applicable regression/device checks, and issue executive review. Explicitly list deferred features. |

## First work sessions

1. **Completed: T01.** First-release requirements and acceptance examples are documented; pending owner/technical decisions are explicit gates for dependent work.
2. **Next session: T02.** Establish the build/device path and record technical decisions. Do not scaffold an arbitrary framework before this is resolved.
3. **Following session: T03.** Establish the durable annotation and workflow model to reduce later rework.
4. **Then T04.** Deliver the first runnable foundation; use actual consumption from these sessions to revise the daily pace.

Do not interpret these as guaranteed calendar days. There is no defensible completion date until initial implementation and tooling risks are measured.

## Session handoff template

```text
Task ID and status:
Goal and acceptance criteria:
Relevant files and decisions:
What changed:
Checks performed and results:
Known limitations / blockers:
Base and candidate commit:
Executive review verdict:
Exact next step:
```

Keep handoffs short and specific so the next session can continue from repository evidence without replaying the entire conversation. Reuse established tests and fixtures, inspect only relevant code, and avoid broad refactors or unrelated improvements inside a bounded task.
