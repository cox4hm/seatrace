# SeaTrace

**Live page: https://cox4hm.github.io/seatrace/**

SeaTrace is a research prototype for spotting deep-sea mining and seabed-survey activity from ship tracking (AIS) data. It was built for a James Madison University capstone project sponsored by the National Geospatial-Intelligence Agency (NGA).

It is a single web page with no server and no installation. The data, the map library and the coastlines are built in, so `index.html` also works offline.

## What it does
- Finds **seabed surveys** (lawnmower line patterns) and **stationkeeping** (ships holding position offshore for days), and ranks each one by priority.
- Puts each detection in context:
  - ISA mining contract areas;
  - exclusive economic zones (EEZs) and the nearest foreign boundary;
  - water depth;
  - ports used, who owns each port, and Chinese port financing;
  - submarine cables;
  - US sanctions.
- **Port ownership:** the owner and operators of all 182 ports the ships called at, researched with linked sources (state, local government, private, military, oil-field or trust port). It adds Wikidata owners for 169 other ports. There is a searchable table, and you can copy it as CSV.
- **Cable watch:** ranks ships that slowed, stopped or switched off AIS over submarine cables, and lists ships near publicly reported cable faults.
- **Analyst tools:**
  - a daily summary;
  - MGRS coordinates;
  - KML and GeoJSON export;
  - draft imagery requests;
  - a draw-an-area filter;
  - a case log;
  - a briefing written in standard intelligence likelihood and confidence language;
  - checks of the detectors against known cases.
- **Load data:** analyse your own AIS file in the browser.

## Data
All real, no simulated tracks.

| Data | Source | Licence |
|---|---|---|
| Ship positions (1 May–29 Oct 2026, 86 ships), events and registry | Global Fishing Watch | CC BY-NC 4.0 |
| Submarine cables | TeleGeography Submarine Cable Map | CC BY-NC-SA 3.0 |
| Chinese-financed ports | AidData CPORTS 2.0 (Thome, Fedorochko and Zhang 2026) | Cite the dataset |
| Port ownership (182 fleet ports) | Team research, Nov 2026: port authorities, governments, company filings, news | Sources linked in the app |
| Port owners for other ports | Wikidata | CC0 |
| Sanctions | US Treasury OFAC SDN list (dated 9 Nov 2026) | Public domain |
| ISA contract areas | International Seabed Authority | Public |
| EEZs | Marine Regions EEZ v12 | CC BY 4.0 |
| Depth | GEBCO 2020 | Public |
| Map library | Leaflet 1.9.4 (BSD-2), ECharts world outline (Apache 2.0) | |

Non-commercial use only, because of the Global Fishing Watch and TeleGeography licences.

## Source code
The page is built from a Python data pipeline plus HTML, CSS and JavaScript. That source rebuilds this exact page and is available from the team on request.

## Caveats
This is a student research prototype, not an operational system. Positions are hourly on a grid of about 1 km, and cable routes are approximate. Port ownership changes, and deals marked as agreed may not have closed. Detections and leads are prompts for analysis, not findings.
