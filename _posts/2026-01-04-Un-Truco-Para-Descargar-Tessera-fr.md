---
author: Juan Sebastian Vinasco Salinas
bibliography:
- /home/juanse/Documents/ext_data/Proyectos/IGAC_CLC/references/igac_clc_biblio.bib
title: 2026 01 04 Une astuce pour télécharger Tessera
---

Récemment, j'ai exploré les représentations spatiales de
Tessera[@feng2025tessera], en particulier comment automatiser leur téléchargement.
Pour ce faire, j'ai bien sûr utilisé la bibliothèque geotessera [^1], décrite
en détail par l'un de ses auteurs, Anil Madhavapeddy [^2].

Cependant, lorsque j'ai essayé de télécharger des zones relativement grandes à l'aide de l'interface de ligne de commande,
la consommation de RAM a explosé et mon pauvre ordinateur, qui ne dispose que de 16 Go de RAM, a planté pendant la
tentative.

En regardant sous le capot, j'ai réalisé que le flux de traitement effectué par
geotessera est le suivant :

```{=ascii}
User Request (lat/lon bbox)
    ↓
Parquet Registry Lookup (find available tiles from registry.parquet)
    ↓
Direct HTTP Downloads to Temp Files
    ├── embedding.npy (quantized) → temp file
    └── embedding_scales.npy → temp file
    ↓
Dequantization (multiply arrays)
    ↓
Automatic Cleanup (delete temp files)
    ↓
Output Format
    ├── NumPy arrays → Direct analysis
    └── GeoTIFF → GIS integration

```
La conversion au format geotiff, en particulier, pose problème en termes de
consommation de RAM.

Pour résoudre ce problème, j'ai dû passer à l'API Python
de geotessera et ne traiter qu'une seule tuile à la fois. De cette façon,
la consommation des ressources de l'ordinateur personnel reste limitée,
au prix d'un temps de traitement plus long.

Voici ma solution, au cas où cela vous intéresserait :

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
            bands=None,  # Export all 128 bands (default)
            compress="lzw"  # Compression method
        )

if __name__ == "__main__":
    descarga_masiva()

```

# Footnotes

[^1]: <https://github.com/ucam-eo/geotessera/>

[^2]: <https://anil.recoil.org/notes/geotessera-python>
