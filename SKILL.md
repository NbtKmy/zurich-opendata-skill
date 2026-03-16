---
name: zurich-opendata
description: "Access Zurich city's open data APIs. Use when the user asks about Zurich city data such as: weather/air quality/water conditions, parking availability, pedestrian traffic, VBZ public transit, geodata (schools, playgrounds, sports facilities, districts), parliamentary motions and council members, Stadtratsbeschlüsse (city council decisions), tourism attractions/restaurants/events, or searching the 900+ open datasets at data.stadt-zuerich.ch. Triggers on queries like Zürich Wetter, Zurich parking, Gemeinderat, Stadtratsbeschlüsse, VBZ Fahrgastzahlen, offene Daten Zürich, Zurich schools, Zurich tourism, etc."
---

# Zurich Open Data Skill

You have access to Zurich city's open data via direct HTTP calls. Use Python with `httpx` (already installed in this project) or `curl` via the Bash tool.

## Available Data & Which API to Use

| User wants | API to use | Reference |
|---|---|---|
| Search/browse datasets | CKAN Search | [api_reference.md](references/api_reference.md#ckan-dataset-discovery) |
| Query tabular data from a dataset | CKAN DataStore | [api_reference.md](references/api_reference.md#ckan-datastore) |
| Current weather, temperature, humidity | UGZ DataStore (weather resource) | [api_reference.md](references/api_reference.md#real-time-environmental) |
| Air quality, NO₂, O₃, PM10 | UGZ DataStore (air quality resource) | [api_reference.md](references/api_reference.md#real-time-environmental) |
| Lake Zurich water temp/level | UGZ DataStore (water resource) | [api_reference.md](references/api_reference.md#real-time-environmental) |
| Parking availability | ParkenDD API | [api_reference.md](references/api_reference.md#mobility) |
| Bahnhofstrasse pedestrian counts | CKAN DataStore | [api_reference.md](references/api_reference.md#mobility) |
| VBZ public transit ridership | CKAN DataStore | [api_reference.md](references/api_reference.md#mobility) |
| School locations, playgrounds, geo layers | Geoportal WFS | [api_reference.md](references/api_reference.md#geodata-wfs) |
| City council motions / Gemeinderat | Paris API | [api_reference.md](references/api_reference.md#parliamentary) |
| Stadtratsbeschlüsse (STRB decisions) | CKAN DataStore | [api_reference.md](references/api_reference.md#stadtratsbeschluesse) |
| Tourism attractions, restaurants, hotels | Zürich Tourismus API | [api_reference.md](references/api_reference.md#tourism) |

## Workflow

1. **Identify** which API/data source fits the user's question (table above)
2. **Read** the relevant section of [references/api_reference.md](references/api_reference.md) for endpoint details
3. **Execute** the HTTP request using Bash (Python one-liner or curl)
4. **Present** results clearly in Markdown

## Quick Examples

### Search for datasets
```bash
python3 -c "
import urllib.request, json, urllib.parse
params = urllib.parse.urlencode({'q': 'schulen', 'rows': 5})
url = f'https://data.stadt-zuerich.ch/api/3/action/package_search?{params}'
with urllib.request.urlopen(url) as r:
    data = json.loads(r.read())
for d in data['result']['results']:
    print(d['name'], '-', d.get('title',''))
"
```

### Get real-time parking
```bash
python3 -c "
import urllib.request, json
with urllib.request.urlopen('https://api.parkendd.de/Zuerich') as r:
    data = json.loads(r.read())
for lot in data['lots'][:5]:
    free = lot.get('free', '?')
    total = lot.get('total', '?')
    print(f\"{lot['name']}: {free}/{total} frei\")
"
```

### Query DataStore (e.g., weather)
```bash
python3 -c "
import urllib.request, json, urllib.parse
params = urllib.parse.urlencode({
    'resource_id': 'f9aa1373-404f-443b-b623-03ff02d2d0b7',
    'limit': 5,
    'sort': 'Timestamp desc'
})
url = f'https://data.stadt-zuerich.ch/api/3/action/datastore_search?{params}'
with urllib.request.urlopen(url) as r:
    data = json.loads(r.read())
for rec in data['result']['records']:
    print(rec)
"
```

## Notes

- All data is **CC0 licensed** (public domain, no restrictions)
- CKAN base URL: `https://data.stadt-zuerich.ch/api/3/action/`
- DataStore supports JSON filters: `{"Quartier": "Wiedikon"}`, SQL queries, full-text search
- SPARQL endpoint (`ld.stadt-zuerich.ch`) is **not yet productive** — use DataStore SQL instead
- For geographic data, Geoportal WFS returns GeoJSON — useful for maps and spatial queries
- Response language for Paris API is German; tourism API supports `de`, `en`, `fr`, `it`
