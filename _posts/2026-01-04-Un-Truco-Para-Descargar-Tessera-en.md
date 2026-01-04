---
author: Juan Sebastian Vinasco Salinas
bibliography:
- /home/juanse/Documents/ext_data/Proyectos/IGAC_CLC/references/igac_clc_biblio.bib
title: 2026 01 04 Un Trick to Download Tessera
---

Recently, I was exploring the spatial representations of
Tessera[@feng2025tessera], particularly how to automate their download.
To do this, I obviously used the geotessera library [^1], described
in depth by one of its authors, Anil Madhavapeddy [^2].

However, when I tried to download relatively large areas using the command line interface,
RAM consumption skyrocketed, and my poor computer, with only 16GB of RAM, crashed in the
attempt.

Looking under the hood, I realized that the processing flow performed by
geotessera is as follows:

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
Conversion to geotiff, in particular, is problematic in terms of
RAM consumption.

To solve the problem, I had to switch to the Python API
of geotessera and process only one tile at a time. This way,
the consumption of personal computer resources remains limited,
at the cost of increasing processing time.

Here is my solution, in case you are interested:

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

    for tile in tqdm(tiles_to_fetch, 
                     total=len(tiles_to_fetch),
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
