# PPT Master Skill

Claude Code skill for AI-driven presentation generation. Contains role definitions, workflow instructions, layout/chart/icon templates, and the skill manifest used by `ppm skill install`.

## Structure

```
skill/S              # Installed skill manifest (used by ppm skill install)
  SKILL.md           # Claude Code skill definition (ppm-command version)
SKILL.md             # Source-repo skill definition (raw python3 paths)
references/          # AI role definitions and technical specs (11 files)
workflows/           # Supplementary workflow instructions
templates/           # Layout templates, chart templates, 640+ vector icons
  layouts/           # Slide layout SVGs + layouts_index.json
  charts/            # Visualization SVGs + charts_index.json
  icons/             # Icon libraries (chunk/, tabler-filled/, tabler-outline/)
```

## Usage with ppm

```bash
# Install skill into a target project
ppm skill install --skill-dir /path/to/ppt-master-skill --target-dir /your/project

# Or copy .claude/skills/ppt-master/ manually
```

## Related

- [pptmaster-cli](https://github.com/user/ppt-master) — The core engine (`ppm` CLI, converters, exporters)
