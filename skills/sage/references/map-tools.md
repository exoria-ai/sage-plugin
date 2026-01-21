# Map Tools Reference

SAGE provides two distinct mapping tools for different purposes:

| Tool | Output | Purpose | Best For |
|------|--------|---------|----------|
| `capture_map_view` | Static image | AI spatial analysis | Verifying locations, assessing spatial relationships |
| `get_interactive_map_url` | URL to live map | User exploration | Panning, zooming, toggling layers, detailed exploration |

---

## When to Use Which Tool

### Use `capture_map_view` when:
- You need to visually verify a location or parcel
- You're assessing spatial relationships ("is this near the freeway?")
- You want to see the layout of parcels in a buffer/notification area
- You need to confirm what layers/features are present in an area
- You're making a spatial decision that requires visual context
- User needs a static image for a document or report

### Use `get_interactive_map_url` when:
- User wants to explore the map themselves
- User needs to toggle layers on/off
- User wants to change basemaps
- User is looking at multiple properties and wants to navigate
- After showing a static map, offer the interactive link for further exploration

---

## capture_map_view (Static Maps)

Generates a map image that is returned directly to you (the AI) for visual analysis. An imageUrl is also generated for sharing with the user.

### Location Input (provide ONE)

| Parameter | Description | Example |
|-----------|-------------|---------|
| `apn` | Center on and highlight a single parcel | `"0030-251-020"` |
| `apns` | Highlight multiple parcels | `["0030-251-020", "0030-251-021"]` |
| `center` | Center on coordinates (shows marker if no APN) | `{ latitude: 38.25, longitude: -122.04 }` |
| `bbox` | Explicit bounding box in WGS84 | `{ xmin, ymin, xmax, ymax }` |
| `buffer` | Buffer visualization for notification maps | See buffer section |
| `extent` | Full county view | `"county"` |

### Layer Options

```javascript
layers: {
  parcels: true,       // Parcel boundaries WITH APN labels (default: true at zoom >= 14)
  aerial2025: false,   // Solano County 2025 high-res aerial imagery
  addressPoints: false,
  cityBoundary: false,
  countyBoundary: false,
  garbageAreas: false
}
```

**Tip**: Enable `aerial2025` when zoomed in on individual parcels - it provides much clearer detail than the default basemap imagery.

### Basemap Options

| Basemap | Description | Best For |
|---------|-------------|----------|
| `topographic` | Default - hillshade + topo labels | General use, orientation |
| `imagery` | World satellite imagery | Regional context |
| `imagery-hybrid` | Satellite + labels | Satellite with reference labels |
| `navigation` | Street map style | Address/road context |

### Buffer Visualization

For notification radius maps (e.g., "parcels within 300 feet"):

```javascript
// Using APN as center
capture_map_view({
  buffer: {
    apn: "0030-251-020",
    radius_feet: 300,
    show_ring: true  // default: true
  }
})

// Using coordinates as center
capture_map_view({
  buffer: {
    latitude: 38.25,
    longitude: -122.04,
    radius_feet: 500
  }
})
```

The buffer visualization shows:
- **Orange fill**: Source parcel (if APN provided)
- **Dashed orange ring**: Buffer radius circle
- **Blue highlight**: Target parcels within buffer (if also specified via `apns`)

### Adding External Layers

Overlay any ArcGIS service using `additionalLayers`:

```javascript
capture_map_view({
  apn: "0030-251-020",
  additionalLayers: [
    {
      url: "https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer/28",
      title: "FEMA Flood Zones",
      opacity: 0.6
    }
  ]
})
```

**Layer types** (auto-detected from URL, or specify):
- `ArcGISFeatureLayer` - FeatureServer layers
- `ArcGISMapServiceLayer` - MapServer dynamic layers
- `ArcGISTiledMapServiceLayer` - Tiled/cached layers

**Optional `where` clause** to filter features:
```javascript
additionalLayers: [{
  url: "https://..../Fire_Stations/FeatureServer/0",
  title: "Fire Stations",
  where: "CITY='Fairfield'"
}]
```

### Using Layer Extent

Set map extent based on a layer's features:

```javascript
capture_map_view({
  extentLayer: {
    url: "https://.../SomeLayer/FeatureServer/0",
    where: "DISTRICT='5'",  // optional filter
    padding: 0.1            // 10% padding (default)
  },
  additionalLayers: [...] // the layer to display
})
```

### Layout Templates

For maps with scale bars and titles:

```javascript
capture_map_view({
  apn: "0030-251-020",
  layout: "Letter ANSI A Landscape",  // or "A4 Landscape"
  layoutOptions: {
    title: "Property Location Map",
    author: "Solano County GIS",
    scalebarUnit: "Feet"  // Feet, Miles, Meters, Kilometers
  }
})
```

| Layout | Output |
|--------|--------|
| `MAP_ONLY` | Just the map (default) |
| `Letter ANSI A Landscape` | 8.5x11" with title, scale bar, north arrow |
| `A4 Landscape` | A4 size with marginalia |

### What You Get Back

The tool returns:
1. **Image**: Displayed directly for your visual analysis
2. **Spatial context metadata**:
   - `extent`: Bounding box coordinates
   - `scale`: Approximate map scale (e.g., "1:5K")
   - `layersShown`: List of visible layers
   - `highlightedApns`: APNs highlighted on map
   - `approximateArea`: Area shown (e.g., "~25 acres")
3. **imageUrl**: Public URL for sharing with user

### Common Patterns

**Verify a parcel location:**
```javascript
capture_map_view({ apn: "0030-251-020" })
```

**Notification buffer map:**
```javascript
capture_map_view({
  buffer: { apn: "0030-251-020", radius_feet: 300 }
})
```

**Check flood zones:**
```javascript
capture_map_view({
  apn: "0030-251-020",
  additionalLayers: [{
    url: "https://hazards.fema.gov/arcgis/rest/services/public/NFHL/MapServer/28",
    title: "Flood Zones"
  }]
})
```

**Detailed aerial view:**
```javascript
capture_map_view({
  apn: "0030-251-020",
  layers: { aerial2025: true }
})
```

**County overview:**
```javascript
capture_map_view({
  extent: "county",
  layers: { countyBoundary: true, cityBoundary: true }
})
```

---

## get_interactive_map_url (Live Maps)

Generates a URL to open the SAGE interactive map application where users can pan, zoom, toggle layers, and explore.

### Map Presets

| Preset | Name | Layers Included |
|--------|------|-----------------|
| `base` | Property Research | Parcels, addresses, basic layers (default) |
| `hazards` | Hazard Assessment | Flood zones, fire hazard severity, risk layers |
| `zoning` | Zoning Analysis | Zoning districts, land use designations |
| `environmental` | Environmental Review | Wetlands, habitat, environmental constraints |
| `districts` | District Lookup | Supervisor districts, special districts, service areas |

### Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `preset` | Map preset/theme | `"hazards"` |
| `apn` | Highlight and zoom to parcel | `"0001-011-180"` |
| `address` | Geocode and highlight | `"675 Texas St, Fairfield, CA"` |
| `center` | Initial map center | `{ longitude: -122.0, latitude: 38.25 }` |
| `zoom` | Initial zoom (1-20) | `17` |
| `origin` | Route start point | `{ longitude, latitude, label? }` |
| `destination` | Route end point | `{ longitude, latitude, label? }` |

### Examples

**Simple parcel view:**
```javascript
get_interactive_map_url({ apn: "0001-011-180" })
// → Opens map centered on parcel, highlighted
```

**Hazard assessment:**
```javascript
get_interactive_map_url({
  apn: "0001-011-180",
  preset: "hazards"
})
// → Opens hazard layers, centered on parcel
```

**Address lookup:**
```javascript
get_interactive_map_url({
  address: "675 Texas St, Fairfield, CA"
})
// → Geocodes address, highlights parcel
```

**Show driving route:**
```javascript
get_interactive_map_url({
  origin: { longitude: -122.0, latitude: 38.2, label: "Start" },
  destination: { longitude: -122.1, latitude: 38.3, label: "End" }
})
// → Opens map with route and distance/time overlay
```

### Using list_map_presets

To show available presets:
```javascript
list_map_presets()
// Returns all presets with descriptions
```

---

## Combining Map Tools

A typical workflow:

1. **Quick verification** with `capture_map_view`:
   ```javascript
   capture_map_view({ apn: "0030-251-020" })
   ```
   "I can see this parcel is located north of Texas Street..."

2. **Offer interactive exploration** with `get_interactive_map_url`:
   ```javascript
   get_interactive_map_url({ apn: "0030-251-020", preset: "hazards" })
   ```
   "Here's an interactive map where you can explore the hazard layers in more detail: [URL]"

---

## Technical Notes

### Coordinate Systems

| System | EPSG | Usage |
|--------|------|-------|
| WGS84 | 4326 | All tool inputs/outputs (lat/long) |
| Web Mercator | 3857 | Internal rendering |
| CA State Plane III | 103004 | County internal storage |

### Zoom Levels

| Zoom | Approximate Scale | View |
|------|-------------------|------|
| 10 | 1:600K | County overview |
| 12 | 1:150K | City area |
| 14 | 1:40K | Neighborhood |
| 16 | 1:10K | Blocks |
| 17 | 1:5K | Parcel detail (default) |
| 18 | 1:2.5K | Building detail |
| 19 | 1:1.2K | Maximum detail |

### Available Aerial Imagery Years

Solano County has historical aerial imagery available (though currently only 2025 is exposed in the tool):

| Year | Coverage |
|------|----------|
| 2025 | Countywide (current) |
| 2024 | Urban areas |
| 2023 | Urban areas |
| 2022 | Countywide |
| 2021 | Urban areas |
| 2019 | Countywide |
| 2017 | Countywide |
| 2015 | Countywide |
| 2008 | Countywide |
| 2004 | Countywide (oldest) |

### Print Service

Static maps are rendered using ESRI's hosted print service:
```
https://utility.arcgisonline.com/arcgis/rest/services/Utilities/PrintingTools/GPServer/Export%20Web%20Map%20Task/execute
```

The service composites:
1. Basemap tiles
2. Operational layers (parcels, boundaries, etc.)
3. Graphics layers (highlights, buffers, markers)
4. Layout elements (scale bar, title, north arrow)

### Symbol Colors

| Element | Color | RGBA |
|---------|-------|------|
| Highlighted parcel fill | Blue 30% | [59, 130, 246, 77] |
| Highlighted parcel outline | Blue solid | [59, 130, 246, 255] |
| Buffer source fill | Orange 40% | [249, 115, 22, 100] |
| Buffer ring | Orange dashed | [249, 115, 22, 230] |
| Center marker | Red | [220, 38, 38, 255] |

---

## Troubleshooting

### "APN not found"
- Verify APN format (e.g., "003-025-1020" or "0030251020")
- The tool normalizes APNs internally, but unusual formats may fail
- Try using coordinates instead: `center: { latitude, longitude }`

### Buffer map not showing parcels
- Buffer visualization only shows the ring and source parcel
- To highlight parcels WITHIN the buffer, use `find_nearby_parcels` first to get APNs, then pass them to `capture_map_view` via `apns`

### Layer not appearing
- Check that the service URL is accessible
- Verify the layerType matches the service type
- Check opacity isn't set too low
- Some layers only render at certain zoom levels

### Map extent too tight/loose
- Explicitly set `zoom` to override auto-calculation
- Use `bbox` for precise extent control
- For `extentLayer`, adjust `padding` (default: 0.1 = 10%)
