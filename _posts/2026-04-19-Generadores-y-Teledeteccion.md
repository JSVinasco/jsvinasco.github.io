# Generadores y Teledetección

En los últimos días, en mi *feed*, han aparecido varias referencias a
los generadores en Python. Hace un tiempo pensé que sería interesante
escribir sobre este tema, pero aplicado a algo más cercano a mi trabajo:
el procesamiento de datos geográficos tipo raster.

Siguiendo la filosofía de aprendizaje de [FastAI](https://www.fast.ai/),
quiero empezar con la idea central antes de entrar en detalles: el
objetivo aquí es poder procesar grandes volúmenes de datos geográficos
de manera eficiente, especialmente cuando no tenemos memoria suficiente
para cargar todo al mismo tiempo.

Y aquí es donde entran los generadores[^1].

Un generador es un tipo especial de función en Python que usa la palabra
clave *yield*. Esta palabra permite algo bastante interesante: pausar la
ejecución de la función y retomarla después desde el mismo punto.

En la práctica, esto significa que no necesitamos cargar todos los datos
de una vez en memoria. En lugar de eso, podemos ir procesándolos poco a
poco, lo cual es clave cuando trabajamos con imágenes satelitales
grandes, que fácilmente pueden superar la memoria RAM disponible.

Un ejemplo simple de generador en Python es el siguiente:

``` python
def generator_function():
    for element in range(5):
        yield element


def main():
    g = generator_function()

    print(next(g))  # 0
    print(next(g))  # 1
    print(next(g))  # 2
    print(next(g))  # 3
    print(next(g))  # 4
    print(next(g))  # StopIteration


if __name__ == "__main__":
    main()
```

¿Qué está pasando aquí?

-   La función usa *yield* en lugar de return
-   Creamos un generador llamado \"g\"
-   Podemos ir pidiendo valores uno por uno usando next()
-   Cuando ya no hay más datos, Python lanza una excepción llamada
    StopIteration

Esto puede parecer simple, pero es extremadamente poderoso cuando lo
aplicamos a datos reales.

---

Ahora veamos un ejemplo más cercano a la teledetección.

Primero, revisemos el tamaño de una imagen satelital Sentinel-2:

``` bash
du -sh .../B02.jp2
```

En este caso, estamos hablando de una imagen de aproximadamente 70 MB (y
esto es solo una banda).

El problema es que si intentamos cargarla completa en memoria muchas
veces, podríamos quedarnos sin RAM fácilmente.

Por eso, en lugar de leer toda la imagen de una vez, vamos a procesarla
por partes (ventanas de 1024 × 1024 píxeles) usando Rasterio.

``` python
from pathlib import Path
import numpy as np
import rasterio as rio


def read_raster_file_by_windows(filename):
    if filename.exists():
        with rio.open(Path(filename)) as raster:

            num_windows = len(list(raster.block_windows(1)))
            yield num_windows

            for _, window in raster.block_windows(1):
                yield raster.read(window=window)


def main():
    file_b4 = "..._B04.jp2"
    file_b8 = "..._B08.jp2"

    raster_b4 = read_raster_file_by_windows(file_b4)
    raster_b8 = read_raster_file_by_windows(file_b8)

    n_windows = next(raster_b4)
    next(raster_b8)

    for b4, b8 in zip(raster_b4, raster_b8):
        b4 = b4 / 10_000
        b8 = b8 / 10_000

        ndvi = (b8 - b4) / (b8 + b4 + 1e-10)

        print(ndvi.mean())


if __name__ == "__main__":
    main()
```

La idea clave aquí es sencilla:

en lugar de procesar toda la imagen, la dividimos en pequeños bloques y
los procesamos uno a uno.

Esto nos permite trabajar con imágenes mucho más grandes que la memoria
disponible.

Y obteniendo un resultado como el que vemos en la siguiente imagen:

![NDVI example with 1024 x 1024 pixels](../../../images/example_windows_read_raster_file.png)


---

Ahora viene una pregunta importante: ¿cómo guardamos el resultado si
estamos procesando por partes?

Para eso podemos combinar procesamiento por ventanas con paralelización.

La idea es simple:

-   dividimos la imagen en bloques
-   procesamos cada bloque en paralelo
-   escribimos el resultado de forma segura en disco

``` python
import concurrent.futures
import threading
import rasterio
import numpy as np


def process_window(src1, src2, dst, window, read_lock, write_lock):

    with read_lock:
        a = src1.read(window=window)
        b = src2.read(window=window)

    result = (b - a) / (b + a + 1e-10)

    with write_lock:
        dst.write(result, window=window)


def main(in1, in2, out):

    with rasterio.open(in1) as src1, rasterio.open(in2) as src2:

        profile = src1.profile
        profile.update(tiled=True, blockxsize=1024, blockysize=1024)

        with rasterio.open(out, "w", **profile) as dst:

            windows = [w for _, w in dst.block_windows()]

            read_lock = threading.Lock()
            write_lock = threading.Lock()

            with concurrent.futures.ThreadPoolExecutor(max_workers=4) as executor:
                for window in windows:
                    executor.submit(
                        process_window,
                        src1, src2, dst,
                        window,
                        read_lock,
                        write_lock
                    )
```

Aquí aparece un concepto importante: el procesamiento en paralelo.

Pero cuando varios procesos intentan escribir al mismo tiempo en un
archivo, pueden ocurrir problemas conocidos como **race conditions**,
que pueden corromper los datos.

Por eso usamos locks (bloqueos), que básicamente aseguran que solo un
proceso pueda escribir o leer una sección crítica a la vez.

---

Y con esto cerramos la idea principal:

-   los generadores nos permiten trabajar con datos grandes sin cargar
    todo en memoria
-   el procesamiento por ventanas hace posible trabajar con imágenes
    satelitales enormes
-   y la paralelización permite acelerar estos procesos sin perder
    control sobre los datos

En la siguiente entrada voy a mostrar cómo extender esta idea para hacer
inferencia con redes neuronales sobre imágenes satelitales a gran
escala.

---

# Footnotes

[^1]: <https://github.com/dabeaz/generators/blob/master/Generators.pdf>
