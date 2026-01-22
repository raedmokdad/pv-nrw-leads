# PV-NRW-Leads: Skalierbare Lead-Generierung für ganz NRW

**Automatisierte PV-Lead-Generierung auf Basis von NRW Open Data – kostenfrei, skalierbar, mit B2B-Features**

---

## 📋 Vision & Problemstellung

### Aktuelles System (Google Solar API)

Das bestehende System (`src/pv_roof_leads/`) nutzt die Google Solar API für Dortmund:

- **Kosten:** $85/1.000 Gebäude (Building Insights $10 + Data Layers $75)
- **Beschränkung:** Nur Dortmund, manuelle Skalierung aufwändig
- **Abhängigkeit:** Google API, Rate Limits, monatliche Kosten bei Skalierung

**Hochrechnung für NRW (50.000 Gebäude):**
- Google Solar API: **$4.600**
- Google Places (Top 200 Leads × 10 Städte): $64
- **Gesamtkosten: $4.664**

### Neue Lösung (NRW Open Data)

Das neue System (`src/pv_nrw_leads/`) nutzt kostenlose Open Data Quellen:

- **Kosten:** $0 für Basisdaten, optional $32-64 für Google Places API (nur Top-Leads)
- **Skalierung:** Ganz NRW (397 Städte/Gemeinden) automatisiert verarbeitbar
- **Kontrolle:** Alle Daten lokal, keine Rate Limits, keine monatlichen API-Kosten
- **B2B-Features:** Firmenerkennung, Netzanschlussanalyse, Stromverbrauchsschätzung

**Hochrechnung für NRW (50.000 Gebäude):**
- Open Data: **$0**
- Google Places (optional, Top 200 × 10): $64
- **Gesamtkosten: $64** → **98,6% Ersparnis**

---

## 🗺️ Datenquellen

### Primäre Quellen (NRW Open Data, kostenfrei)

| Datenquelle | Beschreibung | Format | URL | Ersetzt |
|-------------|--------------|--------|-----|---------|
| **Solarkataster NRW** | LiDAR-basierte Dachanalyse:<br>- Dachneigung (pitch)<br>- Ausrichtung (azimuth)<br>- Verschattung<br>- Solarpotenzial (kWp)<br>- Jahresertrag (kWh/Jahr) | Shapefile/GeoJSON | [opengeodata.nrw.de](https://www.opengeodata.nrw.de/produkte/umwelt_klima/energie/solarkataster/) | **Google Solar API** ✅ |
| **Hausumringe NRW** | Gebäudeumrisse (Footprints) | WFS | [opengeodata.nrw.de](https://www.opengeodata.nrw.de/produkte/geobasis/lk/akt/hausumringe/) | ALKIS |
| **INSPIRE Adressen** | Straße, Hausnummer, PLZ | WFS | [wfs.nrw.de](https://www.wfs.nrw.de/geobasis/wfs_nw_inspire-adressen) | ALKIS Adressen |
| **Energieatlas NRW** | Bestehende PV-Anlagen (WFS) | WFS | [energieatlas.nrw.de](https://www.energieatlas.nrw.de/site/wfs) | Zusätzlich zu MaStR |
| **MaStR** | Marktstammdatenregister | CSV | [marktstammdatenregister.de](https://www.marktstammdatenregister.de/) | Behalten |
| **OSM Power** | Transformatoren & Umspannwerke | Overpass API | [overpass-turbo.eu](https://overpass-turbo.eu/) | **NEU: Netzanschluss** ✅ |
| **OSM POI** | Firmen/Gewerbe (office, shop, industrial) | Overpass API | [overpass-turbo.eu](https://overpass-turbo.eu/) | **NEU: B2B-Leads** ✅ |
| **DOP10 NRW** | 10cm Luftbilder (Orthophotos) | WMS | [wms.nrw.de](https://www.wms.nrw.de/geobasis/wms_nw_dop) | Google Aerial Imagery |

### Sekundäre Quellen (optional, kostenpflichtig)

| Datenquelle | Beschreibung | Kosten | Wann nutzen? |
|-------------|--------------|--------|--------------|
| **Google Places API** | Firmenname, Kontaktdaten, Website | $32/1.000 Requests | Nur für Top 100-200 Leads pro Stadt |
| **Gemini Vision AI** | Dachzustand, Material, Hindernisse | ~$0.0003/Bild | Nur für finale Top-Leads (wie bisher) |

---

## 🏗️ System-Architektur

### Modul-Übersicht

```
src/pv_nrw_leads/                    # Neues System (parallel zu pv_roof_leads/)
│
├── core/                             # Basis-Utilities
│   ├── __init__.py
│   ├── geo_utils.py                  # CRS-Transformation, spatial joins
│   ├── download.py                   # WFS/WMS/HTTP Downloader
│   └── config_loader.py              # YAML Stadt-Configs laden
│
├── ingest/                           # Daten-Ingestion (Phase 1)
│   ├── __init__.py
│   ├── hausumringe_loader.py         # REPLACES: normalize_alkis.py
│   ├── solarkataster_loader.py       # REPLACES: enrich_with_google_solar.py
│   ├── inspire_addresses_loader.py   # NEW: INSPIRE WFS
│   ├── energieatlas_loader.py        # NEW: Energieatlas WFS (bestehende PV)
│   ├── osm_power_loader.py           # NEW: OSM Transformatoren
│   └── osm_poi_loader.py             # NEW: OSM Firmen/POI
│
├── enrich/                           # Datenanreicherung (Phase 2-3)
│   ├── __init__.py
│   ├── pv_checker.py                 # Triple-Check: MaStR + Energieatlas + OSM
│   ├── grid_connection_enricher.py   # NEW: Entfernung zu Trafo berechnen
│   ├── company_enricher.py           # NEW: Firmen identifizieren + anreichern
│   └── consumption_estimator.py      # NEW: Stromverbrauch schätzen (heuristisch)
│
├── scoring/                          # Lead-Bewertung (Phase 4)
│   ├── __init__.py
│   ├── lead_scorer.py                # NEW: Multi-Faktor-Scoring-Algorithmus
│   └── economic_calculator.py        # REUSE: ROI, Amortisation (erweitert)
│
├── export/                           # Reports & Exports (Phase 5)
│   ├── __init__.py
│   ├── excel_exporter.py             # REUSE: Excel mit 4 Tabs (angepasst)
│   └── report_generator.py           # REUSE: HTML-Report (angepasst)
│
├── config/                           # Stadt-Konfigurationen
│   ├── cities.yaml                   # Stadt-Metadaten (AGS, Bbox, PLZ)
│   └── scoring_weights.yaml          # Scoring-Parameter
│
└── cli.py                            # CLI-Interface (Click)
```

### Code-Wiederverwendung (70%)

| Modul (alt) | Modul (neu) | Status |
|-------------|-------------|--------|
| `normalize_alkis.py` | `hausumringe_loader.py` | Anpassen (WFS statt Shapefile) |
| `enrich_with_google_solar.py` | `solarkataster_loader.py` | Ersetzen (Open Data) |
| `check_existing_pv.py` | `enrich/pv_checker.py` | Erweitern (3 Quellen) |
| `acquire_osm.py` | `ingest/osm_*.py` | Wiederverw. + erweitern |
| `filter_area.py` | Integriert in `scoring/lead_scorer.py` | Vereinfachen |
| `economic_calculator.py` | `scoring/economic_calculator.py` | Wiederverw. + Netzkosten |
| `create_complete_report.py` | `export/report_generator.py` | Anpassen (neue Felder) |
| `export_to_excel.py` | `export/excel_exporter.py` | Anpassen (B2B-Felder) |

---

## 🔄 Datenfluss-Pipeline

### Phase 1: Daten-Acquisition (Data Ingestion)

```
┌─────────────────────────────────────────────────────────────────┐
│                    DATENQUELLEN (NRW Open Data)                 │
├─────────────┬──────────────┬──────────────┬────────────────────┤
│ Solarkataster│ Hausumringe  │   INSPIRE    │   OSM (Power, POI) │
│   NRW        │   NRW        │   Adressen   │   Energieatlas     │
└──────┬───────┴──────┬───────┴──────┬───────┴──────┬─────────────┘
       │              │              │              │
       ▼              ▼              ▼              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   INGEST-LAYER (Loader-Module)                  │
│  solarkataster_    hausumringe_    inspire_      osm_power_     │
│  loader.py         loader.py       addresses_    loader.py      │
│                                    loader.py     osm_poi_       │
│                                                  loader.py      │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              DATA CACHE (data/nrw_cache/*.parquet)              │
│  - solarkataster_<city>.parquet                                 │
│  - hausumringe_<city>.parquet                                   │
│  - adressen_<city>.parquet                                      │
│  - transformers_<city>.parquet                                  │
│  - poi_<city>.parquet                                           │
└─────────────────────────────────────────────────────────────────┘
```

**Output:** 5 Parquet-Dateien pro Stadt (kompakt, schnell, GeoPandas-ready)

---

### Phase 2: Räumliche Joins (Spatial Operations)

```
┌─────────────────────────────────────────────────────────────────┐
│                    SPATIAL JOIN ENGINE                          │
└─────────────────────────────┬───────────────────────────────────┘
                              │
      ┌───────────────────────┼───────────────────────┐
      │                       │                       │
      ▼                       ▼                       ▼
┌─────────────┐      ┌─────────────┐       ┌─────────────┐
│ Hausumringe │      │Solarkataster│       │  Adressen   │
│  (Polygon)  │ JOIN │  (Polygon)  │ JOIN  │   (Point)   │
│             │──────│             │───────│             │
│   (contains)│      │(intersects) │       │  (within)   │
└─────────────┘      └─────────────┘       └─────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────────┐
│         buildings_with_solar_<city>.parquet                     │
│  Felder: geometry, address, roof_area_m2, pitch, azimuth,      │
│          solar_potential_kwp, annual_yield_kwh                  │
└─────────────────────────────────────────────────────────────────┘
```

**Output:** `buildings_with_solar_<city>.parquet`

---

### Phase 3: Datenanreicherung (Enrichment)

```
┌─────────────────────────────────────────────────────────────────┐
│         buildings_with_solar_<city>.parquet                     │
└──────────────────────────┬──────────────────────────────────────┘
                           │
      ┌────────────────────┼────────────────────┐
      │                    │                    │
      ▼                    ▼                    ▼
┌─────────────┐   ┌─────────────────┐   ┌─────────────────┐
│ PV-Check    │   │ Grid Connection │   │ Company Data    │
│ (Triple)    │   │ Enrichment      │   │ Enrichment      │
│             │   │                 │   │                 │
│ MaStR       │   │ OSM             │   │ OSM POI         │
│ Energieatlas│   │ Transformers    │   │ (optional       │
│ OSM Solar   │   │ (nearest        │   │ Google Places)  │
│             │   │  neighbor)      │   │                 │
└─────┬───────┘   └────────┬────────┘   └────────┬────────┘
      │                    │                     │
      └────────────────────┼─────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ PV-Status   │ Grid Distance │ Company Name  │ Consumption Est.  │
│ (boolean)   │ (meters)      │ (string)      │ (kWh/year)        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│            buildings_enriched_<city>.parquet                    │
│  + has_pv, grid_distance_m, company_name, est_consumption_kwh   │
└─────────────────────────────────────────────────────────────────┘
```

**Stromverbrauch-Heuristik:**
```
est_consumption_kwh = building_area_m2 × building_type_factor × industry_multiplier

building_type_factor:
  - Wohngebäude: 50 kWh/m²/Jahr
  - Büro/Gewerbe: 150 kWh/m²/Jahr
  - Industrie: 300 kWh/m²/Jahr
  - Handel: 200 kWh/m²/Jahr

industry_multiplier (falls OSM Tags vorhanden):
  - Bäcker, Metzger: 2.0
  - Supermarkt, Restaurant: 1.5
  - Hotel: 1.8
  - Default: 1.0
```

**Output:** `buildings_enriched_<city>.parquet`

---

### Phase 4: Lead-Scoring (Bewertung)

```
┌─────────────────────────────────────────────────────────────────┐
│            buildings_enriched_<city>.parquet                    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                 LEAD SCORING ALGORITHMUS                        │
│                                                                 │
│  lead_score = Σ (faktor_i × gewicht_i)                         │
│                                                                 │
│  Faktoren (0-100 normalisiert):                                │
│  1. Solarpotenzial (40%):                                      │
│     - annual_yield_kwh / building_area_m2                      │
│  2. Stromverbrauch (25%):                                      │
│     - est_consumption_kwh                                      │
│  3. Eigenverbrauchsquote (20%):                                │
│     - min(annual_yield / consumption, 1.0) × 100               │
│  4. Kein bestehendes PV (10%):                                 │
│     - 100 wenn has_pv=false, sonst 0                           │
│  5. Netzanschluss-Qualität (5%):                               │
│     - 100 - (grid_distance_m / 500) × 100  [max 500m]          │
│                                                                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              buildings_scored_<city>.parquet                    │
│  + lead_score (0-100), priority_tier (A/B/C/D)                 │
└─────────────────────────────────────────────────────────────────┘
```

**Priority Tiers:**
- **A (85-100):** Top-Leads, sofort kontaktieren
- **B (70-84):** Gute Leads, Kampagne
- **C (50-69):** Potenzielle Leads, Nurturing
- **D (<50):** Niedrige Priorität

**Output:** `buildings_scored_<city>.parquet`

---

### Phase 5: Export & Reporting

```
┌─────────────────────────────────────────────────────────────────┐
│              buildings_scored_<city>.parquet                    │
└──────────────────────────┬──────────────────────────────────────┘
                           │
      ┌────────────────────┼────────────────────┐
      │                    │                    │
      ▼                    ▼                    ▼
┌─────────────┐   ┌─────────────────┐   ┌─────────────────┐
│ Excel       │   │ HTML Report     │   │ Optional:       │
│ Export      │   │ (Leaflet Map)   │   │ - GeoJSON       │
│             │   │                 │   │ - CSV           │
│ 4 Sheets:   │   │ - Alle Gebäude  │   │ - JSON          │
│ 1. All      │   │ - Top 100 Pins  │   │                 │
│ 2. No PV    │   │ - Klickbar      │   │                 │
│ 3. Top 100  │   │ - Statistiken   │   │                 │
│ 4. Stats    │   │                 │   │                 │
└─────────────┘   └─────────────────┘   └─────────────────┘
```

**Excel-Struktur (4 Sheets):**

1. **All Buildings** (alle gescorten Gebäude)
   - Adresse, Solarpotenzial, Lead Score, Priorität, Firma, Kontakt
2. **No Existing PV** (gefiltert: has_pv=false)
   - Gleiche Spalten wie Sheet 1
3. **Top 100 Leads** (sortiert nach lead_score DESC)
   - Alle Details + ROI-Berechnungen
4. **Statistics**
   - Anzahl Gebäude, Durchschnitts-Potenzial, Top-Bezirke, etc.

**Output:**
- `data/exports/<city>/<city>_leads.xlsx`
- `data/exports/<city>/<city>_report.html`

---

## 🎯 Wirtschaftlichkeitsrechnung

### Erweiterte Kennzahlen

```python
# Basis (wie bisher)
investment_eur = solar_potential_kwp × 1200  # €/kWp
annual_savings_eur = annual_yield_kwh × 0.30  # €/kWh Strompreis
payback_years = investment_eur / annual_savings_eur

# NEU: Netzanschluss-Kosten einbeziehen
grid_connection_cost_eur = 0 if grid_distance_m < 50 else (
    5000 + (grid_distance_m - 50) × 100  # €5k Basis + €100/Meter
)
total_investment_eur = investment_eur + grid_connection_cost_eur
adjusted_payback_years = total_investment_eur / annual_savings_eur

# NEU: Eigenverbrauchsquote
self_consumption_ratio = min(annual_yield_kwh / est_consumption_kwh, 1.0)
self_consumption_savings = annual_yield_kwh × self_consumption_ratio × 0.30
feed_in_revenue = (annual_yield_kwh × (1 - self_consumption_ratio)) × 0.08
total_annual_revenue = self_consumption_savings + feed_in_revenue

# NEU: ROI mit Eigenverbrauch
roi_percentage = (total_annual_revenue / total_investment_eur) × 100
```

**Beispiel-Rechnung:**

| Parameter | Wert |
|-----------|------|
| Solarpotenzial | 50 kWp |
| Jahresertrag | 45.000 kWh/Jahr |
| Geschätzter Verbrauch | 40.000 kWh/Jahr |
| Eigenverbrauchsquote | 89% (40k/45k) |
| Investition (PV-Anlage) | 60.000 € (50 kWp × 1.200 €) |
| Netzanschluss-Distanz | 120m |
| Netzanschluss-Kosten | 12.000 € (5k + 70m × 100€) |
| **Gesamtinvestition** | **72.000 €** |
| Eigenverbrauch-Ersparnis | 10.800 € (36k kWh × 0.30 €) |
| Einspeisevergütung | 360 € (4,5k kWh × 0.08 €) |
| **Jahresertrag gesamt** | **11.160 €** |
| **Amortisation** | **6,5 Jahre** |
| **ROI** | **15,5% p.a.** |

---

## 💾 Datenspeicherung

### Dateistruktur

```
data/
├── nrw_cache/                       # Rohdaten-Cache (Phase 1)
│   ├── solarkataster/
│   │   ├── dortmund.parquet
│   │   ├── koeln.parquet
│   │   └── essen.parquet
│   ├── hausumringe/
│   │   ├── dortmund.parquet
│   │   └── ...
│   ├── adressen/
│   ├── transformers/
│   └── poi/
│
├── processed/                       # Verarbeitete Daten (Phase 2-4)
│   ├── dortmund/
│   │   ├── buildings_with_solar.parquet
│   │   ├── buildings_enriched.parquet
│   │   └── buildings_scored.parquet
│   ├── koeln/
│   └── ...
│
└── exports/                         # Finale Outputs (Phase 5)
    ├── dortmund/
    │   ├── dortmund_leads.xlsx
    │   ├── dortmund_report.html
    │   └── dortmund_leads.geojson
    └── ...
```

### Parquet vs. CSV

| Kriterium | CSV | Parquet |
|-----------|-----|---------|
| Dateigröße (5.000 Gebäude) | 50 MB | 5 MB (10x kleiner) |
| Ladezeit (GeoPandas) | 15s | 1s (15x schneller) |
| Datentypen | String (alles) | Typisiert (int, float, bool) |
| Kompression | Keine | Automatisch (Snappy/GZIP) |
| Geometrien | WKT-String | Binär (WKB) |
| SQL-Queries | Nein | Ja (mit DuckDB) |
| Menschenlesbar | Ja | Nein |

**Empfehlung:** Parquet für interne Verarbeitung, CSV/Excel nur für finale Exports.

---

## 🚀 CLI-Interface

### Befehle

```bash
# Stadt vollständig verarbeiten (alle 5 Phasen)
python -m src.pv_nrw_leads.cli process dortmund

# Nur spezifische Phase ausführen
python -m src.pv_nrw_leads.cli ingest dortmund
python -m src.pv_nrw_leads.cli enrich dortmund
python -m src.pv_nrw_leads.cli score dortmund
python -m src.pv_nrw_leads.cli export dortmund

# Mehrere Städte parallel
python -m src.pv_nrw_leads.cli batch --cities dortmund,koeln,essen --workers 3

# Batch-Verarbeitung aller NRW-Städte >100k Einwohner
python -m src.pv_nrw_leads.cli batch --filter "population>100000" --workers 5

# Export-Formate
python -m src.pv_nrw_leads.cli export dortmund --format excel
python -m src.pv_nrw_leads.cli export dortmund --format html
python -m src.pv_nrw_leads.cli export dortmund --format geojson
python -m src.pv_nrw_leads.cli export dortmund --format all

# Cache löschen (Neustart)
python -m src.pv_nrw_leads.cli cache-clear dortmund
python -m src.pv_nrw_leads.cli cache-clear --all
```

### Stadt-Konfiguration (YAML)

```yaml
# config/cities.yaml
cities:
  dortmund:
    ags: "05913000"               # Amtlicher Gemeindeschlüssel
    name: "Dortmund"
    state: "NRW"
    population: 587010
    bbox:                         # Bounding Box (WGS84)
      minx: 7.3082
      miny: 51.4075
      maxx: 7.6283
      maxy: 51.5927
    postal_codes:                 # Alle PLZ in Dortmund
      - "44135"
      - "44137"
      # ... (weitere PLZ)
    solarkataster_url: "https://www.opengeodata.nrw.de/.../dortmund.zip"
    hausumringe_wfs: "https://www.wfs.nrw.de/geobasis/wfs_nw_alkis_aaa-modell-basiert"
    
  koeln:
    ags: "05315000"
    name: "Köln"
    state: "NRW"
    population: 1083498
    bbox:
      minx: 6.7725
      miny: 50.8283
      maxx: 7.1618
      maxy: 51.0849
    # ...
```

---

## 📊 Multi-City Skalierung

### Automatisierung

**Batch-Verarbeitung mit parallel-Funktion:**

```python
# Pseudo-Code
def process_city_batch(city_list, max_workers=5):
    with ProcessPoolExecutor(max_workers=max_workers) as executor:
        futures = {
            executor.submit(process_city, city): city
            for city in city_list
        }
        for future in as_completed(futures):
            city = futures[future]
            try:
                result = future.result()
                logger.info(f"✓ {city}: {result['total_leads']} leads")
            except Exception as e:
                logger.error(f"✗ {city}: {e}")
```

**Performance-Hochrechnung (10 Städte, je 5.000 Gebäude):**

| Phase | Zeit/Stadt | Zeit/10 Städte (parallel, 5 Workers) | Gesamt |
|-------|------------|--------------------------------------|--------|
| 1. Ingest | 5 min | 10 min (2 Batches) | 10 min |
| 2. Spatial Joins | 3 min | 6 min | 6 min |
| 3. Enrichment | 8 min | 16 min | 16 min |
| 4. Scoring | 1 min | 2 min | 2 min |
| 5. Export | 2 min | 4 min | 4 min |
| **GESAMT** | **19 min** | **38 min** | **38 min** |

**NRW-weit (397 Städte/Gemeinden):**
- Mit 10 Workers parallel: ~15 Stunden
- Einmalig ausführbar, danach nur Updates (neue Gebäude)

---

## 💰 Kosten-Nutzen-Analyse

### Detaillierte Kostenvergleichung

**Szenario 1: 10 Städte (50.000 Gebäude)**

| Position | Google Solar Ansatz | NRW Open Data Ansatz | Ersparnis |
|----------|---------------------|----------------------|-----------|
| Google Solar API (Building Insights) | $500 | $0 | $500 |
| Google Solar API (Data Layers) | $3.750 | $0 | $3.750 |
| Google Aerial Imagery | $0 (inkl.) | $0 (DOP10 NRW) | $0 |
| Google Places API (Top 200 × 10) | $64 | $64 (optional) | $0 |
| Gemini Vision (500 Bilder) | $0.15 | $0.15 | $0 |
| Serverkosten (Railway) | $20/Monat | $20/Monat | $0 |
| **GESAMT (1. Monat)** | **$4.334** | **$84** | **$4.250 (98,1%)** |
| **GESAMT (12 Monate)** | **$4.574** | **$324** | **$4.250 (92,9%)** |

**Szenario 2: NRW-weit (200.000 Gebäude)**

| Position | Google Solar Ansatz | NRW Open Data Ansatz | Ersparnis |
|----------|---------------------|----------------------|-----------|
| Google Solar API | $17.000 | $0 | $17.000 |
| Google Places (Top 500) | $16 | $16 | $0 |
| Gemini Vision (500 Bilder) | $0.15 | $0.15 | $0 |
| Server (höher wg. Traffic) | $50/Monat | $50/Monat | $0 |
| **GESAMT (1. Monat)** | **$17.066** | **$66** | **$17.000 (99,6%)** |
| **GESAMT (12 Monate)** | **$17.666** | **$666** | **$17.000 (96,2%)** |

### ROI für Entwicklung

**Entwicklungsaufwand:**
- 4 Wochen × 6h/Tag × 5 Tage = 120 Stunden
- Stundensatz: 80 €/h
- **Entwicklungskosten: 9.600 €**

**Break-Even:**
- Bei 10 Städten (50k Gebäude): Nach 2,2 Monaten
- Bei NRW-weit (200k Gebäude): Nach 1 Monat

**Langfristiger Nutzen (12 Monate):**
- 10 Städte: 9.600 € Invest vs. 4.250 € Ersparnis → **Verlust im 1. Jahr**
- NRW-weit: 9.600 € Invest vs. 17.000 € Ersparnis → **+7.400 € Gewinn im 1. Jahr**

**Empfehlung:** System lohnt sich ab 25-30 Städten oder bei geplanter NRW-weiter Skalierung.

---

## 📈 Erfolgsmetriken

### KPIs (Key Performance Indicators)

| Metrik | Zielwert | Messung |
|--------|----------|---------|
| **Datenqualität** | | |
| PV-Erkennungsrate | >95% | Triple-Check (MaStR + Energieatlas + OSM) |
| Vollständige Adressen | >90% | INSPIRE Adressen-Coverage |
| Vollständige Firmendaten (Top 100) | >80% | OSM + Google Places |
| **Performance** | | |
| Verarbeitungszeit (5.000 Gebäude) | <20 Min | Single City Benchmark |
| Batch-Throughput | >250 Gebäude/Min | Multi-City Parallel |
| Cache-Hit-Rate | >70% | Wiederholte Ausführungen |
| **Lead-Qualität** | | |
| Durchschn. Lead-Score (Top 100) | >75 | Scoring-Algorithmus |
| PV-freie Quote (Top 100) | >90% | Triple-Check Accuracy |
| Kontaktierbare Leads (Email/Tel) | >60% | Google Places Enrichment |
| **Wirtschaftlichkeit** | | |
| Kosten/1.000 Gebäude | <$5 | Open Data + optional Places |
| Durchschn. ROI (Top 100 Leads) | >12% p.a. | Economic Calculator |
| Durchschn. Payback (Top 100) | <8 Jahre | Investment Calculation |

### Reporting-Dashboard

**Statistiken pro Stadt (JSON-Output):**

```json
{
  "city": "dortmund",
  "date": "2026-01-21",
  "statistics": {
    "total_buildings": 5201,
    "buildings_with_solar_data": 5180,
    "buildings_no_existing_pv": 4823,
    "buildings_scored": 4823,
    "avg_lead_score": 62.3,
    "top_100_avg_score": 87.5,
    "avg_solar_potential_kwp": 38.7,
    "avg_annual_yield_kwh": 35420,
    "total_potential_mwp": 186.5,
    "total_annual_potential_gwh": 171.2,
    "company_buildings": 1247,
    "avg_grid_distance_m": 87.3,
    "processing_time_seconds": 1134
  },
  "priority_distribution": {
    "A": 342,
    "B": 891,
    "C": 2145,
    "D": 1445
  }
}
```

---

## 🔮 Erweiterungsmöglichkeiten

### Phase 1 (MVP - 4 Wochen) ✅

- [x] Solarkataster NRW Integration
- [x] Triple-PV-Check (MaStR + Energieatlas + OSM)
- [x] Netzanschluss-Analyse (OSM Transformatoren)
- [x] Firmen-Identifikation (OSM POI + Google Places)
- [x] Stromverbrauch-Schätzung (heuristisch)
- [x] Multi-Faktor Lead-Scoring
- [x] Excel/HTML Export
- [x] Multi-City Batch-Verarbeitung

### Phase 2 (Optimierungen - 2 Wochen)

- [ ] **DuckDB-Integration:**
  - SQL-Queries über alle Städte hinweg
  - Aggregierte Reports (Top 100 NRW-weit)
  - Spatial Queries ohne GeoPandas

- [ ] **Erweiterte OSM-Daten:**
  - Dachtyp (flat, gabled, hipped) aus OSM `building:roof:shape`
  - Gebäudealter aus OSM `building:year` → Sanierungsbedarf
  - Stockwerke aus OSM `building:levels` → Verbrauchsschätzung

- [ ] **Visualisierungen:**
  - Heatmaps (Solarpotenzial pro Bezirk)
  - Vergleichsdiagramme (Stadt A vs. Stadt B)
  - Zeitreihen (monatliche Neu-Leads)

### Phase 3 (ML & Advanced - 4 Wochen)

- [ ] **Machine Learning:**
  - Trainiere Modell: "Wahrscheinlichkeit für PV-Installation"
  - Features: Gebäudetyp, Alter, Bezirk, Nachbarschaft-PV-Quote
  - Output: `ml_conversion_probability` (0-100%)

- [ ] **Stromnetz-Analyse:**
  - Last-Analyse: Trafo-Kapazität aus OSM `power=transformer` + `transformer:rating`
  - Netzauslastung: Bestehende PV-Anlagen pro Trafo aggregieren
  - Ampelsystem: Grün (<50% Auslastung), Gelb (50-80%), Rot (>80%)

- [ ] **EEG-Förderung:**
  - Automatische Prüfung: Gebäude-PLZ → Förderprogramme-API
  - Berechnung: Investition mit Zuschüssen, aktualisierte ROI
  - Beispiel: NRW Progres.nrw, KfW 270, BEG-Förderung

- [ ] **Satelliten-Zeitreihen:**
  - Copernicus Sentinel-2 (kostenlos): Zeitreihen-Analyse
  - Erkennung: Wann wurde PV-Anlage installiert? (Change Detection)
  - Validierung: MaStR-Inbetriebnahme vs. Satellitendaten

### Phase 4 (Produktisierung - 2 Wochen)

- [ ] **REST API:**
  - FastAPI-Backend: `GET /api/leads?city=dortmund&min_score=70`
  - Authentifizierung: API-Keys für Kunden
  - Rate Limiting: 1000 Requests/Tag

- [ ] **Web-Dashboard:**
  - React-Frontend mit Mapbox/Leaflet
  - Filterbare Tabelle (Datatable.js)
  - CRM-Integration (HubSpot, Salesforce API)

- [ ] **Automatisierte Updates:**
  - Cronjob: Wöchentliches Update (neue MaStR-Einträge)
  - Delta-Processing: Nur neue Gebäude verarbeiten
  - Notification: Email bei neuen High-Priority Leads

---

## 🔧 Technologie-Stack

### Python-Bibliotheken

```toml
[tool.poetry.dependencies]
python = "^3.11"

# Geo-Daten
geopandas = "^0.14"
pyproj = "^3.6"          # CRS-Transformation
shapely = "^2.0"         # Geometrie-Operationen
rtree = "^1.2"           # Spatial Index (schnellere Joins)

# Daten-I/O
pyarrow = "^15.0"        # Parquet Reader/Writer
duckdb = "^0.10"         # SQL auf Parquet (optional)
requests = "^2.31"       # HTTP Downloads
owslib = "^0.30"         # WFS/WMS Client

# Verarbeitung
pandas = "^2.2"
numpy = "^1.26"
scikit-learn = "^1.4"    # ML (Phase 3)

# CLI & Config
click = "^8.1"           # CLI Framework
pyyaml = "^6.0"          # YAML Parser
python-dotenv = "^1.0"   # .env Support

# Logging & Monitoring
loguru = "^0.7"          # Besseres Logging
tqdm = "^4.66"           # Progress Bars

# Export
openpyxl = "^3.1"        # Excel Writer
jinja2 = "^3.1"          # HTML Templates

# Optional (Phase 2+)
fastapi = "^0.109"       # REST API
uvicorn = "^0.27"        # ASGI Server
```

### Externe Services (optional)

| Service | Zweck | Kosten |
|---------|-------|--------|
| Google Places API | Firmendaten (Top Leads) | $32/1.000 |
| Google Gemini | Dachanalyse (Final Leads) | ~$0.0003/Bild |
| Railway.app | Hosting (Reports) | $5-20/Monat |
| GitHub Actions | CI/CD (Tests, Deploy) | Gratis |

---

## 📚 Ressourcen & Links

### NRW Open Data Portale

- **Open.NRW:** https://open.nrw/ (Hauptportal)
- **OpenGeoData.NRW:** https://www.opengeodata.nrw.de/ (Geodaten)
- **Energieatlas NRW:** https://www.energieatlas.nrw.de/
- **WFS.NRW.DE:** https://www.wfs.nrw.de/ (Web Feature Services)
- **WMS.NRW.DE:** https://www.wms.nrw.de/ (Web Map Services)

### Solarkataster

- **Produktseite:** https://www.opengeodata.nrw.de/produkte/umwelt_klima/energie/solarkataster/
- **Dokumentation:** https://www.lanuv.nrw.de/landesamt/veroeffentlichungen/solarkataster
- **Methodik:** LiDAR-basiert, 1m Auflösung, Verschattungsanalyse

### Weitere Datenquellen

- **Marktstammdatenregister:** https://www.marktstammdatenregister.de/MaStR
- **OpenStreetMap Overpass API:** https://overpass-turbo.eu/
- **INSPIRE Addressen WFS:** https://www.wfs.nrw.de/geobasis/wfs_nw_inspire-adressen

### Tutorials

- **GeoPandas Spatial Joins:** https://geopandas.org/en/stable/gallery/spatial_joins.html
- **Parquet mit Python:** https://arrow.apache.org/docs/python/parquet.html
- **DuckDB Spatial Extension:** https://duckdb.org/docs/extensions/spatial.html

---

## 📞 Support & Kontakt

Bei Fragen oder Problemen:

- **Entwickler:** Raed Mokdad
- **Projekt:** PV-NRW-Leads
- **Repository:** (internes GitLab/GitHub)
- **Todoist-Projekt:** [PV-NRW-System (4 Wochen)](https://todoist.com/)

---

## 📝 Change-Log

| Datum | Version | Änderungen |
|-------|---------|------------|
| 2026-01-21 | 1.0.0 | Initial-Konzept (README-NRW.md) |

---

*Dieses Dokument beschreibt das Konzept und die geplante Implementierung. Der tatsächliche Code befindet sich in `src/pv_nrw_leads/` und wird schrittweise über 4 Wochen entwickelt (siehe Todoist-Plan).*

**Entwicklungsstatus:** 🟡 Planung abgeschlossen, Implementierung startet
