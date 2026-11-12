# SeaTrace: source code

SeaTrace is a single web page (`SeaTrace.html`). It is not a Python app, and nothing has to run on a server.

- **In the browser:** HTML, CSS and JavaScript draw the map and score the detections. The briefing, gallery, timeline, season replay and Load data also run here.
- **Ahead of time:** Python reads the raw data files. It works out port calls, AIS gaps and stationkeeping, merges the survey detector's results and the Global Fishing Watch events, and writes one `data.json`. That file is embedded inside the page.

```
raw data files ──► pipeline/*.py (Python) ──► data/data.json ──┐
                                                               ├──► web/build_page.py ──► SeaTrace.html
web/shell.html + web/engine.js + web/ui.js (HTML/CSS/JS) ──────┘
```

## Files

| File | Language | Lines | What it does |
|---|---|---|---|
| `web/shell.html` | HTML + CSS | 571 | Page layout (header, fleet list, map, panels, timeline bar) and all styling. `/*ENGINE*/` and `/*UI*/` mark where the JavaScript goes. |
| `web/engine.js` | JavaScript | 422 | Analysis engine. It decodes the embedded tracks and builds voyages and port calls, then scores every detection. **Confidence** says "is this really a survey/stationkeeping?" and **priority** (8 weighted factors) says "how much does it matter?". It also holds the Chikyu reference case and the regression results, and **Cable watch**: it ranks ships that were slow, stopped or AIS-dark over submarine cable routes, and lists who was near each reported cable fault (`cableWatch`, `faultCheck`, `FAULTS`). |
| `web/ui.js` | JavaScript | 1,809 | Everything you see and click: fleet browser, alert queue, Cable watch tab, Summary view and vessel panel (with identity history, sanctions, MGRS and the case log). Also the area filter, KML/GeoJSON export and imagery-request drafts. Also the map layers, briefing, gallery, fleet timeline and season replay, the Validation and Method sheets, exports, and **Load data**. Load data is an in-browser copy of the Python port-call, gap and stationkeeping rules, with a small ZIP reader. |
| `web/build_page.py` | Python | 27 | Assembles `SeaTrace.html` from the three files above, the two libraries in `web/lib/` and `data/data.json`. The result works with no internet connection. |
| `web/lib/` | JavaScript | – | Leaflet 1.9.4 (BSD-2) and the ECharts 4.9 world outline (Apache 2.0), copied from their public CDNs on 10 Nov 2026 so the page can run offline. |
| `pipeline/build_data.py` | Python | 319 | Main data build. It reads the vessel tracker's "Everything (ZIP)" export, the WPI port list, the ISA contract-area shapefiles and the EEZ files. It finds port calls (speed under 2 kn within 12 km of a port) and AIS gaps (over 12 h). It finds stationkeeping: at least 48 h within 3 km, median speed under 1 kn, more than 60 km from a port, not during a survey. It imports the survey detector's results, adds context to every detection, and writes `data.json`. |
| `pipeline/gfw_merge.py` | Python | 147 | Adds Global Fishing Watch loitering, port-visit and gap events and registry data. Before using a GFW identity, it checks it against our own track. It also computes the GFW cross-check numbers shown in Validation. Run it after `build_data.py`. It also stores every GFW identity seen on each ship's MMSI, with the track-test verdict, for the identity history. |
| `pipeline/ext_merge.py` | Python | 114 | Adds port details (harbor size and type) and AidData CPORTS 2.0 Chinese port financing, ownership and naval-activity records, matched to the port list. Also adds TeleGeography submarine cables (simplified to about 1 km) and landing stations. Run it after `gfw_merge.py`. |
| `pipeline/shp2json.py` | Python | 32 | Minimal shapefile (.shp/.dbf) reader used for the ISA contract areas. |
| `pipeline/sanctions_merge.py` | Python | 37 | Checks every ship's IMO number, name and registry owner, and every ISA contractor, against the US Treasury OFAC SDN list (`SDN.CSV`, `ALT.CSV`). Run it last. |
| `pipeline/cable_screening.py` | Python | 90 | Offline version of the Cable watch screening for analysis outside the page. Writes `cable_screening.json`. |
| `pipeline/stationkeeping_detector.py` | Python | 276 | Stand-alone stationkeeping detector with a command-line interface, for the team's backend. It uses the same rules as above and gives the same 44 operations on the May–Oct data. |

Not included: the team's survey detector. Its file is `survey_pattern_detector.py` in the `deep_sea_vessel_monitor` folder, and it includes the 6 Nov revisit fix. SeaTrace uses its *results* from the ZIP export; it does not run it.

## Rebuilding SeaTrace.html

You need Python 3 with `numpy` and `pandas`. Put the input files in `data/`:

| Put in `data/` | Where it comes from |
|---|---|
| `survey_results/` (unzipped) | "Everything (ZIP)" export from vesseltracker-v2.streamlit.app |
| `ports.csv` | `deep_sea_vessel_monitor/app_data/ports.csv` |
| `isa/PMN`, `isa/PMS`, `isa/CRFC` | ISA contract-area shapefiles from isa.org.jm/maps |
| `seatrace_eez_lines.json` | Marine Regions EEZ v12 boundaries, simplified (in Harper's Downloads) |
| `eez_points.json` | EEZ (Marine Regions) and depth (GEBCO 2020) looked up for each detection centre |
| `seatrace_gfw_events_2026.csv` | GFW events and registry export (in Harper's Downloads) |
| `AidData_CPORTS_Dataset_v2.0.xlsx` | AidData CPORTS 2.0 ZIP from aiddata.org (in Harper's Downloads) |
| `seatrace_submarine_cables.json` | TeleGeography cable routes, landing stations and owners (in Harper's Downloads) |
| `sdn.csv`, `alt.csv` | OFAC SDN list from sanctionslist.ofac.treas.gov (in Harper's Downloads, list dated 9 Nov 2026) |

Then run:

```
python pipeline/build_data.py
python pipeline/gfw_merge.py
python pipeline/ext_merge.py
python -I pipeline/sanctions_merge.py data/sdn.csv data/alt.csv data/data.json
python web/build_page.py
```

Checked 10 Nov 2026 (after adding the analyst features): with the current `data.json`, `build_page.py` reproduces the published SeaTrace page byte for byte.

## Libraries used by the page

- Leaflet 1.9.4 (map) and the ECharts 4.9 world outline (coastlines), both built into the page from `web/lib/`. The page runs with no internet connection; only the IBM Plex web fonts are fetched when online.
- No other frameworks: the rest is plain JavaScript.

## Data licence

- Ship positions and GFW events: Global Fishing Watch, CC BY-NC 4.0 (non-commercial use only).
- Submarine cables: TeleGeography, CC BY-NC-SA 3.0.
- Chinese port financing: AidData CPORTS 2.0. Cite as Thome, Fedorochko and Zhang (2026).
- Sanctions: US Treasury OFAC SDN list (public domain).
