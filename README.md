# Myanmar Earthquakes & Active Faults Map

An interactive web map visualizing earthquake events and active fault lines in Myanmar. Built with [Leaflet.js](https://leafletjs.com/), powered by open geoscience data from the [USGS Earthquake Catalog](https://earthquake.usgs.gov/earthquakes/search/) and the [GEM Global Active Faults Database](https://github.com/GEMScienceTools/gem-global-active-faults).

![Leaflet](https://img.shields.io/badge/Leaflet-1.9.4-199900?logo=leaflet&logoColor=white)
![GeoJSON](https://img.shields.io/badge/Format-GeoJSON-blue)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## Table of Contents

- [Overview](#overview)
- [Live Demo](#live-demo)
- [Project Structure](#project-structure)
- [Data Sources](#data-sources)
  - [Earthquake Data](#earthquake-data)
  - [Active Faults Data](#active-faults-data)
- [Data Processing](#data-processing)
- [Map Features](#map-features)
- [Getting Started](#getting-started)
- [Reproducing the Data](#reproducing-the-data)
- [Technology Stack](#technology-stack)
- [Data Schema](#data-schema)
- [License](#license)
- [Acknowledgments](#acknowledgments)

---

## Overview

On March 28, 2025, a devastating M7.7 earthquake struck Myanmar, triggering widespread destruction and a renewed focus on seismic hazard in the region. This project compiles all recorded seismic events in and around Myanmar from that date forward and overlays them on the mapped active fault network, providing a spatial context for ongoing seismicity.

The project produces three artifacts:

| File | Description | Size |
|------|-------------|------|
| `earthquakes_myanmar.geojson` | USGS earthquake catalog extract | 140 KB |
| `myanmar_faults.geojson` | GEM active faults filtered to Myanmar region | 302 KB |
| `myanmar_earthquake_map.html` | Self-contained interactive web map | 454 KB |

---

## Live Demo

Open `myanmar_earthquake_map.html` directly in any modern browser — no server, build step, or internet connection required beyond the initial tile load (map tiles are fetched from CARTO).

---

## Project Structure

```
.
├── README.md                                     # This documentation
├── earthquakes_myanmar.geojson                   # USGS earthquake data (GeoJSON)
├── myanmar_faults.geojson                        # GEM active faults for Myanmar (GeoJSON)
├── gem_active_faults.geojson                     # Full GEM global faults dataset (source file)
└── myanmar_earthquake_map.html (index.html)      # Interactive Leaflet web map
```

---

## Data Sources

### Earthquake Data

| Property | Value |
|----------|-------|
| **Source** | [USGS Earthquake Hazards Program — Search Earthquake Catalog](https://earthquake.usgs.gov/earthquakes/search/) |
| **API Endpoint** | `https://earthquake.usgs.gov/fdsnws/event/1/query` |
| **Format** | GeoJSON (USGS native output) |
| **Date Range** | 2025-03-28 to 2026-03-19 |
| **Geographic Bounds** | Latitude 9.5°N – 28.5°N, Longitude 92.0°E – 101.5°E |
| **Total Events** | 201 |
| **Magnitude Range** | M3.4 – M7.7 |
| **Depth Range** | 4.5 – 191.4 km |

**Magnitude breakdown:**

| Category | Count |
|----------|-------|
| M7.0+ | 1 |
| M6.0 – 6.9 | 2 |
| M5.0 – 5.9 | 20 |
| M4.0 – 4.9 | 130 |
| M3.0 – 3.9 | 48 |

The full USGS API query used:

```
https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson
  &starttime=2025-03-28
  &endtime=2026-03-19
  &minlatitude=9.5
  &maxlatitude=28.5
  &minlongitude=92.0
  &maxlongitude=101.5
```

### Active Faults Data

| Property | Value |
|----------|-------|
| **Source** | [GEM Global Active Faults Database](https://github.com/GEMScienceTools/gem-global-active-faults) |
| **Source File** | `geojson/gem_active_faults.geojson` (version of record) |
| **Format** | GeoJSON |
| **Global Features** | 16,195 fault segments |
| **Myanmar-filtered Features** | 407 fault segments |
| **Geometry Type** | LineString |
| **Filter Bounding Box** | Latitude 9.0°N – 29.0°N, Longitude 91.5°E – 102.0°E |
| **Regional Sources** | Southeast Asia faults (Chan et al., 2017) |

**Slip type distribution (Myanmar region):**

| Slip Type | Count |
|-----------|-------|
| Dextral (right-lateral strike-slip) | 150 |
| Sinistral (left-lateral strike-slip) | 148 |
| Reverse | 47 |
| Normal | 38 |
| Subduction Thrust | 16 |
| Spreading Ridge | 4 |
| Anticline | 2 |
| Dextral Transform | 2 |

---

## Data Processing

The data pipeline involves two steps:

1. **Earthquake data** — Downloaded directly from the USGS FDSN Event Web Service as GeoJSON. No transformation needed; the API natively returns a valid GeoJSON `FeatureCollection`.

2. **Fault data** — The full global GEM dataset (`gem_active_faults.geojson`, 12.3 MB, 16,195 features) is downloaded from GitHub. A spatial filter extracts all fault segments where **at least one coordinate** falls within the Myanmar bounding box (lat 9.0°–29.0°, lon 91.5°–102.0°). The slightly wider bounding box (compared to the earthquake query) ensures faults that cross Myanmar's borders are included. The filtered result is written to `myanmar_faults.geojson`.

Both GeoJSON outputs conform to [RFC 7946](https://tools.ietf.org/html/rfc7946).

---

## Map Features

The interactive map (`myanmar_earthquake_map.html`) includes:

### Earthquake Layer
- Earthquakes rendered as **circle markers** with size proportional to magnitude
- **Color scale** from yellow (M3) through orange (M4–5) to deep red (M7+)
- **Click popup** showing:
  - Magnitude (color-coded)
  - Place name
  - Date/time (UTC)
  - Depth (km)
  - Felt reports (if available)
  - Tsunami alert flag
  - Geographic coordinates
  - Link to the USGS event detail page

### Active Faults Layer
- Fault lines rendered as **colored polylines** by slip type:
  - **Red** — Dextral / Dextral Transform
  - **Blue** — Sinistral
  - **Teal** — Normal
  - **Gold** — Reverse
  - **Purple** — Subduction Thrust
  - **Orange** — Spreading Ridge
  - **Gray** — Anticline / Other
- **Click popup** showing:
  - Slip type
  - Net slip rate (mm/yr)
  - Average dip and rake
  - Seismogenic depth range
  - Catalog source

### Map Controls
- **Layer toggle** (top-right) to show/hide earthquake and fault layers independently
- **Zoom controls** with mouse wheel and button support
- **Statistics panel** (top-left) with summary counts and magnitude statistics
- **Legend** (bottom-right) with color keys for both layers
- **Basemap**: CARTO Positron (light, clean cartographic style)

---

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Firefox, Safari, Edge)
- Python 3.6+ (only if you want to reproduce or update the data)

### Viewing the Map

Simply open the HTML file in your browser:

```bash
# macOS
open myanmar_earthquake_map.html

# Linux
xdg-open myanmar_earthquake_map.html

# Windows
start myanmar_earthquake_map.html
```

No web server is required. The HTML file is fully self-contained — both GeoJSON datasets are embedded inline. The only external dependency is map tile loading from CARTO's CDN and the Leaflet library from unpkg.

### Viewing the GeoJSON Files

The GeoJSON files can be viewed in:

- [geojson.io](https://geojson.io) — drag and drop the file
- [GitHub](https://github.com) — renders GeoJSON natively when viewing files
- **QGIS**, **ArcGIS**, or any GIS software
- Any text editor (they are valid JSON)

---

## Reproducing the Data

To regenerate the datasets from scratch:

### 1. Download Earthquake Data

```bash
curl -o earthquakes_myanmar.geojson \
  "https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=2025-03-28&endtime=$(date +%Y-%m-%d)&minlatitude=9.5&maxlatitude=28.5&minlongitude=92.0&maxlongitude=101.5"
```

> **Note:** Update the `endtime` parameter to your desired end date. The USGS API returns all matching events up to a maximum of 20,000.

### 2. Download and Filter Fault Data

```bash
# Download the full GEM global active faults dataset
curl -L -o gem_active_faults.geojson \
  "https://raw.githubusercontent.com/GEMScienceTools/gem-global-active-faults/master/geojson/gem_active_faults.geojson"
```

Then filter to Myanmar using Python:

```python
import json

MIN_LAT, MAX_LAT = 9.0, 29.0
MIN_LON, MAX_LON = 91.5, 102.0

with open('gem_active_faults.geojson') as f:
    data = json.load(f)

def in_bbox(geometry):
    for lon, lat, *_ in geometry['coordinates']:
        if MIN_LON <= lon <= MAX_LON and MIN_LAT <= lat <= MAX_LAT:
            return True
    return False

myanmar = [f for f in data['features'] if in_bbox(f['geometry'])]

with open('myanmar_faults.geojson', 'w') as f:
    json.dump({"type": "FeatureCollection", "features": myanmar}, f)

print(f"Extracted {len(myanmar)} fault segments")
```

### 3. Rebuild the HTML Map

To rebuild the map with updated data, inject the GeoJSON into the HTML template:

```python
import json

with open('myanmar_earthquake_map.html', 'r') as f:
    html = f.read()

# Replace the inline JSON data between the JavaScript variable assignments
# (or use the template approach described in the HTML source)
```

---

## Technology Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Map rendering | [Leaflet.js](https://leafletjs.com/) | 1.9.4 |
| Basemap tiles | [CARTO Positron](https://carto.com/basemaps/) | — |
| Data format | [GeoJSON (RFC 7946)](https://tools.ietf.org/html/rfc7946) | — |
| Earthquake API | [USGS FDSN Event Web Service](https://earthquake.usgs.gov/fdsnws/event/1/) | 1.0 |
| Fault database | [GEM Global Active Faults](https://github.com/GEMScienceTools/gem-global-active-faults) | Latest |
| Data processing | Python 3 (standard library `json` only) | 3.6+ |

No build tools, package managers, or frameworks are required.

---

## Data Schema

### Earthquake GeoJSON Properties

Each earthquake feature includes properties from the [USGS GeoJSON Summary Format](https://earthquake.usgs.gov/earthquakes/feed/v1.0/geojson.php):

| Property | Type | Description |
|----------|------|-------------|
| `mag` | float | Magnitude |
| `place` | string | Text description of named geographic region near the event |
| `time` | long | Unix timestamp in milliseconds |
| `url` | string | Link to USGS event detail page |
| `depth` (in coordinates) | float | Depth in kilometers |
| `felt` | int | Number of felt reports submitted |
| `tsunami` | int | 1 if tsunami alert was issued |
| `type` | string | Event type (e.g., "earthquake") |
| `title` | string | Human-readable title |

### Fault GeoJSON Properties

Each fault feature includes properties from the [GEM GAF-DB schema](https://github.com/GEMScienceTools/gem-global-active-faults):

| Property | Type | Description |
|----------|------|-------------|
| `slip_type` | string | Kinematic classification (e.g., Dextral, Normal, Reverse) |
| `net_slip_rate` | float | Net slip rate in mm/yr |
| `average_dip` | float | Average dip angle in degrees |
| `average_rake` | float | Average rake angle in degrees |
| `upper_seis_depth` | float | Upper seismogenic depth in km |
| `lower_seis_depth` | float | Lower seismogenic depth in km |
| `catalog_id` | string | Source catalog identifier |
| `catalog_name` | string | Source catalog name |
| `name` | string | Fault name (where available) |
| `dip_dir` | string | Dip direction |

---

## License

This project is released under the [MIT License](LICENSE).

### Data Licenses

- **USGS Earthquake Data**: Public domain. USGS data is produced by the U.S. Government and is not subject to copyright in the United States.
- **GEM Global Active Faults Database**: Released under the [Creative Commons Attribution-ShareAlike 4.0 International License (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/). Any redistribution of the fault data must include proper attribution and maintain the same license.

---

## Acknowledgments

- **USGS Earthquake Hazards Program** for providing open, real-time access to the global earthquake catalog via the FDSN Event Web Service.
- **GEM Foundation (Global Earthquake Model)** and the contributors to the [Global Active Faults Database](https://github.com/GEMScienceTools/gem-global-active-faults) for compiling and openly publishing global active fault traces.
- **Leaflet.js** contributors for the lightweight, open-source mapping library.
- **CARTO** for providing free basemap tiles.

### Citation

If using the GEM fault data in academic work, please cite:

> Styron, R., and M. Pagani (2020), The GEM Global Active Faults Database, *Earthquake Spectra*, 36(S1), 218–239, doi:10.1177/8755293020944182.
