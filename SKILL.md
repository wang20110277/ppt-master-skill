---
name: ppt-master
description: >
  AI-driven multi-format SVG content generation system. Converts source documents
  (PDF/DOCX/URL/Markdown) into high-quality SVG pages and exports to PPTX through
  multi-role collaboration. Use when user asks to "create PPT", "make presentation",
  "生成PPT", "做PPT", "制作演示文稿", or mentions "ppt-master".
---

# PPT Master Skill

> AI-driven multi-format SVG content generation system. Converts source documents into high-quality SVG pages through multi-role collaboration and exports to PPTX.

**Core Pipeline**: `Source Document → Create Project → Template Option → Strategist → [Image_Generator] → Executor → Post-processing → Export`

> [!CAUTION]
> ## Global Execution Discipline (MANDATORY)
>
> **This workflow is a strict serial pipeline. The following rules have the highest priority — violating any one of them constitutes execution failure:**
>
> 1. **SERIAL EXECUTION** — Steps MUST be executed in order; the output of each step is the input for the next. Non-BLOCKING adjacent steps may proceed continuously once prerequisites are met, without waiting for the user to say "continue"
> 2. **BLOCKING = HARD STOP** — Steps marked ⛔ BLOCKING require a full stop; the AI MUST wait for an explicit user response before proceeding and MUST NOT make any decisions on behalf of the user
> 3. **NO CROSS-PHASE BUNDLING** — Cross-phase bundling is FORBIDDEN. (Note: the Eight Confirmations in Step 4 are ⛔ BLOCKING — the AI MUST present recommendations and wait for explicit user confirmation before proceeding. Once the user confirms, all subsequent non-BLOCKING steps — design spec output, SVG generation, speaker notes, and post-processing — may proceed automatically without further user confirmation)
> 4. **GATE BEFORE ENTRY** — Each Step has prerequisites (🚧 GATE) listed at the top; these MUST be verified before starting that Step
> 5. **NO SPECULATIVE EXECUTION** — "Pre-preparing" content for subsequent Steps is FORBIDDEN (e.g., writing SVG code during the Strategist phase)
> 6. **NO SUB-AGENT SVG GENERATION** — Executor Step 6 SVG generation is context-dependent and MUST be completed by the current main agent end-to-end. Delegating page SVG generation to sub-agents is FORBIDDEN
> 7. **SEQUENTIAL PAGE GENERATION ONLY** — In Executor Step 6, after the global design context is confirmed, SVG pages MUST be generated sequentially page by page in one continuous pass. Grouped page batches (for example, 5 pages at a time) are FORBIDDEN

> [!IMPORTANT]
> ## Language & Communication Rule
>
> - **Response language**: Always match the language of the user's input and provided source materials. For example, if the user asks in Chinese, respond in Chinese; if the source material is in English, respond in English.
> - **Explicit override**: If the user explicitly requests a specific language (e.g., "请用英文回答" or "Reply in Chinese"), use that language instead.
> - **Template format**: The `design_spec.md` file MUST always follow its original English template structure (section headings, field names), regardless of the conversation language. Content values within the template may be in the user's language.

> [!IMPORTANT]
> ## Compatibility With Generic Coding Skills
>
> - `ppt-master` is a repository-specific workflow skill, not a general application scaffold
> - Do NOT create or require `.worktrees/`, `tests/`, branch workflows, or other generic engineering structure by default
> - If another generic coding skill suggests repository conventions that conflict with this workflow, follow this skill first unless the user explicitly asks otherwise

## CLI Commands

This skill uses the `ppm` CLI (installed via `pip install ppt-master-cli`).

| Command | Purpose |
|---------|---------|
| `ppm create <input_path> [--format ppt169] [-o dir]` | One-step: create hidden project from source |
| `ppm convert <file_or_url>` | Auto-detect and convert to Markdown |
| `ppm init <name> [--format ppt169]` | Initialize a new project |
| `ppm import <project> <sources...> --move` | Import sources into project |
| `ppm validate <project>` | Validate project structure |
| `ppm info <project>` | Show project info |
| `ppm finalize <project>` | SVG post-processing |
| `ppm export <project> [-s final]` | Export to PPTX |
| `ppm split <project>` | Split speaker notes |
| `ppm check <project>` | SVG quality check |
| `ppm image <prompt> [--aspect-ratio 16:9]` | AI image generation |
| `ppm config list-formats` | List canvas formats |

## Template & Reference Paths

Templates and references are available at the skill directory (`.claude/skills/ppt-master/`):

| Resource | Path |
|----------|------|
| Layout templates | `templates/layouts/layouts_index.json` |
| Visualization templates | `templates/charts/charts_index.json` |
| Icon library | `templates/icons/` (libraries: `chunk/`, `tabler-filled/`, `tabler-outline/`) |
| Role definitions | `references/` (strategist.md, executor-base.md, etc.) |
| Shared constraints | `references/shared-standards.md` |
| Canvas formats | `references/canvas-formats.md` |

---

## Workflow

### Step 1: Source Content Processing

🚧 **GATE**: User has provided source material (PDF / DOCX / EPUB / URL / Markdown file / text description / conversation content — any form is acceptable).

When the user provides non-Markdown content, convert immediately:

| User Provides | Command |
|---------------|---------|
| PDF / DOCX / PPTX / other doc | `ppm convert <file>` |
| Web link / URL | `ppm convert <URL>` |
| Markdown | Read directly |

**Checkpoint — Confirm source content is ready, proceed to Step 2.**

---

### Step 2: Project Initialization

🚧 **GATE**: Step 1 complete; source content is ready.

**Option A — One-step creation (recommended for file/URL sources):**

```bash
ppm create <input_file_or_url> [--format ppt169] [-o output_dir]
```

Creates a hidden `.project` directory alongside the input file. Source is referenced in-place (no copy/move). PPTX defaults to the same directory as the source file.

**Option B — Two-step init + import:**

```bash
ppm init <project_name> --format <format>
```

Format options: `ppt169` (default), `ppt43`, `xhs`, `story`, etc.

Import source content:

| Situation | Action |
|-----------|--------|
| Has source files | `ppm import <project_path> <source_files...> --move` |
| User provided text in conversation | No import needed |

> **MUST use `--move`**: All source files MUST be moved into the project.

**Checkpoint — Confirm project structure created. Proceed to Step 3.**

---

### Step 3: Template Selection

🚧 **GATE**: Step 2 complete; project directory structure is ready.

⛔ **BLOCKING**: If the user has not yet clearly expressed whether to use a template, you MUST present options and **wait for an explicit user response** before proceeding.

**Early-exit**: If the user has already stated "no template" / "不使用模板" / "自由设计", skip directly to Step 4.

**Template recommendation flow** (only when the user has NOT yet decided):
Query `templates/layouts/layouts_index.json` to list available templates.
Present a professional recommendation based on the PPT topic and content.

After the user confirms option A (use template), copy template files:
```bash
cp .claude/skills/ppt-master/templates/layouts/<template_name>/*.svg <project_path>/templates/
cp .claude/skills/ppt-master/templates/layouts/<template_name>/design_spec.md <project_path>/templates/
```

After the user confirms option B (free design), proceed directly to Step 4.

**Checkpoint — User has responded with template selection. Proceed to Step 4.**

---

### Step 4: Strategist Phase (MANDATORY)

🚧 **GATE**: Step 3 complete; user has confirmed template selection.

First, read the role definition:
```
Read references/strategist.md
```

> **Mandatory gate**: Before writing `design_spec.md`, Strategist MUST read `templates/design_spec_reference.md` and produce the spec following its full I–XI section structure.

**Must complete the Eight Confirmations**:

⛔ **BLOCKING**: The Eight Confirmations MUST be presented to the user as a bundled set of recommendations, and you MUST **wait for the user to confirm or modify** before proceeding.

1. Canvas format
2. Page count range
3. Target audience
4. Style objective
5. Color scheme
6. Icon usage approach
7. Typography plan
8. Image usage approach

If the user has provided images, run the analysis script **before outputting the design spec**:
```bash
ppm image --analyze <project_path>/images
```
Or use: `python3 -c "from pptmaster.scripts.analyze_images import main; import sys; sys.argv=['analyze_images.py','<project_path>/images']; main()"`

> **Image handling rule**: The AI must NEVER directly read, open, or view image files. All image information must come from the analysis script output or the Design Specification's Image Resource List.

**Output**: `<project_path>/design_spec.md`

**Checkpoint — Phase deliverables complete, auto-proceed**:

---

### Step 5: Image_Generator Phase (Conditional)

🚧 **GATE**: Step 4 complete; Design Specification generated and user confirmed.

> **Trigger condition**: Image approach includes "AI generation". If not triggered, skip to Step 6.

Read `references/image-generator.md`

1. Extract all images with status "pending generation" from the design spec
2. Generate prompt document → `<project_path>/images/image_prompts.md`
3. Generate images:
   ```bash
   ppm image "prompt" --aspect-ratio 16:9 -o <project_path>/images
   ```

**Checkpoint — Confirm all images are ready, proceed to Step 6.**

---

### Step 6: Executor Phase

🚧 **GATE**: Step 4 (and Step 5 if triggered) complete.

Read the role definition based on the selected style:
```
Read references/executor-base.md          # REQUIRED
Read references/executor-general.md       # General style
Read references/executor-consultant.md    # Consulting style
Read references/executor-consultant-top.md # MBB consulting style
```

**Design Parameter Confirmation (Mandatory)**: Before generating the first SVG, review and output key design parameters from the Design Specification.

> **Main-agent only rule**: SVG generation MUST remain with the current main agent. Do NOT delegate to sub-agents.
> **Sequential generation rule**: Generate pages one at a time in a continuous pass. Grouped batches are FORBIDDEN.

**Visual Construction**: Generate SVG pages → `<project_path>/svg_output/`

**Logic Construction**: Generate speaker notes → `<project_path>/notes/total.md`

**Checkpoint — All SVGs and notes generated. Proceed to Step 7.**

---

### Step 7: Post-processing & Export

🚧 **GATE**: Step 6 complete; all SVGs and notes generated.

> The following three sub-steps MUST be **executed individually one at a time**.

**Step 7.1** — Split speaker notes:
```bash
ppm split <project_path>
```

**Step 7.2** — SVG post-processing:
```bash
ppm finalize <project_path>
```

**Step 7.3** — Export PPTX:
```bash
ppm export <project_path> -s final
# Output: <source_file_dir>/<project_name>_<timestamp>.pptx
```

> **NEVER** use `cp` as a substitute for finalize.
> **NEVER** export directly from `svg_output/` — MUST use `-s final`.

---

## Role Switching Protocol

Before switching roles, you **MUST first read** the corresponding reference file. Output marker:

```markdown
## [Role Switch: <Role Name>]
Reading role definition: references/<filename>.md
Current task: <brief description>
```

---

## Notes

- Do NOT add extra flags to the post-processing commands — run them as-is
- Local preview: `python3 -m http.server -d <project_path>/svg_final 8000`
- **Troubleshooting**: For layout overflow, export errors, blank images, see the project docs
