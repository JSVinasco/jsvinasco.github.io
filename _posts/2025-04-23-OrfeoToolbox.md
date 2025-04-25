
1.  [Orfeo Toolbox, una biblioteca para procesamiento de imagenes de satelite](#orgbcc99b2)



<a id="orgbcc99b2"></a>

# Orfeo Toolbox, una biblioteca para procesamiento de imagenes de satelite

A pesar de que orfeo toolbox es una de las bibliotecas para procesamiento de imagenes de satelites mas completas en el mercado, me soprende el poco uso o conocimiento de la herramienta que tiene por fuera de la agencia espacial frances (CNES por sus siglas en frances) y el ecosistema de empresas, centrado en Toulouse, y de algunas empresas italianas.

OTB es una herramienta de codigo abierto, tanto en el sentido literal, es decir el codigo fuente esta disponible<sup><a id="fnr.1" class="footref" href="#fn.1" role="doc-backlink">1</a></sup>, como en el sentido mas amplio y es que sigue la filosofía de desarrollo colaborativa de codigo abierto<sup><a id="fnr.2" class="footref" href="#fn.2" role="doc-backlink">2</a></sup> , valga la redundancia.

Las capacidades de la biblioteca son variadas, y en mi opinion permiten tener un control bastante preciso sobre el que operaciones se desea aplicar sobre una imagen de satelite. Las principales abstracciones de la biblioteca son heredadas de una biblioteca de tratamiento de imagenes medicas, llamata ITK, escrita y desarrollada en C++<sup><a id="fnr.3" class="footref" href="#fn.3" role="doc-backlink">3</a></sup> . Lo anterior permite que segun el nivel de abstraccion en el que el usuario este interesado, pueda escribir algoritmos de procesamiento desde cero en C++, o beneficiarce de la portabilidad en python de la biblioteca.

En mi opinion, la parte mas dificil de OTB, es su instalación, sus desarrolladores en aras de las flexibilidad y el deseo de compilar unicamente las herramientas necesarias, ha hecho que la instalacion via compilacion sea engorrosa. Obviamente se ha trabajado en esto, y mecanismos como el *superbuild* han ayudado a mejorar la experiencia de usuario al momento de instalar la biblioteca.

Otro punto debil es la necesidad de recompilar toda la biblioteca para que se compile para una version de python especifica, lo que complica a sobre manera el usar esta herramienta como dependencia en algun proyecto basado en python, que a dia de hoy es casi un estandar en la industria de teledetección.

Mi forma favorita de superar los problemas arriba mensionados, es simplemente instalar OTB, mediante la compilación realizada por el equipo desarrollador de Iota-2<sup><a id="fnr.4" class="footref" href="#fn.4" role="doc-backlink">4</a></sup>. Esto permite generar un ambiente de python con un minimo de dependencias, que te permite usar OTB al instante. Hacerlo es tan facil como hacer ejecutar las siguientes lineas<sup><a id="fnr.5" class="footref" href="#fn.5" role="doc-backlink">5</a></sup> :

    # create an empty micromamba environment: otb-env
    micromamba create --name otb-env
    # enable it
    micromamba activate otb-env
    # install recent python 3.11
    micromamba install python=3.11
    # install mamba
    micromamba install mamba
    # install otb
    mamba install otb -c iota2-2
    # disable environment and reload it
    micromamba deactivate
    micromamba activate otb-env

Para verificar si la instalacion ha sido correcta, puedes ejecutar las siguientes lineas en una repl de python, una vez hayas verificado que estes apuntando a la repl de python correcta.

    import otbApplication as otb
    
    print(otb.Registry_GetAvailableApplications())
    
    print(len(otb.Registry_GetAvailableApplications()))

Como se puede ver en la lista anterior, hay 118 applicaciones de otb instaladas, esto nos permite hacer un monton de cosas !!!

Para mantener las cosas simples, podemos simplemente leer, la informacion basica de una imagen, usando la otb application *ReadImageInfo*, una ventaja de usar esta aplicaciones, es que podemos obtener una información, similar con la aplciación *gdalinfo* y asi validar que la información sea correcta.

    
    image_path = "~/Documents/datasets/Venus/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0_FRE_B2.tif"
    
    import otbApplication as otb
    
    # Let's create the application with codename "Smoothing"
    app = otb.Registry.CreateApplication("ReadImageInfo")
    
    # We set its parameters
    app.SetParameterString("in", image_path)
    
    # This will execute the application and save the output file
    app.ExecuteAndWriteOutput()

Puedes obtener con el codigo anterior informacion como la siguiente:

Para efectuar una comparacion es posible usar un comando de gdal, como el siguiente:

    gdalinfo ~/Documents/datasets/Venus/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0/VENUS-XS_20201030-114859-000_L2A_BAMBENW2_C_V3-0_FRE_B2.tif

Obteniendo a su vez, una salida como la siguiente:

Por supuesto, todo esto es una abre bocas, la biblioteca permite realizar muchisimas tareas, como mosaicos, calibración optica, rasterización, extracción de caracteristicas, indices radiometricos, clasificaciones supervizadas y no supervizadas, tratamiento de imagenes de radar, tratamiento de imagenes basado en objetos (segmentacion), procesamiento 3D, entre otros.

Desde este pequeño blog, te animo a probar la biblioteca y a explotarla en tur proyectos personales e industriales.


# Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> [Main Repositories / otb · GitLab](https://gitlab.orfeo-toolbox.org/orfeotoolbox/otb)

<sup><a id="fn.2" href="#fnr.2">2</a></sup> [Código abierto - Wikipedia, la enciclopedia libre](https://es.wikipedia.org/wiki/C%C3%B3digo_abierto)

<sup><a id="fn.3" href="#fnr.3">3</a></sup> <https://itk.org/>

<sup><a id="fn.4" href="#fnr.4">4</a></sup> <https://docs.iota2.net/master/>

<sup><a id="fn.5" href="#fnr.5">5</a></sup> Ejemplo modificado de la documentacion de Iota-2

