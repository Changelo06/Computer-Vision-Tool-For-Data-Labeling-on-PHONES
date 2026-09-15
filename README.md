# Computer Vision Tool for Data Labeling on Phones

> Initial project plan — planning stage. No application features have been implemented yet.

## Purpose

Build a simple, offline mobile application for manually labeling computer vision datasets and exporting them for local model training. The project takes inspiration from dataset preparation workflows found in tools such as Roboflow, while focusing on accessible, precise manual work on a phone.

Users should be able to import images or extract video frames, inspect incoming data, draw annotations, review their work, prepare dataset splits, and export compatible training files without a server, account, or internet connection.

The application prepares datasets. Training models inside the app is outside the initial scope. Suitable training data depends on annotation quality, coverage, and the requirements of the chosen training pipeline; an export alone cannot guarantee model quality or industrial readiness.

## Product principles

- **Offline operation:** core importing, labeling, reviewing, and exporting must work without network access.
- **Manual control:** no AI labeling, automatic object detection, or automatic segmentation.
- **Clear progress:** every image has an understandable workflow status.
- **Recoverable work:** save edits reliably, support undo, and preserve original source data.
- **Touchscreen precision:** keep navigation, annotation, and inspection usable on a small screen.
- **Defined exports:** document and validate the exact conventions supported by each exporter.
- **Focused scope:** prioritize a dependable end-to-end workflow over server infrastructure and feature count.

## Intended workflow

**Import → Import Review → Labeling Queue → Final Review → Export**

The proposed main navigation is **Dataset · Label · Review · Export**, with project settings and backups accessible separately.

### 1. Create a project and import data

- Define project name, annotation task, and class list.
- Import images from local storage or the camera/gallery, subject to platform support.
- Accept videos and extract selected frames locally.
- Offer extraction by time interval, selected time range, evenly spaced frame count, or manual frame selection.
- Preview the number of frames and estimated storage impact before extraction.
- Retain source identifiers and video timestamps for review and dataset splitting.

Supported file types, maximum practical input sizes, and device requirements remain to be established through implementation testing.

### 2. Import Review

Incoming images enter a staging area before joining the labeling queue. Show a thumbnail grid and a large preview with these actions:

| Action | Meaning |
| --- | --- |
| Keep | Accept the image and send it to the labeling queue. |
| Mark empty / No target objects | Retain an intentionally negative example with no target annotations. |
| Exclude | Leave the image out of the active dataset and exports. |
| Undo | Reverse an accidental review decision. |

Support bulk selection and explicit confirmation of the import batch. Use “empty” rather than “null” to distinguish a deliberate negative example from missing labels or an unreadable image. Empty images still require review before approval.

Removing an item from a project must not delete the original gallery or source file. The exact project storage and retention policy will be documented before implementation.

### 3. Labeling Queue

Track **Viewed / Unseen** independently from annotation progress. Opening an image does not mean its annotations are finished.

Proposed annotation states are **To label → In progress → Ready for review → Approved**. Track **Empty / No target objects** as a separate explicit annotation disposition so an empty image can also be reviewed and approved. Exclusion is a dataset membership decision, not an annotation status.

Thumbnails should display viewed state, progress, annotation overlays, and annotation counts. Filters should make unseen, unfinished, empty, and flagged images easy to locate.

The editor should provide:

- Previous/next image navigation and a clear completion action.
- Bounding box drawing, selection, movement, and resizing.
- Polygon creation and point editing.
- Segmentation painting and erasing with adjustable brush size as part of the planned scope.
- Class selection and distinct annotation colors.
- Undo/redo, autosave, and recovery after interruption.
- Two-finger pan and pinch zoom, with precise editing assistance to be evaluated on devices.

Choose project task explicitly: object detection, semantic segmentation, instance segmentation, or image classification. The first-release task subset is still open; tools and export choices must match the selected task.

### 4. Inspect images and annotations

Provide three display modes: **Image only**, **Image + labels**, and **Labels only**. Consider press-and-hold to temporarily hide annotations, mask opacity adjustment, and outline-only viewing.

Provide fit-to-screen, original-resolution inspection, and a bounded maximum zoom. Determine the maximum through device testing rather than choosing an arbitrary number now. Magnification beyond source detail should be indicated clearly; an overview map can preserve full-image context. Viewing controls must not modify stored annotations.

### 5. Final Review

- Allow approval, flagging, or return to the labeling queue.
- Detect zero-area boxes, out-of-bounds geometry, unfinished polygons, missing classes, and contradictions between empty status and existing annotations.
- Distinguish structural errors from advisory dataset-quality notices.
- Return an approved image to review when its annotations are changed.
- Keep one stable project-wide class mapping; renaming must not change class identity.
- Require an explicit reassign/remove decision when deleting a class already used by annotations.
- Define background and ignored pixels explicitly for semantic segmentation.

### 6. Prepare and export a dataset

Configure train/validation/test ratios with a recorded random seed for repeatability. Keep images from the same source video or related capture group together by default to reduce leakage between splits. If a single video must be divided by time ranges, surface the limitation that those ranges may still be visually similar.

Separate these controls:

| Preprocessing: standardize inputs | Augmentation: create training variations |
| --- | --- |
| Resize and padding | Rotation and flipping |
| Orientation handling | Saturation adjustment |
| Optional grayscale conversion | Brightness or simulated exposure adjustment |
| | Warmer/cooler color temperature |

Augmentation controls should include toggles, strength ranges, previews, and limits on generated copies. Split original data first, then augment the training split by default. Every derived image must remain in its source image's split. Geometric transformations must also correctly transform boxes, polygons, and masks. Preserve original images and annotations.

Before export, summarize included images, empty images, class counts, split counts, unresolved work, selected transformations, and estimated size. Default to approved images; any inclusion of unfinished work must be explicit. Structurally invalid annotations must block the affected export.

## Export compatibility plan

YOLO is a model family, CNN describes an architecture category, and computer vision is the application field. There is no universal “CNN format” or “CV format.” Compatibility must be defined by annotation task and the consuming training pipeline.

These are candidate targets, not current support claims:

| Task | Candidate export |
| --- | --- |
| Object detection | A documented YOLO detection convention; COCO detection |
| Instance segmentation | COCO instance annotations; a specifically supported YOLO segmentation convention |
| Semantic segmentation | Paired images and class-ID PNG masks with a class map |
| Single-label classification | Class folders and a split manifest |
| Multi-label classification | Images and a documented label manifest |

Each implemented exporter must specify coordinate conventions, class identifiers, empty-image representation, folder structure, supported geometry, and limitations. Do not silently discard mask holes, merge object instances, or otherwise lose annotation meaning during conversion.

Validate export fixtures with the intended training loader before claiming compatibility. Export metadata should record app/schema version, class mapping, source grouping, split assignments, and transformation settings. Provide a separate full editable project backup and restore path.

## Proposed delivery stages

1. **Design and technical decisions:** choose target operating systems, mobile framework, initial task subset, storage approach, internal annotation model, and first export target. Prototype touch interaction and assess realistic memory/storage limits.
2. **Complete detection workflow:** implement local projects, image import review, boxes, status tracking, autosave/recovery, final review, splitting, and one validated export. This is a proposed implementation sequence, not a decision to remove segmentation from scope.
3. **Segmentation:** implement the chosen first segmentation workflow, then add the other needed tools and validated exports. Decide whether semantic or instance segmentation comes first; brush/eraser may move earlier if painting is the primary use case.
4. **Video and transformations:** implement frame extraction, source grouping, preprocessing, and training-only augmentation. Video extraction may move earlier if essential to initial users.
5. **Reliability and release readiness:** verify backup/restore, interruption recovery, large-project behavior, device interaction, export compatibility, and user documentation.

## Initial scope boundaries

Included in the overall plan: local computer vision datasets, manual annotation, image and video-frame input, import review, labeling and final review, dataset splitting, optional transformations, and exports.

Outside the initial scope: AI-assisted labeling, cloud synchronization, hosted datasets, user accounts, team collaboration, server-side processing, model hosting, and in-app model training. Tracking, keypoints, rotated boxes, and other specialized annotation types require a separate scope decision.

## Codebase documentation process

Documentation is part of implementation work and should describe actual behavior, including limitations.

1. Keep this README as the project entry point; update status and links as features become real.
2. Before substantial coding, document the chosen architecture and initial data model.
3. Record significant choices as short architecture decision records containing context, decision, alternatives, and consequences.
4. Update related documentation in the same change as behavior or data-format changes.
5. Maintain a changelog for user-visible changes and migration notes for persisted project formats.
6. Use issues and pull requests with a concrete problem, scope, acceptance criteria, implementation summary, and verification evidence.
7. Keep code comments focused on non-obvious reasoning and constraints; avoid repeating what the code says.

Create these documents as their content becomes concrete rather than filling the repository with empty placeholders:

| Document | Responsibility |
| --- | --- |
| `docs/product-requirements.md` | Detailed workflows, scope, and acceptance criteria |
| `docs/architecture.md` | Module boundaries, storage, data flow, and offline operation |
| `docs/data-model.md` | Image identity, classes, geometry, statuses, source groups, schema versions |
| `docs/export-formats.md` | Exact supported conventions and compatibility evidence |
| `docs/testing.md` | Test strategy, device checks, fixtures, and verification commands |
| `docs/user-guide.md` | Import, label, review, recover, and export instructions |
| `docs/decisions/` | Numbered architecture decision records |
| `CONTRIBUTING.md` | Setup, development workflow, and contribution requirements |
| `CHANGELOG.md` | Meaningful changes by release |

### Definition of done for implemented features

- The intended workflow functions offline on the supported target device(s).
- Relevant edits persist and remain consistent after an interruption.
- Validation covers the feature's actual risks, especially geometry, masks, splitting, and exported files.
- Relevant documentation describes current behavior and known limitations.
- Export changes include representative fixtures and consuming-loader checks.
- No feature is advertised as supported until its implementation and verification are complete.

## Decisions still open

- Android first, iOS first, or both, and minimum supported devices.
- Framework and local storage strategy, including backup format and schema migrations.
- First segmentation mode and priority of brush painting versus polygons.
- First YOLO training implementation/version and other export targets to validate.
- Zoom bounds, supported input types, and practical project size limits.
- Licensing and distribution approach.

Resolve these through focused discussion and small technical prototypes. This plan authorizes no particular technology choice and makes no claim that a feature already exists.
