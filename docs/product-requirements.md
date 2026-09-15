# Product requirements — first release

Status: T01 requirements baseline. These are intended behaviors, not implemented features. Open decisions below must be resolved before their dependent implementation; documenting them completes T01 without pretending that they have been decided.

Related documents: [project purpose](../README.md), [implementation tasks](../IMPLEMENTATION-TASKS.md), and [review policy](../Executive-Code-Reviewer.md).

## 1. User and outcome

The primary user is a person preparing computer vision training data on a phone, including situations without internet access. They need to inspect their own images or video frames, manually label them precisely, recover their work, and move a correctly formatted dataset to an external training environment.

Success means a user can complete **Import → Import Review → Labeling Queue → Final Review → Export** offline, understand which images are unfinished or intentionally empty, and load exported data with the explicitly supported training loader. Model accuracy and industrial suitability are not guaranteed by the application.

## 2. Release boundaries

Distinguish two deliverables:

- **Detection prototype (Milestone C):** local image import, review, boxes, persistence, splitting, and one verified detection export. This is an intermediate validation milestone, not the complete first release.
- **First release baseline:** the shared workflow plus manual segmentation tools, video frame input, optional preprocessing/augmentation, editable backups, and reliability checks. The first segmentation mode and exact export consumers remain decisions below. Unsupported modes must not appear as working features.

Required tool coverage for the first-release baseline includes boxes, polygon editing, brush painting, and erasing. D02 defines their segmentation meaning; these tools alone do not imply support for both semantic and instance segmentation.

Owner decisions: **Android first** and **semantic segmentation with brush painting and erasing prioritized**. For this release, segmentation labels represent class regions, not separate object identities. Polygon tools edit/fill class regions; instance segmentation is deferred. Develop and validate the painting workflow before polygon conveniences within Milestone D. A semantic mask export is required alongside the detection export; the exact consumers remain D03.

Deferred unless explicitly added to the release: a second segmentation mode, single-/multi-label image classification, extra exporters beyond those selected for the release, camera capture inside the app, and ZIP/folder import conveniences. Importing supported local photos remains required.

Excluded: AI labeling, automatic segmentation/detection, object tracking, keypoints, rotated boxes, user accounts, hosted datasets, cloud synchronization, collaboration servers, model hosting, and in-app model training.

## 3. Workflow state rules

| Property | Meaning and permitted behavior |
| --- | --- |
| Dataset membership | Staged, included, or excluded. Only confirmed imports enter the active workflow; excluded items never export. |
| Viewed | Unseen or viewed. Opening an image only changes this property. |
| Progress | To label, in progress, ready for review, or approved. Saving is independent of completion. |
| Annotation disposition | Normal or explicitly empty/no target objects. Empty is not the same as unlabeled. |
| Attention flag | Optional flag with a reason; does not silently approve or exclude an image. |

Confirming a kept staged image sends it to **To label**. Confirming an explicitly empty staged image sends it to **Ready for review**, retaining the empty disposition. Final Review must approve it before the default export includes it.

The first annotation edit sets progress to **In progress**. Any annotation edit on an approved or ready-for-review image returns it to **In progress**. “Done” requests **Ready for review** after structural checks. Adding an annotation to an empty image must explicitly clear the empty disposition; marking an annotated image empty requires an explicit, undoable annotation-removal action.

## 4. Functional requirements and acceptance examples

Each acceptance example defines a future verification obligation; none has been executed against an application yet.

| ID | Requirement | Acceptance example | Tasks |
| --- | --- | --- | --- |
| FR01 | Create and reopen local projects with a task and stable class list. | Create two classes, label an image, rename a class, reopen the project: the annotation keeps the same class identity. Class deletion requires reassignment/removal when used. | T03–T05 |
| FR02 | Import supported local images into staging with clear error reporting. | Import valid images and a corrupt file: valid images are previewable, the failed item is identified, and source files remain unchanged. Orientation is consistent in preview, editor, and export. | T06 |
| FR03 | Review incoming data before accepting it. | In one batch, keep A, mark B empty, exclude C, and undo C's exclusion. Nothing enters the active queue until confirmation; confirmed B requires final review and is distinguishable from unlabeled A. Bulk actions give the same results as individual actions. | T07 |
| FR04 | Show image previews and independent progress indicators. | Open an unlabeled image and return to the grid: it is viewed but still To label. Thumbnails show overlays/counts when annotations exist. Filters and counts match stored states. | T08 |
| FR05 | Provide reliable mobile inspection controls. | Pan/zoom then switch among image-only, image-plus-labels, and labels-only: annotation data is unchanged. Fit and original-resolution inspection are available; the maximum is bounded and documented after device testing. | T09 |
| FR06 | Support precise manual box editing. | Create, select, move, resize, reclassify, and delete a box; reopen it and inspect at different zooms: geometry remains aligned and in valid image bounds. | T10 |
| FR07 | Save edits and support undo/redo and recovery. | Draw, resize, delete, undo, and redo, then reopen: saved state is consistent. Interrupt a save: recover a valid last committed state, without reporting unsaved work as saved. | T05, T11 |
| FR08 | Review annotations explicitly. | Attempt completion with a zero-area box or missing class: explain and locate the issue. Approve valid data, then edit it: approval is removed and re-review is required. | T12 |
| FR09 | Create repeatable train/validation/test splits. | With identical input membership, groups, seed, and settings, assignments repeat. No capture group spans splits by default. Ratios must be nonnegative and total 100%; impossible small-dataset allocations are reported honestly. | T13 |
| FR10 | Export compatible detection data. | Export approved positives and empty examples, then load them using the named supported training loader/version: coordinates, class mapping, split membership, and empty-image representation are correct. | T14 |
| FR11 | Preflight and write exports locally. | Summary counts match exported contents; excluded images are absent; unresolved work is identified. Invalid included annotations block export. A failed or canceled write is not presented as a completed dataset. | T15 |
| FR12 | Manually edit segmentation polygons. | Create/close a polygon, edit a vertex, undo and reload: geometry and selected class/instance identity remain correct. Invalid geometry is reported before completion/export. | T17, T18 |
| FR13 | Paint and erase segmentation masks. | At multiple zoom levels, paint and erase with adjustable brush size, undo, save, and reopen: intended pixels and class/instance identities persist. Opacity and viewing modes never modify the mask. | T17, T19a, T19b |
| FR14 | Export the selected segmentation mode without silent information loss. | Export representative boundaries, overlapping objects where supported, and holes where supported. The named consumer loads the results; unsupported geometry produces an explicit restriction rather than a misleading success. Background/ignore meaning is documented. | T20 |
| FR15 | Extract video frames locally into Import Review. | Select a range/interval or individual frames, inspect the extraction preview, then confirm: source ID/timestamps persist. Cancel extraction safely. Related frames remain grouped for splitting. | T21a, T21b |
| FR16 | Apply optional preprocessing consistently. | Resize/pad a labeled fixture: exported image and annotation dimensions align; mask class values remain discrete. Originals stay unchanged and the preview matches the exported transformation. | T22 |
| FR17 | Generate optional geometric training variations. | Rotate/flip a fixture: transformed boxes/polygons/masks match the image, settings are recorded, generated copies obey the configured cap and stay in their source training split. | T23 |
| FR18 | Generate optional color training variations. | Toggle saturation, brightness/exposure, and warmer/cooler temperature independently: preview reflects chosen ranges; disabled changes are absent; labels/masks remain unchanged. | T24 |
| FR19 | Back up and restore an editable project. | Back up a mixed-status project and restore it separately: images, annotations, class IDs, statuses, and provenance agree. Corrupt archives do not replace a valid project. | T25 |

Default exports include approved images only, including approved empty images. Inclusion of unfinished images is an optional later capability, not required for the first release; if implemented it must be explicit and must not bypass structural validation.

Split originals before creating augmented variants. Validation/test data receive required standardizing preprocessing but no augmentation by default. Record the relevant settings and provenance in export metadata. If a single source video must be divided by time ranges, require an explicit choice and explain residual similarity; exact ratios must not override default group integrity silently.

## 5. Non-functional requirements

| ID | Requirement | Verification obligation |
| --- | --- | --- |
| NF01 | Offline and private operation | Exercise all implemented core stages without network connectivity after installation/setup; no account or remote service is required and user media is not uploaded. |
| NF02 | Data integrity | Exercise interruption/recovery, class changes, orientation, and save/reload with representative fixtures; never delete original gallery files when excluding project items. |
| NF03 | Clear errors and resource limits | Permission denial, corrupt inputs, cancellation, and insufficient storage have actionable messages and leave valid existing work intact. Establish supported dimensions/video limits and measured behavior during implementation. |
| NF04 | Practical touchscreen use | On named target devices, verify gesture separation, visible selection/status, precise editing, and reachable controls. Do not rely on color alone for state or class identification. No untested zoom or performance number is a release claim. |
| NF05 | Compatibility evidence | For each advertised format, record consumer/version, conventions, fixtures, and loader result; actual model training is not required merely to verify that a loader accepts data. |
| NF06 | Maintainability | Implemented behavior, significant decisions, persistence schemas, and format changes have matching documentation and appropriate tests. Every pushed change follows the executive review policy. |

## 6. First-release acceptance journey

Use owned or synthetic fixtures with known annotations; do not commit private datasets. Include differently oriented images, empty examples, invalid input, known box/polygon/mask boundaries, and a short video with known timestamps.

1. On a named supported phone, disable connectivity and create a project.
2. Import images, preview them, confirm keep/empty/exclude choices, and verify originals remain intact.
3. Annotate with each supported task/tool, exercise viewing modes, undo/redo, and reopen the project.
4. Review positives and empty images; verify invalid annotations are rejected and changed approved work needs review again.
5. Import selected video frames; confirm timestamps and source grouping survive review and splitting.
6. Export with recorded splits and toggled transformations; inspect label alignment and load outputs with each advertised consumer.
7. Restore a backup into a separate project; compare meaningful content and state.
8. Record relevant error/resource/device checks, resolve blocking findings, and obtain a release-scoped executive verdict.

Completing a detection-only subset proves the prototype milestone, not all first-release requirements. Any deliberate release deferral must name the requirement and update release claims and the backlog together.

## 7. Open decisions and their implementation gates

| ID | Decision | Current status | Must be resolved before |
| --- | --- | --- | --- |
| D01 | Target platform; available test phone(s) and OS | **Android first — owner confirmed.** Minimum Android version and test device inventory remain for T02. | T02 device/tooling decision and T04 scaffolding |
| D02 | First segmentation mode and priority | **Semantic masks, brush painting and erasing first — owner confirmed.** Polygons represent class regions; instance segmentation is deferred. | Use this decision in T03 and T17–T20 |
| D03 | Exact first YOLO detection consumer/version and first segmentation consumer | Open; choose and verify the intended loader during technical planning. No generic YOLO/CNN compatibility claim. | Export contract/schema decisions in T03 and exporter implementation T14/T20 |
| D04 | Framework, persistence, migrations, and backup format | T02/T03 technical decisions pending. | Dependent foundation/persistence work |
| D05 | Supported input types, maximum tested sizes, zoom bounds, and device performance targets | Establish measurable targets through target-device feasibility work; verify before advertising support. | Relevant import/editor/resource acceptance and release |
| D06 | License and distribution route | Open; does not block requirements documentation. | Public application distribution |

Unanswered decisions remain open; elapsed time is not owner approval. T01's deliverable is a testable requirements baseline with explicit decision gates. T02 must resolve the decisions it needs before dependent coding starts.

## 8. T01 completion and next step

T01 covers the primary user, workflow states, prototype versus release scope, explicit exclusions, functional/non-functional acceptance examples, task traceability, and decisions that require resolution. It does not select a framework or implement the app.

Next: **T02 — Select platform and technical approach**, using the confirmed Android-first target and semantic-mask priority. Inspect the available build/device environment, record architecture decisions, and establish export-target feasibility. D03 and the remaining device/version details are still open.
