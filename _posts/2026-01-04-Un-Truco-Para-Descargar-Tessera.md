---
author: Juan Sebastian Vinasco Salinas
bibliography:
- /home/juanse/Documents/ext_data/Proyectos/IGAC_CLC/references/igac_clc_biblio.bib
title: 2026 01 04 Un Truco Para Descargar Tessera
---

Recientemente, estuve explorando las representaciones espaciales de
Tessera[@feng2025tessera], en particular como automatizar su descarga,
para ello me serví obviamente de la biblioteca geotessera [^1], descrita
a profundidad por uno de sus autores Anil Madhavapeddy [^2].

Sin embargo, al intentar realizar la descarga mediante la interfaz de
línea de comando para areas relativamente grandes, el consumo de la RAM
se disparaba y mi pobre computadora de solo 16Gb de RAM moría en su
intento.

Revisando bajo el capo, me doy cuenta que el flujo de procesamiento que
realiza geotessera es el siguiente:

```{=ascii}
Solicitud del usuario (caja contenedora lat/lon )
    ↓
Búsqueda en el registro de parquet (buscar tiles disponibles desde el registro(registry.parquet)
    ↓
Descargas HTTP directas a archivos temporales
    ├── embedding.npy (cuantizado) → archivo temporal
    └── embedding_scales.npy → archivo temporal
    ↓
Descuantización (matrices multiplicadas)
    ↓
Limpieza automática (eliminar archivos temporales)
    ↓
Formato de salida
    ├── NumPy arrays →  Análisis directo
    └── GeoTIFF → integración con SIG

```
Particularmente la conversión a geotiff, es problematica en cuanto a
consumo de RAM.

Para solucionar el problema me vi avocado a cambiar a la API de python
de geotessera y a procesar unicamente un tile cada vez, de esta manera
el consumo de recursos de la computadora personal se mantiene acotado,
acosta de incrementar el tiempo de procesamiento.

He aquí mi solución, por si te interesa:

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
            bands=None,  # Exporta todas las bandas (por defecto)
            compress="lzw"  # Metodo de compresion
        )

if __name__ == "__main__":
    descarga_masiva()

```

# Footnotes

[^1]: <https://github.com/ucam-eo/geotessera/>

[^2]: <https://anil.recoil.org/notes/geotessera-python>
