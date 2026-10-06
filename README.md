# Heat-Klang-Valley – SDS320

Exploratives Fernerkundungsprojekt zur urbanen Expansion im Elmina-Expansionskorridor im Klang Valley, Malaysia. Ziel ist, Veränderungen von Vegetation und bebauten Flächen sichtbar zu machen und anschließend deren Zusammenhang mit der Landoberflächentemperatur zu untersuchen.

## Aktueller Stand (6. Oktober 2026)

Der bisherige Arbeitsstand umfasst die Auswahl eines Landsat-Vergleichspaars, einen lokalen Wolkencheck im Untersuchungsgebiet und einen openEO/STAC-Workflow zum Laden, Exportieren und Anzeigen der RGB-Aufnahmen. Eine quantitative Analyse der urbanen Expansion oder Temperatur ist noch nicht abgeschlossen.

| Aufnahme | Satellit | RGB-Bänder (Rot, Grün, Blau) |
| --- | --- | --- |
| 31.05.2004 | Landsat 5 | `TM_B3`, `TM_B2`, `TM_B1` |
| 17.05.2025 | Landsat 9 | `OLI_B4`, `OLI_B3`, `OLI_B2` |

Die Aufnahmen liegen rund 21 Jahre auseinander und stammen beide aus dem Mai. Dadurch werden saisonale Unterschiede begrenzt; Wetter- und Sensorunterschiede müssen bei weiteren Auswertungen berücksichtigt werden.

## Untersuchungsgebiet

Elmina-Ausschnitt in geografischen Koordinaten (WGS84, EPSG:4326):

```python
bbox_list = [101.50, 3.16, 101.54, 3.20]
bbox_stac = {
    "west": 101.50,
    "south": 3.16,
    "east": 101.54,
    "north": 3.20,
}
```

`bbox_list` wird für die STAC-Suche und den lokalen Wolkencheck verwendet; `bbox_stac` ist das Dictionary für `connection.load_stac(spatial_extent=...)`.

## Daten und Wolkencheck

Datenquelle: Landsat Collection 2 Level-2 über den [Microsoft Planetary Computer STAC-Katalog](https://planetarycomputer.microsoft.com/api/stac/v1/collections/landsat-c2-l2).

Die Auswahl beruht auf einem Wolkencheck mit `QA_PIXEL` innerhalb des Elmina-Ausschnitts. Im bisherigen Projektverlauf wurden für beide ausgewählten Aufnahmen **0,0 % Wolken im ROI** berichtet. Das unterscheidet sich vom Wolkenanteil der gesamten Landsat-Szene. Die berichteten Werte wurden für diese README nicht erneut berechnet; ihre genaue Aussage hängt von den verwendeten QA-Bits, der Behandlung ungültiger Pixel und dem Nenner der Berechnung ab.

## Workflow in Jupyter

Benötigt werden eine Python-/Jupyter-Umgebung, ein Zugang zu einem openEO-Backend mit Unterstützung für `load_stac` und eine authentifizierte openEO-Verbindung namens `connection`. Das konkrete Backend und die Anmeldung sind im Projekt-Notebook festzuhalten.

Für den RGB-Schritt werden `openeo`, `rasterio`, `numpy` und `matplotlib` benötigt; für die STAC-Suche und den ROI-Wolkencheck können weitere Pakete erforderlich sein.

1. Verbindung zum openEO-Backend herstellen und authentifizieren.
2. Landsat-Szenen über STAC suchen und `QA_PIXEL` im ROI prüfen.
3. Die beiden ausgewählten Aufnahmen mit demselben räumlichen Ausschnitt laden.
4. RGB-Daten als GeoTIFF exportieren.
5. Beide Bilder in Jupyter nebeneinander anzeigen.

Der zuletzt festgelegte Lade- und Exportblock lautet:

```python
landsat_url = (
    "https://planetarycomputer.microsoft.com/api/stac/v1/"
    "collections/landsat-c2-l2"
)

old_final = connection.load_stac(
    landsat_url,
    spatial_extent=bbox_stac,
    temporal_extent=["2004-05-31", "2004-06-01"],
    bands=["TM_B3", "TM_B2", "TM_B1"],
    properties={"platform": lambda x: x == "landsat-5"},
)

new_final = connection.load_stac(
    landsat_url,
    spatial_extent=bbox_stac,
    temporal_extent=["2025-05-17", "2025-05-18"],
    bands=["OLI_B4", "OLI_B3", "OLI_B2"],
    properties={"platform": lambda x: x == "landsat-9"},
)

old_final.execute_batch("elmina_2004_05_31_rgb.tif", out_format="GTiff")
new_final.execute_batch("elmina_2025_05_17_rgb.tif", out_format="GTiff")
```

Die GeoTIFF-Dateinamen sind die vorgesehenen Exportnamen; sie belegen allein keinen abgeschlossenen Download. Das Erstellen eines Data Cubes bestätigt ebenfalls noch keine erfolgreiche Ausführung auf dem Backend.

Zur Visualisierung werden die drei RGB-Bänder mit Rasterio eingelesen und mit Matplotlib nebeneinander angezeigt. Der bisherige Ansatz verwendet einen Kontraststretch zwischen dem 2. und 98. Perzentil je Bild. Dieser dient der Darstellung; unterschiedliche Bildstreckungen erlauben keinen direkten quantitativen Vergleich von Farben oder Helligkeit.

## Nächste Schritte

- RGB-Aufnahmen visuell prüfen und den ROI-Wolkencheck mit den konkreten STAC-Item-IDs dokumentieren.
- Vegetationsveränderungen mit NDVI und/oder bebaute Flächen mit einer geeigneten Klassifikation untersuchen; dafür zusätzliche Spektralbänder laden.
- Für quantitative Vergleiche Skalierung, NoData-Masken, räumliche Ausrichtung und Sensorunterschiede berücksichtigen.
- Anschließend Landoberflächentemperatur mit den dafür geeigneten thermischen Produkten auswerten. RGB-Bänder allein liefern keine Temperatur.

## Reproduzierbarkeit

Diese README dokumentiert den zuletzt in der Projektbesprechung festgelegten Stand. Für eine vollständige Reproduktion gehören das aktuelle Notebook, die Backend-Konfiguration ohne Zugangsdaten, Paketversionen, STAC-Item-IDs und die ausführbare Definition des ROI-Wolkenchecks ins Repository. Zugangsdaten und Tokens dürfen nicht mit veröffentlicht werden.
