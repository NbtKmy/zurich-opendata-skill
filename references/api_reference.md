# Zurich Open Data – API Reference

## Table of Contents
1. [CKAN Dataset Discovery](#ckan-dataset-discovery)
2. [CKAN DataStore](#ckan-datastore)
3. [Real-Time Environmental](#real-time-environmental)
4. [Mobility](#mobility)
5. [Geodata WFS](#geodata-wfs)
6. [Parliamentary](#parliamentary)
7. [Stadtratsbeschlüsse](#stadtratsbeschluesse)
8. [Tourism](#tourism)

---

## CKAN Dataset Discovery

Base: `https://data.stadt-zuerich.ch/api/3/action/`

### Search datasets
```
GET /package_search
  q=<solr query>        # supports AND, OR, NOT, wildcards, fuzzy (~)
  rows=10               # max 50
  start=0               # pagination offset
  sort=score desc       # or: metadata_modified desc, name asc
  fq=groups:<group_id>  # filter by category (see categories below)
```
Returns: list of datasets with `name`, `title`, `notes`, `resources[]`, `metadata_modified`

### Get dataset details
```
GET /package_show?id=<dataset_id_or_name>
```
Returns: full metadata, all resources with `id`, `name`, `url`, `format`, `datastore_active`

### List categories (19 total)
```
GET /group_list?all_fields=true
GET /group_show?id=<group_id>
```
Categories: `arbeit-und-erwerb`, `basiskarten`, `bauen-und-wohnen`, `bevolkerung`, `bildung`, `energie`, `finanzen`, `freizeit`, `gesundheit`, `kriminalitat`, `kultur`, `mobilitat`, `politik`, `preise`, `soziales`, `tourismus`, `umwelt`, `verwaltung`, `volkswirtschaft`

### Search tags
```
GET /tag_list?query=<term>&limit=30
```

### Catalog statistics
```
GET /package_search?q=*&rows=0         # total dataset count in numFound
GET /package_search?facet.field=groups # category distribution
```

---

## CKAN DataStore

Base: `https://data.stadt-zuerich.ch/api/3/action/`

### Basic query
```
GET /datastore_search
  resource_id=<uuid>           # required
  filters={"Field": "Value"}   # JSON object (URL-encoded)
  q=<full-text search>         # searches all fields
  sort=<field> asc|desc
  limit=20                     # max 100
  offset=0
```
Returns: `fields[]` (schema), `records[]` (data), `total` (count)

### SQL query (SELECT only)
```
GET /datastore_search_sql
  sql=SELECT * FROM "<resource_id>" WHERE ... LIMIT 100
```
- Only SELECT allowed (no INSERT/UPDATE/DELETE)
- Use double-quotes around resource UUID
- Supports JOINs, GROUP BY, aggregations, LIKE, BETWEEN

### Example: Filter by district
```python
params = {
    'resource_id': '<uuid>',
    'filters': json.dumps({"Quartier": "Wiedikon"}),
    'limit': 20
}
```

### Example: SQL aggregation
```python
sql = 'SELECT "Quartier", COUNT(*) as n FROM "<resource_id>" GROUP BY "Quartier" ORDER BY n DESC LIMIT 10'
```

---

## Real-Time Environmental

All via CKAN DataStore. Sort by `Timestamp desc` for latest values.

### Weather (hourly, 5 stations)
**Resource ID:** `f9aa1373-404f-443b-b623-03ff02d2d0b7`

Key fields: `Timestamp`, `Standort`, `Parameter`, `Wert`, `Einheit`

Stations: `Stampfenbachstrasse`, `Schimmelstrasse`, `Rosengartenstrasse`, `Heubeeribüel`, `Kaserne`

Parameters: `T` (temperature °C), `Hr` (humidity %), `p` (pressure hPa), `RainDur` (rain min/h), `StrGlo` (solar radiation)

```python
params = {
    'resource_id': 'f9aa1373-404f-443b-b623-03ff02d2d0b7',
    'filters': json.dumps({"Parameter": "T"}),
    'sort': 'Timestamp desc',
    'limit': 5
}
```

### Air Quality (hourly, 5 stations)
**Resource ID:** `90410203-4b4f-4a65-9015-1fca2792e04d`

Key fields: `Timestamp`, `Standort`, `Parameter`, `Wert`, `Einheit`

Stations: `Zch_Stampfenbachstrasse`, `Zch_Schimmelstrasse`, `Zch_Rosengartenstrasse`, `Zch_Heubeeribüel`, `Zch_Kaserne`

Parameters: `NO2` (µg/m³, WHO limit 40), `O3` (µg/m³), `PM10` (µg/m³, WHO limit 50), `PM2.5` (µg/m³, WHO limit 25)

### Lake Zurich Water (every 10 min)
- **Tiefenbrunnen:** `f86b3581-6fbc-4337-ab1a-b6ead9d15daf`
- **Mythenquai:** `61e26c94-c521-473f-b7bf-bb0d73f21e9f`

Key fields: `Timestamp`, `Parameter`, `Wert`, `Einheit`

Parameters: `Wassertemperatur` (°C), `Seespiegel` (m ü.M.), `Windgeschwindigkeit` (m/s), `Windrichtung` (°)

---

## Mobility

### Parking (real-time, 36 facilities)
**API:** `https://api.parkendd.de/Zuerich`

```python
import urllib.request, json
with urllib.request.urlopen('https://api.parkendd.de/Zuerich') as r:
    data = json.loads(r.read())
# data['lots'] = list of parking facilities
# Each lot: name, id, total, free, state ('open'/'closed'), coords
```

### Pedestrian Traffic – Bahnhofstrasse (hourly)
**Resource ID:** `ec1fc740-8e54-4116-aab7-3394575b4666`

Key fields: `Timestamp`, `Standort` (Nord/Mitte/Süd), `Ost` (eastbound count), `West` (westbound count), `Fussverkehr` (total)

### VBZ Public Transit Ridership (annual data, 800k+ records)
Dataset: `fahrgastzahlen-der-vbz`
Key fields: `Linie`, `Haltestelle`, `Richtung`, `FahrtenProTag`, `EinsteigerProTag`, `AusteigerProTag`

```python
# Filter by line
params = {
    'resource_id': '<resource_id_from_dataset>',
    'filters': json.dumps({"Linie": "4"}),
    'limit': 20
}
```

---

## Geodata WFS

Base: `https://www.ogd.stadt-zuerich.ch/wfs/geoportal/<ServiceName>`

```
GET ?service=WFS&version=1.1.0&request=GetFeature
  &typeName=<typename>
  &outputFormat=GeoJSON
  &maxFeatures=50
  &CQL_FILTER=<cql expression>   # optional
```

> **Note:** The URL path includes the WFS service name (e.g. `POI_oeffentliche_Spielplaetze`), and `typeName` is the view name inside that service — these are not always the same as the `layer_id` key. Use `GetCapabilities` to discover the correct `typeName` if unsure.

### Available Layers

| layer_id | Description |
|---|---|
| `schulanlagen` | Schools, kindergartens, care centers |
| `schulkreise` | School district boundaries (polygons) |
| `schulwege` | School routes and danger zones |
| `stadtkreise` | City district boundaries (12 districts) |
| `spielplaetze` | Public playgrounds (typename: `poi_spielplatz_view`) |
| `kreisbuero` | District office locations |
| `sammelstelle` | Waste/recycling collection points |
| `sport` | Sports facilities and swimming pools |
| `klimadaten` | Climate data raster (heat islands) |
| `lehrpfade` | Educational trails and paths |
| `stimmlokale` | Polling and voting stations |
| `sozialzentrum` | Social services centers |
| `velopruefstrecken` | Bike test courses (for school children) |
| `familienberatung` | Family counseling meeting points |

### Example: Get all schools
```bash
python3 -c "
import urllib.request, json, urllib.parse
params = urllib.parse.urlencode({
    'service': 'WFS', 'version': '1.1.0', 'request': 'GetFeature',
    'typeName': 'schulanlagen', 'outputFormat': 'GeoJSON', 'maxFeatures': 100
})
url = f'https://ogd.stadt-zuerich.ch/wfs/geoportal?{params}'
with urllib.request.urlopen(url) as r:
    data = json.loads(r.read())
for f in data['features'][:3]:
    print(f['properties'])
"
```

### Example: Get all playgrounds (Spielplätze)
```bash
python3 -c "
import urllib.request, json, urllib.parse
params = urllib.parse.urlencode({
    'service': 'WFS', 'version': '1.1.0', 'request': 'GetFeature',
    'typeName': 'poi_spielplatz_view', 'outputFormat': 'GeoJSON', 'maxFeatures': 500
})
url = f'https://www.ogd.stadt-zuerich.ch/wfs/geoportal/POI_oeffentliche_Spielplaetze?{params}'
req = urllib.request.Request(url, headers={'User-Agent': 'zurich-opendata-mcp/0.2.0'})
with urllib.request.urlopen(req) as r:
    data = json.loads(r.read())
for f in data['features'][:3]:
    print(f['properties'].get('name'), f['properties'].get('adresse'))
"
```
Key properties: `name`, `adresse`, `eignung` (age range), `geraete` (equipment), `besonderes` (highlights), `zvv_link`

### CQL Filter examples
- `NAME LIKE '%Wiedikon%'` – name contains substring
- `INTERSECTS(geometry, POINT(8.54 47.37))` – spatial intersection
- `Schulkreis = 'Letzi'` – exact field match

---

## Parliamentary

Base: `https://www.gemeinderat-zuerich.ch/api`

Uses XML responses. Parse with Python's `xml.etree.ElementTree`.

### Search motions/interpellations
```
GET /search/<index>?cql=<CQL>&start=0&size=10
```
Indexes: `geschaefte` (motions), `mitglieder` (members), `kommissionen` (commissions)

CQL examples:
- `title LIKE '%Klimaschutz%'`
- `datumEinreichung >= "2024-01-01" AND datumEinreichung <= "2024-12-31"`
- `geschaeftsart = "Interpellation"`
- `partei = "SP"`

```bash
python3 -c "
import urllib.request
import xml.etree.ElementTree as ET
import urllib.parse

query = urllib.parse.quote('title LIKE \"%Klimaschutz%\"')
url = f'https://www.gemeinderat-zuerich.ch/api/search/geschaefte?cql={query}&start=0&size=5'
with urllib.request.urlopen(url) as r:
    tree = ET.parse(r)
root = tree.getroot()
for hit in root.findall('.//{*}hit'):
    title = hit.findtext('.//{*}title', default='')
    date = hit.findtext('.//{*}datumEinreichung', default='')
    print(f'{date}: {title}')
"
```

### Search council members
```
GET /search/mitglieder?cql=name LIKE "%<name>%" AND partei="<party>"
```
Returns: member name, party, commissions, contact info

### Business types (geschaeftsart)
`Interpellation`, `Motion`, `Postulat`, `Anfrage`, `Dringliche Anfrage`, `Initiative`, `Petition`

---

## Stadtratsbeschlüsse

City council (Stadtrat) public decisions since Feb 2025, via CKAN DataStore.

**Resource ID:** `35c97bec-f8de-4521-814e-704dc98f71a2`

Key fields: `Beschlussnummer`, `Beschlusstitel`, `Beschlussdatum`, `Departement`, `Link`

### Departments
| Short | Full Name |
|---|---|
| `SSD` | Schul- und Sportdepartement |
| `FD` | Finanzdepartement |
| `PRD` | Präsidialdepartement |
| `TED` | Tiefbau- und Entsorgungsdepartement |
| `HBD` | Hochbaudepartement |
| `GUD` | Gesundheits- und Umweltdepartement |
| `SID` | Sicherheitsdepartement |
| `SD` | Sozialdepartement |
| `SKZ` | Stadtschreiber |
| `DIB` | Departement der Industriellen Betriebe |
| `RK` | Stadtrat (Stadtpräsidium / RK) |

### Search decisions by keyword
```python
import urllib.request, json, urllib.parse

params = urllib.parse.urlencode({
    'resource_id': '35c97bec-f8de-4521-814e-704dc98f71a2',
    'q': 'Klimaschutz',           # full-text search
    'sort': 'Beschlussdatum desc',
    'limit': 10
})
url = f'https://data.stadt-zuerich.ch/api/3/action/datastore_search?{params}'
with urllib.request.urlopen(url) as r:
    data = json.loads(r.read())
for rec in data['result']['records']:
    print(f"{rec['Beschlussdatum']} [{rec['Departement']}] {rec['Beschlusstitel']}")
    print(f"  {rec.get('Link', '')}")
```

### Filter by department and date range (SQL)
```python
sql = '''
SELECT "Beschlussnummer", "Beschlusstitel", "Beschlussdatum", "Link"
FROM "35c97bec-f8de-4521-814e-704dc98f71a2"
WHERE "Departement" = 'SSD'
  AND "Beschlussdatum" >= '2025-01-01'
ORDER BY "Beschlussdatum" DESC
LIMIT 20
'''
params = urllib.parse.urlencode({'sql': sql})
url = f'https://data.stadt-zuerich.ch/api/3/action/datastore_search_sql?{params}'
```

### Get specific decision by number
```python
params = urllib.parse.urlencode({
    'resource_id': '35c97bec-f8de-4521-814e-704dc98f71a2',
    'filters': json.dumps({"Beschlussnummer": "1203/2025"})
})
```

---

## Tourism

Base: `https://www.zuerich.com/en/api/v2/data`

Uses Schema.org JSON-LD format.

### Get all categories
```
GET /categories
```
Returns list of category objects with `id`, `name`

### Get data for a category
```
GET /categories/<category_id>/items?lang=<lang>
```
Language: `de` (default), `en`, `fr`, `it`

### Categories
| ID | Content |
|---|---|
| `uebernachten` | Hotels and accommodation |
| `aktivitaeten` | Activities and experiences |
| `restaurants` | Dining |
| `shopping` | Shopping |
| `nachtleben` | Nightlife and bars |
| `kultur` | Culture and arts |
| `events` | Events (time-based) |
| `touren` | Tours and guided walks |
| `natur` | Nature and outdoor |
| `sport` | Sports |
| `familien` | Family-friendly attractions |
| `museen` | Museums |

### Example: Get restaurants in English
```bash
python3 -c "
import urllib.request, json
url = 'https://www.zuerich.com/en/api/v2/data/categories/restaurants/items?lang=en'
with urllib.request.urlopen(url) as r:
    data = json.loads(r.read())
for item in data[:3]:
    print(item.get('name',''), '-', item.get('description','')[:80])
"
```
