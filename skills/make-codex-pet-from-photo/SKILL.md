---
name: make-codex-pet-from-photo
description: Create, repair, resume, validate, package, and install a custom animated Codex desktop pet from one or more pet photos. Use when a user asks to turn their cat, dog, or other pet photo into a Codex pet, wants a photo-faithful 9-state pet spritesheet, or wants to continue an interrupted pet-generation run.
---

# Make Codex Pet From Photo

Create a photo-faithful Codex pet while keeping the user loop simple. Use the installed
`hatch-pet` workflow for atlas geometry, state semantics, validation, visual QA, and packaging.

## Load The Authoritative Workflow

Read and follow:

```text
${CODEX_HOME:-$HOME/.codex}/skills/hatch-pet/SKILL.md
```

Treat `hatch-pet` as authoritative when it is stricter than this wrapper. Use `$imagegen` for
visual generation. Do not draw or synthesize missing animation strips locally.

## Collect Inputs

Ask only for missing choices that materially affect the result:

- reference photo path or attached image
- display name, or infer a short friendly name
- preferred style, defaulting to `sticker`

For a real pet photo, default to a semi-realistic sticker illustration that preserves the
specific animal's face, coat markings, eye color, proportions, and expression. Do not add
collars, accessories, props, or decorative effects unless the user requests them.

Use a stable ASCII pet id derived from the display name. Save the working run under:

```text
<workspace>/<pet-id>-pet/run
```

Copy temporary or chat-app reference images into:

```text
<workspace>/<pet-id>-pet/references/
```

## Keep Visible Progress

Use `update_plan` with the four `hatch-pet` checklist items:

1. Getting `<Pet>` ready.
2. Imagining `<Pet>`'s main look.
3. Picturing `<Pet>`'s poses.
4. Hatching `<Pet>`.

Resume from existing files when a run was interrupted. Inspect `imagegen-jobs.json`, `decoded/`,
and any generated output directories before restarting work. Preserve completed rows.

## Prepare The Run

Use the Codex bundled Python runtime when available because the system Python may not contain
Pillow. Call `load_workspace_dependencies` when the runtime path is unknown.

Initialize with `prepare_pet_run.py`. Include:

- the copied reference photo
- explicit `--pet-name`, `--pet-id`, and `--display-name`
- `--style-preset sticker`
- photo-faithful semi-realistic sticker notes
- `--chroma-key auto`

Let the script choose a safe removable key color. Use the built-in `$imagegen` chroma-key path
first. Switch to native-transparency CLI fallback only after local extraction fails and the user
explicitly approves the fallback.

## Generate Visual Jobs

Generate the canonical base image first. Copy the selected output into `decoded/base.png` and
`references/canonical-base.png`, then mark the manifest job complete.

Generate pose rows through lightweight subagents when the user authorizes subagents. Keep at
most two generation workers active at once. Start with `idle` and `running-right`.

After visually checking `running-right`, derive `running-left` with the official framewise mirror
script only when the pet has no asymmetric marking, accessory, or prop whose meaning would
change. Otherwise generate `running-left` separately.

Generate the remaining distinct states independently:

```text
waving jumping failed waiting running review
```

Copy every selected row into its manifest output path before marking it complete. Keep generated
workers scoped to one visual job. Do not ask a worker to edit manifests, process frames, package,
or clean up.

## Process And Review

After all rows are complete, run the official deterministic pipeline from `hatch-pet`:

```text
extract_strip_frames.py
inspect_frames.py --require-components
compose_atlas.py
validate_atlas.py
make_contact_sheet.py
render_animation_previews.py
```

Inspect `qa/contact-sheet.png` and preview GIFs with one lightweight visual-QA worker. Reject:

- identity drift or generic redesign
- clipped poses, guide marks, shadows, chroma residue, or detached effects
- directional rows with wrong facing or stagnant gait
- `failed` rows with detached smoke, symbols, or aura
- non-directional `running` rows that read as travel instead of in-place focused task work
- idle loops that are visually static

Repair only failing rows, rerun the deterministic pipeline, and repeat visual QA until it passes.

## Package And Install

Install the finished pet into:

```text
${CODEX_HOME:-$HOME/.codex}/pets/<pet-id>/
  pet.json
  spritesheet.webp
```

Write `pet.json` with:

```json
{
  "id": "<pet-id>",
  "displayName": "<display name>",
  "description": "<short description>",
  "spritesheetPath": "spritesheet.webp"
}
```

Validate the installed `spritesheet.webp`, not only the run copy. Retain the final WebP,
validation JSON, contact sheet, preview GIFs, QA JSON, run summary, request JSON, canonical base,
and copied source photo. Remove prompts, layout guides, decoded strips, extracted frames, the
PNG atlas, and the completed generation manifest after successful installation.

Report the installed paths and render the retained contact sheet in the final response.
