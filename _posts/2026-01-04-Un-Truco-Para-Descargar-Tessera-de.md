---
author: Juan Sebastian Vinasco Salinas
bibliography:
- /home/juanse/Documents/ext_data/Proyectos/IGAC_CLC/references/igac_clc_biblio.bib
title: 2026 01 04 Ein Tipp zum Herunterladen von Tessera
---

Kürzlich habe ich mich mit den räumlichen Darstellungen von
Tessera[@feng2025tessera] beschäftigt, insbesondere damit, wie man deren Download automatisieren kann.
Dazu habe ich natürlich die Bibliothek geotessera [^1] verwendet, die
von einem ihrer Autoren, Anil Madhavapeddy [^2], ausführlich beschrieben wurde.

Als ich jedoch versuchte, relativ große Bereiche über die Befehlszeilenschnittstelle herunterzuladen,
explodierte der RAM-Verbrauch und mein armer Computer, der nur über 16 GB RAM verfügt, stürzte während des
Versuches ab.

Als ich mir das Ganze genauer ansah, stellte ich fest, dass der von
geotessera durchgeführte Verarbeitungsablauf wie folgt aussieht:

```{=ascii}
Benutzeranforderung  (lat/lon bbox)
    ↓
Parquet-Registrierungssuche (verfügbare Kacheln aus registry.parquet suchen)
    ↓
Direkte HTTP-Downloads in temporäre Dateien
    ├── embedding.npy (quantisiert) → temporäre Datei
    └── embedding_scales.npy → temporäre Datei
    ↓
Entquantisierung (Arrays multiplizieren)
    ↓
Automatische Bereinigung (temporäre Dateien löschen)
    ↓
Ausgabeformat
    ├── NumPy arrays → Direkte Analyse
    └── GeoTIFF → GIS-Integration

```
Insbesondere die Konvertierung in das Geotiff-Format ist hinsichtlich des
RAM-Verbrauchs problematisch.

Um dieses Problem zu lösen, musste ich auf die Python-API
von Geotessera umsteigen und jeweils nur einen Kachel nach dem anderen verarbeiten. Auf diese Weise
bleibt der Verbrauch der Ressourcen des PCs begrenzt,
allerdings auf Kosten einer längeren Verarbeitungszeit.

Hier ist meine Lösung, falls Sie daran interessiert sind:

``` {.python exports="code"}
import glob
from pathlib import Path
import shutil
import numpy as np
from tqdm import tqdm
from geotessera import GeoTessera

def descarga_masiva():
    gt = GeoTessera()
    # (min_lon, min_lat, max_lon, max_lat)
    bbox = (-76.5145301279999330,
            7.3470872690000419,
            -74.7809453929999108,
            9.4477481020000482)
    tiles_to_fetch = gt.registry.load_blocks_for_region(bounds=bbox, year=2024)
    tiles_to_fetch.reverse()

    for tile in tqdm(tiles_to_fetch,  #_no_duplicated[25:100],
                     total=len(tiles_to_fetch), #_no_duplicated[25:100]),
                     desc="Descargando tiles"):
        gt.export_embedding_geotiffs(
            tiles_to_fetch=[tile],
            output_dir=".",
            bands=None,  #  Alle 128 Bänder exportieren (Standard)
            compress="lzw"  # Komprimierungsmethode
        )
if __name__ == "__main__":
    descarga_masiva()
```

# Footnotes

[^1]: <https://github.com/ucam-eo/geotessera/>

[^2]: <https://anil.recoil.org/notes/geotessera-python>


