# DatenaufbereitungBa

Dieses Repository enthält die Datenaufbereitung für die Bachelorarbeit zur automatisierten Bestimmung des Window-to-Wall Ratio (WWR) von Gebäudefassaden in Winterthur. Die Notebooks bereiten den ursprünglichen Winterthurer Gebäudedatensatz auf, prüfen Baujahre über die geo.admin.ch/GWR-API, erstellen einen Ground-Truth-Datensatz und erzeugen zusätzlich einen vorbereiteten Datensatz für die spätere Modellanwendung beziehungsweise Street-View-Bildabfrage.

## Ziel des Repositories

Das Repository dient dazu,

1. den Rohdatensatz der Winterthurer Gebäude einzulesen und zu analysieren,
2. die vorhandenen Baujahre mit Daten aus geo.admin.ch beziehungsweise dem Gebäude- und Wohnungsregister (GWR) abzugleichen,
3. einen nachvollziehbar ausgewählten Ground-Truth-Datensatz für die manuelle beziehungsweise modellbasierte Evaluation zu erstellen,
4. eine Tour-Datei für die Begehung der Ground-Truth-Gebäude zu erzeugen,
5. den gesamten Winterthurer Gebäudedatensatz so vorzubereiten, dass er später für die API-basierte Bildabfrage und Modellanwendung verwendet werden kann.

## Repository-Struktur

```text
DatenaufbereitungBa/
│
├── README.md
├── 01_Datenanalyse.ipynb
├── 02_Baujahr_Abfrage_API.ipynb
├── 03_Aufbereitung_Daten.ipynb
├── 04_GT_Tour.ipynb
├── WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb
│
└── Data/
    ├── Adressen_Winti_Dep_Arch.xlsx
    ├── 00_Adressen_Winti_Dep_Arch.xlsx
    ├── 01_Datenanalyse_Ergebnis.xlsx
    ├── 02_API_Baujahr_Adressen.xlsx
    ├── 03_Ground_Truth_Datensatz.xlsx
    ├── 04_GT_Tour.xlsx
    │
    └── WT01_modellanwendung/
        ├── WT01_geocoding_results.csv
        ├── streetview_results.csv              # optional, falls Resultate aus OPT-06 vorhanden sind
        └── WT01_Winterthur_vorbereiteter_Datensatz_<timestamp>.xlsx
```

Die Dateien im Ordner `Data/` werden nicht zwingend alle direkt mit dem Repository mitgeliefert. Die Rohdaten müssen lokal in diesem Ordner abgelegt werden. Die übrigen Dateien entstehen durch das Ausführen der Notebooks.

## Voraussetzungen

Benötigte Python-Pakete:

```bash
pip install pandas openpyxl numpy matplotlib seaborn requests
```

Für die Notebook-Ausführung wird eine lokale Python- oder Anaconda-Umgebung empfohlen. Die API-Abfragen benötigen eine Internetverbindung.

## Eingabedaten

Für die Ground-Truth-Aufbereitung wird im Notebook `01_Datenanalyse.ipynb` folgende Datei erwartet:

```text
Data/Adressen_Winti_Dep_Arch.xlsx
```

Für die separate Vorbereitung des gesamten Winterthur-Datensatzes in `WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb` wird folgende Datei erwartet:

```text
Data/00_Adressen_Winti_Dep_Arch.xlsx
```

Im WT-01-Notebook wird das Tabellenblatt `Gebäude Screening` eingelesen. Falls nur eine Rohdatei vorhanden ist, muss sie entweder entsprechend benannt oder der Dateipfad im jeweiligen Notebook angepasst werden.

## Workflow 1: Ground-Truth-Datensatz aufbereiten

Die Notebooks `01` bis `04` bauen aufeinander auf und sollten in dieser Reihenfolge ausgeführt werden.

### 1. `01_Datenanalyse.ipynb`

Dieses Notebook liest den ursprünglichen Excel-Datensatz ein und führt eine erste Datenanalyse durch.

Eingabe:

```text
Data/Adressen_Winti_Dep_Arch.xlsx
```

Wichtige Schritte:

- Einlesen des Rohdatensatzes
- Anzeigen der ersten Datenzeilen
- Prüfung von Spalten, Datentypen und fehlenden Werten
- Ermittlung der Anzahl eindeutiger EGIDs
- Analyse relevanter Merkmale wie `BAUJAHR`, `HAUPTNUTZUNG`, `GS_EIGENTUMSKATEGORIE` und `Fenster`
- Export einer ersten Ergebnisdatei

Output:

```text
Data/01_Datenanalyse_Ergebnis.xlsx
```

Im ausgeführten Notebook umfasst der Rohdatensatz 846 Zeilen, 55 Spalten und 770 eindeutige EGIDs. Mehrere Zeilen können zur gleichen EGID gehören, da einzelne Gebäude mehrere Adresszeilen aufweisen können.

### 2. `02_Baujahr_Abfrage_API.ipynb`

Dieses Notebook ergänzt und validiert die Baujahre über die geo.admin.ch/GWR-API.

Eingabe:

```text
Data/01_Datenanalyse_Ergebnis.xlsx
```

Wichtige Schritte:

- Aufbau einer vollständigen Adresse aus Strasse, Hausnummer, Hausnummerzusatz, PLZ und Ort
- Abfrage der Adresse über geo.admin.ch
- Zugriff auf das GWR-Feature über die erhaltene `feature_id`
- Extraktion von `egid_gwr`, `baujahr_gwr`, Koordinaten und API-Label
- Vergleich zwischen lokalem `BAUJAHR` und `baujahr_gwr`
- Ausschluss von Zeilen, bei denen das lokale Baujahr und das API-Baujahr nicht übereinstimmen

Output:

```text
Data/02_API_Baujahr_Adressen.xlsx
```

Im ausgeführten Notebook wurden aus 846 Zeilen nach dem Entfernen der Baujahr-Abweichungen 814 Zeilen weiterverwendet.

### 3. `03_Aufbereitung_Daten.ipynb`

Dieses Notebook bereitet die validierten Daten weiter auf und erstellt den finalen Ground-Truth-Datensatz.

Eingabe:

```text
Data/02_API_Baujahr_Adressen.xlsx
```

Wichtige Schritte:

- Bildung von `BAUJAHR_API` aus `baujahr_gwr`
- Einteilung der Baujahre in Baualtersklassen
- Reduktion auf Gebäudeebene durch eindeutige EGIDs
- Verdichtung von Nutzungs-, Eigentums- und Fensterkategorien
- Filterung auf relevante Hauptnutzungen
- Filterung auf relevante Eigentumskategorien
- Ausschluss unklarer Fensterangaben
- Erstellung einer repräsentativen Auswahl von 15 Ground-Truth-Gebäuden
- Prüfung der Verteilungen nach Baujahr, Hauptnutzung, Nutzung, Eigentum und Fensterkategorie
- Export des finalen Ground-Truth-Datensatzes

Output:

```text
Data/03_Ground_Truth_Datensatz.xlsx
```

Der finale Ground-Truth-Datensatz enthält 15 Gebäude. Die Auswahl berücksichtigt insbesondere die Hauptnutzungen `Verwaltungsgebäude und Gebäude mit öffentlichem Charakter`, `Wohngebäude` sowie `Industrie und Gerwerbe`. Zusätzlich werden Baujahrkategorien, Nutzungskategorien, Eigentumskategorien und Fensterkategorien berücksichtigt.

### 4. `04_GT_Tour.ipynb`

Dieses Notebook erstellt aus dem Ground-Truth-Datensatz eine vereinfachte Tour-Datei.

Eingabe:

```text
Data/03_Ground_Truth_Datensatz.xlsx
```

Wichtige Schritte:

- Einlesen des finalen Ground-Truth-Datensatzes
- Erzeugung einer reduzierten Tabelle mit Adresse, Latitude und Longitude
- Export als Excel-Datei für die Planung oder Durchführung einer Begehung

Output:

```text
Data/04_GT_Tour.xlsx
```

Die erzeugte Tour-Datei enthält 15 Zeilen und die Spalten `Nr`, `Adresse`, `lat` und `lon`.

## Workflow 2: Gesamten Winterthur-Datensatz für Modellanwendung vorbereiten

Das Notebook `WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb` ist ein separater Workflow für die spätere Anwendung der Bildabfrage und Modellpipeline auf den gesamten Winterthurer Gebäudedatensatz.

### `WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb`

Eingabe:

```text
Data/00_Adressen_Winti_Dep_Arch.xlsx
```

Erwartetes Tabellenblatt:

```text
Gebäude Screening
```

Wichtige Schritte:

- Einlesen des Rohdatensatzes
- Eindeutige Benennung mehrfach vorkommender Spaltennamen
- Reduktion auf Pflichtspalten:
  - `EGID`
  - `STRASSENNAME`
  - `HAUSNR`
  - `HAUSNRZUSATZ`
  - `PLZ4`
  - `ORT`
  - optional `BAUJAHR`
- Bereinigung von Text-, Nummern- und Adressfeldern
- Aufbau einer einheitlichen Adressspalte
- Erzeugung stabiler Dateinamen über `address_slug`
- Markierung mehrfach vorkommender EGIDs
- Prüfung und Kategorisierung des Baujahrs
- Ergänzung von Koordinaten über geo.admin.ch, falls `RUN_GEOCODING = True`
- Statuslogik für die spätere Bildabfrage
- Erzeugung eines adressbasierten Datensatzes für die Street-View-Bildabfrage
- optionale Übernahme von Resultaten aus `streetview_results.csv`, falls diese bereits aus der Bildabfrage vorliegen
- Ableitung einer gebäudebasierten Tabelle mit einer bevorzugten Adresszeile pro EGID
- Erstellung eines Qualitätsberichts
- Export des vorbereiteten Datensatzes

Outputs:

```text
Data/WT01_modellanwendung/WT01_geocoding_results.csv
Data/WT01_modellanwendung/WT01_Winterthur_vorbereiteter_Datensatz_<timestamp>.xlsx
```

Optionaler Input, falls bereits Bildabfrage-Resultate vorhanden sind:

```text
Data/WT01_modellanwendung/streetview_results.csv
```

Im ausgeführten Notebook umfasst der vorbereitete Datensatz 846 adressbasierte Zeilen und 770 eindeutige EGIDs. Alle 846 adressbasierten Einträge konnten geocodiert werden und sind für die spätere Street-View-Bildextraktion vorbereitet. 813 Einträge besitzen ein plausibles Baujahr; 33 Einträge wurden wegen fehlendem oder unplausiblem Baujahr als Qualitätsnotiz markiert, aber nicht ausgeschlossen.

## Statuslogik im WT-01-Workflow

Das WT-01-Notebook erzeugt unter anderem folgende Statusspalten:

| Spalte | Bedeutung |
|---|---|
| `WT01_status` | Status der Grunddaten, z. B. `ready_for_image_request`, `needs_geocoding` oder `exclude_missing_core_data` |
| `model_application_status` | Status für die spätere Modellpipeline, z. B. `needs_image_extraction` oder `ready_for_model` |
| `data_issue_reason` | Ausschluss- oder Problemgründe, z. B. fehlende EGID oder ungültige Adresse |
| `data_quality_note` | Qualitätsnotizen, z. B. fehlendes oder unplausibles Baujahr |
| `image_request_ready` | Gibt an, ob die Zeile für die Bildabfrage vorbereitet ist |
| `image_status` | Status der späteren Bildabfrage, initial z. B. `not_requested` |

Diese Statusspalten sollen verhindern, dass problematische Zeilen stillschweigend gelöscht werden. Stattdessen bleibt nachvollziehbar, weshalb ein Eintrag weiterverwendet, geprüft oder ausgeschlossen wird.

## Manuelle Ergänzungen für Ground Truth

Die automatisierte Datenaufbereitung erzeugt noch keine vollständigen Ground-Truth-WWR-Werte. Für die tatsächliche Ground-Truth-Erhebung müssen die ausgewählten Gebäude zusätzlich manuell geprüft und vermessen werden.

Die Gebäudefläche beziehungsweise Fassadenfläche kann über den 3D-Stadtplan Winterthur bestimmt werden:

```text
https://stadt-winterthur.maps.arcgis.com/apps/webappviewer3d/index.html?id=f8903fec3cde443f8716dad805e45aa5
```

Dazu wird im 3D-Viewer die Messfunktion verwendet. Die so bestimmten Fassaden- und Fensterflächen können anschliessend für die manuelle WWR-Berechnung verwendet werden.

## Reihenfolge der Ausführung

Für die Ground-Truth-Aufbereitung:

```text
01_Datenanalyse.ipynb
02_Baujahr_Abfrage_API.ipynb
03_Aufbereitung_Daten.ipynb
04_GT_Tour.ipynb
```

Für die Vorbereitung des gesamten Winterthur-Datensatzes:

```text
WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb
```

Der WT-01-Workflow ist unabhängig von den Notebooks `01` bis `04`, verwendet aber denselben ursprünglichen Datenkontext.

## Hinweise zur Reproduzierbarkeit

- Die Rohdaten müssen im Ordner `Data/` liegen.
- Die Dateinamen müssen mit den Pfaden im jeweiligen Notebook übereinstimmen.
- Die geo.admin.ch-Abfragen benötigen eine Internetverbindung.
- Die geo.admin.ch-API benötigt keinen eigenen API-Key.
- Bei erneuter Ausführung können API-Antworten oder Geocoding-Ergebnisse leicht variieren.
- Die Ground-Truth-Auswahl enthält Zufallsauswahlen mit `random_state=42`, damit die Auswahl reproduzierbar bleibt.
- Der WT-01-Export enthält einen Zeitstempel im Dateinamen und erzeugt deshalb bei jeder Ausführung eine neue Excel-Datei.

## Wichtigste Outputs

| Output | Entsteht in | Zweck |
|---|---|---|
| `Data/01_Datenanalyse_Ergebnis.xlsx` | `01_Datenanalyse.ipynb` | bereinigte Grundlage nach erster Datenanalyse |
| `Data/02_API_Baujahr_Adressen.xlsx` | `02_Baujahr_Abfrage_API.ipynb` | Datensatz nach API-Baujahrabgleich |
| `Data/03_Ground_Truth_Datensatz.xlsx` | `03_Aufbereitung_Daten.ipynb` | finaler Ground-Truth-Datensatz mit 15 Gebäuden |
| `Data/04_GT_Tour.xlsx` | `04_GT_Tour.ipynb` | vereinfachte Tour-Datei mit Adresse und Koordinaten |
| `Data/WT01_modellanwendung/WT01_geocoding_results.csv` | `WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb` | Geocoding-Ergebnisse für den Gesamtbestand |
| `Data/WT01_modellanwendung/WT01_Winterthur_vorbereiteter_Datensatz_<timestamp>.xlsx` | `WT-01_Winterthur_Datensatz_Modellanwendung_vorbereiten.ipynb` | vorbereiteter Datensatz für Bildabfrage und Modellpipeline |

## Abgrenzung

Dieses Repository führt keine WWR-Segmentierung und keine Modellinferenz aus. Es bereitet die Daten so vor, dass sie in nachgelagerten Repositories oder Notebooks für Bildabfrage, Modellanwendung und Evaluation verwendet werden können.
