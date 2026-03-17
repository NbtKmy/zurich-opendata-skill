# Zurich Open Data – Claude Code Skill

Dieses Repository enthält einen **Claude Code Skill**, der den Zugriff auf die offenen Daten der Stadt Zürich ermöglicht.

## Dank

Dieser Skill basiert auf dem MCP-Server [zurich-opendata-mcp](https://github.com/malkreide/zurich-opendata-mcp) von **malkreide**. Ein herzlicher Dank geht an den Autor für die sorgfältige Aufbereitung der Zürich-Open-Data-APIs und die Bereitstellung als Open-Source-Projekt. Ohne diese Vorarbeit wäre dieser Skill nicht entstanden.

## Was ist dieser Skill?

Ein Claude Code Skill ist eine Erweiterung für das CLI-Tool [Claude Code](https://claude.ai/code), die Claude beibringt, mit einem bestimmten Themengebiet umzugehen. Dieser Skill ermöglicht es Claude, auf die öffentlichen Daten der Stadt Zürich zuzugreifen – ohne zusätzliche Software oder laufende Server.

## Verfügbare Daten

| Thema | Datenquelle |
|---|---|
| Wetter, Temperatur, Luftfeuchtigkeit | UGZ DataStore |
| Luftqualität (NO₂, O₃, PM10) | UGZ DataStore |
| Zürichsee-Wassertemperatur und -pegel | UGZ DataStore |
| Parkhaus-Verfügbarkeit (Echtzeit) | ParkenDD API |
| Fussgängerverkehr Bahnhofstrasse | CKAN DataStore |
| VBZ Fahrgastzahlen | CKAN DataStore |
| Schulen, Spielplätze, Geodaten | Geoportal WFS |
| Gemeinderats-Vorstösse und Mitglieder | Gemeinderat Zürich API |
| Stadtratsbeschlüsse | CKAN DataStore |
| Tourismus: Hotels, Restaurants, Events | Zürich Tourismus API |
| Datensatz-Suche (900+ Datensätze) | CKAN Dataset Discovery |

Alle Daten sind unter **CC0** lizenziert (gemeinfrei, keine Einschränkungen).

## Installation

### Voraussetzungen

- [Claude Code](https://claude.ai/code) muss installiert sein
- Python 3 muss verfügbar sein

### Persönliche Installation (für alle Projekte)

Klone dieses Repository direkt in das persönliche Skills-Verzeichnis von Claude Code:

```bash
git clone https://github.com/NbtKmy/zurich-opendata-skill.git ~/.claude/skills/zurich-opendata
```

Der Skill ist danach in jeder Claude Code-Sitzung verfügbar.

### Projekt-Installation (nur für ein bestimmtes Projekt)

Klone dieses Repository in das Skills-Verzeichnis deines Projekts:

```bash
git clone https://github.com/NbtKmy/zurich-opendata-skill.git .claude/skills/zurich-opendata
```

Der Skill ist dann nur innerhalb dieses Projekts verfügbar.

## Verwendung

### Automatische Aktivierung

Claude erkennt selbständig, wann dieser Skill relevant ist. Stelle einfach eine Frage zu Zürich:

```
Was ist die aktuelle Luftqualität in Zürich?
```

```
Zeige mir freie Parkhäuser in der Innenstadt.
```

```
Welche Stadtratsbeschlüsse gibt es zum Thema Klimaschutz?
```

```
Liste die Schulen im Kreis Wiedikon auf.
```

### Direkte Aktivierung

Du kannst den Skill auch direkt per Slash-Befehl aufrufen:

```
/zurich-opendata Fahrgastzahlen der Linie 4
```

## Projektstruktur

```
zurich-opendata/
├── SKILL.md                    # Skill-Definition (Einstiegspunkt)
└── references/
    └── api_reference.md        # Vollständige API-Dokumentation
```

## Weiterführende Links

- [data.stadt-zuerich.ch](https://data.stadt-zuerich.ch) – Offizielle Open-Data-Plattform der Stadt Zürich
- [zurich-opendata-mcp](https://github.com/malkreide/zurich-opendata-mcp) – Ursprünglicher MCP-Server von malkreide
- [Claude Code Skills-Dokumentation](https://code.claude.com/docs/en/skills)
