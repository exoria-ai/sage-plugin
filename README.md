# SAGE Plugin for Claude Code

**SAGE - Solano Agent for Geographic Enquiry**

A Claude Code plugin that provides AI-powered GIS assistant capabilities for Solano County, California.

## What This Plugin Provides

- **Skill**: Domain knowledge for interpreting GIS data, zoning codes, flood zones, fire hazards, and Solano County organizational structure
- **MCP Server**: Connection to the hosted SAGE API with 40+ GIS and county data tools

## Quick Start

**Requires [Claude Code](https://code.claude.com/docs/en/quickstart) or [Claude Desktop](https://code.claude.com/docs/en/desktop)**

```shell
/plugin marketplace add exoria-ai/sage-plugin
/plugin install sage@exoria-ai-sage-plugin
```

That's it! Start asking questions about Solano County properties, zoning, hazards, budgets, and more.

## Alternative: Manual Installation

For development or testing:

```bash
claude --plugin-dir /path/to/sage-plugin
```

## Available Tools

Once installed, you'll have access to these MCP tools:

### GIS & Property
- `geocode_address` - Convert address to coordinates and APN
- `get_parcel_details` - Property info, values, ownership
- `get_zoning` - Zoning with automatic jurisdiction routing
- `get_flood_zone` - FEMA flood zone designation
- `get_fire_hazard_zone` - CAL FIRE FHSZ classification
- `get_supervisor_district` - Board of Supervisors district
- `get_special_districts` - Fire, water, school districts
- `get_nearby` - Find schools, parks, fire stations nearby
- `search_parcels` - Search parcels by criteria
- `get_parcels_in_buffer` - Parcels within radius (notification lists)
- `render_map` - Generate static map images

### GIS Layer Discovery
- `list_gis_categories` - Browse 11 layer categories
- `list_gis_layers` - List layers by category or priority
- `search_gis_layers` - Keyword search across 67 layers
- `get_gis_layer_details` - Service URL, fields, metadata
- `find_layers_for_question` - Match questions to relevant layers

### County Code
- `get_county_code_sections` - Full text of code sections
- `list_county_code_chapters` - Available chapters
- `list_county_code_sections` - Sections in a chapter
- `search_county_code` - Keyword search

### Budget
- `search_budget` - Semantic search of FY25-26 budget
- `get_budget_chunk` - Full chunk by ID
- `list_budget_departments` - All departments
- `get_department_budget` - Department budget details

### Visualization
- `generate_infographic` - Create diagrams and visualizations
- `edit_image` - Edit or combine images

## Example Usage

After installing the plugin, ask Claude questions like:

- "What's the zoning for 123 Main St, Fairfield?"
- "Show me a map of parcel 003-025-1020"
- "What parcels are within 300 feet of this property?"
- "What are the ADU requirements for unincorporated Solano County?"
- "What's the Sheriff's department budget?"

## Reference Materials

The plugin includes detailed reference materials on:

- GIS layers, discovery, and querying
- Jurisdiction routing (city vs county)
- Zoning code interpretation
- Proposition 13 / assessed values
- ADU/JADU requirements
- FEMA flood zones
- CAL FIRE fire hazard zones
- Special districts
- County contacts
- Standard disclaimers

## Requirements

- Claude Code v1.0.33 or later
- Internet connection (tools query live GIS services)

## License

MIT

## Author

[Exoria AI](https://github.com/exoria-ai)
