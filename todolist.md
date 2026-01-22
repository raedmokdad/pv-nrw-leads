# 📋 TODOIST-PLAN: PV-NRW-Leads-System (4 Wochen)

**Gesamtaufwand:** 120 Stunden (20 Tage × 6h)  
**Start:** 22. Januar 2026  
**Ziel:** Produktionsreifes MVP für Dortmund + 4 weitere Städte

---

## 🗓️ WOCHE 1: Setup + Datenquellen (Tag 1-5)

### ✅ Tag 1: Projekt-Setup (6h) `#prio1` `#setup`

**Deliverable:** Lauffähige Projekt-Struktur

**Tasks:**
- [ ] Virtual Environment erstellen (`.venv`)
  ```bash
  python -m venv .venv
  .venv\Scripts\activate
  ```
- [ ] `requirements.txt` updaten
  ```txt
  geopandas>=0.14
  pyarrow>=15.0
  requests>=2.31
  pyyaml>=6.0
  click>=8.1
  loguru>=0.7
  tqdm>=4.66
  owslib>=0.30
  openpyxl>=3.1
  jinja2>=3.1
  python-dotenv>=1.0
  ```
- [ ] Git-Branch anlegen: `git checkout -b nrw-system`
- [ ] Ordnerstruktur anlegen:
  ```bash
  mkdir -p src/pv_nrw_leads/{core,ingest,enrich,scoring,export,pipeline}
  mkdir -p config
  mkdir -p data/nrw_cache/{solarkataster,hausumringe,adressen,transformers,poi}
  mkdir -p data/cities/{dortmund,koeln,essen,bochum,duisburg}/{raw,processed,final}
  mkdir -p docs
  mkdir -p tests
  ```
  cmd : mkdir -Force -Path 'src\pv_nrw_leads\core','src\pv_nrw_leads\ingest','src\pv_nrw_leads\enrich','src\pv_nrw_leads\scoring','src\pv_nrw_leads\export','src\pv_nrw_leads\pipeline'
  '''
  
- [ ] `config/cities.yaml` Template erstellen (siehe unten)
- [ ] `.gitignore` erweitern:
  ```gitignore
  *.parquet
  data/nrw_cache/
  data/cities/*/processed/
  data/cities/*/final/
  ```
- [ ] Commit: `"Initial NRW system structure"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 2: NRW-Datenquellen recherchieren + downloaden (6h) `#prio1` `#data`

**Deliverable:** Solarkataster Dortmund lokal verfügbar

**Tasks:**
- [ ] Solarkataster Dortmund downloaden (105 MB Shapefile)
  - URL: https://www.opengeodata.nrw.de/produkte/umwelt_klima/energie/solarkataster/
  - Datei: `Solarpotenzial_Dortmund.zip`
- [ ] Unzip nach `data/nrw_cache/solarkataster/dortmund_raw/`
- [ ] Zu GeoJSON konvertieren (QGIS oder Python):
  ```python
  import geopandas as gpd
  gdf = gpd.read_file('data/nrw_cache/solarkataster/dortmund_raw/Solarpotenzial.shp')
  gdf.to_file('data/nrw_cache/solarkataster/dortmund.geojson', driver='GeoJSON')
  ```
- [ ] Felder analysieren (mit QGIS oder `gdf.columns`)
- [ ] Energieatlas WFS testen (Browser: GetCapabilities)
  - URL: https://www.wfs.nrw.de/umwelt/erneuerbare_energien_wfs?SERVICE=WFS&REQUEST=GetCapabilities
  - Layer-Name notieren: `rea:photovoltaik_dach` oder ähnlich
- [ ] MaStR CSV aktualisieren (falls nötig)
  - Download: https://www.marktstammdatenregister.de/MaStR/Datendownload
  - Speichern: `data/raw/dortmund/mastr_pv.csv`
- [ ] Datenquellen-Doku schreiben: `docs/DATA_SOURCES_NRW.md`
  - Alle URLs dokumentieren
  - Feldnamen-Mapping (z.B. Solarkataster → Unser Modell)

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 3: Solarkataster-Loader entwickeln (6h) `#prio1` `#code`

**Deliverable:** Solarkataster-Daten in Python geladen

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/solarkataster_loader.py`
- [ ] Funktion implementieren:
  ```python
  def load_solarkataster_for_city(city_name: str) -> gpd.GeoDataFrame:
      """
      Lädt Solarkataster-Daten für eine Stadt.
      
      Returns:
          GeoDataFrame mit Feldern:
          - geometry (Polygon)
          - roof_area_m2
          - pitch (Neigung in Grad)
          - azimuth (Ausrichtung in Grad, 180=Süd)
          - solar_potential_kwp
          - annual_yield_kwh
          - shading_factor (0-1)
      """
  ```
- [ ] Feld-Mapping dokumentieren (Solarkataster-Feldnamen → Unsere Feldnamen)
- [ ] Test-Script schreiben: `tests/test_solarkataster_loader.py`
  ```python
  gdf = load_solarkataster_for_city('dortmund')
  print(f"Loaded {len(gdf)} roof segments")
  print(gdf.head(10))
  ```
- [ ] Visualisierung (optional): Folium-Karte mit 100 besten Dachflächen
- [ ] Commit: `"Add solarkataster loader"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 4a: ALKIS Dortmund laden (3h) `#prio1` `#code`

**Deliverable:** ALKIS-Gebäudedaten in Python

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/alkis_loader.py`
- [ ] Funktion implementieren:
  ```python
  def load_alkis_for_dortmund() -> gpd.GeoDataFrame:
      """
      Lädt bestehende ALKIS-Daten für Dortmund.
      Nutzt: data/raw/dortmund/alkis_buildings/
      
      Returns:
          GeoDataFrame mit Feldern:
          - geometry (Polygon)
          - building_id
          - building_function (ALKIS-Code)
          - address_street
          - address_number
          - postal_code
          - area_m2
      """
  ```
- [ ] Normalisierung (minimal, basierend auf `normalize_alkis.py`):
  - Geometrie-Validierung
  - CRS-Transformation zu EPSG:4326
  - Flächen-Berechnung
- [ ] Test: Wie viele Gebäude? Welche Felder vorhanden?
- [ ] Commit: `"Add ALKIS loader for Dortmund"`

**Zeitschätzung:** 3 Stunden

---

### ✅ Tag 4b: Spatial Join Engine (3h) `#prio1` `#code`

**Deliverable:** Gebäude mit Solar-Potenzial (Parquet)

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/core/spatial_joiner.py`
- [ ] Funktion implementieren:
  ```python
  def join_buildings_with_solar(
      buildings: gpd.GeoDataFrame,
      solarkataster: gpd.GeoDataFrame
  ) -> gpd.GeoDataFrame:
      """
      Spatial Join: Gebäude × Solarkataster-Dachflächen
      
      Methode: intersects oder contains
      Aggregation: Wenn mehrere Dachflächen pro Gebäude → summieren
      
      Returns:
          GeoDataFrame mit allen Gebäude-Feldern + Solar-Feldern
      """
  ```
- [ ] Implementierung:
  - `gpd.sjoin(buildings, solarkataster, how='left', predicate='intersects')`
  - Aggregation: `groupby('building_id').agg({'solar_potential_kwp': 'sum', ...})`
- [ ] Output speichern: `data/cities/dortmund/processed/buildings_with_solar.parquet`
- [ ] Test: Match-Quote? (sollte >95% sein)
  ```python
  print(f"Buildings with solar data: {(gdf['solar_potential_kwp'] > 0).sum()}")
  ```
- [ ] Commit: `"Add spatial join engine"`

**Zeitschätzung:** 3 Stunden

---

### ✅ Tag 5a: OSM Trafos laden (3h) `#prio2` `#code`

**Deliverable:** Transformatoren-Daten für Dortmund

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/osm_power_loader.py`
- [ ] Overpass Query schreiben:
  ```python
  query = """
  [out:json][timeout:60];
  (
    node["power"="transformer"](51.4075,7.3082,51.5927,7.6283);
    node["power"="substation"](51.4075,7.3082,51.5927,7.6283);
  );
  out body;
  """
  ```
- [ ] Funktion implementieren:
  ```python
  def load_osm_transformers_for_city(city_name: str) -> gpd.GeoDataFrame:
      """
      Lädt Transformatoren aus OSM.
      
      Returns:
          GeoDataFrame mit Feldern:
          - geometry (Point)
          - osm_id
          - power_type (transformer/substation)
          - voltage (falls vorhanden)
      """
  ```
- [ ] Output: `data/nrw_cache/osm/transformers_dortmund.geojson`
- [ ] Test: Wie viele Trafos gefunden?
- [ ] Commit: `"Add OSM power loader"`

**Zeitschätzung:** 3 Stunden

---

### ✅ Tag 5b: OSM POI laden (3h) `#prio2` `#code`

**Deliverable:** Firmen-POI-Daten für Dortmund

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/osm_poi_loader.py`
- [ ] Overpass Query schreiben:
  ```python
  query = """
  [out:json][timeout:60];
  (
    node["office"](51.4075,7.3082,51.5927,7.6283);
    way["office"](51.4075,7.3082,51.5927,7.6283);
    node["shop"](51.4075,7.3082,51.5927,7.6283);
    way["shop"](51.4075,7.3082,51.5927,7.6283);
    node["industrial"](51.4075,7.3082,51.5927,7.6283);
    way["industrial"](51.4075,7.3082,51.5927,7.6283);
  );
  out body;
  >;
  out skel qt;
  """
  ```
- [ ] Funktion implementieren:
  ```python
  def load_osm_poi_for_city(city_name: str) -> gpd.GeoDataFrame:
      """
      Lädt Firmen/POI aus OSM.
      
      Returns:
          GeoDataFrame mit Feldern:
          - geometry (Point oder Polygon)
          - osm_id
          - name (Firmenname, falls vorhanden)
          - poi_type (office/shop/industrial)
          - tags (Dict mit allen OSM-Tags)
      """
  ```
- [ ] Output: `data/nrw_cache/osm/poi_dortmund.geojson`
- [ ] Test: Wie viele POIs gefunden?
- [ ] Commit: `"Add OSM POI loader"`

**Zeitschätzung:** 3 Stunden

---

## 🗓️ WOCHE 2: Enrichment + PV-Detection (Tag 6-10)

### ✅ Tag 6: INSPIRE Adressen + Energieatlas (6h) `#prio1` `#code`

**Deliverable:** Beide WFS-Module funktionieren

**Tasks (Teil 1 - INSPIRE, 3h):**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/inspire_addresses_loader.py`
- [ ] WFS-Request implementieren:
  ```python
  from owslib.wfs import WebFeatureService
  
  def load_inspire_addresses_for_city(city_name: str) -> gpd.GeoDataFrame:
      """
      Lädt Adressen aus INSPIRE WFS.
      
      WFS: https://www.wfs.nrw.de/geobasis/wfs_nw_inspire-adressen
      Layer: ad:Address
      """
      wfs = WebFeatureService(url=WFS_URL, version='2.0.0')
      # GetFeature mit BBox-Filter
  ```
- [ ] Spatial Join: Gebäude × Adressen (nearest neighbor, max 50m)
- [ ] Feld hinzufügen: `address_full` (Straße + Hausnr + PLZ)

**Tasks (Teil 2 - Energieatlas, 3h):**
- [ ] Modul anlegen: `src/pv_nrw_leads/ingest/energieatlas_loader.py`
- [ ] WFS-Request implementieren:
  ```python
  def load_energieatlas_pv_for_city(city_name: str) -> gpd.GeoDataFrame:
      """
      Lädt bestehende PV-Anlagen aus Energieatlas NRW.
      
      WFS: https://www.wfs.nrw.de/umwelt/erneuerbare_energien_wfs
      Layer: rea:photovoltaik_dach (oder ähnlich)
      """
  ```
- [ ] Output: `data/nrw_cache/energieatlas/pv_dortmund.geojson`
- [ ] Test: Wie viele PV-Anlagen in Dortmund?
- [ ] Commit: `"Add INSPIRE addresses + Energieatlas loaders"`

**Zeitschätzung:** 6 Stunden (3h + 3h)

---

### ✅ Tag 7: PV-Detection kombinieren (6h) `#prio1` `#code`

**Deliverable:** Triple-Check PV-Erkennung (MaStR + Energieatlas + OSM)

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/enrich/pv_detector.py`
- [ ] 3 Quellen laden:
  ```python
  mastr_pv = load_mastr_pv()  # Aus bestehendem check_existing_pv.py
  energieatlas_pv = load_energieatlas_pv_for_city('dortmund')
  osm_pv = load_osm_pv_for_city('dortmund')  # Aus extract_osm_pv.py
  ```
- [ ] Deduplizierung:
  ```python
  def detect_existing_pv(buildings: gpd.GeoDataFrame) -> gpd.GeoDataFrame:
      """
      Spatial Join mit allen 3 Quellen (Buffer 50m).
      Deduplizierung: Wenn 2+ Quellen gleichen Punkt haben → 1 PV.
      
      Returns:
          buildings + Felder:
          - has_existing_pv (bool)
          - pv_detection_sources (list: ["mastr", "energieatlas", "osm"])
          - pv_capacity_kwp (aus MaStR/Energieatlas)
      """
  ```
- [ ] Test: Vergleich mit altem `check_existing_pv.py`:
  - Wie viele PV-Anlagen mehr gefunden?
  - Falsch-Positiv-Rate?
- [ ] Commit: `"Combine PV detection sources (triple-check)"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 8: Netzanschluss-Enrichment (6h) `#prio1` `#code`

**Deliverable:** Netzanschluss-Daten pro Gebäude

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/enrich/grid_connection_enricher.py`
- [ ] Funktion implementieren:
  ```python
  def enrich_grid_connection(
      buildings: gpd.GeoDataFrame,
      transformers: gpd.GeoDataFrame
  ) -> gpd.GeoDataFrame:
      """
      Findet nächsten Trafo für jedes Gebäude.
      
      Methode: Spatial nearest neighbor (Shapely nearest_points)
      
      Returns:
          buildings + Felder:
          - grid_nearest_trafo_id (OSM-ID)
          - grid_distance_m (Luftlinie)
          - grid_voltage_level (aus OSM, falls vorhanden)
          - grid_connection_cost_eur (Heuristik: 5000 + distance × 100)
      """
  ```
- [ ] Implementierung:
  ```python
  from shapely.ops import nearest_points
  for idx, building in buildings.iterrows():
      nearest_trafo = transformers.distance(building.geometry).idxmin()
      distance = building.geometry.distance(transformers.loc[nearest_trafo].geometry)
      # In Meter umrechnen (haversine formula)
  ```
- [ ] Test: Durchschnittliche Distanz? (sollte 50-150m sein)
- [ ] Visualisierung (optional): Top 10 Gebäude mit Linien zu Trafos
- [ ] Commit: `"Add grid connection enrichment"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 9: Firmendaten-Enrichment (6h) `#prio2` `#code`

**Deliverable:** Basis-Firmendaten (OSM, kostenlos)

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/enrich/company_enricher.py`
- [ ] Teil 1: ALKIS Gebäudefunktion auswerten (2h)
  ```python
  BUILDING_TYPE_MAPPING = {
      '31001': 'residential',
      '31002': 'office',
      '31003': 'industrial',
      '31004': 'retail',
      # ... (siehe ALKIS-Katalog)
  }
  ```
- [ ] Teil 2: OSM POI matchen (3h)
  ```python
  def enrich_company_data(
      buildings: gpd.GeoDataFrame,
      osm_poi: gpd.GeoDataFrame
  ) -> gpd.GeoDataFrame:
      """
      Spatial Join: Gebäude × OSM-POI (20m Buffer).
      
      Returns:
          buildings + Felder:
          - building_type (residential/office/industrial/retail)
          - company_name (aus OSM, wenn vorhanden)
          - company_type (aus OSM tags: office=company, shop=bakery, ...)
          - osm_poi_id
      """
  ```
- [ ] Test: Wie viele Gebäude mit Firmenname? (Erwartung: ~10-15%)
- [ ] Commit: `"Add company data enrichment (OSM-based)"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 10: Stromverbrauch-Heuristik (6h) `#prio2` `#code`

**Deliverable:** Stromverbrauch-Schätzung pro Gebäude

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/enrich/consumption_estimator.py`
- [ ] Lookup-Tabelle definieren:
  ```python
  CONSUMPTION_FACTORS = {
      'residential': 50,    # kWh/m²/Jahr
      'office': 150,
      'industrial': 300,
      'retail': 200,
      'hotel': 180,
      'school': 80,
      'hospital': 250,
      'default': 100
  }
  
  INDUSTRY_MULTIPLIERS = {
      'bakery': 2.0,
      'butcher': 2.0,
      'supermarket': 1.5,
      'restaurant': 1.5,
      'hotel': 1.8,
      'default': 1.0
  }
  ```
- [ ] Funktion implementieren:
  ```python
  def estimate_consumption(buildings: gpd.GeoDataFrame) -> gpd.GeoDataFrame:
      """
      Schätzt Stromverbrauch basierend auf:
      - Gebäudetyp (ALKIS)
      - Fläche (m²)
      - Branche (OSM-Tags, falls vorhanden)
      
      Formel:
      consumption = area_m2 × consumption_factor × industry_multiplier
      
      Returns:
          buildings + Felder:
          - estimated_consumption_kwh (int)
          - consumption_confidence (low/medium/high)
      """
  ```
- [ ] Confidence-Level:
  - `high`: Gebäudetyp bekannt + OSM-Branche
  - `medium`: Nur Gebäudetyp bekannt
  - `low`: Default-Wert
- [ ] Test: Plausibilität prüfen (Bürogebäude 1000m² → 150.000 kWh/Jahr)
- [ ] Commit: `"Add consumption estimation (heuristic)"`

**Zeitschätzung:** 6 Stunden

---

## 🗓️ WOCHE 3: Scoring + Reporting (Tag 11-15)

### ✅ Tag 11: Lead-Scoring-Algorithmus (6h) `#prio1` `#code`

**Deliverable:** Multi-Faktor-Scoring funktioniert

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/scoring/lead_scorer.py`
- [ ] Score-Komponenten definieren:
  ```python
  SCORING_WEIGHTS = {
      'solar_potential': 0.40,      # 40%
      'consumption': 0.25,           # 25%
      'self_consumption_ratio': 0.20, # 20%
      'no_existing_pv': 0.10,        # 10%
      'grid_connection': 0.05        # 5%
  }
  ```
- [ ] Normalisierungs-Funktionen (0-100):
  ```python
  def normalize_solar_potential(annual_yield_kwh, building_area_m2):
      """Normiert auf 0-100 (Best: 150 kWh/m²/Jahr = 100)"""
      return min((annual_yield_kwh / building_area_m2) / 150 * 100, 100)
  
  def normalize_consumption(consumption_kwh):
      """Normiert auf 0-100 (Best: >100.000 kWh/Jahr = 100)"""
      return min(consumption_kwh / 100000 * 100, 100)
  
  def normalize_self_consumption_ratio(yield_kwh, consumption_kwh):
      """Eigenverbrauchsquote: min(yield/consumption, 1.0) × 100"""
      return min(yield_kwh / consumption_kwh, 1.0) * 100
  
  def normalize_grid_connection(distance_m):
      """100 - (distance / 500) × 100  [max 500m]"""
      return max(100 - (distance_m / 500) * 100, 0)
  ```
- [ ] Haupt-Funktion:
  ```python
  def calculate_lead_score(building: pd.Series) -> float:
      """
      Berechnet Lead-Score (0-100).
      
      Returns:
          float: Gewichteter Score
      """
      score = 0
      score += normalize_solar_potential(...) * SCORING_WEIGHTS['solar_potential']
      score += normalize_consumption(...) * SCORING_WEIGHTS['consumption']
      # ...
      return round(score, 2)
  ```
- [ ] Priority Tiers:
  ```python
  def assign_priority_tier(score: float) -> str:
      if score >= 85: return 'A'
      elif score >= 70: return 'B'
      elif score >= 50: return 'C'
      else: return 'D'
  ```
- [ ] Test: Top 10 Leads anschauen, macht Score Sinn?
- [ ] Commit: `"Add lead scoring algorithm"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 12: Wirtschaftlichkeits-Berechnung (6h) `#prio1` `#code`

**Deliverable:** ROI mit Netzanschluss-Kosten

**Tasks:**
- [ ] Modul erweitern: `src/pv_nrw_leads/scoring/economic_calculator.py`
  (basiert auf bestehendem `economic_calculator.py`, aber erweitert)
- [ ] Neue Funktionen:
  ```python
  def calculate_grid_connection_cost(distance_m: float) -> float:
      """
      Netzanschluss-Kosten:
      - < 50m: 0 € (Hausanschluss)
      - > 50m: 5.000 € Basis + (distance - 50) × 100 €/m
      """
      if distance_m < 50:
          return 0
      return 5000 + (distance_m - 50) * 100
  
  def calculate_self_consumption_savings(
      annual_yield_kwh: float,
      consumption_kwh: float,
      electricity_price_per_kwh: float = 0.30
  ) -> tuple[float, float, float]:
      """
      Berechnet Eigenverbrauch + Einspeisung.
      
      Returns:
          (self_consumption_kwh, self_consumption_savings_eur, feed_in_revenue_eur)
      """
      self_consumption_kwh = min(annual_yield_kwh, consumption_kwh)
      self_consumption_savings = self_consumption_kwh * electricity_price_per_kwh
      
      feed_in_kwh = annual_yield_kwh - self_consumption_kwh
      feed_in_revenue = feed_in_kwh * 0.08  # 8 Cent/kWh
      
      return (self_consumption_kwh, self_consumption_savings, feed_in_revenue)
  
  def calculate_economics_with_grid(building: pd.Series) -> dict:
      """
      Vollständige Wirtschaftlichkeitsrechnung.
      
      Returns:
          {
              'pv_investment_eur': ...,
              'grid_connection_cost_eur': ...,
              'total_investment_eur': ...,
              'annual_self_consumption_savings_eur': ...,
              'annual_feed_in_revenue_eur': ...,
              'total_annual_revenue_eur': ...,
              'payback_period_years': ...,
              'roi_percentage': ...,
          }
      """
  ```
- [ ] Integration in Pipeline:
  ```python
  buildings['economics'] = buildings.apply(calculate_economics_with_grid, axis=1)
  # Dann dict zu Spalten expandieren
  ```
- [ ] Test: Beispiel-Rechnung (siehe README-NRW.md Beispiel)
- [ ] Commit: `"Add economic calculations with grid costs"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 13: Top-500 Leads filtern + sortieren (6h) `#prio1` `#code`

**Deliverable:** Top 500 Leads Dortmund (Parquet)

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/pipeline/lead_processor.py`
- [ ] Filter-Funktion:
  ```python
  def filter_leads(buildings: gpd.GeoDataFrame) -> gpd.GeoDataFrame:
      """
      Filter-Kriterien:
      1. has_existing_pv == False (keine PV)
      2. building_type in ['office', 'industrial', 'retail'] (B2B-Fokus)
      3. lead_score >= 40 (Mindest-Score)
      4. solar_potential_kwp >= 10 (mind. 10 kWp)
      
      Returns:
          Gefiltertes GeoDataFrame
      """
      filtered = buildings[
          (~buildings['has_existing_pv']) &
          (buildings['building_type'].isin(['office', 'industrial', 'retail'])) &
          (buildings['lead_score'] >= 40) &
          (buildings['solar_potential_kwp'] >= 10)
      ]
      return filtered
  ```
- [ ] Sortierung:
  ```python
  def rank_leads(leads: gpd.GeoDataFrame, top_n: int = 500) -> gpd.GeoDataFrame:
      """
      Sortiert nach lead_score DESC, nimmt Top N.
      Fügt Feld 'rank' hinzu (1-500).
      """
      ranked = leads.sort_values('lead_score', ascending=False).head(top_n)
      ranked['rank'] = range(1, len(ranked) + 1)
      return ranked
  ```
- [ ] Statistiken berechnen:
  ```python
  def calculate_statistics(leads: gpd.GeoDataFrame) -> dict:
      return {
          'total_leads': len(leads),
          'avg_lead_score': leads['lead_score'].mean(),
          'avg_solar_potential_kwp': leads['solar_potential_kwp'].mean(),
          'total_potential_mwp': leads['solar_potential_kwp'].sum() / 1000,
          'avg_payback_years': leads['payback_period_years'].mean(),
          # ...
      }
  ```
- [ ] Output: `data/cities/dortmund/final/leads_top500.parquet`
- [ ] Test: Liste anschauen, macht Rang-Reihenfolge Sinn?
- [ ] Commit: `"Add lead filtering and ranking"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 14: Excel-Export (6h) `#prio1` `#code`

**Deliverable:** Verkaufsfähige Excel-Datei

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/export/excel_exporter.py`
- [ ] Spalten-Definition:
  ```python
  EXCEL_COLUMNS = {
      'Rang': 'rank',
      'Adresse': 'address_full',
      'PLZ': 'postal_code',
      'Stadt': 'city',
      'Gebäudetyp': 'building_type',
      'Firma': 'company_name',
      'Dachfläche (m²)': 'roof_area_m2',
      'Solar-Potenzial (kWp)': 'solar_potential_kwp',
      'Jahresertrag (kWh)': 'annual_yield_kwh',
      'Geschätzter Verbrauch (kWh)': 'estimated_consumption_kwh',
      'Eigenverbrauch-Quote (%)': 'self_consumption_ratio',
      'Netzanschluss-Distanz (m)': 'grid_distance_m',
      'Investition (€)': 'total_investment_eur',
      'Amortisation (Jahre)': 'payback_period_years',
      'ROI (%)': 'roi_percentage',
      'Lead-Score': 'lead_score',
      'Priorität': 'priority_tier',
  }
  ```
- [ ] 4 Sheets erstellen:
  ```python
  def export_to_excel(
      all_buildings: gpd.GeoDataFrame,
      leads_top500: gpd.GeoDataFrame,
      output_path: str
  ):
      with pd.ExcelWriter(output_path, engine='openpyxl') as writer:
          # Sheet 1: Alle Leads (ohne PV)
          all_leads = all_buildings[~all_buildings['has_existing_pv']]
          all_leads[EXCEL_COLUMNS.values()].to_excel(writer, sheet_name='Alle Leads', index=False)
          
          # Sheet 2: Nur ohne PV + Gewerblich
          no_pv = all_leads[all_leads['building_type'].isin(['office', 'industrial', 'retail'])]
          no_pv[EXCEL_COLUMNS.values()].to_excel(writer, sheet_name='Ohne PV (Gewerbe)', index=False)
          
          # Sheet 3: Top 500
          leads_top500[EXCEL_COLUMNS.values()].to_excel(writer, sheet_name='Top 500', index=False)
          
          # Sheet 4: Statistiken
          stats = calculate_statistics(leads_top500)
          pd.DataFrame([stats]).T.to_excel(writer, sheet_name='Statistiken')
  ```
- [ ] Formatierung:
  ```python
  # Header: Bold, Hintergrund grau
  # Zahlen: Tausender-Trenner, 2 Dezimalstellen
  # Euro: € Symbol
  # Prozent: % Symbol
  ```
- [ ] Test: Excel öffnen, Qualität prüfen
- [ ] Commit: `"Add Excel export with 4 sheets"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 15: Basis HTML-Report (4h) `#prio1` `#code`

**Deliverable:** Interaktiver HTML-Report (vereinfacht)

**Tasks:**
- [ ] Template anlegen: `src/pv_nrw_leads/templates/report_nrw.html` (Jinja2)
- [ ] Leaflet-Karte:
  ```html
  <div id="map" style="height: 600px;"></div>
  <script>
    var map = L.map('map').setView([51.5135, 7.4653], 12);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);
    
    // Nur Top 100 Gebäude (Performance!)
    {% for building in top_100_buildings %}
      var marker = L.circleMarker([{{ building.lat }}, {{ building.lon }}], {
        radius: 8,
        fillColor: '{{ building.score_color }}',  // Grün > 85, Gelb > 70, Orange > 50
        color: '#000',
        weight: 1,
        opacity: 1,
        fillOpacity: 0.8
      }).addTo(map);
      
      marker.bindPopup(`
        <b>Rang {{ building.rank }}</b><br>
        {{ building.address }}<br>
        Potenzial: {{ building.solar_potential_kwp }} kWp<br>
        Score: {{ building.lead_score }}
      `);
    {% endfor %}
    
    // Trafos als ⚡-Icons
    {% for trafo in transformers %}
      L.marker([{{ trafo.lat }}, {{ trafo.lon }}], {
        icon: L.divIcon({html: '⚡', className: 'trafo-icon'})
      }).addTo(map);
    {% endfor %}
    
    // KEINE Verbindungslinien (zu komplex für MVP)
  </script>
  ```
- [ ] Tabelle: Top 100 Leads (DataTables.js für Sortierung/Filterung)
- [ ] Statistiken-Box:
  ```html
  <div class="stats">
    <h3>Statistiken Dortmund</h3>
    <p>Gesamt-Potenzial: {{ total_potential_mwp }} MWp</p>
    <p>Durchschnitt-Score: {{ avg_score }}</p>
    <p>Top 100 Investition: {{ total_investment_mio }} Mio. €</p>
  </div>
  ```
- [ ] Modul: `src/pv_nrw_leads/export/report_generator.py`
  ```python
  def generate_html_report(
      leads_top100: gpd.GeoDataFrame,
      transformers: gpd.GeoDataFrame,
      output_path: str
  ):
      from jinja2 import Environment, FileSystemLoader
      env = Environment(loader=FileSystemLoader('src/pv_nrw_leads/templates'))
      template = env.get_template('report_nrw.html')
      
      html = template.render(
          top_100_buildings=leads_top100.to_dict('records'),
          transformers=transformers.to_dict('records'),
          statistics=calculate_statistics(leads_top100)
      )
      
      with open(output_path, 'w', encoding='utf-8') as f:
          f.write(html)
  ```
- [ ] Test: Im Browser öffnen (Chrome/Firefox)
- [ ] Commit: `"Add HTML report with map (simplified)"`

**Zeitschätzung:** 4 Stunden

---

### ✅ Tag 15 (Bonus): Advanced Report Features (2h) `#optional`

**Nur wenn Zeit übrig!**

**Tasks:**
- [ ] Verbindungslinien (on-click):
  ```javascript
  marker.on('click', function() {
      // Zeichne Linie zu nächstem Trafo
      var trafo_latlng = [trafo.lat, trafo.lon];
      L.polyline([marker.getLatLng(), trafo_latlng], {color: 'blue'}).addTo(map);
  });
  ```
- [ ] Heatmap: Solarpotenzial-Dichte
  ```javascript
  var heat = L.heatLayer(heatmapData, {radius: 25}).addTo(map);
  ```
- [ ] Marker-Clustering:
  ```javascript
  var markers = L.markerClusterGroup();
  // Alle Marker zu Cluster hinzufügen
  ```
- [ ] Commit: `"Add advanced report features (optional)"`

**Zeitschätzung:** 2 Stunden (optional)

---

## 🗓️ WOCHE 4: Multi-Stadt + Automation (Tag 16-20)

### ✅ Tag 16: Multi-Stadt-Config (6h) `#prio1` `#config`

**Deliverable:** Stadt-Metadaten für 5 Städte

**Tasks:**
- [ ] `config/cities.yaml` erweitern:
  ```yaml
  cities:
    dortmund:
      ags: "05913000"
      name: "Dortmund"
      state: "NRW"
      population: 587010
      bbox:
        minx: 7.3082
        miny: 51.4075
        maxx: 7.6283
        maxy: 51.5927
      postal_codes:
        - "44135"
        - "44137"
        - "44139"
        # ... (alle PLZ)
      solarkataster_url: "https://www.opengeodata.nrw.de/.../dortmund.zip"
      hausumringe_wfs: "https://www.wfs.nrw.de/geobasis/..."
    
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
      postal_codes:
        - "50667"
        - "50668"
        # ...
      solarkataster_url: "https://..."
    
    essen:
      ags: "05113000"
      name: "Essen"
      # ...
    
    bochum:
      ags: "05911000"
      name: "Bochum"
      # ...
    
    duisburg:
      ags: "05112000"
      name: "Duisburg"
      # ...
  ```
- [ ] Config-Loader: `src/pv_nrw_leads/core/config_loader.py`
  ```python
  import yaml
  
  def load_cities_config() -> dict:
      with open('config/cities.yaml') as f:
          return yaml.safe_load(f)
  
  def get_city_config(city_name: str) -> dict:
      config = load_cities_config()
      if city_name not in config['cities']:
          raise ValueError(f"City {city_name} not found in config")
      return config['cities'][city_name]
  ```
- [ ] Test: Alle Städte laden
  ```python
  for city in ['dortmund', 'koeln', 'essen', 'bochum', 'duisburg']:
      config = get_city_config(city)
      print(f"{config['name']}: {config['population']} Einwohner")
  ```
- [ ] Commit: `"Add multi-city config (5 cities)"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 17: Pipeline-Runner (6h) `#prio1` `#code`

**Deliverable:** CLI für vollautomatische Pipeline

**Tasks:**
- [ ] Modul anlegen: `src/pv_nrw_leads/cli.py`
- [ ] CLI mit Click implementieren:
  ```python
  import click
  from loguru import logger
  from tqdm import tqdm
  
  @click.group()
  def cli():
      """PV-NRW-Leads CLI"""
      pass
  
  @cli.command()
  @click.option('--city', required=True, help='Stadt (z.B. dortmund)')
  def process(city: str):
      """Verarbeitet eine Stadt komplett (alle Phasen)"""
      logger.info(f"Processing {city}...")
      
      # Phase 1: Daten laden
      logger.info("Phase 1: Loading data...")
      buildings = load_alkis_for_dortmund() if city == 'dortmund' else load_hausumringe_for_city(city)
      solarkataster = load_solarkataster_for_city(city)
      transformers = load_osm_transformers_for_city(city)
      poi = load_osm_poi_for_city(city)
      
      # Phase 2: Spatial Joins
      logger.info("Phase 2: Spatial joins...")
      buildings = join_buildings_with_solar(buildings, solarkataster)
      
      # Phase 3: Enrichment
      logger.info("Phase 3: Enrichment...")
      buildings = detect_existing_pv(buildings)
      buildings = enrich_grid_connection(buildings, transformers)
      buildings = enrich_company_data(buildings, poi)
      buildings = estimate_consumption(buildings)
      
      # Phase 4: Scoring
      logger.info("Phase 4: Scoring...")
      buildings['lead_score'] = buildings.apply(calculate_lead_score, axis=1)
      buildings['priority_tier'] = buildings['lead_score'].apply(assign_priority_tier)
      buildings = buildings.apply(calculate_economics_with_grid, axis=1, result_type='expand')
      
      # Phase 5: Export
      logger.info("Phase 5: Export...")
      leads_top500 = filter_and_rank_leads(buildings, top_n=500)
      export_to_excel(buildings, leads_top500, f'data/cities/{city}/final/{city}_leads.xlsx')
      generate_html_report(leads_top500.head(100), transformers, f'data/cities/{city}/final/{city}_report.html')
      
      logger.success(f"✓ {city} processed successfully!")
  
  @cli.command()
  @click.option('--cities', required=True, help='Komma-separierte Liste (z.B. dortmund,koeln,essen)')
  def batch(cities: str):
      """Verarbeitet mehrere Städte sequenziell"""
      city_list = cities.split(',')
      for city in city_list:
          try:
              process.invoke(click.Context(process), city=city.strip())
          except Exception as e:
              logger.error(f"✗ {city} failed: {e}")
  
  if __name__ == '__main__':
      cli()
  ```
- [ ] Test:
  ```bash
  python -m src.pv_nrw_leads.cli process --city dortmund
  ```
- [ ] Commit: `"Add pipeline runner CLI"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 18: NRW-Cache-System (6h) `#prio2` `#optimization`

**Deliverable:** Effizientes Caching für Multi-Stadt

**Tasks:**
- [ ] Solarkataster für 5 Städte downloaden:
  - Dortmund (bereits vorhanden)
  - Köln (~150 MB)
  - Essen (~120 MB)
  - Bochum (~80 MB)
  - Duisburg (~90 MB)
- [ ] Alle nach `data/nrw_cache/solarkataster/` speichern
- [ ] Metadata-File: `data/nrw_cache/metadata.json`
  ```json
  {
    "solarkataster": {
      "dortmund": {
        "file": "dortmund.parquet",
        "downloaded": "2026-01-25",
        "source_url": "https://...",
        "row_count": 125483
      },
      "koeln": {
        "file": "koeln.parquet",
        "downloaded": "2026-01-25",
        "source_url": "https://...",
        "row_count": 187234
      }
    },
    "transformers": {
      "nrw_wide": {
        "file": "transformers_nrw.parquet",
        "downloaded": "2026-01-25",
        "row_count": 8734
      }
    }
  }
  ```
- [ ] Cache-Loader erweitern:
  ```python
  def load_solarkataster_for_city(city_name: str, use_cache: bool = True) -> gpd.GeoDataFrame:
      cache_file = f'data/nrw_cache/solarkataster/{city_name}.parquet'
      if use_cache and os.path.exists(cache_file):
          logger.info(f"Loading {city_name} from cache...")
          return gpd.read_parquet(cache_file)
      else:
          logger.info(f"Downloading {city_name}...")
          # Download-Logik
  ```
- [ ] OSM-Daten NRW-weit cachen (alle Trafos auf einmal):
  ```python
  # Overpass Query mit NRW-BBox (nicht pro Stadt)
  # Dann bei Bedarf: spatial filter für Stadt
  ```
- [ ] Test: 2. Stadt läuft schneller (Cache-Hit)
- [ ] Commit: `"Add NRW-wide data caching"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 19: Batch-Processing (6h) `#prio2` `#optimization`

**Deliverable:** Multi-Stadt parallel

**Tasks:**
- [ ] Pipeline-Runner erweitern: `--cities all`
  ```python
  @cli.command()
  @click.option('--cities', default='all', help='all oder komma-separiert')
  @click.option('--workers', default=3, help='Anzahl parallele Prozesse')
  def batch(cities: str, workers: int):
      if cities == 'all':
          config = load_cities_config()
          city_list = list(config['cities'].keys())
      else:
          city_list = cities.split(',')
      
      # Parallel Processing
      from concurrent.futures import ProcessPoolExecutor, as_completed
      with ProcessPoolExecutor(max_workers=workers) as executor:
          futures = {executor.submit(process_city, city): city for city in city_list}
          for future in tqdm(as_completed(futures), total=len(futures)):
              city = futures[future]
              try:
                  result = future.result()
                  logger.success(f"✓ {city}: {result['total_leads']} leads")
              except Exception as e:
                  logger.error(f"✗ {city}: {e}")
  
  def process_city(city: str) -> dict:
      """Wrapper für Multiprocessing"""
      # ... (gleiche Logik wie process-Command)
      return {'total_leads': len(leads_top500), 'city': city}
  ```
- [ ] Error Handling:
  - Stadt fehlgeschlagen → Log-Eintrag, weiter mit nächster
  - Timeout nach 30 Minuten pro Stadt
- [ ] Summary-Report:
  ```python
  @cli.command()
  def summary():
      """Zeigt Übersicht aller verarbeiteten Städte"""
      results = []
      for city_dir in Path('data/cities').iterdir():
          if city_dir.is_dir():
              stats_file = city_dir / 'final' / 'statistics.json'
              if stats_file.exists():
                  with open(stats_file) as f:
                      stats = json.load(f)
                      results.append(stats)
      
      # DataFrame mit Vergleich
      df = pd.DataFrame(results)
      print(df[['city', 'total_leads', 'avg_lead_score', 'total_potential_mwp']])
  ```
- [ ] Test: 3 Städte parallel
  ```bash
  python -m src.pv_nrw_leads.cli batch --cities dortmund,koeln,essen --workers 3
  ```
- [ ] Commit: `"Add parallel batch processing"`

**Zeitschätzung:** 6 Stunden

---

### ✅ Tag 20: Testing + Dokumentation (6h) `#prio1` `#finalize`

**Deliverable:** Produktionsreifes MVP

**Tasks:**
- [ ] Unit-Tests schreiben: `tests/test_solarkataster_loader.py`
  ```python
  import pytest
  from src.pv_nrw_leads.ingest.solarkataster_loader import load_solarkataster_for_city
  
  def test_load_solarkataster():
      gdf = load_solarkataster_for_city('dortmund')
      assert len(gdf) > 0
      assert 'solar_potential_kwp' in gdf.columns
      assert gdf.crs.to_epsg() == 4326
  
  # Weitere Tests: spatial_joiner, lead_scorer, etc.
  ```
- [ ] Integration-Test: Volle Pipeline Dortmund
  ```python
  def test_full_pipeline_dortmund():
      from src.pv_nrw_leads.cli import process_city
      result = process_city('dortmund')
      assert result['total_leads'] > 100
      assert os.path.exists('data/cities/dortmund/final/dortmund_leads.xlsx')
  ```
- [ ] README_NRW.md finalisieren:
  - [ ] Datenquellen-Links (alle URLs)
  - [ ] Ausführungsbeispiele (CLI-Befehle)
  - [ ] Troubleshooting (häufige Fehler)
  - [ ] FAQ (z.B. "Warum hat Stadt X weniger Leads?")
- [ ] `requirements.txt` freezen:
  ```bash
  pip freeze > requirements.txt
  ```
- [ ] Git-Tag: `v1.0.0-nrw-mvp`
  ```bash
  git tag -a v1.0.0-nrw-mvp -m "NRW system MVP complete"
  git push origin v1.0.0-nrw-mvp
  ```
- [ ] Demo-Video aufnehmen (5 Min):
  - Zeige CLI-Befehle
  - Öffne Excel-Datei
  - Zeige HTML-Report im Browser
- [ ] Commit: `"MVP complete - NRW system v1.0"`

**Zeitschätzung:** 6 Stunden

---

## 📊 OPTIONALE TASKS (Nach Woche 4)

### 🎁 Bonus 1: DOP10 Luftbilder (4h) `#optional`

**Tasks:**
- [ ] Modul: `src/pv_nrw_leads/ingest/dop_downloader.py`
- [ ] WMS GetMap Requests für Top 100 Gebäude
  ```python
  from owslib.wms import WebMapService
  
  wms = WebMapService('https://www.wms.nrw.de/geobasis/wms_nw_dop', version='1.3.0')
  
  for building in top_100:
      bbox = building.geometry.bounds  # (minx, miny, maxx, maxy)
      img = wms.getmap(
          layers=['nw_dop_rgb'],
          srs='EPSG:4326',
          bbox=bbox,
          size=(512, 512),
          format='image/jpeg'
      )
      with open(f'data/cities/dortmund/final/dop_images/{building.id}.jpg', 'wb') as f:
          f.write(img.read())
  ```
- [ ] Commit: `"Add DOP10 image download"`

---

### 🎁 Bonus 2: Gemini Dach-Analyse (4h) `#optional`

**Tasks:**
- [ ] Modul wiederverwenden: `src/pv_roof_leads/analyze_roof_with_gemini.py`
- [ ] Nur Top 50 Gebäude analysieren (Budget ~€0.15)
- [ ] Felder hinzufügen:
  - `roof_condition` (good/fair/poor)
  - `obstacles` (list: "chimney", "dormer", "skylight")
  - `roof_material` (tiles/metal/bitumen)
- [ ] In Excel-Export integrieren (neue Spalten)
- [ ] Commit: `"Add Gemini analysis for top leads"`

---

### 🎁 Bonus 3: Google Places Test (4h) `#optional`

**Tasks:**
- [ ] Modul: `src/pv_nrw_leads/enrich/google_places_enricher.py`
- [ ] Top 50 Leads: Firmenname + Kontakt holen
  ```python
  import googlemaps
  
  gmaps = googlemaps.Client(key=GOOGLE_API_KEY)
  
  for building in top_50:
      place = gmaps.places_nearby(
          location=(building.lat, building.lon),
          radius=50,
          type='establishment'
      )
      if place['results']:
          building['company_name'] = place['results'][0]['name']
          building['company_phone'] = place['results'][0].get('formatted_phone_number')
  ```
- [ ] Kosten-Tracking: Wie teuer ist es wirklich?
- [ ] Entscheidung: Lohnt sich Google Places? (Vergleich mit OSM-Daten)
- [ ] Commit: `"Test Google Places for top leads"`

---

## 📌 Zusammenfassung

**Gesamt-Zeitplan:**
- **Woche 1:** Setup + Daten (30h)
- **Woche 2:** Enrichment + PV-Detection (30h)
- **Woche 3:** Scoring + Reporting (30h)
- **Woche 4:** Multi-Stadt + Automation (30h)
- **Gesamt:** 120 Stunden

**MVP-Deliverables (Ende Woche 4):**
- ✅ Dortmund: 500 Leads (Excel + HTML)
- ✅ 5 Städte: Dortmund, Köln, Essen, Bochum, Duisburg
- ✅ CLI-Tool: `python -m src.pv_nrw_leads.cli batch --cities all`
- ✅ Kosten: €0 (Base) oder <€100 (mit Google Places)
- ✅ Processing-Zeit: <2h für 5 Städte

**Erfolgsmetriken:**
- PV-Erkennungsrate: >95% (Triple-Check)
- Vollständige Adressen: >90% (INSPIRE)
- Lead-Qualität: Avg. Score >70 (Top 100)
- Performance: <20 Min/Stadt

---

*Erstellt: 22. Januar 2026*  
*Status: 🟢 Bereit zum Start*
