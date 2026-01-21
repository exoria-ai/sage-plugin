# Contributing to SAGE Plugin

This document is written for LLMs (and humans) working inside this codebase. It explains the plugin architecture, how to make changes, and best practices for extending SAGE.

---

## What is This Plugin?

SAGE (Solano Agent for Geographic Enquiry) is a Claude Code plugin that transforms Claude into a specialized GIS assistant for Solano County, California. When activated, Claude gains access to:

- **40+ MCP tools** for property lookups, zoning queries, hazard assessments, budget analysis, and more
- **Domain expertise** through the SKILL.md instructions and reference files
- **County-specific context** including General Plan policies, County Code regulations, and organizational structure

## Plugin Architecture

This plugin follows the [Claude Code Plugin specification](https://code.claude.com/docs/en/plugins). Understanding this architecture is critical for making changes.

### Directory Structure

```
sage-plugin/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest (name, version, description)
│   └── marketplace.json     # Marketplace listing metadata
├── skills/
│   └── sage/
│       ├── SKILL.md         # Main skill instructions (the "brain")
│       └── references/      # Supporting documentation files
│           ├── map-tools.md
│           ├── gis-layers.md
│           ├── fire-hazard.md
│           └── ... (15+ reference files)
└── README.md
```

### How Skills Work

Skills use **progressive disclosure** - Claude loads content in stages:

1. **Level 1 (Always loaded)**: The YAML frontmatter (`name`, `description`) is injected into Claude's system prompt at startup. This lets Claude know the skill exists.

2. **Level 2 (Loaded when triggered)**: When the user asks something matching the description (or invokes `/sage`), Claude reads the full SKILL.md body.

3. **Level 3 (Loaded as needed)**: Reference files in `references/` are only read when SKILL.md mentions them and they're relevant to the task.

This means:
- Keep SKILL.md focused on **when to use tools** and **how to interpret results**
- Put detailed specifications in reference files
- Claude reads reference files via bash when needed - they don't consume context until accessed

### The SKILL.md File

`skills/sage/SKILL.md` is the main instruction file. It has two parts:

**YAML Frontmatter** (between `---` markers):
```yaml
---
name: sage
description: AI GIS assistant for Solano County, California. Use for property lookups, zoning queries, flood/fire hazard assessments...
---
```

The `description` field is critical - Claude uses it to decide when to activate this skill. Include keywords users might say.

**Markdown Body**: Contains:
- Core principles and behavioral guidelines
- Tool tables organized by category
- Integration patterns (how to combine tools)
- Response format guidelines
- Sensitive topic handling
- Common mistake warnings

### Reference Files

Files in `references/` provide detailed documentation that Claude loads on-demand:

| File | Purpose |
|------|---------|
| `map-tools.md` | capture_map_view and get_interactive_map_url parameters |
| `gis-layers.md` | Layer discovery, querying, and download workflows |
| `fire-hazard.md` | CAL FIRE FHSZ methodology and response guidance |
| `flood-zones.md` | FEMA flood zone designations and implications |
| `aumentum-parcels.md` | Parcel data fields and assessment system |
| `zoning-codes.md` | Zoning district codes and meanings |
| `county-code-chapters.md` | Available County Code chapters |
| `general-plan.md` | General Plan structure and usage |
| `org-structure.md` | County departments and leadership |
| `prop13.md` | Proposition 13 and assessed values |
| `adu-rules.md` | ADU/JADU regulations |
| `jurisdiction.md` | City vs. county jurisdiction routing |
| `spatial-grounding.md` | Geographic orientation for the county |
| `solano-county-encyclopedia.md` | County history and demographics |
| `special-districts.md` | Fire, water, school districts |
| `contacts.md` | Department contact information |
| `disclaimers.md` | Standard GIS data disclaimers |

### Plugin Manifest

`.claude-plugin/plugin.json` defines the plugin identity:

```json
{
  "name": "sage",
  "description": "SAGE - Solano Agent for Geographic Enquiry...",
  "version": "1.8.0",
  "author": { "name": "Exoria AI" },
  "homepage": "https://github.com/exoria-ai/sage-plugin",
  "repository": "https://github.com/exoria-ai/sage-plugin",
  "license": "MIT"
}
```

The `name` field becomes the skill namespace (skills are invoked as `/sage:sage`).

---

## Making Changes

### Adding a New Tool

When a new MCP tool is added to the SAGE server:

1. **Update SKILL.md tool tables**: Add a row to the appropriate category table
2. **Add to Quick Reference**: If it's a common operation, add a pattern to the Quick Reference table at the bottom
3. **Update relevant reference file**: If the tool needs detailed documentation, add a section to the appropriate reference file
4. **Increment version**: Bump the patch version in `plugin.json`

Example: Adding `list_gis_downloads`:

```markdown
### GIS Layer Discovery Tools
| Tool | Purpose |
|------|---------|
| `list_gis_downloads` | List datasets available for file download (Shapefile, GDB, LiDAR) |
```

### Updating Tool Documentation

If a tool's behavior changes (renamed, new parameters, different output):

1. **Update SKILL.md**: Change the tool name/description in tables
2. **Update reference files**: Update any detailed documentation
3. **Check for obsolete patterns**: Search for old tool names in cross-tool patterns
4. **Increment version**: Bump patch version

### Adding a Reference File

When you need detailed documentation that shouldn't load by default:

1. Create the file in `references/` with a descriptive name
2. Add a link in SKILL.md under "Knowledge Domains"
3. In the reference file, use headers that match what users might ask about

Structure reference files with:
- Clear headers for easy navigation
- Tables for quick lookup
- Code examples where relevant
- Links back to related tools

### Modifying Behavioral Guidelines

The SKILL.md contains behavioral guidelines like:
- Jurisdiction routing rules
- Sensitive topic handling
- Response format standards

When modifying these:
1. Consider the impact on all response types
2. Test with representative queries
3. Update the "Common Mistakes to Avoid" section if relevant

---

## Best Practices

### Keep SKILL.md Concise

SKILL.md loads into context when the skill activates. Every token counts.

**DO:**
- Use tables for tool listings (scannable, compact)
- Link to reference files for details
- Use bullet points over paragraphs
- Include only "when to use" and "how to interpret" guidance

**DON'T:**
- Duplicate information from reference files
- Include lengthy examples in SKILL.md
- Over-explain concepts Claude already knows

### Write for Progressive Disclosure

Reference files should be self-contained:

```markdown
# Fire Hazard Reference

## Quick Reference
[Essential info Claude needs most often]

## Detailed Methodology
[Deeper explanation - Claude reads this only when needed]

## Response Templates
[How to explain to users - read when crafting responses]
```

### Maintain Tool-to-Reference Links

SKILL.md should clearly indicate which reference file to consult:

```markdown
### Hazards & Environment
- **Fire hazard zones (CAL FIRE)** → `references/fire-hazard.md`
- **Flood zones (FEMA)** → `references/flood-zones.md`
```

This helps Claude navigate to the right documentation.

### Version Semantics

Follow semantic versioning:
- **MAJOR** (1.0.0 → 2.0.0): Breaking changes to skill behavior
- **MINOR** (1.7.0 → 1.8.0): New tools or features
- **PATCH** (1.8.0 → 1.8.1): Documentation fixes, clarifications

---

## Testing Changes

### Local Testing

Test plugin changes with `--plugin-dir`:

```bash
claude --plugin-dir /path/to/sage-plugin
```

### Test Queries

After making changes, verify with representative queries:

**Property lookups:**
- "What's the zoning for 123 Main St, Fairfield?"
- "Tell me about APN 0001-011-180"

**Hazard assessments:**
- "Is this property in a flood zone?"
- "What's the fire hazard rating?"

**Policy questions:**
- "What are the setback requirements for A-40 zoning?"
- "How does the county implement agricultural preservation?"

**New tools (if applicable):**
- Test the specific functionality you added
- Verify Claude uses the tool appropriately

---

## MCP Tools Reference

The SAGE MCP server provides these tool categories. When working on this plugin, understand what each tool returns:

### Property Tools
- `geocode_address` → coordinates, APN, jurisdiction
- `get_parcel_details` → ownership, values, use codes, links
- `get_zoning` → zone code, jurisdiction routing
- `get_flood_zone` → FEMA designation, SFHA status
- `get_fire_hazard_zone` → CAL FIRE FHSZ classification
- `get_supervisor_district` → BOS district
- `get_special_districts` → fire, water, school districts
- `find_nearby` → nearby POI (schools, parks, etc.)
- `search_parcels` → parcels matching criteria with aggregations
- `find_nearby_parcels` → parcels within radius (notification lists)

### GIS Layer Tools
- `list_gis_categories` → 11 categories with counts
- `list_gis_layers` → layers by category/priority
- `search_gis_layers` → keyword search
- `get_gis_layer_details` → full metadata, fields, downloads
- `suggest_layers` → match questions to layers
- `list_gis_downloads` → downloadable datasets by format

### Map Tools
- `capture_map_view` → static map image for AI analysis
- `get_interactive_map_url` → URL for user exploration
- `list_map_presets` → available preset configurations

### Geoprocessing Tools
- `dissolve_layer` → merge polygons by attribute
- `inspect_layer` → examine fields and dissolve candidates

### Document Tools
- `search_general_plan` → full-text search
- `search_general_plan_policies` → policy-specific search
- `get_general_plan_chunk` → retrieve specific chunk
- `get_general_plan_chapter` → entire chapter
- `search_county_code` → code keyword search
- `get_county_code_sections` → full section text
- `search_budget` → budget document search
- `get_department_budget` → department details

### Organization Tools
- `get_org_overview` → all departments with FTEs
- `get_department` → department structure
- `search_positions` → find positions by title
- `get_position_distribution` → job class allocation
- `compare_departments` → side-by-side comparison

### Visualization Tools
- `generate_infographic` → AI-generated diagrams
- `edit_image` → modify/combine images

---

## Common Patterns

When extending the plugin, these patterns appear frequently:

### Tool Chaining
Many workflows require multiple tools:
```
geocode_address → get_zoning → get_flood_zone → get_fire_hazard_zone
```

Document these patterns in the "Cross-Tool Integration Patterns" section.

### Jurisdiction Routing
Always check jurisdiction first - city addresses use different zoning sources than unincorporated areas.

### Grounded Responses
For sensitive topics (fire hazard, flood zones), defer to authoritative sources. Don't claim independent analysis.

---

## Questions?

If you're an LLM working on this codebase and something is unclear:
1. Read the relevant reference file
2. Check the main SKILL.md for context
3. Look at how similar tools are documented
4. When in doubt, keep changes minimal and focused
